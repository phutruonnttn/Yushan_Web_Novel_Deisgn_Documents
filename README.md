# Yushan Web Novel Platform - Design Documents

This repository contains comprehensive design documentation for the **Yushan Microservices Web Novel Platform** - a distributed, gamified web novel reading and publishing system.

## 📚 Documentation Structure

### [1. Overview](./OVERVIEW.md) 
**Context, Business Problem, Platform Thinking & Project Scope**

- **Context & Business Problem**: Market challenges ($4.5B web novel market) and solution vision
- **Platform Thinking**: Multi-sided marketplace (readers, authors, content ecosystem)
- **Network Effects**: Cross-side and reader-side network effects
- **Project Scope**: In-scope features, out-of-scope items, and boundaries
- **Success Metrics**: Business and technical indicators

**Key Topics**:
- Global web novel market growth
- Multi-sided platform design
- Network effects strategy
- Scalable architecture rationale
- Platform differentiation

📄 **See**: [OVERVIEW.md](./OVERVIEW.md)

---

### [2. Logical Architecture & Design](./LOGICAL_ARCHITECTURE_DESIGN.md)
**Architectural Decisions, Components & System Design**

- **Key Architectural Decisions**: 11 architectural decisions explained
  - Microservices Architecture
  - Service Discovery (Eureka)
  - API Gateway Pattern
  - Event-Driven Architecture with Kafka
  - **Resilience4j Circuit Breaker** ⭐
  - Database-Per-Service Pattern
  - Caching with Redis
  - Containerization with Docker
  - Infrastructure as Code (Terraform)
  - Observability Stack

- **Logical Components & Deployment**: 6-layer architecture diagram
  - Client Layer
  - API Gateway Layer
  - Infrastructure Services Layer
  - Business Services Layer
  - Data Storage Layer
  - Observability Layer

- **Domain Driven Design**: Bounded contexts (User, Content, Engagement, Gamification, Analytics)
- **Producer-Consumer Interactions**: Kafka event flows
- **Resilience Patterns**: Circuit Breaker, Retry, Rate Limiting with Resilience4j

📄 **See**: [LOGICAL_ARCHITECTURE_DESIGN.md](./LOGICAL_ARCHITECTURE_DESIGN.md)

---

### [3. Project Conduct](./PROJECT_CONDUCT.md)
**Project Status, Issues, Milestones & Team Effort**

- **Current Status**: Service-by-service progress (75% complete)
- **Outstanding Issues**: 
  - High Priority: Circuit Breaker fallback, Analytics optimization
  - Medium Priority: Content moderation, Rate limiting
  - Low Priority: Documentation, Dashboards
- **Project Milestones**: 5-phase timeline (Oct 2024 - March 2025)
  - Phase 1: Foundation ✅ Complete
  - Phase 2: Core Services ✅ Complete
  - Phase 3: Engagement ✅ Complete
  - Phase 4: Gamification & Analytics 🔄 In Progress
  - Phase 5: Testing & Deployment ⏸️ Planned
- **Team Effort Summary**: 2,240 total hours breakdown
- **Risk Assessment**: Mitigation strategies

📄 **See**: [PROJECT_CONDUCT.md](./PROJECT_CONDUCT.md)

---

### [4. Physical Architecture & Design](./PHYSICAL_ARCHITECTURE_DESIGN.md)
**Network, Nodes, Technologies & Cloud Services**

- **Physical Architecture Decisions**:
  - Digital Ocean cloud provider (13 droplets)
  - Container-based deployment (Docker)
  - Network segmentation (Docker networks)
  - Persistent storage (Docker volumes)

- **Physical Architecture**:
  - Network topology
  - Node specifications (Droplet sizes and resources)
  - Technology stack by layer
  - Cloud services integration (Digital Ocean Spaces)
  - Deployment layout (Singapore region)

- **Persistence Design**:
  - Database architecture (PostgreSQL per service)
  - Caching strategy (Redis per service)
  - Data migration (Flyway)
  - Storage allocation

- **Representative User Story**: 
  - Detailed physical flow for "User Publishes a Novel Chapter"
  - Physical resource utilization
  - Scalability considerations

