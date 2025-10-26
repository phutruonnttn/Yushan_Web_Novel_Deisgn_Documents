# Yushan Platform - UML Deployment Diagrams

## Diagram 1: Physical Deployment Architecture (Deployment Elements)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        <<location>> Internet                                │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │                         Client PC                               │      │
│  │                                                                 │      │
│  │  Note:                                                          │      │
│  │  - Web Browser (Chrome, Firefox, etc.)                       │      │
│  │  - Mobile Apps                                                  │      │
│  │  - User Interface                                                │      │
│  └──────────────────────────────────────────────────────────────────┘      │
└────────────────────────────────────┬─────────────────────────────────────────┘
                                     │
                                     │ HTTPS
                                     │
┌────────────────────────────────────┴─────────────────────────────────────────┐
│                       <<location>> Digital Ocean Cloud                      │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │                    API Gateway Server                            │      │
│  │                                                                 │      │
│  │  Note:                                                          │      │
│  │  Model = DigitalOcean Droplet                                  │      │
│  │  CPU = 2 vCPU                                                   │      │
│  │  RAM = 2 GB                                                     │      │
│  │  OS = Ubuntu 22.04                                              │      │
│  │  SW = Spring Cloud Gateway, Docker                             │      │
│  │  Port = 8080                                                    │      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │              Infrastructure Services Droplet                     │      │
│  │                                                                 │      │
│  │  Note:                                                          │      │
│  │  Model = DigitalOcean Droplet                                  │      │
│  │  CPU = 4 vCPU                                                   │      │
│  │  RAM = 8 GB                                                     │      │
│  │  OS = Ubuntu 22.04                                              │      │
│  │  SW = Eureka, Config Server, Kafka, Docker                     │      │
│  │  Ports = 8761, 8888, 9092                                       │      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │                   User Service Droplet                          │      │
│  │                                                                 │      │
│  │  Note:                                                          │      │
│  │  Model = DigitalOcean Droplet                                  │      │
│  │  CPU = 2 vCPU                                                   │      │
│  │  RAM = 4 GB                                                     │      │
│  │  OS = Ubuntu 22.04                                              │      │
│  │  SW = Spring Boot 3.4.10, MyBatis, Docker                     │      │
│  │  Port = 8081                                                     │      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │               Content Service Droplet                           │      │
│  │                                                                 │      │
│  │  Note:                                                          │      │
│  │  Model = DigitalOcean Droplet                                  │      │
│  │  CPU = 4 vCPU                                                   │      │
│  │  RAM = 8 GB                                                     │      │
│  │  OS = Ubuntu 22.04                                              │      │
│  │  SW = Spring Boot 3.4.10, MyBatis, Elasticsearch, Docker      │      │
│  │  Port = 8082                                                     │      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │            Engagement Service Droplet                         │      │
│  │                                                                 │      │
│  │  Note:                                                          │      │
│  │  Model = DigitalOcean Droplet                                  │      │
│  │  CPU = 2 vCPU                                                   │      │
│  │  RAM = 4 GB                                                     │      │
│  │  OS = Ubuntu 22.04                                              │      │
│  │  SW = Spring Boot 3.4.10, MyBatis, Docker                     │      │
│  │  Port = 8084                                                     │      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │    Gamification & Analytics Services Droplet (Shared)           │      │
│  │                                                                 │      │
│  │  Note:                                                          │      │
│  │  Model = DigitalOcean Droplet                                  │      │
│  │  CPU = 2 vCPU                                                   │      │
│  │  RAM = 4 GB                                                     │      │
│  │  OS = Ubuntu 22.04                                              │      │
│  │  SW = Spring Boot 3.4.10, MyBatis, Docker                     │      │
│  │  Ports = 8083, 8085                                             │      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │                    Database Droplets                            │      │
│  │                                                                 │      │
│  │  User DB:                                                       │      │
│  │  - PostgreSQL 16 + Redis (Session)                             │      │
│  │  - Port 5432, 6379                                              │      │
│  │                                                                 │      │
│  │  Content DB:                                                    │      │
│  │  - PostgreSQL 16 + Redis (Cache) + Elasticsearch 8.11          │      │
│  │  - Port 5432, 6379, 9200                                       │      │
│  │                                                                 │      │
│  │  Engagement DB:                                                 │      │
│  │  - PostgreSQL 16 + Redis                                        │      │
│  │  - Port 5432, 6379                                              │      │
│  │                                                                 │      │
│  │  Analytics DB:                                                  │      │
│  │  - PostgreSQL 16 + Redis                                        │      │
│  │  - Port 5432, 6379                                              │      │
│  │                                                                 │      │
│  │  Gamification DB:                                                │      │
│  │  - PostgreSQL 16 + Redis (Leaderboards)                        │      │
│  │  - Port 5432, 6379                                              │      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │                Monitoring Droplets                              │      │
│  │                                                                 │      │
│  │  ELK Stack:                                                     │      │
│  │  - Elasticsearch, Logstash, Kibana                             │      │
│  │                                                                 │      │
│  │  Prometheus/Grafana:                                            │      │
│  │  - Prometheus, Grafana                                          │      │
│  └──────────────────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Diagram 2: Component Diagram - Functional Elements (Create Novel Use Case)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                          <<JSP>> Novel Creation UI                         │
│                                                                             │
│                          +createNovel()                                     │
│                                 │                                           │
│                          <<Servlet>> NovelController                        │
│                                 │                                           │
│                          <<Service>> NovelService                          │
│                                 │                                           │
├─────────────────────────────────┼───────────────────────────────────────────┤
│                                 │                                           │
│                     <<FeignClient>> UserServiceClient                       │
│                          +getUserProfile()                                  │
│                                 │                                           │
│                     <<FeignClient>> ContentServiceClient                    │
│                          +validateAuthor()                                  │
│                                 │                                           │
├─────────────────────────────────┼───────────────────────────────────────────┤
│                                 │                                           │
│                    <<MyBatis>> NovelMapper                                  │
│                          +insertNovel()                                      │
│                                 │                                           │
│                    <<PostgreSQL>> Novel Table                               │
│                                                                             │
├─────────────────────────────────┴───────────────────────────────────────────┤
│                                                                             │
│                    <<Elasticsearch>> Novel Index                            │
│                          +indexNovel()                                      │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                    <<Redis>> Novel Cache                                    │
│                          +cacheNovelMetadata()                             │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│           <<DigitalOceanSpaces>> Image Storage (S3 Compatible)            │
│                          +uploadCoverImage()                                │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│              <<Kafka>> Kafka Event Producer                                │
│                          +publishNovelCreatedEvent()                         │
│                                                                             │
│               Topic: novel-events                                           │
│               Partition Key: novelId                                       │
│                                                                             │
│               ┌─────────────┬─────────────┬─────────────┐                   │
│               │   Analytics│Gamification│Engagement   │                   │
│               │   Consumer │  Consumer  │  Consumer  │                   │
│               └─────────────┴─────────────┴─────────────┘                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Diagram 3: Detailed Deployment Diagram - Create Novel Use Case

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    <<location>> Digital Ocean Cloud                         │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  API Gateway Server  ──deploy────────────────────┐                         │
│    Port 8080                                      │                         │
│    Spring Cloud Gateway                           │                         │
│    Docker Container                               │                         │
│                                                   │                         │
│         └──deploy────→ <<JAR>> GatewayRouting     │                         │
│                         manifest                  │                         │
│                         <<Route>> User Service Route                        │
│                         <<Route>> Content Service Route                     │
│                         <<Route>> Engagement Service Route                   │
│                         <<Route>> Gamification Service Route                 │
│                         <<Route>> Analytics Service Route                    │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│  Content Service Droplet  ──deploy────────────┐                            │
│    Port 8082                                   │                            │
│    Spring Boot 3.4.10                          │                            │
│    Docker Container                             │                            │
│                                                 │                            │
│    └──deploy────→ <<JAR>> NovelController      │                            │
│                   manifest                      │                            │
│                   <<RestController>> NovelController                        │
│                   +createNovel()                │                            │
│                   +getNovel()                   │                            │
│                   +updateNovel()                 │                            │
│                   +searchNovels()                │                            │
│                                                 │                            │
│    └──deploy────→ <<JAR>> NovelService         │                            │
│                   manifest                      │                            │
│                   <<Service>> NovelService      │                            │
│                   <<Service>> ChapterService    │                            │
│                   <<Service>> SearchService     │                            │
│                   <<Service>> KafkaEventProducerService                      │
│                                                 │                            │
│    └──deploy────→ <<JAR>> NovelData            │                            │
│                   manifest                      │                            │
│                   <<MyBatis>> NovelMapper       │                            │
│                   <<MyBatis>> ChapterMapper     │                            │
│                   <<MyBatis>> GenreMapper      │                            │
│                                                 │                            │
│    └──deploy────→ <<JAR>> Integration          │                            │
│                   manifest                      │                            │
│                   <<Elasticsearch>> ElasticsearchConfig                     │
│                   +indexNovel()                 │                            │
│                   +searchNovels()               │                            │
│                                                 │                            │
│                   <<DigitalOceanSpaces>> S3Config                            │
│                   +uploadCoverImage()           │                            │
│                   +uploadChapterContent()        │                            │
│                                                 │                            │
│                   <<Redis>> RedisConfig         │                            │
│                   +cacheNovelMetadata()         │                            │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│  Kafka Message Broker  ──deploy───────────────┐                            │
│    Port 9092                                  │                            │
│    Apache Kafka                               │                            │
│    Docker Container                            │                            │
│                                               │                            │
│    └──deploy────→ <<Topic>> novel-events      │                            │
│                   manifest                    │                            │
│                   <<Event>> NovelCreatedEvent │                            │
│                   <<Event>> ChapterCreatedEvent                             │
│                   <<Event>> NovelPublishedEvent                              │
│                                               │                            │
│    └──deploy────→ <<Topic>> comment-events    │                            │
│                   manifest                    │                            │
│                   <<Event>> CommentCreatedEvent                              │
│                                               │                            │
│    └──deploy────→ <<Topic>> review-events     │                            │
│                   manifest                    │                            │
│                   <<Event>> ReviewCreatedEvent │                            │
│                                               │                            │
│    └──deploy────→ <<Topic>> vote-events       │                            │
│                   manifest                    │                            │
│                   <<Event>> VoteCreatedEvent │                            │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│  Content Database  ──deploy────────────┐                                    │
│    Port 5432                           │                                    │
│    PostgreSQL 16                       │                                    │
│    Docker Container                     │                                    │
│                                        │                                    │
│    └──deploy────→ <<Table>> novels     │                                    │
│                   manifest             │                                    │
│                   <<Entity>> Novel     │                                    │
│                   id, title, synopsis, author_id, status, created_at        │
│                                        │                                    │
│    └──deploy────→ <<Table>> chapters   │                                    │
│                   manifest             │                                    │
│                   <<Entity>> Chapter │                                    │
│                   id, novel_id, title, content, order, created_at          │
│                                        │                                    │
│    └──deploy────→ <<Table>> genres      │                                    │
│                   manifest             │                                    │
│                   <<Entity>> Genre    │                                    │
│                   id, name, description                                    │
│                                        │                                    │
│    └──deploy────→ <<Table>> novel_genres                                   │
│                   manifest             │                                    │
│                   <<Entity>> NovelGenre                                    │
│                   novel_id, genre_id                                       │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│  Elasticsearch Cluster  ──deploy────────────┐                             │
│    Port 9200                                 │                             │
│    Elasticsearch 8.11                        │                             │
│    Docker Container                          │                             │
│                                              │                             │
│    └──deploy────→ <<Index>> novels           │                             │
│                   manifest                   │                             │
│                   <<Document>> NovelDocument │                             │
│                   title, synopsis, author, genres, status, tags            │
│                                              │                             │
│    └──deploy────→ <<Index>> chapters          │                             │
│                   manifest                   │                             │
│                   <<Document>> ChapterDocument                              │
│                   novel_id, title, content   │                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│  Engagement Service Droplet  ──deploy────────┐                            │
│    Port 8084                                 │                            │
│    Spring Boot 3.4.10                        │                            │
│    Docker Container                           │                            │
│                                              │                            │
│    └──deploy────→ <<JAR>> EventListener     │                            │
│                   manifest                   │                            │
│                   <<KafkaListener>> NovelCreatedListener                   │
│                   @KafkaListener(topics = "novel-events")                  │
│                   +handleNovelCreatedEvent()                               │
│                   └─ Update reading recommendations                        │
│                   └─ Notify followers                                       │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│  Gamification Service Droplet  ──deploy───────┐                           │
│    Port 8085                                  │                           │
│    Spring Boot 3.4.10                         │                           │
│    Docker Container                            │                           │
│                                               │                           │
│    └──deploy────→ <<JAR>> EventListener      │                           │
│                    manifest                  │                           │
│                    <<KafkaListener>> NovelCreatedListener                  │
│                    @KafkaListener(topics = "novel-events")                │
│                    +handleNovelCreatedEvent()                             │
│                    └─ Award "Content Creator" achievement                  │
│                    └─ Award points for publishing                         │
│                    └─ Update leaderboard                                    │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│  Analytics Service Droplet  ──deploy─────────┐                           │
│    Port 8083                                 │                           │
│    Spring Boot 3.4.10                        │                           │
│    Docker Container                          │                           │
│                                              │                           │
│    └──deploy────→ <<JAR>> EventListener     │                           │
│                    manifest                 │                           │
│                    <<KafkaListener>> NovelCreatedListener                 │
│                    @KafkaListener(topics = "novel-events")               │
│                    +handleNovelCreatedEvent()                            │
│                    └─ Track publishing metrics                            │
│                    └─ Update content rankings                             │
│                    └─ Update author statistics                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Flow Description: Create Novel Use Case

