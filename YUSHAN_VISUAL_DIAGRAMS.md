# Yushan Platform - Visual Deployment Diagrams (Mermaid)

## 1. Physical Deployment Architecture

```mermaid
graph TB
    subgraph Internet["🌐 Internet - Location"]
        ClientPC["💻 Client PC<br/>Web Browser, Mobile Apps"]
    end
    
    subgraph DigitalOcean["☁️ Digital Ocean Cloud"]
        
        subgraph APIGateway["🚪 API Gateway Server"]
            Gateway["Spring Cloud Gateway<br/>Port: 8080<br/>Docker Container"]
        end
        
        subgraph Infrastructure["🔧 Infrastructure Services Droplet"]
            Eureka["Eureka Service Registry<br/>Port: 8761<br/>Discovery & Health"]
            ConfigServer["Config Server<br/>Port: 8888<br/>Centralized Config"]
            Kafka["Apache Kafka<br/>Port: 9092<br/>Event Streaming"]
        end
        
        subgraph UserService["👤 User Service Droplet"]
            UserApp["Spring Boot 3.4.10<br/>Port: 8081<br/>Auth, Profiles, Library"]
        end
        
        subgraph ContentService["📚 Content Service Droplet"]
            ContentApp["Spring Boot 3.4.10<br/>Port: 8082<br/>Novels, Chapters, Search"]
            Elasticsearch["Elasticsearch 8.11<br/>Port: 9200<br/>Full-text Search"]
        end
        
        subgraph EngagementService["💬 Engagement Service Droplet"]
            EngagementApp["Spring Boot 3.4.10<br/>Port: 8084<br/>Comments, Reviews, Bookmarks"]
        end
        
        subgraph GamificationService["🎮 Gamification & Analytics Droplet"]
            GamificationApp["Gamification Service<br/>Port: 8085<br/>Points, Achievements"]
            AnalyticsApp["Analytics Service<br/>Port: 8083<br/>Rankings, Analytics"]
        end
        
        subgraph Databases["🗄️ Database Droplets"]
            UserDB["User DB<br/>PostgreSQL + Redis<br/>Sessions, Profiles"]
            ContentDB["Content DB<br/>PostgreSQL + Redis<br/>Novels, Metadata"]
            EngagementDB["Engagement DB<br/>PostgreSQL + Redis<br/>Comments, Reviews"]
            AnalyticsDB["Analytics DB<br/>PostgreSQL + Redis<br/>Rankings, Stats"]
            GamificationDB["Gamification DB<br/>PostgreSQL + Redis<br/>Leaderboards"]
        end
        
        subgraph Monitoring["📊 Monitoring Droplets"]
            ELK["ELK Stack<br/>Logging & Search<br/>Elasticsearch, Logstash, Kibana"]
            PrometheusGrafana["Prometheus & Grafana<br/>Metrics & Dashboards<br/>Prometheus, Grafana"]
        end
        
        subgraph Storage["💾 External Storage"]
            DOSpaces["Digital Ocean Spaces<br/>S3-Compatible<br/>Cover Images, Files"]
        end
    end
    
    ClientPC -->|HTTPS Requests| Gateway
    Gateway -->|Service Discovery| Eureka
    Gateway -->|Route| UserApp
    Gateway -->|Route| ContentApp
    Gateway -->|Route| EngagementApp
    Gateway -->|Route| GamificationApp
    Gateway -->|Route| AnalyticsApp
    
    UserApp -->|Query| UserDB
    UserApp -->|Events| Kafka
    UserApp -->|Register/Discover| Eureka
    UserApp -->|Config| ConfigServer
    
    ContentApp -->|Query| ContentDB
    ContentApp -->|Search| Elasticsearch
    ContentApp -->|Events| Kafka
    ContentApp -->|Upload/Download| DOSpaces
    ContentApp -->|Register/Discover| Eureka
    ContentApp -->|Config| ConfigServer
    
    EngagementApp -->|Query| EngagementDB
    EngagementApp -->|Events| Kafka
    EngagementApp -->|Feign Client| UserApp
    EngagementApp -->|Feign Client| ContentApp
    EngagementApp -->|Register/Discover| Eureka
    EngagementApp -->|Config| ConfigServer
    
    GamificationApp -->|Query| GamificationDB
    GamificationApp -->|Events| Kafka
    GamificationApp -->|Consumer| Kafka
    GamificationApp -->|Register/Discover| Eureka
    GamificationApp -->|Config| ConfigServer
    
    AnalyticsApp -->|Query| AnalyticsDB
    AnalyticsApp -->|Events| Kafka
    AnalyticsApp -->|Consumer| Kafka
    AnalyticsApp -->|Register/Discover| Eureka
    AnalyticsApp -->|Config| ConfigServer
    
    Kafka -->|Events| EngagementApp
    Kafka -->|Events| GamificationApp
    Kafka -->|Events| AnalyticsApp
    
    ELK -.->|Collect Logs| UserApp
    ELK -.->|Collect Logs| ContentApp
    ELK -.->|Collect Logs| EngagementApp
    ELK -.->|Collect Logs| GamificationApp
    ELK -.->|Collect Logs| AnalyticsApp
    
    PrometheusGrafana -.->|Collect Metrics| UserApp
    PrometheusGrafana -.->|Collect Metrics| ContentApp
    PrometheusGrafana -.->|Collect Metrics| EngagementApp
    PrometheusGrafana -.->|Collect Metrics| GamificationApp
    PrometheusGrafana -.->|Collect Metrics| AnalyticsApp
    
    style ClientPC fill:#e1f5ff
    style Gateway fill:#ffd700
    style Infrastructure fill:#90ee90
    style UserService fill:#87ceeb
    style ContentService fill:#87ceeb
    style EngagementService fill:#87ceeb
    style GamificationService fill:#87ceeb
    style Databases fill:#ffb6c1
    style Monitoring fill:#dda0dd
    style Storage fill:#f0e68c
```