📄 **See**: [PHYSICAL_ARCHITECTURE_DESIGN.md](./PHYSICAL_ARCHITECTURE_DESIGN.md)

---

### [5. DevOps & Development Lifecycle](./DEVOPS_DEVELOPMENT_LIFECYCLE.md)
**CI/CD Pipeline, Source Management & Technical Findings**

- **Source Code Management**:
  - Multi-repository pattern
  - GitHub Container Registry (ghcr.io)
  - Branching strategy
  - Artifact management

- **DevOps Pipeline** (GitHub Actions):
  1. **unit-tests**: H2 database, ~2-3 minutes
  2. **integration-tests**: TestContainers, ~5-8 minutes
  3. **lint-quality**: SpotBugs, Checkstyle, JaCoCo, SonarCloud
  4. **security-scan**: OWASP Dependency Check, Snyk
  5. **docker-build**: Multi-platform (amd64, arm64), Trivy scan
  6. **production-deploy**: SSH to Digital Ocean (Content Service only)
  7. **dast-zap**: OWASP ZAP baseline security test
  8. **generate-reports**: Collect all CI/CD reports

- **Technical Findings & Issues**:
  - OWASP database corruption (fixed)
  - Quality checks blocking deployment (fixed)
  - Docker layer caching optimization
  - Multi-platform build performance
  - Health check failures (fixed with retry logic)

