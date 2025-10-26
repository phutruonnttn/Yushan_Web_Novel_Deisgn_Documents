# Yushan Web Novel Platform - Physical Architecture & Design

## Table of Contents
1. [Key Architectural Decisions](#key-architectural-decisions)
2. [Physical Architecture](#physical-architecture)
3. [Persistence Design](#persistence-design)
4. [Representative User Story Design](#representative-user-story-design)

---

## 1. Key Architectural Decisions

### A. Digital Ocean as Cloud Provider

**Decision**: Deploy on Digital Ocean cloud infrastructure.

**Rationale**:
- **Cost-Effective**: Affordable pricing at ~$144/month for 12 droplets
- **Region Selection**: Singapore region (sgp1) for low latency in Asian markets
- **Simple Droplet Model**: Easy to understand and manage for small teams
- **Docker Support**: Native Docker support on Ubuntu 22.04
- **Flexibility**: Easy scaling by adding/removing droplets

**Impact on Physical Architecture**:
- 13 total droplets distributed across services
- Each service runs in its own droplet for isolation
- Database-per-service pattern with dedicated droplets
- Load balancers deployed per cluster

### B. Container-Based Deployment

**Decision**: Package all services as Docker containers using multi-stage builds.

**Implementation**:
```dockerfile
# Example from User Service Dockerfile
FROM eclipse-temurin:21-jdk-alpine AS builder
# Build stage with Maven cache
RUN ./mvnw clean package

FROM eclipse-temurin:21-jre-alpine
# Runtime stage with minimal JRE
COPY --from=builder /app/target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Rationale**:
- **Portability**: Consistent environment across dev/staging/production
- **Security**: Non-root user execution (appuser with UID 1001)
- **Efficiency**: Multi-stage builds reduce image size by 60%+
- **Caching**: Docker layer caching speeds up builds
- **Health Checks**: Built-in container health monitoring

**Physical Impact**:
- All services run as Docker containers on droplets
- Container orchestration via Docker daemon
- Health checks every 30s for all services
- Memory allocation: Max 75% of container RAM

### C. Network Segmentation

**Decision**: Use Docker networks for service isolation.

**Architecture**:
```yaml
# Example network from docker-compose.yml
networks:
  yushan-platform-network:
    external: true  # Shared network across services
```

**Network Layers**:
1. **yushan-platform-network**: Shared network for all microservices
2. **yushan-elk-network**: Isolated ELK stack network
3. **yushan-monitoring-network**: Isolated Prometheus/Grafana network

**Rationale**:
- **Service Discovery**: Services communicate via service names
- **Security**: Network-level isolation
- **Performance**: Docker bridge networking for speed
- **Scalability**: Easy to add/remove services

### D. Persistent Storage Strategy

**Decision**: Use Docker volumes for stateful data.

**Implementation**:
- PostgreSQL data: `/var/lib/postgresql/data`
- Redis data: `/data` with AOF persistence
- Elasticsearch: `/usr/share/elasticsearch/data`
- Log data: Centralized via syslog driver

**Rationale**:
- **Data Persistence**: Survives container restarts
- **Backup**: Easy to backup volume mounts
- **Performance**: Local storage is faster than network storage
- **Cost**: Included in droplet pricing

---

## 2. Physical Architecture

### A. Network Topology

```
┌────────────────────────────────────────────────────────────┐
│              DIGITAL OCEAN CLOUD - SGP1                    │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌──────────────────────────────────────────────────┐    │
│  │         PUBLIC INTERNET                         │    │
│  └──────────────────┬───────────────────────────────┘    │
│                     │                                      │
│                     ▼                                      │
│  ┌──────────────────────────────────────────────────┐    │
│  │         INFRASTRUCTURE DROPLET (1 droplet)        │    │
│  │  Size: s-2vcpu-4gb | IP: X.X.X.X                 │    │
│  ├─────────────────────────────────────────────────  │    │
│  │  Services:                                        │    │
│  │  • Nginx (Port 80, 443) - Reverse Proxy         │    │
│  │  • Eureka Registry (Port 8761)                   │    │
│  │  • Config Server (Port 8888)                     │    │
│  │  • API Gateway (Port 8080)                       │    │
│  │  • Kafka (Port 9092)                            │    │
│  │  • Zookeeper (Port 2181)                        │    │
│  └──────────────────┬──────────────────────────────┘    │
│                     │                                      │
│         ┌───────────┼───────────┬───────────┐             │
│         │           │           │           │             │
│         ▼           ▼           ▼           ▼             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐       │
│  │USER_SVC │  │CONT_SVC│  │ENG_SVC │  │GAM_SVC  │       │
│  │:8081    │  │:8082   │  │:8084   │  │:8085    │       │
│  └────┬────┘  └────┬───┘  └────┬───┘  └────┬───┘       │
│       │            │            │            │            │
│       ▼            ▼            ▼            ▼            │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐       │
│  │USER_DB │  │CONT_DB │  │ENG_DB  │  │GAM_DB  │       │
│  │:5432   │  │:5432   │  │:5432   │  │:5432   │       │
│  └────────┘  └────────┘  └────────┘  └────────┘       │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │    MONITORING DROPLETS (2 droplets)               │  │
│  ├───────────────────────────────────────────────── │  │
│  │  • ELK Stack (Elasticsearch, Logstash, Kibana)   │  │
│  │  • Prometheus, Grafana, Alertmanager             │  │
│  └──────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

### B. Node Specifications

#### Infrastructure Droplet
- **Size**: s-2vcpu-4gb
- **OS**: Ubuntu 22.04 x64
- **RAM**: 4GB
- **CPU**: 2 vCPUs
- **Storage**: 80GB SSD
- **Services**: 6 containers (Eureka, Config, API Gateway, Nginx, Kafka, Zookeeper)
- **Network**: Single public IP
- **Cost**: $24/month

#### Business Service Droplets (4 droplets)
Each service:
- **Size**: s-1vcpu-2gb (small), s-2vcpu-4gb (large)
- **OS**: Ubuntu 22.04 x64
- **RAM**: 2GB-4GB
- **CPU**: 1-2 vCPUs
- **Storage**: 40GB SSD each
- **Network**: Public IP + Docker networking
- **Cost**: $12-24/month each

**Examples**:
- User Service: s-1vcpu-2gb
- Content Service: s-1vcpu-2gb
- Engagement Service: s-1vcpu-2gb
- Gamification + Analytics: s-2vcpu-4gb (shared droplet)

#### Database Droplets (5 droplets)
- **Size**: s-1vcpu-1gb to s-2vcpu-4gb
- **Services**: PostgreSQL 16 + Redis 7 per droplet
- **Storage**: Docker volumes for persistence
- **Backup**: Manual via pg_dump + Redis RDB

**Database Allocations**:
```
User DB Droplet:
├─ PostgreSQL:16-alpine (Port 5432)
└─ Redis:7-alpine (Port 6379)

Content DB Droplet:
├─ PostgreSQL:16-alpine (Port 5432)
├─ Redis:7-alpine (Port 6379)
└─ Elasticsearch:7.17.9 (Port 9200)

Engagement DB Droplet:
├─ PostgreSQL:16-alpine (Port 5432)
└─ Redis:7-alpine (Port 6379)

Gamification DB Droplet:
├─ PostgreSQL:16-alpine (Port 5432)
├─ PostgreSQL:16-alpine (Analytics, Port 5433)
├─ Redis:7-alpine (Port 6379)
└─ Redis:7-alpine (Analytics, Port 6380)
```

### C. Technology Stack by Layer

#### Application Layer
- **Runtime**: Eclipse Temurin JRE 21 (Alpine Linux)
- **Framework**: Spring Boot 3.4.10
- **Container Base**: `eclipse-temurin:21-jre-alpine`
- **Security**: Non-root user (appuser)
- **Health Checks**: Built-in Docker healthcheck every 30s

#### Database Layer
- **Primary DB**: PostgreSQL 16-alpine
- **Cache**: Redis 7-alpine with AOF persistence
- **Search**: Elasticsearch 7.17.9
- **Message Queue**: Kafka 7.4.0

#### Infrastructure Layer
- **Reverse Proxy**: Nginx (installed on droplet)
- **Service Registry**: Eureka Server 2024.0.2
- **Config Server**: Spring Cloud Config Server
- **API Gateway**: Spring Cloud Gateway
- **Orchestration**: Docker Engine 24.x

#### Monitoring Layer
- **Metrics**: Prometheus 2.x
- **Visualization**: Grafana 10.x
- **Logging**: ELK Stack (Elasticsearch 8.11, Logstash 8.11, Kibana 8.11)
- **Alerting**: Alertmanager 2.x

### D. Cloud Services Integration

**Digital Ocean Services Used**:
1. **Droplets**: 13 virtual private servers
2. **Spaces (S3-compatible)**: Object storage for images
   - Endpoint: `https://sgp1.digitaloceanspaces.com`
   - Bucket: `yushan-content`
   - Access: AWS SDK for Java (compatible API)
3. **SSH Keys**: For secure droplet access
4. **Load Balancers**: For high availability (planned)

**Physical Deployment Layout**:
```
Region: Singapore (sgp1)
├─ Droplet Size Distribution:
│  ├─ Small (s-1vcpu-2gb): 9 droplets @ $12/month = $108/month
│  ├─ Medium (s-2vcpu-4gb): 2 droplets @ $24/month = $48/month
│  └─ Large (s-4vcpu-8gb): 2 droplets @ $48/month = $96/month (planned for scale)
└─ Total Current: ~$156/month
```

---

## 3. Persistence Design

### A. Database Architecture

#### **Pattern: Database-Per-Service**

Each microservice owns its database, ensuring:
- **Data Isolation**: Services cannot directly access other services' data
- **Independent Scaling**: Each database can be scaled independently
- **Technology Flexibility**: Choose appropriate database for each service
- **Deployment Independence**: Services can be deployed separately

#### **User Service Database**

**Schema**: PostgreSQL 16
**Location**: `/user/src/main/resources/db/migration/`

**Tables**:
```sql
-- users table (Primary entity)
CREATE TABLE users(
    uuid UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) NOT NULL,
    hash_password VARCHAR(255) NOT NULL,
    avatar_url TEXT NOT NULL,
    status INTEGER DEFAULT 1,
    is_author BOOLEAN DEFAULT FALSE,
    is_admin BOOLEAN DEFAULT FALSE,
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    update_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- library table (User's saved novels)
CREATE TABLE library(
    id SERIAL PRIMARY KEY,
    uuid UUID NOT NULL,
    user_id UUID NOT NULL,
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- novel_library table (Many-to-many relationship)
CREATE TABLE novel_library(
    id SERIAL PRIMARY KEY,
    library_id INTEGER NOT NULL,
    novel_id INTEGER NOT NULL,
    progress INTEGER,  -- Reading progress
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Volume**: `user_pg_data` (Docker volume)
**Backup**: Automated daily via `pg_dump`

#### **Content Service Database**

**Schema**: PostgreSQL 16 + Elasticsearch 7.17.9

**PostgreSQL Tables**:
```sql
-- novel table
CREATE TABLE novel(
    id SERIAL PRIMARY KEY,
    uuid UUID UNIQUE NOT NULL,
    title VARCHAR(255) NOT NULL,
    author_id UUID NOT NULL,
    synopsis TEXT,
    cover_img_url TEXT,
    status INTEGER DEFAULT 0,
    chapter_cnt INTEGER DEFAULT 0,
    avg_rating REAL DEFAULT 0.0,
    view_cnt BIGINT DEFAULT 0
);

-- chapter table
CREATE TABLE chapter(
    id SERIAL PRIMARY KEY,
    novel_id INTEGER REFERENCES novel(id) ON DELETE CASCADE,
    chapter_number INTEGER NOT NULL,
    title VARCHAR(255) NOT NULL,
    content TEXT,  -- Large text field
    word_cnt INTEGER,
    is_premium BOOLEAN DEFAULT FALSE,
    view_cnt BIGINT DEFAULT 0,
    CONSTRAINT unique_novel_chapter UNIQUE (novel_id, chapter_number)
);
```

**Elasticsearch Indexes**:
- `novels`: Full-text search for novel metadata
- `chapters`: Searchable chapter content
- **Mapping**: Custom analyzers for multi-language support

**Storage**:
- PostgreSQL data: `content_pg_data` volume
- Elasticsearch data: `content_es_data` volume
- Image files: Digital Ocean Spaces (S3-compatible)

#### **Engagement Service Database**

**Schema**: PostgreSQL 16

**Tables**:
```sql
-- comment table
CREATE TABLE comment(
    id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL,
    chapter_id INTEGER NOT NULL,
    content TEXT NOT NULL,
    like_cnt INTEGER DEFAULT 0,
    is_spoiler BOOLEAN DEFAULT FALSE,
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- review table (for novels)
CREATE TABLE review(
    id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL,
    novel_id INTEGER NOT NULL,
    rating INTEGER CHECK (rating >= 1 AND rating <= 5),
    title VARCHAR(255),
    content TEXT,
    like_cnt INTEGER DEFAULT 0,
    CONSTRAINT unique_user_novel_review UNIQUE (user_id, novel_id)
);
```

**Volume**: `engagement_pg_data`

#### **Gamification Service Database**

**Schema**: PostgreSQL 16

**Tables**:
```sql
-- exp_transactions (Experience points)
CREATE TABLE exp_transactions(
    id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL,
    amount DOUBLE PRECISION NOT NULL,
    reason VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- achievements
CREATE TABLE achievements(
    id VARCHAR(100) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    criteria_json JSONB,  -- Flexible criteria definition
    icon_url TEXT
);

-- user_achievements
CREATE TABLE user_achievements(
    id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL,
    achievement_id VARCHAR(100) REFERENCES achievements(id),
    unlocked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, achievement_id)
);
```

**Volume**: `gamification_pg_data`

#### **Analytics Service Database**

**Schema**: PostgreSQL 16 (shared droplet with gamification)

**Tables**:
```sql
-- reading_history (User reading tracking)
CREATE TABLE reading_history(
    id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL,
    novel_id INTEGER NOT NULL,
    chapter_id INTEGER NOT NULL,
    read_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    duration_seconds INTEGER  -- Time spent reading
);

-- content_metrics (Aggregated metrics)
CREATE TABLE content_metrics(
    id SERIAL PRIMARY KEY,
    content_id INTEGER NOT NULL,
    view_count BIGINT DEFAULT 0,
    engagement_score REAL,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Volume**: `analytics_pg_data` (Port 5433)

### B. Caching Strategy

#### **Redis Usage per Service**

**User Service** (Port 6379):
- Session management (TTL: 24 hours)
- User profile cache (TTL: 1 hour)
- JWT token blacklist (TTL: token expiration)

**Content Service** (Port 6379):
- Novel metadata cache (TTL: 5 minutes)
- Popular novels list (TTL: 1 hour)
- Search results cache (TTL: 10 minutes)

**Engagement Service** (Port 6379):
- Recent comments cache
- Review aggregations
- Like counters (TTL: Real-time sync to DB)

**Gamification Service** (Port 6379):
- Real-time leaderboards (TTL: Persistent)
- User point balances (TTL: 1 hour)
- Achievement unlock status (TTL: 5 minutes)

**Analytics Service** (Port 6380):
- Aggregated metrics cache
- Ranking calculations (TTL: 15 minutes)

**Redis Configuration**:
```yaml
# redis:7-alpine with AOF persistence
command: ["redis-server", "--appendonly", "yes"]
volumes:
  - redis_data:/data
```

### C. Data Migration Strategy

**Tool**: Flyway Database Migrations

**Migration Pattern**:
```
src/main/resources/db/migration/
├─ V1__Initial_schema.sql       # Base schema
├─ V2__Add_seed_data.sql         # Initial data
├─ V3__Admin_seed_data.sql       # Admin users
└─ V4__Fix_seed_data.sql         # Data corrections
```

**Migration Execution**:
```yaml
# application.yml
spring.flyway:
  enabled: true
  locations: classpath:db/migration
  baseline-on-migrate: true
  validate-on-migrate: true
```

**Benefits**:
- Version control for database schema
- Automatic migration on startup
- Rollback capability via Flyway history
- No manual SQL scripts needed

---

## 4. Representative User Story Design

### **User Story**: User Publishes a Novel Chapter

**Actor**: Author (authenticated user with is_author=true)  
**System**: Content Service + supporting services  
**Precondition**: User is logged in and has created a novel

---

### **Step-by-Step Physical Flow**

#### **Step 1: API Gateway Receives Request**

```
Client (Browser/Mobile App)
  │
  ▼ HTTPS POST
API Gateway Droplet (X.X.X.X:8080)
  │
  ├─ Validate JWT token
  ├─ Extract user_id from token
  └─ Route to Content Service
```

**Physical Location**: Infrastructure Droplet, Port 8080

**Network**: Public IP → API Gateway container

---

#### **Step 2: Content Service Processes Request**

```
API Gateway
  │
  ▼ HTTP POST /api/v1/chapters
Content Service Container (:8082)
  │
  ├─ Security Filter (JWT validation)
  ├─ Controller: ChapterController.createChapter()
  ├─ Service: ChapterService.validateChapter()
  └─ Persistence: ChapterMapper.insert()
```

**Physical Location**: Content Service Droplet
**Container**: yushan-content-service
**JVM**: -XX:MaxRAMPercentage=75.0 (1.5GB from 2GB droplet)

---

#### **Step 3: Database Persistence**

```
ChapterMapper.insert()
  │
  ▼ INSERT INTO chapter(...)
PostgreSQL Container (:5432)
  │
  ├─ Write chapter data
  ├─ Update novel.chapter_cnt
  └─ Commit transaction
```

**Physical Location**: Content DB Droplet
**Container**: yushan-content-postgres
**Volume**: content-pg-data (40GB SSD)
**Query Time**: <50ms for insert

---

#### **Step 4: Elasticsearch Indexing**

```
Chapter Service
  │
  ▼ HTTP POST /chapters/_doc
Elasticsearch Container (:9200)
  │
  ├─ Index chapter content
  ├─ Update novel metadata
  └─ Refresh search index
```

**Physical Location**: Content DB Droplet (same as PostgreSQL)
**Container**: yushan-content-elasticsearch
**Index**: chapters
**Volume**: content-es-data (100GB SSD)
**Index Time**: <200ms for single chapter

---

#### **Step 5: Event Publication**

```
ChapterService.publishEvent()
  │
  ▼ Send to Kafka
Kafka Container (:9092)
  │
  ├─ Topic: novel-events
  ├─ Partition: Based on novel_id
  └─ Persist to disk
```

**Physical Location**: Infrastructure Droplet
**Container**: yushan-kafka
**Topic**: novel-events
**Retention**: 7 days
**Replication**: 1 (single broker for MVP)

---

#### **Step 6: Event Consumption**

**Parallel Event Processing**:

**A. Analytics Service**
```
Kafka Consumer (Analytics Service)
  │
  ├─ Receive chapter-created event
  ├─ Update reading_history table
  ├─ Recalculate content_metrics
  └─ Update ranking algorithms
```

**Physical Location**: Shared droplet with Gamification Service
**Container**: yushan-analytics-service (:8083)
**Database**: yushan-analytics-postgres (:5433)

**B. Gamification Service**
```
Kafka Consumer (Gamification Service)
  │
  ├─ Award points for "Publish Chapter"
  ├─ Check achievements (e.g., "Chapter Master")
  └─ Update leaderboard (exp_transactions)
```

**Physical Location**: Shared droplet with Analytics
**Container**: yushan-gamification-service (:8085)
**Database**: yushan-gamification-postgres (:5432)

**C. Engagement Service**
```
Kafka Consumer (Engagement Service)
  │
  ├─ Notify followers of new chapter
  ├─ Update recommendation algorithms
  └─ Cache popular content list
```

**Physical Location**: Engagement Service Droplet
**Container**: yushan-engagement-service (:8084)

---

#### **Step 7: Cache Invalidation**

```
Content Service
  │
  ├─ Clear novel metadata from Redis
  ├─ Clear chapter list cache
  └─ Update popular novels cache
```

**Physical Location**: Content DB Droplet
**Container**: yushan-content-redis (:6379)
**TTL**: 5 minutes for novel metadata

---

#### **Step 8: Response to User**

```
ChapterController.createChapter()
  │
  ├─ Return chapter_id
  ├─ Return publish_time
  └─ Return novel statistics
      │
      ▼ HTTP 201 Created
API Gateway
  │
  ▼ JSON Response
Client
```

**Physical Path Summary**:
1. Client → API Gateway (Infrastructure droplet)
2. API Gateway → Content Service (Content droplet)
3. Content Service → PostgreSQL + Elasticsearch (Content DB droplet)
4. Content Service → Kafka (Infrastructure droplet)
5. Kafka → Analytics/Gamification/Engagement (Parallel)
6. Content Service → Client via API Gateway

**Total Physical Nodes Involved**: 5 droplets
**Estimated Latency**: 200-500ms (p95)
**Data Replicated**: 3 databases, 1 search index, 3 caches

---

### **Physical Resource Utilization**

**For this user story**:

| Node | Service | CPU Usage | Memory Usage | Network I/O |
|------|---------|-----------|--------------|------------|
| Infrastructure | API Gateway | 5% | 200MB | 10 KB in, 50 KB out |
| Content | Content Service | 15% | 1.2GB | 50 KB in, 5 KB out |
| Content DB | PostgreSQL | 10% | 512MB | 20 KB in, 5 KB out |
| Content DB | Elasticsearch | 20% | 768MB | 100 KB in, 5 KB out |
| Infrastructure | Kafka | 5% | 128MB | 10 KB in, 30 KB out |
| Gamification | Gamification Service | 5% | 512MB | 5 KB in, 2 KB out |
| Engagement | Engagement Service | 5% | 512MB | 5 KB in, 2 KB out |
| Analytics | Analytics Service | 5% | 512MB | 5 KB in, 2 KB out |

**Total Estimated**: 
- CPU: ~65% aggregated
- Memory: ~4GB across all services
- Network: ~100KB total

---

### **Scalability Considerations**

**Current Bottlenecks**:
1. **Elasticsearch**: Single node limits write throughput
2. **PostgreSQL**: No read replicas for query distribution
3. **Kafka**: Single broker limits partition distribution

**Scaling Strategy**:
- **Horizontal**: Add service instances (multiple droplets)
- **Vertical**: Upgrade droplet sizes (s-1vcpu-2gb → s-2vcpu-4gb)
- **Caching**: Increase Redis cache hit ratio (>90%)
- **Database**: PostgreSQL connection pooling (HikariCP: 10 min, 100 max)

---

**Document Version**: 1.0.0  
**Last Updated**: January 2025