---

## 2. Create Novel Use Case - Component Flow

```mermaid
graph LR
    subgraph UI["🎨 UI Layer"]
        UIComponent["Novel Creation UI<br/>(React Frontend)"]
    end
    
    subgraph Gateway["🚪 API Gateway"]
        GatewayRoute["Spring Cloud Gateway<br/>/api/v1/novels/*"]
    end
    
    subgraph Content["📚 Content Service"]
        Controller["NovelController<br/>createNovel()"]
        Service["NovelService<br/>createNovel()"]
        MyBatis["MyBatis Mapper<br/>insertNovel()"]
        Producer["Kafka Producer<br/>novel-events"]
        SearchService["SearchService<br/>indexNovel()"]
        S3Service["S3Service<br/>uploadCoverImage()"]
        CacheService["RedisCache<br/>cacheNovelMetadata()"]
    end
    
    subgraph Database["🗄️ PostgreSQL"]
        NovelTable["novels table<br/>INSERT novel data"]
    end
    
    subgraph Elasticsearch["🔍 Elasticsearch"]
        NovelIndex["novels index<br/>Index document"]
    end
    
    subgraph Cache["⚡ Redis"]
        NovelCache["novel:{id}<br/>Cache metadata"]
    end
    
    subgraph Storage["💾 Digital Ocean Spaces"]
        CoverImage["covers/{novelId}.jpg<br/>Store image"]
    end
    
    subgraph Kafka["📮 Kafka"]
        NovelEventsTopic["novel-events topic<br/>Publish event"]
    end
    
    subgraph Consumers["👂 Event Consumers"]
        EngagementListener["Engagement Service<br/>@KafkaListener<br/>Update recommendations"]
        GamificationListener["Gamification Service<br/>@KafkaListener<br/>Award achievement"]
        AnalyticsListener["Analytics Service<br/>@KafkaListener<br/>Update rankings"]
    end
    
    UIComponent -->|POST /api/v1/novels| GatewayRoute
    GatewayRoute -->|Route to content-service| Controller
    
    Controller --> Service
    Service --> MyBatis
    MyBatis --> NovelTable
    
    Service --> SearchService
    SearchService --> NovelIndex
    
    Service --> CacheService
    CacheService --> NovelCache
    
    Service --> S3Service
    S3Service --> CoverImage
    
    Service --> Producer
    Producer --> NovelEventsTopic
    
    NovelEventsTopic --> EngagementListener
    NovelEventsTopic --> GamificationListener
    NovelEventsTopic --> AnalyticsListener
    
    style UI fill:#e1f5ff
    style Gateway fill:#ffd700
    style Content fill:#87ceeb
    style Database fill:#ffb6c1
    style Elasticsearch fill:#98fb98
    style Cache fill:#f0e68c
    style Storage fill:#f0e68c
    style Kafka fill:#ffd700
    style Consumers fill:#dda0dd
```