📄 **See**: [DEVOPS_DEVELOPMENT_LIFECYCLE.md](./DEVOPS_DEVELOPMENT_LIFECYCLE.md)

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│         YUSHAN MICROSERVICES PLATFORM                  │
├─────────────────────────────────────────────────────────┤
│  Infrastructure Services (3)                           │
│  ├─ Eureka Registry (:8761)                           │
│  ├─ Config Server (:8888)                              │
│  └─ API Gateway (:8080)                                │
├─────────────────────────────────────────────────────────┤
│  Business Services (5)                                │
│  ├─ User Service (:8081)                              │
│  ├─ Content Service (:8082)                           │
│  ├─ Engagement Service (:8084)                         │
│  ├─ Gamification Service (:8085)                       │
│  └─ Analytics Service (:8083)                         │
├─────────────────────────────────────────────────────────┤
│  Supporting Infrastructure                             │
│  ├─ PostgreSQL 16 (Primary Database)                  │
│  ├─ Redis 7 (Caching & Sessions)                      │
│  ├─ Elasticsearch 8.11 (Search)                       │
│  ├─ Apache Kafka (Event Streaming)                    │
│  ├─ Digital Ocean Spaces (File Storage)               │
│  ├─ Prometheus + Grafana (Monitoring)                 │
│  └─ ELK Stack (Logging)                               │
└─────────────────────────────────────────────────────────┘
```

## 🚀 Technology Stack

| Category | Technology | Version |
|----------|-----------|---------|
| **Language** | Java | 21 |
| **Framework** | Spring Boot | 3.4.10 |
| **Cloud** | Spring Cloud | 2024.0.2 |
| **Service Discovery** | Netflix Eureka | - |
| **API Gateway** | Spring Cloud Gateway | - |
| **Circuit Breaker** | Resilience4j | - |
| **Database** | PostgreSQL | 16 |
| **Cache** | Redis | 7 |
| **Search** | Elasticsearch | 8.11 |
| **Message Broker** | Apache Kafka | 7.4.0 |
| **ORM** | MyBatis | 3.0.5 |
| **Security** | JWT + Spring Security | - |
| **Monitoring** | Prometheus + Grafana | - |
| **Logging** | ELK Stack | - |
| **Containerization** | Docker | - |
| **IaC** | Terraform | - |
| **Cloud Provider** | Digital Ocean | - |
| **Region** | Singapore (sgp1) | - |

## 📊 Project Status

**Overall Progress**: 🟢 **75% Complete**

**Phase**: Active Development - Core Services Implementation

| Service | Status | Completion |
|---------|--------|------------|
| User Service | ✅ Production | 100% |
| Content Service | ✅ Production | 95% |
| Engagement Service | ✅ Production | 90% |
| Gamification Service | 🔄 Development | 75% |
| Analytics Service | 🔄 Development | 70% |

## 📖 Reading Guide

### For Stakeholders
1. Start with **[Overview](./OVERVIEW.md)** - Business context and platform thinking
2. Review **[Project Conduct](./PROJECT_CONDUCT.md)** - Timeline and milestones
3. Skim **[Physical Architecture](./PHYSICAL_ARCHITECTURE_DESIGN.md)** - Deployment details

### For Architects
1. **[Logical Architecture & Design](./LOGICAL_ARCHITECTURE_DESIGN.md)** - Core design
2. **[Physical Architecture](./PHYSICAL_ARCHITECTURE_DESIGN.md)** - Infrastructure
3. **Domain Driven Design** section - Service boundaries
4. **Platform Thinking** in Overview

### For Developers
1. **[DevOps & Development Lifecycle](./DEVOPS_DEVELOPMENT_LIFECYCLE.md)** - CI/CD pipeline
2. **[Logical Architecture & Design](./LOGICAL_ARCHITECTURE_DESIGN.md)** - Architecture
3. **Producer-Consumer Interactions** - Service communication
4. **Technical Findings** - Real issues and fixes

### For Project Managers
1. **[Overview](./OVERVIEW.md)** - Business objectives
2. **[Project Conduct](./PROJECT_CONDUCT.md)** - Status and timeline
3. **Risk Assessment** - Mitigation strategies

## 🔗 Quick Links

- **Overview**: [OVERVIEW.md](./OVERVIEW.md)
- **Logical Architecture**: [LOGICAL_ARCHITECTURE_DESIGN.md](./LOGICAL_ARCHITECTURE_DESIGN.md)
- **Project Conduct**: [PROJECT_CONDUCT.md](./PROJECT_CONDUCT.md)
- **Physical Architecture**: [PHYSICAL_ARCHITECTURE_DESIGN.md](./PHYSICAL_ARCHITECTURE_DESIGN.md)
- **DevOps & Lifecycle**: [DEVOPS_DEVELOPMENT_LIFECYCLE.md](./DEVOPS_DEVELOPMENT_LIFECYCLE.md)

## 📝 Document Details

**Repository**: [phutruonnttn/Yushan_Web_Novel_Deisgn_Documents](https://github.com/phutruonnttn/Yushan_Web_Novel_Deisgn_Documents)  
**Version**: 1.0.0  
**Last Updated**: January 2025  
**Total Pages**: 5 comprehensive documents  
**Total Words**: ~25,000 words

## 🎯 Key Highlights

### What Makes Yushan Unique?
1. **Platform Thinking**: Multi-sided marketplace, not just an app
2. **Event-Driven**: Kafka for real-time analytics and gamification
3. **Fault Tolerant**: Resilience4j Circuit Breaker throughout
4. **Observable**: ELK + Prometheus + Grafana for complete visibility
5. **Scalable**: Database-per-service, independent scaling
6. **Modern Stack**: Java 21, Spring Boot 3.4.10, Spring Cloud 2024.0.2

### Technical Achievements
- ✅ 13 Digital Ocean droplets orchestrated via Terraform
- ✅ Multi-platform Docker images (linux/amd64, linux/arm64)
- ✅ Comprehensive CI/CD pipeline (GitHub Actions)
- ✅ Security scanning: OWASP, Snyk, Trivy
- ✅ Code quality: SpotBugs, Checkstyle, SonarCloud
- ✅ 75% test coverage (target: 80%)
- ✅ Zero critical security vulnerabilities

## 📸 Architecture Diagrams

For detailed architecture diagrams, please refer to:
- **[Logical Architecture](./LOGICAL_ARCHITECTURE_DESIGN.md)**: 6-layer system architecture
- **[Physical Architecture](./PHYSICAL_ARCHITECTURE_DESIGN.md)**: Network topology and node details

## 👥 Contributors

- Architecture Team
- Backend Development Team
- DevOps Team
- Product Management Team

---

**Last Updated**: January 2025  
**Documentation Version**: 1.0.0