### Step 1: User Request via Client PC
- **Client** (Browser/Mobile App) sends HTTP POST to API Gateway
- Request includes novel data (title, synopsis, genres, cover image)
- Header includes JWT token for authentication

### Step 2: API Gateway Routing
- **API Gateway** (Port 8080) receives request
- Validates JWT token
- Routes to Content Service based on path `/api/v1/novels`

### Step 3: Content Service Processing
- **NovelController** receives request
- Calls **NovelService.createNovel()**
- Service validates data and author permissions

### Step 4: Database Persistence
- **MyBatis Mapper** inserts into PostgreSQL
- **Novel** entity saved to `novels` table
- Returns novel with generated ID

### Step 5: Elasticsearch Indexing
- **SearchService** indexes novel in Elasticsearch
- Creates **NovelDocument** with searchable fields
- Index name: `novels`

### Step 6: Redis Caching
- Novel metadata cached in Redis
- Key: `novel:{id}`, TTL: 5 minutes
- Improves subsequent read performance

### Step 7: Digital Ocean Spaces Upload
- **S3Client** uploads cover image
- Stores in bucket with key: `covers/{novelId}/{timestamp}.jpg`
- Returns public URL

### Step 8: Kafka Event Publication
- **KafkaEventProducerService** publishes event
- Topic: `novel-events`
- Event Type: `NovelCreatedEvent`
- Payload: {novelId, title, authorId, timestamp}