---

## 3. Detailed Component Architecture

```mermaid
graph TB
    subgraph ContentService["📚 Content Service - Port 8082"]
        
        subgraph Layer1["🌐 API Layer"]
            NovelController["NovelController<br/>+createNovel()<br/>+getNovel()<br/>+updateNovel()<br/>+deleteNovel()"]
            ChapterController["ChapterController<br/>+createChapter()<br/>+getChapter()<br/>+updateChapter()<br/>+deleteChapter()"]
            SearchController["SearchController<br/>+searchNovels()<br/>+searchChapters()"]
        end
        
        subgraph Layer2["📋 Service Layer"]
            NovelService["NovelService<br/>- Novel CRUD operations<br/>- Validation logic"]
            ChapterService["ChapterService<br/>- Chapter management<br/>- Content handling"]
            SearchService["SearchService<br/>- Elasticsearch integration<br/>- Search indexing"]
            StorageService["StorageService<br/>- S3 upload/download<br/>- File management"]
            CacheService["CacheService<br/>- Redis caching<br/>- TTL management"]
            EventService["KafkaEventProducerService<br/>- Event publishing<br/>- Topic management"]
        end
        
        subgraph Layer3["🗄️ Data Layer"]
            NovelMapper["MyBatis NovelMapper<br/>- SELECT, INSERT, UPDATE, DELETE"]
            ChapterMapper["MyBatis ChapterMapper<br/>- Chapter operations"]
            GenreMapper["MyBatis GenreMapper<br/>- Genre operations"]
            AuthorMapper["MyBatis AuthorMapper<br/>- Author operations"]
        end
        
        subgraph Layer4["🔗 Integration Layer"]
            ElasticsearchClient["ElasticsearchRestClient<br/>- Index documents<br/>- Search queries"]
            S3Client["Digital Ocean Spaces Client<br/>- Upload images<br/>- Download files"]
            RedisClient["RedisTemplate<br/>- Cache operations<br/>- Session management"]
            KafkaProducer["KafkaTemplate<br/>- Event publishing"]
        end
        
        NovelController --> NovelService
        ChapterController --> ChapterService
        SearchController --> SearchService
        
        NovelService --> NovelMapper
        NovelService --> EventService
        ChapterService --> ChapterMapper
        ChapterService --> EventService
        SearchService --> ElasticsearchClient
        
        NovelService --> StorageService
        NovelService --> CacheService
        
        StorageService --> S3Client
        CacheService --> RedisClient
        EventService --> KafkaProducer
        
    end
    
    subgraph ExternalSystems["🌍 External Systems"]
        PostgreSQL["PostgreSQL Database<br/>novels, chapters, genres tables"]
        Elasticsearch["Elasticsearch Cluster<br/>novels, chapters indexes"]
        Redis["Redis Cache<br/>novel metadata cache"]
        S3Storage["Digital Ocean Spaces<br/>cover images storage"]
        KafkaBroker["Kafka Broker<br/>novel-events, chapter-events"]
    end
    
    NovelMapper --> PostgreSQL
    ChapterMapper --> PostgreSQL
    GenreMapper --> PostgreSQL
    AuthorMapper --> PostgreSQL
    
    ElasticsearchClient --> Elasticsearch
    S3Client --> S3Storage
    RedisClient --> Redis
    KafkaProducer --> KafkaBroker
    
    style Layer1 fill:#e1f5ff
    style Layer2 fill:#87ceeb
    style Layer3 fill:#98fb98
    style Layer4 fill:#f0e68c
    style ExternalSystems fill:#dda0dd
```

