# Yushan Microservices Platform
## Logical Architecture & Design Documentation

---

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [Key Architectural Decisions](#key-architectural-decisions)
3. [Logical Components & Deployment Diagram](#logical-components--deployment-diagram)
4. [Domain Driven Design Elements](#domain-driven-design-elements)
5. [Producer-Consumer Interactions](#producer-consumer-interactions)

---

## Executive Summary

The Yushan platform is a distributed microservices-based web novel reading and publishing system designed for scalability, resilience, and maintainability. The platform consists of 5 business services (User, Content, Engagement, Gamification, Analytics) and 3 infrastructure services (Eureka Service Registry, Config Server, API Gateway), all deployed on Digital Ocean cloud infrastructure with comprehensive monitoring and logging capabilities.

**Core Philosophy**: Domain-Driven Design (DDD) principles guide service boundaries, with Event-Driven Architecture (EDA) enabling asynchronous communication between bounded contexts.

---

## Key Architectural Decisions

### 1. **Microservices Architecture Pattern**

**Decision**: Adopt a microservices architecture over monolithic architecture.

**Rationale**:
- **Independent Deployment**: Each service can be deployed, scaled, and maintained independently
- **Technology Diversity**: Services can use different technologies (e.g., MyBatis for data persistence vs. Elasticsearch for search)
- **Fault Isolation**: Failures in one service do not cascade to others
- **Team Autonomy**: Different teams can work on different services concurrently
- **Horizontal Scalability**: Services can be scaled independently based on load patterns

**Evidence**: 
- Each service has its own database (Database-Per-Service pattern)
- Services communicate via REST APIs and Kafka events
- Independent port allocation (8081-8085)

### 2. **Service Discovery via Netflix Eureka**

**Decision**: Use Eureka for service discovery and registration.

**Rationale**:
- **Dynamic Service Location**: Services discover each other without hardcoded URLs
- **Load Balancing**: Eureka integrates with Spring Cloud LoadBalancer for client-side load balancing
- **Health Monitoring**: Eureka monitors service health and removes unhealthy instances
- **High Availability**: Eureka supports multiple server instances for redundancy

**Implementation**:
- Eureka Server runs on port 8761
- All microservices register as Eureka clients
- API Gateway uses service discovery for dynamic routing

### 3. **Centralized Configuration Management**

**Decision**: Use Spring Cloud Config Server for centralized configuration.

**Rationale**:
- **Environment Management**: Single source of truth for configuration across environments
- **Version Control**: Configuration changes tracked in Git
- **Hot Reloading**: Services can refresh configuration without restart via Actuator `/refresh` endpoint
- **Secrets Management**: Sensitive data (passwords, API keys) externalized from code

**Implementation**:
- Config Server on port 8888
- Configuration files stored in Git repository
- Native backend for local development

### 4. **API Gateway Pattern**

**Decision**: Implement a centralized API Gateway using Spring Cloud Gateway.

**Rationale**:
- **Single Entry Point**: All client requests enter through one gateway
- **Cross-Cutting Concerns**: Authentication, rate limiting, logging handled centrally
- **Service Aggregation**: Gateway can compose multiple service responses
- **Protocol Translation**: Can translate between HTTP/gRPC/WebSocket
- **Security**: Centralized SSL termination and request filtering

**Implementation**:
- Spring Cloud Gateway on port 8080
- Dynamic routing based on service discovery
- Route definitions for all service endpoints
- Path rewriting for API versioning

### 5. **Event-Driven Architecture with Apache Kafka**

**Decision**: Use Kafka as message broker for asynchronous event-driven communication.

**Rationale**:
- **Decoupling**: Services communicate via events without direct dependencies
- **Scalability**: Kafka handles high-throughput event streams
- **Reliability**: Events are persisted and can be replayed
- **Event Sourcing**: Events serve as immutable audit logs
- **Real-time Processing**: Supports real-time analytics and monitoring

**Event Types**:
- User activity events
- Content publication events (novel, chapter creation)
- Engagement events (comments, reviews, likes)
- Gamification events (achievements, points, badges)
- Analytics events (reading progress, behavior tracking)

**Topics Structure**:
- `novel-events`: Novel/chapter lifecycle events
- `comment-events`: Comment creation/deletion events
- `review-events`: Review and rating events
- `vote-events`: User interaction events
- `active`: User activity tracking

### 6. **Database-Per-Service Pattern**

**Decision**: Each microservice maintains its own database.

**Rationale**:
- **Data Ownership**: Each service owns and manages its data independently
- **Technology Flexibility**: Services can choose appropriate databases (PostgreSQL, Redis, Elasticsearch)
- **Independent Scaling**: Databases can be scaled independently based on workload
- **Data Isolation**: Prevents tight coupling through shared database

**Database Allocation**:
- **User Service**: PostgreSQL (user accounts, profiles) + Redis (session cache)
- **Content Service**: PostgreSQL (novels, chapters) + Redis (metadata cache) + Elasticsearch (full-text search)
- **Engagement Service**: PostgreSQL (comments, reviews, bookmarks)
- **Gamification Service**: PostgreSQL (achievements, points, leaderboards)
- **Analytics Service**: PostgreSQL (analytics data, rankings)

### 7. **Caching Strategy with Redis**

**Decision**: Use Redis for distributed caching and session management.

**Rationale**:
- **Performance**: Reduces database load for frequently accessed data
- **Scalability**: In-memory caching supports high-throughput operations
- **Session Management**: Centralized session storage for distributed systems
- **Pub/Sub**: Supports real-time features (notifications, live updates)

**Caching Layers**:
- **Session Cache**: User sessions, authentication tokens
- **Metadata Cache**: Frequently accessed content metadata
- **Query Cache**: Cached search results and recommendations
- **Leaderboards**: Real-time scoring and rankings

### 8. **Containerization with Docker**

**Decision**: Package each service as a Docker container.

**Rationale**:
- **Consistency**: Same environment across development, testing, and production
- **Portability**: Containers run on any Docker-compatible platform
- **Isolation**: Services run in isolated containers
- **Resource Efficiency**: Lightweight compared to virtual machines

**Container Strategy**:
- Each microservice has its own Docker image
- Multi-stage builds for optimized image sizes
- Health checks configured for orchestration
- Logging via Docker syslog driver

### 9. **Fault Tolerance with Resilience4j Circuit Breaker**

**Decision**: Implement Resilience4j for fault tolerance and resilience patterns.

**Rationale**:
- **Circuit Breaker**: Prevents cascading failures by stopping calls to failing services
- **Retry with Exponential Backoff**: Automatically retries transient failures
- **Rate Limiting**: Protects services from traffic spikes
- **Bulkhead Isolation**: Ensures resource isolation between operations
- **Time Limiter**: Prevents hanging requests from consuming resources indefinitely

**Implementation Strategy**:
- **Integration Point**: Applied to all Feign client calls between services
- **Circuit Breaker Configuration**:
  - Sliding window: COUNT_BASED with 20 requests
  - Failure threshold: 50% of calls failing
  - Half-open state: Tests recovery with 5 permitted calls
  - Wait duration: 10 seconds before attempting recovery
  
**Configuration in Application**:
```yaml
# Resilience4j Dependencies
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>

# Configuration applied automatically via Spring Boot Actuator
management:
  health:
    circuitbreakers:
      enabled: true
  metrics:
    export:
      prometheus:
        enabled: true
```

**Use Cases**:
- **Circuit Breaker**: Applied to Content Service calls when Elasticsearch is slow
- **Retry**: Re-attempt failed database connections
- **Rate Limiter**: Limit calls to third-party APIs (Digital Ocean Spaces)
- **Time Limiter**: Prevent slow searches from blocking UI

**Monitoring**:
- Circuit breaker state exposed via Actuator endpoints
- Metrics integrated with Prometheus
- Grafana dashboards show circuit breaker health
- Alerts configured for OPEN circuit breaker state

### 10. **Infrastructure as Code with Terraform**

**Decision**: Use Terraform for Digital Ocean infrastructure provisioning.

**Rationale**:
- **Reproducibility**: Infrastructure can be recreated from code
- **Version Control**: Infrastructure changes tracked in Git
- **Automation**: Reduces manual provisioning errors
- **Cost Optimization**: Resources provisioned only as needed

**Infrastructure Components**:
- **13 Droplets**: 
  - 1 Infrastructure droplet (Eureka, Config, API Gateway, Kafka)
  - 5 Business service droplets
  - 5 Database droplets
  - 1 Monitoring droplet (ELK Stack)
  - 1 Monitoring droplet (Prometheus/Grafana)
- Load Balancers for high availability
- Docker volumes for persistent storage

### 11. **Observability Stack**

**Decision**: Implement comprehensive monitoring, logging, and metrics collection.

**Rationale**:
- **Debugging**: Centralized logs help diagnose issues
- **Performance**: Metrics enable bottleneck identification
- **Alerting**: Automated alerts for critical issues
- **Business Intelligence**: Analytics dashboards for stakeholders

**Observability Components**:
- **ELK Stack** (Elasticsearch, Logstash, Kibana):
  - Log aggregation and analysis
  - Distributed tracing
  - Full-text log search
- **Prometheus + Grafana**:
  - Metrics collection from all services
  - Custom dashboards for KPIs
  - Alerting for SLA violations
- **Actuator Endpoints**:
  - Health checks (`/actuator/health`)
  - Metrics (`/actuator/metrics`)
  - Prometheus scrape endpoint (`/actuator/prometheus`)

---

## Logical Components & Deployment Diagram

### System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER (External Apps)                         │
│                      Web Apps, Mobile Apps, Admin Panel                     │
└────────────────────────────┬────────────────────────────────────────────────┘
                             │ HTTPS
                             ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         API GATEWAY LAYER                                   │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │         Spring Cloud Gateway (Port 8080)                             │  │
│  │  • Routing & Load Balancing                                          │  │
│  │  • Request Aggregation                                               │  │
│  │  • Rate Limiting                                                     │  │
│  │  • API Versioning                                                    │  │
│  └─────────────────────────────┬────────────────────────────────────────┘  │
└────────────────────────────────┼─────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    INFRASTRUCTURE SERVICES LAYER                            │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐         │
│  │  Eureka Server   │  │ Config Server     │  │  Kafka/Zookeeper │         │
│  │  (Port 8761)     │  │  (Port 8888)      │  │  (Port 9092)     │         │
│  │  • Registration  │  │  • Config Mgmt    │  │  • Event Bus     │         │
│  │  • Discovery     │  │  • Environment   │  │  • Pub/Sub       │         │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘         │
└─────────────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      BUSINESS SERVICES LAYER                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ User Service │  │Content Service│  │Engagement    │  │ Gamification ││  │
│  │  (8081)      │  │  (8082)       │  │ Service      │  │  Service     │  │
│  │              │  │              │  │  (8084)      │  │  (8085)      │  │
│  │ • Auth       │  │ • Novels      │  │ • Comments   │  │ • Points     │  │
│  │ • Users      │  │ • Chapters    │  │ • Reviews    │  │ • Badges     │  │
│  │ • Profiles   │  │ • Search      │  │ • Bookmarks  │  │ • Quests     │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
│         │                  │                  │                  │          │
│         └──────────────────┴──────────────────┴──────────────────┘          │
│                                    │                                          │
│                                    ▼                                          │
│                         ┌──────────────────────────┐                        │
│                         │   Analytics Service      │                        │
│                         │        (8083)            │                        │
│                         │   • Rankings             │                        │
│                         │   • Analytics            │                        │
│                         │   • Reports              │                        │
│                         └──────────────────────────┘                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        DATA STORAGE LAYER                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ PostgreSQL   │  │    Redis      │  │ Elasticsearch │ │   Kafka       │  │
│  │ (Port 5432)  │  │ (Port 6379)   │  │ (Port 9200)   │ │  Topics       │  │
│  │              │  │              │  │              │ │              │  │
│  │ • Primary    │  │ • Cache      │  │ • Search     │ │ • Events     │  │
│  │   Database   │  │ • Sessions   │  │ • Analytics  │ │ • Audit Log  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                     OBSERVABILITY LAYER                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Prometheus  │  │   Grafana     │  │ Elasticsearch │ │   Kibana     │  │
│  │ (Port 9090)  │  │  (Port 3000)  │  │  (Logs)      │ │ (Port 5601)  │  │
│  │              │  │              │  │              │ │              │  │
│  │ • Metrics    │  │ • Dashboards  │  │ • Log Storage │ │ • Log View   │  │
│  │ • Alerting   │  │ • Charts      │  │ • Indexing   │ │ • Discovery  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Detailed Service Architecture

#### 1. User Service (Port 8081)

**Domain**: User management, authentication, authorization

**Responsibilities**:
- User registration and authentication
- Profile management
- Role-based access control (Users, Authors, Admins)
- Library management (user's saved novels)
- JWT token generation and validation

**Key Entities**:
- User (UUID, email, username, roles)
- LibraryItem (user's saved novels)
- Author (extended user profile)

**Dependencies**:
- PostgreSQL for persistent data
- Redis for session management
- Config Server for configuration
- Eureka for service discovery
- Kafka for publishing user events

**Published Events**:
- `user-created`: When new user registers
- `user-updated`: When profile is updated
- `user-activity`: For analytics tracking

---

#### 2. Content Service (Port 8082)

**Domain**: Novel content, chapters, search

**Responsibilities**:
- Novel and chapter CRUD operations
- Full-text search using Elasticsearch
- Content moderation
- Author management
- Collection curation
- Image upload to Digital Ocean Spaces (S3-compatible)

**Key Entities**:
- Novel (title, synopsis, genres, status)
- Chapter (content, order, status)
- Genre, Tag (categorization)
- Author (author profiles)

**Dependencies**:
- PostgreSQL for metadata
- Elasticsearch for search
- Redis for caching
- Digital Ocean Spaces for file storage
- Kafka for publishing content events

**Published Events**:
- `novel-created`: New novel published
- `chapter-created`: New chapter released
- `content-updated`: Novel/chapter modified

---

#### 3. Engagement Service (Port 8084)

**Domain**: User engagement and interaction

**Responsibilities**:
- Comment management
- Review and rating system
- Bookmarks and reading progress
- Follow relationships
- Reading lists
- Notifications
- Content recommendations

**Key Entities**:
- Comment (text, timestamps)
- Review (rating, content)
- Bookmark (saved content)
- ReadingProgress (chapter, position)
- Follow (user-to-user, user-to-novel)

**Dependencies**:
- PostgreSQL for engagement data
- Redis for hot data caching
- Feign clients for user and content service calls
- Kafka for publishing engagement events

**Published Events**:
- `comment-created`: User comments on content
- `review-created`: User reviews content
- `bookmark-added`: User bookmarks content
- `user-activity`: Engagement metrics

---

#### 4. Gamification Service (Port 8085)

**Domain**: Gaming elements and rewards

**Responsibilities**:
- Points and reward system
- Achievement tracking
- Badge management
- Leaderboards
- Quest system
- Streak tracking

**Key Entities**:
- Points (user points, history)
- Achievement (badges earned)
- Leaderboard (rankings)
- Quest (challenges, rewards)

**Dependencies**:
- PostgreSQL for gamification data
- Redis for real-time leaderboards
- Kafka for listening to user activity events
- Feign clients for user service

**Consumed Events**:
- Listens to user activity events from multiple services
- Awards points and achievements based on events

**Published Events**:
- `achievement-unlocked`: User earns achievement
- `leaderboard-updated`: Rankings change

---

#### 5. Analytics Service (Port 8083)

**Domain**: Data analytics and reporting

**Responsibilities**:
- User behavior tracking
- Content popularity analytics
- Ranking algorithms
- Report generation
- Reading history tracking

**Key Entities**:
- ReadingHistory (what users read)
- ContentMetrics (views, engagement)
- UserMetrics (activity, preferences)

**Dependencies**:
- PostgreSQL for analytics data
- Redis for aggregated metrics
- Kafka for consuming events from all services
- Feign clients for cross-service data retrieval

**Consumed Events**:
- Consumes events from all services
- Aggregates data for insights
- Generates rankings and reports

---

## Domain Driven Design Elements

### Bounded Contexts

The platform is organized into five primary bounded contexts, each representing a distinct business domain:

#### 1. **User Context** (Bounded Context)
- **Aggregate Root**: `User`
- **Value Objects**: `Email`, `Username`, `PasswordHash`
- **Services**: `AuthService`, `ProfileService`, `LibraryService`
- **Repository**: `UserRepository`

**Domain Events**:
- `UserRegistered`
- `UserLoggedIn`
- `ProfileUpdated`

**Anti-Corruption Layer**: 
- Feign clients isolate external service dependencies

---

#### 2. **Content Context** (Bounded Context)
- **Aggregate Root**: `Novel`, `Chapter`
- **Value Objects**: `Genre`, `Tag`, `Synopsis`
- **Services**: `NovelService`, `ChapterService`, `SearchService`, `ModerationService`
- **Repository**: `NovelRepository`, `ChapterRepository`

**Domain Events**:
- `NovelPublished`
- `ChapterCreated`
- `ContentModerated`

**Specialized Storage**:
- PostgreSQL for transactional data
- Elasticsearch for search optimization
- Digital Ocean Spaces for media files

---

#### 3. **Engagement Context** (Bounded Context)
- **Aggregate Root**: `Comment`, `Review`, `Bookmark`
- **Value Objects**: `Rating`, `Sentiment`
- **Services**: `CommentService`, `ReviewService`, `RecommendationService`
- **Repository**: `CommentRepository`, `ReviewRepository`

**Domain Events**:
- `CommentPosted`
- `ReviewCreated`
- `BookmarkAdded`
- `RecommendationGenerated`

---

#### 4. **Gamification Context** (Bounded Context)
- **Aggregate Root**: `Points`, `Achievement`, `Leaderboard`
- **Value Objects**: `Badge`, `Quest`, `Reward`
- **Services**: `PointsService`, `AchievementService`, `QuestService`
- **Repository**: `PointsRepository`, `AchievementRepository`

**Domain Events**:
- `PointsAwarded`
- `AchievementUnlocked`
- `LeaderboardUpdated`

---

#### 5. **Analytics Context** (Bounded Context)
- **Aggregate Root**: `Metrics`, `Ranking`
- **Value Objects**: `Aggregation`, `Trend`
- **Services**: `AnalyticsService`, `RankingService`, `ReportService`
- **Repository**: `MetricsRepository`, `RankingRepository`

**Domain Events**:
- `MetricsCalculated`
- `RankingUpdated`
- `ReportGenerated`

---

### Context Mapping

The relationship between bounded contexts is managed through:

1. **Published Language**: Kafka events provide a shared communication protocol
2. **Conformist**: Analytics service conforms to events published by other services
3. **Customer/Supplier**: Engagement service is a customer of Content service
4. **Anti-Corruption Layer**: Each service maintains its own view of data from other services

---

## Producer-Consumer Interactions

### Event-Driven Communication Pattern

The platform uses Kafka as a central event bus for asynchronous communication between microservices:

```
Producer (Publishes Events)          Kafka Topic         Consumer (Subscribes to Events)
───────────────────────────          ───────────         ──────────────────────────────
                                                              
User Service                      ┌──────────────┐     Engagement Service
└─ user-registered ──────────────► user-events  ├────► (Update user stats)
                                    └──────────────┘
                                    │                    Analytics Service
Content Service                     │                    (Track user growth)
└─ novel-published ────────────────►│                     
└─ chapter-created ────────────────►│                    Gamification Service
                                    │                    (Award points)
                                    │                    
Engagement Service                  │                    
└─ comment-created ────────────────►│                    
└─ review-created ─────────────────►│                    Content Service
                                    │                    (Update engagement counts)
                                    ▼                    
                                ┌──────────────┐         
                                │ kafka-topics │         
                                └──────────────┘         
                                                         Analytics Service
                                                         (Aggregate metrics)
```

### Detailed Producer-Consumer Flows

#### 1. **Content Publication Flow**

```
User Action: Author publishes a novel
     │
     ▼
Content Service (Producer)
     │
     ├─ [1] Validates author permissions
     ├─ [2] Stores novel in PostgreSQL
     ├─ [3] Indexes in Elasticsearch
     ├─ [4] Publishes event to Kafka
     │        Topic: novel-events
     │        Event: NovelCreatedEvent{novelId, title, authorId, timestamp}
     └─ [5] Returns success to API Gateway
```

**Consumers of `novel-events`**:

```
Consumer: Engagement Service
  └─ Updates user's reading recommendations
  └─ Notifies followers of new content

Consumer: Gamification Service  
  └─ Awards points to author for publishing
  └─ Triggers "Content Creator" achievement

Consumer: Analytics Service
  └─ Tracks publishing metrics
  └─ Updates content rankings
```

---

#### 2. **User Interaction Flow**

```
User Action: User comments on a chapter
     │
     ▼
Engagement Service (Producer)
     │
     ├─ [1] Validates user authentication
     ├─ [2] Stores comment in PostgreSQL
     ├─ [3] Publishes event to Kafka
     │        Topic: comment-events
     │        Event: CommentCreatedEvent{commentId, userId, chapterId, timestamp}
     └─ [4] Returns success
```

**Consumers of `comment-events`**:

```
Consumer: Content Service
  └─ Updates chapter comment count
  └─ Triggers popularity recalculation

Consumer: Gamification Service
  └─ Awards points for engagement
  └─ Unlocks "Social Butterfly" badge

Consumer: Analytics Service
  └─ Tracks engagement metrics
  └─ Updates trending content rankings
```

---

#### 3. **Cross-Service Data Retrieval Flow**

```
API Request: Get novel details with engagement metrics
     │
     ▼
API Gateway
     │
     ├─ [1] Routes to Content Service
     │        GET /api/v1/novels/{novelId}
     │        └─ Returns: Novel metadata
     │
     ├─ [2] Routes to Engagement Service
     │        GET /api/v1/ratings/novel/{novelId}
     │        └─ Returns: Average rating, review count
     │
     └─ [3] Aggregates responses
           └─ Returns: Complete novel data to client
```

**Feign Clients** (Synchronous Communication):

```java
// Engagement Service uses Feign to call Content Service with Circuit Breaker
@FeignClient(
    name = "content-service",
    fallback = ContentClientFallback.class
)
public interface ContentClient {
    @GetMapping("/api/v1/novels/{novelId}")
    @CircuitBreaker(name = "content-service", fallbackMethod = "getNovelFallback")
    NovelDTO getNovel(@PathVariable Integer novelId);
}

// Sync call when processing engagement data
Novel novel = contentClient.getNovel(chapterId);

// Fallback implementation
@Component
public class ContentClientFallback implements ContentClient {
    @Override
    public NovelDTO getNovel(Integer novelId) {
        return NovelDTO.builder()
            .id(novelId)
            .title("Content temporarily unavailable")
            .build();
    }
}
```

**Circuit Breaker Integration**:
- All Feign clients configured with `@CircuitBreaker` annotation
- Fallback methods return cached or default data
- Metrics exported to Prometheus via Micrometer

---

### Interaction Patterns Summary

#### Synchronous Communication (REST + Feign)
- **Use Case**: Immediate data retrieval needed for response
- **Examples**:
  - Getting user profile for engagement actions
  - Fetching novel metadata for comments
  - Verifying content existence before bookmarking
- **Technology**: Spring Cloud OpenFeign
- **Resilience**: Circuit Breaker (Resilience4j)

#### Asynchronous Communication (Kafka Events)
- **Use Case**: Decoupled operations, eventual consistency
- **Examples**:
  - Publishing events for analytics
  - Triggering gamification rewards
  - Updating aggregated metrics
- **Technology**: Apache Kafka
- **Guarantees**: At-least-once delivery, ordering per partition

#### Hybrid Communication
Many flows use both patterns:
1. Synchronous call for immediate data
2. Asynchronous event for downstream processing

**Example: Comment Creation**
```java
// Synchronous: Validate content exists
@PostMapping("/comments")
public ResponseEntity<Comment> createComment(@RequestBody CommentDTO dto) {
    // Sync call to content service
    Novel novel = contentClient.getNovel(dto.getNovelId());
    
    // Save comment
    Comment comment = commentService.save(dto);
    
    // Async event for downstream services
    kafkaProducer.publishCommentCreatedEvent(comment);
    
    return ResponseEntity.ok(comment);
}
```

---

### Resilience Patterns

#### 1. Circuit Breaker with Resilience4j

**Decision**: Implement Circuit Breaker pattern using Resilience4j for fault tolerance.

**Rationale**:
- **Fault Isolation**: Prevents cascading failures across services
- **Resource Protection**: Limits calls to failing services, reducing resource consumption
- **Graceful Degradation**: Provides fallback responses when services are unavailable
- **Self-Healing**: Automatically attempts to recover when services become healthy again

**Implementation**:
- **Technology**: Resilience4j (Spring Cloud Resilience4j)
- **Configuration** (applied to all service-to-service calls):

```yaml
# Circuit Breaker Configuration
resilience4j.circuitbreaker:
  instances:
    content-service:
      registerHealthIndicator: true
      slidingWindowType: COUNT_BASED
      slidingWindowSize: 20
      minimumNumberOfCalls: 10
      permittedNumberOfCallsInHalfOpenState: 5
      automaticTransitionFromOpenToHalfOpenEnabled: true
      waitDurationInOpenState: 10s
      failureRateThreshold: 50
      slowCallRateThreshold: 100
      slowCallDurationThreshold: 5s
  
  # Rate Limiter
  ratelimiter:
    instances:
      content-service:
        limitForPeriod: 10
        limitRefreshPeriod: 60s
        timeoutDuration: 5s
  
  # Retry
  retry:
    instances:
      content-service:
        maxAttempts: 3
        waitDuration: 1000ms
        retryExceptions:
          - java.net.SocketTimeoutException
          - java.util.concurrent.TimeoutException
```

**Circuit Breaker States**:
1. **CLOSED**: Normal operation, calls pass through
2. **OPEN**: Service failing, requests rejected immediately
3. **HALF_OPEN**: Testing if service recovered

**Configuration Details**:
- **Sliding Window**: 20 requests monitored
- **Failure Threshold**: 50% (10 failures out of 20)
- **Wait Duration**: 10 seconds before attempting recovery
- **Slow Call Threshold**: Calls taking >5s are considered slow
- **Minimum Calls**: 10 calls required before circuit opens

**Fallback Strategies**:
- Return cached data when available
- Return default/empty responses
- Retry with exponential backoff

#### 2. Retry Logic with Exponential Backoff
- **Implementation**: Resilience4j Retry
- **Strategy**: Exponential backoff with jitter
- **Configuration**:
  - Max Retries: 3 attempts
  - Initial Interval: 100ms
  - Multiplier: 2 (double interval each retry)
  - Max Interval: 5 seconds
  - Jitter: Random variation to prevent thundering herd

```yaml
resilience4j.retry:
  instances:
    content-service:
      maxAttempts: 3
      waitDuration: 1000ms
      exponentialBackoffMultiplier: 2
      retryExceptions:
        - org.springframework.web.client.RestClientException
        - java.net.SocketTimeoutException
```

#### 3. Bulkhead Isolation
- **Method**: Separate thread pools for different operations
- **Benefit**: One overloaded service doesn't block others

#### 4. Rate Limiting with Resilience4j
- **Implementation**: Resilience4j Rate Limiter
- **Layer**: 
  - API Gateway level (Spring Cloud Gateway)
  - Service level (Resilience4j)
  - Reverse Proxy (Nginx)
- **Configuration**:
  - API Gateway: 10 requests/second per IP
  - Service-to-Service: 100 requests/second per service
  - User-specific: 50 requests/second per authenticated user

```yaml
resilience4j.ratelimiter:
  instances:
    content-service:
      limitForPeriod: 100
      limitRefreshPeriod: 60s
      timeoutDuration: 0ms  # Non-blocking
      subscribeToPermits: true
```

**Purpose**: 
- Prevent API abuse
- Protect services from traffic spikes
- Ensure fair resource allocation

---

### Data Consistency Model

#### Strong Consistency (Within Service)
- **Scope**: Single microservice's database
- **Mechanism**: ACID transactions
- **Example**: User registration (user data, profile data)

#### Eventual Consistency (Across Services)
- **Scope**: Multi-service data synchronization
- **Mechanism**: Kafka events
- **Example**: 
  - Content Service publishes novel
  - Analytics Service eventually updates rankings
  - Gamification Service eventually awards points

#### Saga Pattern (For Complex Transactions)
- **Use Case**: Cross-service operations requiring rollback
- **Example**: Deleting user account across all services
- **Implementation**: Orchestrated via Kafka events

---

## Conclusion

The Yushan platform exemplifies modern distributed system architecture principles:

1. **Domain-Driven Design**: Services organized around business domains
2. **Event-Driven Architecture**: Asynchronous communication via Kafka
3. **Microservices Patterns**: API Gateway, Service Discovery, Config Server
4. **Observability**: Comprehensive monitoring and logging
5. **Infrastructure as Code**: Terraform-managed cloud resources
6. **Resilience**: Circuit breakers, retries, and graceful degradation

The platform is designed to scale horizontally, handle failures gracefully, and provide a rich, gamified web novel reading experience while maintaining high availability and performance.

---

## Appendix: Technology Stack Summary

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Language** | Java 21 | Platform runtime |
| **Framework** | Spring Boot 3.4.10 | Application framework |
| **Cloud** | Spring Cloud 2024.0.2 | Microservices patterns |
| **Service Discovery** | Netflix Eureka | Dynamic service registration |
| **API Gateway** | Spring Cloud Gateway | Entry point and routing |
| **Configuration** | Spring Cloud Config | Centralized configuration |
| **Circuit Breaker** | Resilience4j | Fault tolerance and resilience |
| **Database** | PostgreSQL 16 | Primary data storage |
| **Cache** | Redis 7 | Session and cache management |
| **Search** | Elasticsearch 8.11 | Full-text search |
| **Message Broker** | Apache Kafka | Event streaming |
| **ORM** | MyBatis 3.0.5 | Database access |
| **Security** | JWT + Spring Security | Authentication/Authorization |
| **Observability** | ELK Stack | Log management |
| **Monitoring** | Prometheus + Grafana | Metrics and dashboards |
| **Containerization** | Docker | Application packaging |
| **Orchestration** | Terraform | Infrastructure provisioning |
| **Cloud Provider** | Digital Ocean | Cloud hosting |

**Development Tools**:
- MapStruct: DTO mapping
- Lombok: Boilerplate reduction
- Flyway: Database migrations
- Checkstyle, SpotBugs, JaCoCo: Code quality
- OWASP Dependency Check: Security scanning

**Resilience4j Features Used**:
- Circuit Breaker: Fault isolation and graceful degradation
- Retry: Automatic retry with exponential backoff
- Rate Limiter: Request throttling
- Time Limiter: Timeout enforcement
- Bulkhead: Resource isolation via thread pool isolation
- Micrometer Integration: Prometheus metrics export