### Step 9: Event Consumption (Parallel)
- **Engagement Service**: Updates reading recommendations
- **Gamification Service**: Awards achievement "Content Creator", gives points
- **Analytics Service**: Updates content rankings, tracks metrics

### Step 10: Response to User
- Content Service returns created novel with ID
- API Gateway returns 201 Created status
- Client receives novel data with URL to cover image

---

## Technology Mapping

| Component Type | Technology | Purpose |
|----------------|------------|---------|
| Web Layer | Spring MVC | REST Controllers |
| Service Layer | Spring Service | Business Logic |
| Integration | Apache Kafka | Event-Driven Communication |
| Search | Elasticsearch 8.11 | Full-text Search |
| Persistence | MyBatis 3.0.5 | Database ORM |
| Database | PostgreSQL 16 | Relational Data |
| Cache | Redis 7 | Session & Metadata Cache |
| Storage | Digital Ocean Spaces | Media Files (S3-compatible) |
| Gateway | Spring Cloud Gateway | Routing & Load Balancing |
| Discovery | Netflix Eureka | Service Registration |
| Container | Docker | Application Packaging |
| Platform | Digital Ocean Droplets | Cloud Infrastructure |

---

## Key Architectural Patterns

1. **Database-Per-Service**: Each service has its own PostgreSQL database
2. **Event-Driven Architecture**: Services communicate via Kafka events
3. **Synchronous Feign**: Immediate data retrieval via REST calls
4. **Circuit Breaker**: Resilience4j for fault tolerance
5. **CQRS**: Elasticsearch for queries, PostgreSQL for commands
6. **Microservices**: Independent deployable services
7. **API Gateway**: Single entry point for all clients
8. **Service Discovery**: Dynamic service location via Eureka