---

## 4. Kafka Event Flow - Create Novel Use Case

```mermaid
sequenceDiagram
    participant UI as 🖥️ Client UI
    participant GW as 🚪 API Gateway
    participant CS as 📚 Content Service
    participant DB as 🗄️ PostgreSQL
    participant ES as 🔍 Elasticsearch
    participant REDIS as ⚡ Redis
    participant S3 as 💾 S3 Storage
    participant KAFKA as 📮 Kafka
    participant ENG as 💬 Engagement
    participant GAM as 🎮 Gamification
    participant ANA as 📊 Analytics
    
    UI->>GW: POST /api/v1/novels
    GW->>CS: Route to content-service
    
    CS->>CS: Validate request
    CS->>CS: Check author permissions
    
    CS->>DB: INSERT INTO novels
    DB-->>CS: Return novel_id
    
    CS->>ES: Index novel document
    ES-->>CS: Index created
    
    CS->>REDIS: Cache novel metadata
    REDIS-->>CS: Cache confirmed
    
    CS->>S3: Upload cover image
    S3-->>CS: Return image URL
    
    CS->>KAFKA: Publish NovelCreatedEvent
    
    Note over ENG,GAM,ANA: Parallel Event Consumption
    
    KAFKA->>ENG: Consume novel-events
    ENG->>ENG: Update reading recommendations
    ENG->>ENG: Notify followers
    
    KAFKA->>GAM: Consume novel-events
    GAM->>GAM: Award "Content Creator" achievement
    GAM->>GAM: Add points to user
    GAM->>GAM: Update leaderboard
    
    KAFKA->>ANA: Consume novel-events
    ANA->>ANA: Track publishing metrics
    ANA->>ANA: Update content rankings
    ANA->>ANA: Calculate author statistics
    
    CS-->>GW: Return novel with ID and image URL
    GW-->>UI: HTTP 201 Created
    
    Note over UI: User sees new novel with cover image
```

---

## 5. Service-to-Service Communication Patterns

```mermaid
graph TB
    subgraph SyncCom["🔄 Synchronous Communication (Feign)"]
        
        subgraph EngagementService["💬 Engagement Service"]
            EC1["CommentController<br/>+createComment()"]
            EC2["ReviewController<br/>+createReview()"]
        end
        
        subgraph UserService["👤 User Service"]
            UserFeign["FeignClient<br/>getUserProfile()"]
        end
        
        subgraph ContentService["📚 Content Service"]
            ContentFeign["FeignClient<br/>getNovel(), getChapter()"]
        end
        
        EC1 -->|Feign Call| UserFeign
        EC1 -->|Feign Call| ContentFeign
        EC2 -->|Feign Call| UserFeign
        EC2 -->|Feign Call| ContentFeign
        
    end
    
    subgraph AsyncCom["📢 Asynchronous Communication (Kafka)"]
        
        subgraph Publishers["📤 Event Publishers"]
            CSPub["Content Service<br/>Publishes novel-events"]
            ESPub["Engagement Service<br/>Publishes comment-events"]
            USPub["User Service<br/>Publishes user.events"]
        end
        
        subgraph KafkaBroker["📮 Kafka Topics"]
            KafkaTopic1["novel-events<br/>chapter-events"]
            KafkaTopic2["comment-events<br/>review-events<br/>vote-events"]
            KafkaTopic3["user.events<br/>active"]
        end
        
        subgraph Consumers["👂 Event Consumers"]
            EGC["Engagement Service<br/>Consumes novel-events"]
            GMC["Gamification Service<br/>Consumes comment-events<br/>review-events<br/>vote-events"]
            ANC["Analytics Service<br/>Consumes all events"]
        end
        
        CSPub --> KafkaTopic1
        ESPub --> KafkaTopic2
        USPub --> KafkaTopic3
        
        KafkaTopic1 --> EGC
        KafkaTopic2 --> GMC
        KafkaTopic3 --> ANC
        
    end
    
    style SyncCom fill:#87ceeb
    style AsyncCom fill:#dda0dd
    style Publishers fill:#90ee90
    style Consumers fill:#ffb6c1
```

