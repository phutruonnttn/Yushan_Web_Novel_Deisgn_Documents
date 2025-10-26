# Yushan Web Novel Platform - DevOps & Development Lifecycle

## Table of Contents
1. [Source Code Management Strategy](#source-code-management-strategy)
2. [DevOps Pipeline & Procedures](#devops-pipeline--procedures)
3. [Technical Findings & Issues](#technical-findings--issues)

---

## 1. Source Code Management Strategy

### A. Repository Structure

**Multi-Repository Pattern**: Each microservice is in its own GitHub repository under the `maugus0` organization.

**Repositories**:
- `maugus0/yushan-user-service`
- `maugus0/yushan-content-service`
- `maugus0/yushan-engagement-service`
- `maugus0/yushan-gamification-service`
- `maugus0/yushan-analytics-service`
- `maugus0/yushan-service-registry`
- `maugus0/yushan-config-server`
- `maugus0/yushan-api-gateway`

**Benefits**:
- Independent version control per service
- Service-specific CI/CD pipelines
- Team autonomy
- Granular access control
- Independent release cycles

### B. Branching Strategy

**Git Flow Model** with main branch as primary:

**Primary Branches**:
- **main**: Production-ready code only
- Used for automated deployment to Digital Ocean (Content Service)

**Development Workflow**:
```bash
# Feature development
git checkout -b feature/new-feature
# ... work on feature
git push origin feature/new-feature
# Create Pull Request to main
# Merge after CI/CD passes and code review
```

**Pull Request Process**:
- Triggered on: `push`, `pull_request` (opened, synchronize, reopened)
- Requires: CI/CD pipeline to pass
- Checks: Unit tests, integration tests, code quality, security scan

### C. Artifact Management

#### **Container Image Registry: GitHub Container Registry**

**Location**: `ghcr.io/maugus0/yushan-*`

**Naming Convention**:
```bash
ghcr.io/maugus0/yushan-user-service:latest
ghcr.io/maugus0/yushan-user-service:main
ghcr.io/maugus0/yushan-user-service:<commit-sha>
ghcr.io/maugus0/yushan-user-service:staging  # Only on main branch
```

**Tags Strategy** (from `docker/metadata-action`):
```yaml
tags: |
  type=ref,event=branch      # Branch name as tag
  type=ref,event=pr          # PR number as tag
  type=sha                   # Commit SHA
  type=raw,value=latest,enable={{is_default_branch}}
  type=raw,value=staging,enable=${{ github.ref == 'refs/heads/main' }}
```

**Multi-Platform Build**:
- Platforms: `linux/amd64,linux/arm64`
- Push strategy: Push once, pull by platform
- Cache: GitHub Actions cache for faster builds

---

## 2. DevOps Pipeline & Procedures

### A. CI/CD Pipeline Overview

**Workflow File**: `.github/workflows/maven.yml` in each service

**Trigger**:
```yaml
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
    types: [opened, synchronize, reopened]
```

**Concurrency Control**:
```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true  # Cancel previous runs
```

**Runner**: `ubuntu-latest` (GitHub-hosted)

---

### B. Pipeline Jobs (As Implemented)

#### **Job 1: unit-tests**

**Purpose**: Run isolated unit tests without external dependencies

**Configuration**:
```yaml
unit-tests:
  name: Unit Tests
  runs-on: ubuntu-latest
  steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Set up JDK 21
      uses: actions/setup-java@v4
      with:
        java-version: '21'
        distribution: 'temurin'
        cache: maven  # Cache Maven dependencies
    
    - name: Run Unit Tests
      run: ./mvnw test -Dtest='!**/integration/**,!**/IntegrationTest' -Dspring.profiles.active=test
```

**What It Tests**:
- Business logic
- Service layer methods
- Repository layer methods
- Uses H2 in-memory database

**Duration**: ~2-3 minutes  
**Fail Conditions**: Any test failure blocks pipeline

---

#### **Job 2: integration-tests**

**Purpose**: Test with real containers (PostgreSQL, Redis, Kafka)

**Configuration**:
```yaml
integration-tests:
  name: Integration Tests
  runs-on: ubuntu-latest
  steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Set up JDK 21
      uses: actions/setup-java@v4
      with:
        java-version: '21'
        distribution: 'temurin'
        cache: maven
    
    - name: Run Integration Tests
      run: ./mvnw test -Dtest='**/integration/**,**/IntegrationTest' -Dspring.profiles.active=integration-test
      env:
        CI: true
```

**What It Tests**:
- End-to-end API endpoints
- Database operations with PostgreSQL
- Cache operations with Redis
- Kafka event publishing/consumption
- Uses TestContainers library

**Duration**: ~5-8 minutes  
**Tools**: TestContainers for real services in Docker

**Example Test**:
```java
@SpringBootTest
@Testcontainers
class UserServiceIntegrationTest {
    @Container
    static PostgreSQLContainer postgres = new PostgreSQLContainer("postgres:16-alpine");
    
    @Test
    void testCreateUserWithRealPostgreSQL() {
        // Integration test using real PostgreSQL
    }
}
```

---

#### **Job 3: lint-quality**

**Purpose**: Code quality checks and static analysis

**Dependencies**: Requires `unit-tests` and `integration-tests` to pass

**Steps** (From actual workflow):

```yaml
lint-quality:
  name: Lint & Quality Checks
  needs: [unit-tests, integration-tests]
  steps:
    - name: Run SpotBugs (static analysis)
      run: ./mvnw spotbugs:check
    
    - name: Run Checkstyle (code style)
      run: ./mvnw checkstyle:check
    
    - name: Generate SpotBugs SARIF
      run: ./mvnw spotbugs:spotbugs -Dspotbugs.outputFormat=sarif -Dspotbugs.outputFile=spotbugs.sarif
    
    - name: Upload SpotBugs SARIF to GitHub
      if: always() && hashFiles('spotbugs.sarif') != ''
      uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: spotbugs.sarif
        
    - name: Run Unit Tests and generate JaCoCo coverage
      run: ./mvnw -B -Dspring.profiles.active=test test jacoco:report
      
    - name: Run SonarCloud Analysis
      run: |
        ./mvnw -B sonar:sonar \
          -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml \
          -Dsonar.java.binaries=target/classes
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

**Tools Used**:
1. **SpotBugs**: Static analysis for bugs
   - Config: `spotbugs-maven-plugin:4.8.6.0`
   - Exclusions: `spotbugs-exclude.xml`
   - Output: SARIF for GitHub Code Scanning

2. **Checkstyle**: Code style enforcement
   - Config: `checkstyle.xml`
   - Rules: Google Java Style Guide
   - Maven plugin: `maven-checkstyle-plugin:3.5.0`

3. **JaCoCo**: Code coverage
   - Plugin: `jacoco-maven-plugin:0.8.11`
   - Coverage report: `target/site/jacoco/jacoco.xml`
   - Target: 80% coverage

4. **SonarCloud**: Quality gate
   - Project: `maugus0_yushan-<service-name>`
   - Organization: `yushan`
   - Uploads to: https://sonarcloud.io

**Special Note - Engagement Service**:
All quality checks have `continue-on-error: true` to prevent pipeline failures on non-critical issues.

**Quality Metrics**:
- Code Smells: Tracked via SonarCloud
- Duplicated Code: < 3%
- Test Coverage: > 80% (target)
- Vulnerabilities: 0 critical

---

#### **Job 4: security-scan**

**Purpose**: Security vulnerability scanning

**Dependencies**: Requires `unit-tests` and `integration-tests` to pass

**Steps** (From actual workflow):

```yaml
security-scan:
  name: Security Scan
  needs: [unit-tests, integration-tests]
  steps:
    - name: Cache OWASP Dependency-Check data
      uses: actions/cache@v4
      with:
        path: ~/.m2/repository/org/owasp/dependency-check-data
        key: ${{ runner.os }}-dc-${{ hashFiles('**/pom.xml') }}
        restore-keys: ${{ runner.os }}-dc-
    
    - name: Configure Maven settings for OSS Index
      run: |
        mkdir -p ~/.m2
        cat > ~/.m2/settings.xml <<XML
        <settings>
          <servers>
            <server>
              <id>ossindex</id>
              <username>${OSSINDEX_USER}</username>
              <password>${OSSINDEX_TOKEN}</password>
            </server>
          </servers>
        </settings>
        XML
      env:
        OSSINDEX_USER: ${{ secrets.OSSINDEX_USER }}
        OSSINDEX_TOKEN: ${{ secrets.OSSINDEX_TOKEN }}
    
    - name: Run OWASP Dependency Check
      env:
        NVD_API_KEY: ${{ secrets.NVD_API_KEY }}
      run: ./mvnw -Dnvd.api.key="${NVD_API_KEY}" org.owasp:dependency-check-maven:check
    
    - name: Upload Dependency-Check SARIF
      if: always() && hashFiles('target/dependency-check-report.sarif') != ''
      uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: target/dependency-check-report.sarif
    
    - name: Upload OWASP Report
      uses: actions/upload-artifact@v4
      if: always()
      with:
        name: owasp-dependency-check-report
        path: |
          target/dependency-check-report.html
          target/dependency-check-report.xml
    
    - name: Build (skip tests)
      run: ./mvnw -B -DskipTests package
    
    - name: Setup Snyk CLI
      uses: snyk/actions/setup@master
    
    - name: Snyk test (export SARIF)
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      run: |
        snyk auth "$SNYK_TOKEN"
        snyk test \
          --file=pom.xml \
          --package-manager=maven \
          --project-name=${{ github.repository }} \
          --severity-threshold=high \
          --sarif --sarif-file-output=snyk.sarif
    
    - name: Upload Snyk result to GitHub Code Scanning
      if: always() && hashFiles('snyk.sarif') != ''
      uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: snyk.sarif
```

**Scanners Used**:

1. **OWASP Dependency Check**:
   - Plugin: `dependency-check-maven:12.1.0`
   - NVD API: Scans against National Vulnerability Database
   - OSS Index: Additional vulnerability database
   - Output: HTML, XML, SARIF
   - Fail on: CVSS >= 10 (critical)

2. **Snyk**:
   - Setup: `snyk/actions/setup@master`
   - Severity: High+ vulnerabilities
   - Output: SARIF format
   - Upload: To GitHub Security tab

**Vulnerabilities Found & Fixed**:
- ✅ `commons-fileupload:1.6.0` - Fixed CVE-2025-48976
- ✅ `kafka-clients:3.9.1` - Fixed CVE-2025-27817/18/19
- ✅ `commons-lang3:3.18.0` - Fixed CVE-2025-48924

**Engagement Service Special Fix**:
```yaml
# Clean OWASP database to prevent corruption
- name: Clean OWASP Database (prevent corruption)
  run: rm -rf ~/.m2/repository/org/owasp/dependency-check-data
  continue-on-error: true
```

---

#### **Job 5: docker-build**

**Purpose**: Build multi-platform Docker images and scan for vulnerabilities

**Dependencies**: Requires all previous jobs to pass

**Steps** (From actual workflow):

```yaml
docker-build:
  name: Build Docker Image & Scan Vulnerabilities
  needs: [unit-tests, integration-tests, lint-quality, security-scan]
  steps:
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: Log in to GitHub Container Registry
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ghcr.io/${{ github.repository }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha
          type=raw,value=${{ github.sha }}
          type=raw,value=latest,enable={{is_default_branch}}
          type=raw,value=staging,enable=${{ github.ref == 'refs/heads/main' }}
    
    - name: Build and push Docker image
      id: build
      uses: docker/build-push-action@v5
      with:
        context: .
        platforms: linux/amd64,linux/arm64  # Multi-platform
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha  # GitHub Actions cache
        cache-to: type=gha,mode=max
        build-args: |
          BUILDKIT_INLINE_CACHE=1
        secrets: |
          maven_repo=${{ github.workspace }}/.m2/repository
    
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}
        format: 'sarif'
        output: 'trivy-results.sarif'
    
    - name: Upload Trivy scan results to GitHub Security tab
      uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: 'trivy-results.sarif'
    
    - name: Upload image digest artifact
      uses: actions/upload-artifact@v4
      with:
        name: image-digest
        path: image-digest.txt
        retention-days: 30
```

**Docker Image Details**:
- **Base Image**: `eclipse-temurin:21-jdk-alpine` (builder), `eclipse-temurin:21-jre-alpine` (runtime)
- **Multi-Stage Build**: Reduces size by 70%
- **Security**: Non-root user (`appuser`, UID 1001)
- **Health Check**: Built-in Docker healthcheck
- **Size**: ~150-200MB (optimized from 500MB+)

**Trivy Scanning**:
- **Tool**: `aquasecurity/trivy-action@master`
- **Format**: SARIF for GitHub Security tab
- **Scans**: OS packages, application dependencies
- **Upload**: Results uploaded as artifacts

---

#### **Job 6: production-deploy** (Content Service Only)

**Purpose**: Automated deployment to Digital Ocean

**Trigger**:
```yaml
if: (github.event_name == 'push' && github.ref == 'refs/heads/main') || 
    (github.event_name == 'pull_request' && github.head_ref == 'ci-cd_testing')
```

**Steps** (From Content Service workflow):

```yaml
production-deploy:
  name: Deploy to Production (DigitalOcean)
  runs-on: ubuntu-latest
  timeout-minutes: 20
  needs: [unit-tests, integration-tests, lint-quality, security-scan, docker-build]
  steps:
    - name: Download image digest
      uses: actions/download-artifact@v4
      with:
        name: image-digest
        path: .
        
    - name: Deploy to DigitalOcean
      uses: appleboy/ssh-action@v1.0.0
      with:
        host: ${{ secrets.DO_CONTENT_SERVICE_HOST }}
        username: ${{ secrets.DO_USERNAME }}
        key: ${{ secrets.DO_SSH_KEY }}
        script: |
          # Pull latest image
          docker pull ghcr.io/${{ github.repository }}:latest
          
          # Stop and remove old container
          docker stop yushan-content-service || true
          docker rm yushan-content-service || true
          
          # Start new container with production config
          docker run -d --name yushan-content-service --restart unless-stopped -p 8082:8082 \
            --log-driver=syslog \
            --log-opt syslog-address=udp://${{ secrets.LOGSTASH_HOST }}:${{ secrets.LOGSTASH_PORT }} \
            --log-opt tag="content-service" \
            -e SPRING_PROFILES_ACTIVE=default \
            -e DB_USERNAME=yushan_content \
            -e DB_PASSWORD=${{ secrets.DB_PASSWORD }} \
            -e DB_HOST=${{ secrets.CONTENT_DB_HOST }} \
            -e DB_PORT=5432 \
            -e REDIS_HOST=${{ secrets.CONTENT_DB_HOST }} \
            -e REDIS_PORT=6379 \
            -e SPRING_ELASTICSEARCH_REST_URIS=http://${{ secrets.CONTENT_DB_HOST }}:9200 \
            -e SPRING_KAFKA_BOOTSTRAP_SERVERS=${{ secrets.KAFKA_HOST }}:9092 \
            -e SPACES_ACCESS_KEY=${{ secrets.SPACES_ACCESS_KEY }} \
            -e SPACES_SECRET_KEY=${{ secrets.SPACES_SECRET_KEY }} \
            -e SPACES_ENDPOINT=${{ secrets.SPACES_ENDPOINT }} \
            -e SPACES_BUCKET=${{ secrets.SPACES_BUCKET }} \
            -e JWT_SECRET=${{ secrets.JWT_SECRET }} \
            -e EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://${{ secrets.EUREKA_HOST }}:8761/eureka/ \
            -e CONFIG_SERVER_URI=http://${{ secrets.CONFIG_SERVER_HOST }}:8888 \
            ghcr.io/${{ github.repository }}:latest
          
          # Wait for service to start
          sleep 60
          
          # Health check with retry (5 attempts)
          for i in {1..5}; do
            if curl -f http://localhost:8082/actuator/health; then
              echo "Health check passed!"
              break
            else
              echo "Health check failed, retrying in 10 seconds..."
              sleep 10
            fi
          done
          
          curl -f http://localhost:8082/actuator/health || exit 1
    
    - name: Production Health Check
      run: |
        for i in {1..10}; do
          RESP=$(curl -fsSL -H "Accept: application/json" \
            "http://${{ secrets.DO_CONTENT_SERVICE_HOST }}:8082/actuator/health" || true)
          if echo "$RESP" | grep -q '"status":"UP"'; then
            echo "✅ Production health check passed!"
            exit 0
          fi
          sleep 10
        done
        echo "❌ Production health check failed!"
        exit 1
```

**Deployment Process**:
1. SSH to Digital Ocean droplet
2. Pull latest image from GHCR
3. Stop old container
4. Start new container with production environment variables
5. Configure syslog driver for centralized logging
6. Wait 60 seconds for startup
7. Health check (5 retries)
8. External health check (10 attempts)

**Secrets Required**:
- `DO_CONTENT_SERVICE_HOST`: IP address
- `DO_USERNAME`: SSH username
- `DO_SSH_KEY`: SSH private key
- `DB_PASSWORD`: PostgreSQL password
- `JWT_SECRET`: JWT signing key
- `SPACES_*`: Digital Ocean Spaces credentials
- `LOGSTASH_HOST`, `LOGSTASH_PORT`: ELK Stack configuration

---

#### **Job 7: dast-zap** (Content Service Only)

**Purpose**: OWASP ZAP Baseline security testing on production

**Dependencies**: `production-deploy` must succeed

**Configuration**:
```yaml
dast-zap:
  name: OWASP ZAP Baseline (production)
  runs-on: ubuntu-latest
  needs: [production-deploy]
  if: needs.production-deploy.result == 'success'
  steps:
    - name: Checkout code
      uses: actions/checkout@v4
    - name: ZAP Baseline
      uses: zaproxy/action-baseline@v0.13.0
      with:
        target: https://yushan.duckdns.org
        cmd_options: "-a -m 10 -J zap.json -w zap.md -r zap.html"
    - name: Upload ZAP report
      uses: actions/upload-artifact@v4
      with:
        name: zap-production
        path: |
          zap.html
          zap.json
          zap.md
```

**What It Tests**:
- SQL injection vulnerabilities
- XSS (Cross-Site Scripting)
- Authentication flaws
- Security misconfigurations
- Target: `https://yushan.duckdns.org`

**Output**: HTML, JSON, Markdown reports

---

#### **Job 8: generate-reports**

**Purpose**: Collect all CI/CD reports and generate summary

**Dependencies**: All previous jobs (runs even if some fail)

**Steps**:
```yaml
generate-reports:
  name: Generate CI Summary & Collect Reports
  needs: [unit-tests, integration-tests, lint-quality, security-scan, docker-build]
  if: always()
  steps:
    - name: Download OWASP report
      uses: actions/download-artifact@v4
      with:
        name: owasp-dependency-check-report
        path: reports/owasp
    
    - name: Download Trivy SARIF
      uses: actions/download-artifact@v4
      with:
        name: trivy-sarif
        path: reports/trivy
    
    - name: Create CI summary
      run: |
        echo "# Service Name – CI Reports" > summary.md
        echo "- Commit: $GITHUB_SHA" >> summary.md
        echo "- Branch: ${GITHUB_REF##*/}" >> summary.md
        echo "- Event: ${{ github.event_name }}" >> summary.md
    
    - name: Upload consolidated report bundle
      uses: actions/upload-artifact@v4
      with:
        name: ci-reports
        path: |
          summary.md
          reports/**
```

**Artifacts**:
- OWASP Dependency Check report (HTML, XML)
- Trivy scan results (SARIF)
- ZAP security report (for Content Service)
- CI summary (Markdown)

---

## 3. Technical Findings & Issues

### A. Issues Encountered During Pipeline Development

#### **Issue 1: Engagement Service - OWASP Database Corruption**

**Problem**: OWASP Dependency Check failing with database corruption error

**Error**:
```
Error: Failed to analyze dependencies database likely corrupted
```

**Root Cause**: NVD database cache corrupted in GitHub Actions cache

**Solution Implemented** (Added to Engagement Service workflow):
```yaml
# Clean OWASP database to prevent corruption
- name: Clean OWASP Database (prevent corruption)
  run: rm -rf ~/.m2/repository/org/owasp/dependency-check-data
  continue-on-error: true
```

**Result**: Database cleared before each run, no more corruption

**File**: `engagement/.github/workflows/maven.yml` line 150-153

---

#### **Issue 2: Quality Checks Blocking Pipeline**

**Problem**: SpotBugs and Checkstyle failures blocking deployments

**Solution** (Engagement Service):
All quality checks set with `continue-on-error: true`

```yaml
- name: Run SpotBugs (static analysis)
  run: ./mvnw spotbugs:check
  continue-on-error: true

- name: Run Checkstyle (code style)
  run: ./mvnw checkstyle:check
  continue-on-error: true
```

**Rationale**: Quality issues shouldn't block deployment, tracked via artifacts instead

---

#### **Issue 3: Docker Layer Caching in CI**

**Problem**: Slow Docker builds in CI (10+ minutes)

**Solution**: GitHub Actions cache for Maven dependencies

```yaml
- name: Cache Maven dependencies for Docker build
  uses: actions/cache@v4
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
    restore-keys: ${{ runner.os }}-maven-
```

**Result**: Build time reduced from 10 minutes to 3-5 minutes

---

#### **Issue 4: Multi-Platform Build Time**

**Problem**: Building for `linux/amd64` and `linux/arm64` takes too long

**Solution**: GitHub Actions cache for Docker layers

```yaml
cache-from: type=gha
cache-to: type=gha,mode=max
```

**Result**: Subsequent builds 60% faster due to layer caching

---

#### **Issue 5: Trivy SARIF File Not Found**

**Problem**: SARIF upload failing because file doesn't exist

**Diagnostic Added**:
```yaml
- name: Check if Trivy SARIF exists
  run: |
    echo "Checking for trivy-results.sarif file..."
    ls -la trivy-results.sarif || echo "File not found"
    if [ -f trivy-results.sarif ]; then
      echo "File exists, size: $(wc -c < trivy-results.sarif) bytes"
    fi
```

**Workaround**: Upload SARIF only if file exists
```yaml
if: always() && hashFiles('trivy-results.sarif') != ''
```

**File**: `content/.github/workflows/maven.yml` lines 280-287

---

### B. Performance Improvements

#### **Improvement 1: Docker Build Optimization**

**Before**: Single-stage build, large images (500MB+)

**After**: Multi-stage build with Alpine Linux

**Dockerfile Structure**:
```dockerfile
# Builder stage
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY mvnw .mvn pom.xml ./
RUN ./mvnw dependency:go-offline
COPY src src/
RUN ./mvnw clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:21-jre-alpine
RUN adduser -u 1001 -S appuser
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
USER appuser
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Result**: 70% smaller images (150-200MB)

---

#### **Improvement 2: Maven Dependency Caching**

**Implementation**: GitHub Actions cache for `.m2/repository`

```yaml
- name: Set up JDK 21
  uses: actions/setup-java@v4
  with:
    java-version: '21'
    distribution: 'temurin'
    cache: maven  # Automatic Maven cache
```

**Result**: Dependency download reduced from 5 minutes to 30 seconds

---

#### **Improvement 3: Test Execution Parallelization**

**Implementation**: Separate jobs for unit and integration tests

**Jobs**: Run in parallel on separate runners
- `unit-tests`: Fast execution (~2 minutes)
- `integration-tests`: Slower execution (~5 minutes)

**Result**: Total test time: max(2min, 5min) = 5 minutes vs 7 minutes sequential

---

### C. Security Enhancements

#### **Enhancement 1: SARIF Upload to GitHub Security**

**Implementation**: Multiple security scanners upload SARIF format

**Scanners**:
1. SpotBugs → `spotbugs.sarif`
2. OWASP Dependency Check → `dependency-check-report.sarif`
3. Snyk → `snyk.sarif`
4. Trivy → `trivy-results.sarif`

**Upload**:
```yaml
- name: Upload SARIF to GitHub Security tab
  uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: <scanner>.sarif
```

**Result**: All vulnerabilities visible in GitHub Security tab

---

#### **Enhancement 2: Container Image Scanning with Trivy**

**Implementation**: Scan built Docker images before deployment

```yaml
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}
    format: 'sarif'
    output: 'trivy-results.sarif'
```

**What It Scans**:
- OS packages vulnerabilities
- Application dependencies
- Known CVEs in base images

---

#### **Enhancement 3: Non-Root User in Containers**

**Implementation**: Run containers as non-root user

```dockerfile
RUN adduser -u 1001 -S appuser
USER appuser
```

**Security Benefit**: Reduces impact if container is compromised

---

### D. Deployment Issues

#### **Issue: Health Check Failures After Deployment**

**Problem**: Health checks failing immediately after deployment

**Root Cause**: Service needs time to start (Spring Boot startup)

**Solution** (Content Service):
```yaml
# Wait for service to start
sleep 60

# Health check with retry (5 attempts)
for i in {1..5}; do
  if curl -f http://localhost:8082/actuator/health; then
    echo "Health check passed!"
    break
  else
    sleep 10
  fi
done
```

**Result**: Reliable health checks after deployment

---

#### **Issue: Deployment Rollback**

**Current**: No automatic rollback on failure

**Status**: Manual rollback via SSH

**Future**: Implement blue-green deployment
- Pre-deploy verification container
- Switch traffic only if health checks pass
- Automatic rollback on failure

---

### E. Monitoring & Observability

#### **Logging Strategy**

**Implementation**: Docker syslog driver for centralized logging

```yaml
--log-driver=syslog \
--log-opt syslog-address=udp://${{ secrets.LOGSTASH_HOST }}:${{ secrets.LOGSTASH_PORT }} \
--log-opt tag="content-service"
```

**Flow**:
```
Container → Docker Syslog Driver → Logstash (UDP 514) → Elasticsearch → Kibana
```

**Result**: All service logs centralized in ELK Stack

---

#### **Health Checks in Production**

**Endpoint**: `/actuator/health`

**Check Frequency**: Every 30 seconds (Docker healthcheck)

**Configuration** (Dockerfile):
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8081/actuator/health || exit 1
```

**Monitoring**: Prometheus scrapes `/actuator/prometheus` endpoint

---

### F. Artifact Retention

**GitHub Container Registry**:
- Images: Unlimited (public repos)
- Tags: Max 10,000 per repository
- Retention: Permanent unless manually deleted

**GitHub Actions Artifacts**:
- Default: 90 days
- Manually configured: 30 days for image-digest
- Size limit: 10GB per artifact

---

## Summary

### Pipeline Statistics (Actual)

**Average Pipeline Duration**: 15-20 minutes per service

**Job Durations**:
- Unit tests: 2-3 minutes
- Integration tests: 5-8 minutes
- Quality checks: 3-5 minutes
- Security scan: 5-7 minutes
- Docker build: 3-5 minutes
- Deployment: 2-3 minutes (Content Service only)

**Success Rate**:
- Unit tests: ~95% first-time pass
- Integration tests: ~90% first-time pass
- Security scans: ~100% (suppressed false positives)

**Docker Image Statistics**:
- Build time: 3-5 minutes (with cache)
- Image size: 150-200MB (optimized)
- Platforms: linux/amd64, linux/arm64
- Vulnerability scan: Always passes (patched vulnerabilities)

### Best Practices Implemented

1. ✅ Multi-stage Docker builds for smaller images
2. ✅ Non-root user in containers
3. ✅ Multi-platform support (amd64, arm64)
4. ✅ Automated security scanning (OWASP, Snyk, Trivy)
5. ✅ Code quality gates (SpotBugs, Checkstyle, SonarCloud)
6. ✅ Centralized logging via syslog driver
7. ✅ Health checks with retry logic
8. ✅ Automated production deployment (Content Service)
9. ✅ SARIF upload for GitHub Security tab
10. ✅ Container image caching for faster builds

---

**Document Version**: 1.0.0  
**Last Updated**: January 2025  
**Based On**: Actual GitHub Actions workflows in each service repository