---

## 6. Infrastructure Services - Eureka & Config Server

```mermaid
graph TB
    subgraph Clients["📱 Eureka Clients (Service Registry)"]
        UserReg["User Service<br/>Port 8081<br/>Registered as: user-service"]
        ContentReg["Content Service<br/>Port 8082<br/>Registered as: content-service"]
        EngagementReg["Engagement Service<br/>Port 8084<br/>Registered as: engagement-service"]
        GamificationReg["Gamification Service<br/>Port 8085<br/>Registered as: gamification-service"]
        AnalyticsReg["Analytics Service<br/>Port 8083<br/>Registered as: analytics-service"]
        GatewayReg["API Gateway<br/>Port 8080<br/>Registered as: api-gateway"]
        ConfigReg["Config Server<br/>Port 8888<br/>Registered as: config-server"]
    end
    
    subgraph EurekaServer["🏛️ Eureka Server - Port 8761"]
        EurekaRegistry["Service Registry<br/>- Health Monitoring<br/>- Load Balancing<br/>- Instance Management"]
    end
    
    subgraph ConfigServer["⚙️ Config Server - Port 8888"]
        ConfigRepo["Configuration Repository<br/>- application.yml<br/>- Database configs<br/>- Feature flags<br/>- Secrets management"]
    end
    
    UserReg -->|Register & Heartbeat| EurekaRegistry
    ContentReg -->|Register & Heartbeat| EurekaRegistry
    EngagementReg -->|Register & Heartbeat| EurekaRegistry
    GamificationReg -->|Register & Heartbeat| EurekaRegistry
    AnalyticsReg -->|Register & Heartbeat| EurekaRegistry
    GatewayReg -->|Register & Heartbeat| EurekaRegistry
    ConfigReg -->|Register & Heartbeat| EurekaRegistry
    
    UserReg -.->|Fetch Config| ConfigRepo
    ContentReg -.->|Fetch Config| ConfigRepo
    EngagementReg -.->|Fetch Config| ConfigRepo
    GamificationReg -.->|Fetch Config| ConfigRepo
    AnalyticsReg -.->|Fetch Config| ConfigRepo
    
    GatewayReg -.->|Discover Services| EurekaRegistry
    GatewayReg -->|Route Requests| UserReg
    GatewayReg -->|Route Requests| ContentReg
    GatewayReg -->|Route Requests| EngagementReg
    GatewayReg -->|Route Requests| GamificationReg
    GatewayReg -->|Route Requests| AnalyticsReg
    
    style EurekaServer fill:#ffd700
    style ConfigServer fill:#90ee90
    style Clients fill:#87ceeb
```

---

## Summary of Deployment Architecture

| Service | Port | Technologies | Dependencies |
|---------|------|--------------|--------------|
| API Gateway | 8080 | Spring Cloud Gateway | Eureka |
| Config Server | 8888 | Spring Cloud Config | Git Repository |
| Eureka Server | 8761 | Netflix Eureka | - |
| User Service | 8081 | Spring Boot, MyBatis, JWT | PostgreSQL, Redis, Kafka |
| Content Service | 8082 | Spring Boot, MyBatis, Elasticsearch | PostgreSQL, Redis, Elasticsearch, S3, Kafka |
| Engagement Service | 8084 | Spring Boot, MyBatis, Feign | PostgreSQL, Redis, Kafka |
| Gamification Service | 8085 | Spring Boot, MyBatis | PostgreSQL, Redis, Kafka |
| Analytics Service | 8083 | Spring Boot, MyBatis | PostgreSQL, Redis, Kafka |

### Key Patterns:
- ✅ **Database-Per-Service**: Each service has its own PostgreSQL database
- ✅ **Event-Driven Architecture**: Asynchronous communication via Kafka
- ✅ **Synchronous Communication**: Feign clients for immediate data needs
- ✅ **Circuit Breaker**: Resilience4j for fault tolerance
- ✅ **Service Discovery**: Eureka for dynamic service location
- ✅ **Centralized Config**: Config Server for environment management
- ✅ **API Gateway**: Single entry point for all clients

