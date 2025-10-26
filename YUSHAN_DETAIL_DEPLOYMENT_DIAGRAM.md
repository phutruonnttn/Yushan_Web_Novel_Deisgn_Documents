# Yushan Platform - Detail Deployment Elements

## Detailed Deployment Diagram for "Create Novel" Use Case

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    <<location>> Digital Ocean Cloud                         │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Content Service Droplet  ──deploy────────┐                                │
│    (yushan-content-service:8082)           │                                │
│    Docker Container                         │                                │
│    Spring Boot 3.4.10                      │                                │
│                                             │                                │
│         └──deploy────→ <<JAR>> ContentServiceApplication                   │
│                         manifest                                            │
│                                                                             │
│                         ┌─ <<RestController>> NovelController              │
│                         │   +createNovel() : ResponseEntity                │
│                         │   +getNovel() : ResponseEntity                    │
│                         │   +updateNovel() : ResponseEntity                │
│                         │   +deleteNovel() : ResponseEntity                │
│                         │                                                   │
│                         ├─ <<RestController>> ChapterController            │
│                         │   +createChapter() : ResponseEntity               │
│                         │   +getChapter() : ResponseEntity                  │
│                         │   +updateChapter() : ResponseEntity               │
│                         │   +deleteChapter() : ResponseEntity                │
│                         │                                                   │
│                         ├─ <<RestController>> SearchController              │
│                         │   +searchNovels() : ResponseEntity                │
│                         │   +searchChapters() : ResponseEntity              │
│                         │                                                   │
│                         ├─ <<Service>> NovelService                         │
│                         │   +createNovel() : Novel                          │
│                         │   +getNovel() : Novel                             │
│                         │   +updateNovel() : Novel                          │
│                         │   +deleteNovel() : void                           │
│                         │   +validateNovel() : boolean                      │
│                         │                                                   │
│                         ├─ <<Service>> ChapterService                       │
│                         │   +createChapter() : Chapter                      │
│                         │   +getChapter() : Chapter                         │
│                         │   +updateChapter() : Chapter                     │
│                         │                                                   │
│                         ├─ <<Service>> SearchService                        │
│                         │   +indexNovel() : void                            │
│                         │   +searchNovels() : List<Novel>                    │
│                         │   +deleteFromIndex() : void                       │
│                         │                                                   │
│                         ├─ <<Service>> KafkaEventProducerService            │
│                         │   +publishNovelCreatedEvent() : void               │
│                         │   +publishChapterCreatedEvent() : void             │
│                         │   +publishNovelViewEvent() : void                  │
│                         │                                                   │
│                         ├─ <<Service>> StorageService                       │
│                         │   +uploadCoverImage() : String                    │
│                         │   +downloadImage() : byte[]                       │
│                         │   +deleteImage() : void                           │
│                         │                                                   │
│                         ├─ <<Service>> CacheService                         │
│                         │   +cacheNovelMetadata() : void                    │
│                         │   +getCachedNovel() : Novel                       │
│                         │   +invalidateCache() : void                       │
│                         │                                                   │
│                         ├─ <<MyBatis>> NovelMapper                          │
│                         │   +selectById() : Novel                            │
│                         │   +insertNovel() : int                             │
│                         │   +updateNovel() : int                             │
│                         │   +deleteNovel() : int                             │
│                         │   +selectAll() : List<Novel>                       │
│                         │                                                   │
│                         ├─ <<MyBatis>> ChapterMapper                        │
│                         │   +selectByNovelId() : List<Chapter>              │
│                         │   +insertChapter() : int                           │
│                         │   +updateChapter() : int                           │
│                         │   +deleteChapter() : int                           │
│                         │                                                   │
│                         ├─ <<MyBatis>> GenreMapper                          │
│                         │   +selectAll() : List<Genre>                       │
│                         │   +selectById() : Genre                            │
│                         │                                                   │
│                         ├─ <<MyBatis>> AuthorMapper                        │
│                         │   +selectById() : Author                          │
│                         │   +selectAll() : List<Author>                      │
│                         │                                                   │
│                         ├─ <<Configuration>> ElasticsearchConfig             │
│                         │   +elasticsearchRestClient() : RestClient           │
│                         │                                                   │
│                         ├─ <<Configuration>> RedisConfig                    │
│                         │   +redisTemplate() : RedisTemplate                │
│                         │   +redisConnectionFactory() : ConnectionFactory     │
│                         │                                                   │
│                         ├─ <<Configuration>> KafkaConfig                    │
│                         │   +kafkaTemplate() : KafkaTemplate                 │
│                         │   +producerFactory() : ProducerFactory             │
│                         │                                                   │
│                         ├─ <<Configuration>> S3Config                       │
│                         │   +amazonS3() : AmazonS3                          │
│                         │   +s3Client() : S3Client                           │
│                         │                                                   │
│                         ├─ <<Configuration>> MyBatisConfig                  │
│                         │   +sqlSessionFactory() : SqlSessionFactory         │
│                         │   +dataSource() : DataSource                       │
│                         │                                                   │
│                         └─ <<Configuration>> SecurityConfig                 │
│                             +jwtAuthenticationFilter() : Filter             │
│                             +jwtTokenProvider() : JwtTokenProvider           │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Engagement Service Droplet  ──deploy────┐                                  │
│    (yushan-engagement-service:8084)       │                                  │
│    Docker Container                       │                                  │
│    Spring Boot 3.4.10                     │                                  │
│                                           │                                  │
│         └──deploy────→ <<JAR>> EngagementServiceApplication                │
│                         manifest                                            │
│                                                                             │
│                         ┌─ <<RestController>> CommentController            │
│                         │   +createComment() : ResponseEntity                │
│                         │   +getComment() : ResponseEntity                  │
│                         │   +updateComment() : ResponseEntity               │
│                         │   +deleteComment() : ResponseEntity                │
│                         │                                                   │
│                         ├─ <<RestController>> ReviewController              │
│                         │   +createReview() : ResponseEntity                  │
│                         │   +getReview() : ResponseEntity                    │
│                         │   +getReviewsByNovel() : ResponseEntity           │
│                         │                                                   │
│                         ├─ <<RestController>> BookmarkController            │
│                         │   +addBookmark() : ResponseEntity                  │
│                         │   +removeBookmark() : ResponseEntity              │
│                         │   +getBookmarks() : ResponseEntity                 │
│                         │                                                   │
│                         ├─ <<Service>> CommentService                       │
│                         │   +createComment() : Comment                       │
│                         │   +getComment() : Comment                         │
│                         │   +updateComment() : Comment                      │
│                         │   +deleteComment() : void                         │
│                         │                                                   │
│                         ├─ <<Service>> ReviewService                        │
│                         │   +createReview() : Review                         │
│                         │   +getReview() : Review                           │
│                         │   +calculateAverageRating() : double               │
│                         │                                                   │
│                         ├─ <<Service>> KafkaEventProducerService            │
│                         │   +publishCommentCreatedEvent() : void            │
│                         │   +publishReviewCreatedEvent() : void             │
│                         │                                                   │
│                         ├─ <<KafkaListener>> NovelCreatedListener          │
│                         │   @KafkaListener(topics = "novel-events")         │
│                         │   +handleNovelCreatedEvent() : void                │
│                         │                                                   │
│                         ├─ <<MyBatis>> CommentMapper                        │
│                         │   +insertComment() : int                           │
│                         │   +selectById() : Comment                          │
│                         │   +selectByNovelId() : List<Comment>              │
│                         │                                                   │
│                         ├─ <<MyBatis>> ReviewMapper                         │
│                         │   +insertReview() : int                            │
│                         │   +selectById() : Review                           │
│                         │   +selectByNovelId() : List<Review>                │
│                         │                                                   │
│                         ├─ <<MyBatis>> BookmarkMapper                       │
│                         │   +insertBookmark() : int                          │
│                         │   +deleteBookmark() : int                         │
│                         │   +selectByUserId() : List<Bookmark>               │
│                         │                                                   │
│                         ├─ <<FeignClient>> UserServiceClient               │
│                         │   +getUser() : UserProfile                         │
│                         │   +getUsernameById() : String                      │
│                         │                                                   │
│                         ├─ <<FeignClient>> ContentServiceClient            │
│                         │   +getNovel() : NovelDTO                           │
│                         │   +getChapter() : ChapterDTO                       │
│                         │                                                   │
│                         └─ <<Configuration>> FeignConfig                    │
│                             +feignRequestInterceptor() : RequestInterceptor   │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Gamification Service Droplet  ──deploy──┐                                  │
│    (yushan-gamification-service:8085)     │                                  │
│    Docker Container                       │                                  │
│    Spring Boot 3.4.10                     │                                  │
│                                           │                                  │
│         └──deploy────→ <<JAR>> GamificationServiceApplication             │
│                         manifest                                            │
│                                                                             │
│                         ┌─ <<RestController>> GamificationController      │
│                         │   +getUserPoints() : ResponseEntity                │
│                         │   +getUserAchievements() : ResponseEntity          │
│                         │   +getLeaderboard() : ResponseEntity              │
│                         │                                                   │
│                         ├─ <<Service>> GamificationService                   │
│                         │   +awardPoints() : void                            │
│                         │   +processUserComment() : void                     │
│                         │   +processUserReview() : void                      │
│                         │   +processUserVote() : void                        │
│                         │   +checkAchievements() : void                     │
│                         │                                                   │
│                         ├─ <<Service>> AchievementService                   │
│                         │   +checkAndUnlockAchievement() : void              │
│                         │   +getUserAchievements() : List<Achievement>       │
│                         │                                                   │
│                         ├─ <<Service>> PointsService                        │
│                         │   +addPoints() : void                              │
│                         │   +deductPoints() : void                           │
│                         │   +getUserBalance() : int                          │
│                         │                                                   │
│                         ├─ <<Service>> LeaderboardService                  │
│                         │   +updateLeaderboard() : void                     │
│                         │   +getTopUsers() : List<UserRank>                  │
│                         │                                                   │
│                         ├─ <<KafkaListener>> CommentEventListener          │
│                         │   @KafkaListener(topics = "comment-events")      │
│                         │   +handleCommentCreatedEvent() : void              │
│                         │                                                   │
│                         ├─ <<KafkaListener>> ReviewEventListener            │
│                         │   @KafkaListener(topics = "review-events")        │
│                         │   +handleReviewCreatedEvent() : void              │
│                         │                                                   │
│                         ├─ <<KafkaListener>> VoteEventListener              │
│                         │   @KafkaListener(topics = "vote-events")          │
│                         │   +handleVoteCreatedEvent() : void                 │
│                         │                                                   │
│                         ├─ <<MyBatis>> PointsMapper                         │
│                         │   +insertPointsTransaction() : int                  │
│                         │   +selectByUserId() : List<PointsTransaction>      │
│                         │   +getUserBalance() : int                           │
│                         │                                                   │
│                         ├─ <<MyBatis>> AchievementMapper                    │
│                         │   +insertUserAchievement() : int                   │
│                         │   +selectByUserId() : List<Achievement>            │
│                         │                                                   │
│                         └─ <<Redis>> LeaderboardCache                        │
│                             +updateRank() : void                              │
│                             +getTopRankings() : List<LeaderboardEntry>        │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  User Service Droplet  ──deploy──────┐                                     │
│    (yushan-user-service:8081)          │                                     │
│    Docker Container                     │                                     │
│    Spring Boot 3.4.10                   │                                     │
│                                        │                                     │
│         └──deploy────→ <<JAR>> UserServiceApplication                      │
│                         manifest                                            │
│                                                                             │
│                         ┌─ <<RestController>> AuthController                │
│                         │   +login() : ResponseEntity                        │
│                         │   +register() : ResponseEntity                      │
│                         │   +refreshToken() : ResponseEntity                 │
│                         │                                                   │
│                         ├─ <<RestController>> UserController               │
│                         │   +getProfile() : ResponseEntity                    │
│                         │   +updateProfile() : ResponseEntity                 │
│                         │                                                   │
│                         ├─ <<Service>> AuthService                          │
│                         │   +login() : JwtToken                              │
│                         │   +register() : User                               │
│                         │   +validateToken() : boolean                       │
│                         │                                                   │
│                         ├─ <<Service>> UserService                           │
│                         │   +getUser() : User                                │
│                         │   +updateUser() : User                             │
│                         │   +deleteUser() : void                             │
│                         │                                                   │
│                         ├─ <<Service>> JwtTokenProvider                      │
│                         │   +generateToken() : String                       │
│                         │   +validateToken() : boolean                       │
│                         │   +getUserIdFromToken() : UUID                     │
│                         │                                                   │
│                         ├─ <<Service>> KafkaEventProducerService            │
│                         │   +publishUserRegisteredEvent() : void             │
│                         │   +publishUserLoggedInEvent() : void              │
│                         │                                                   │
│                         ├─ <<MyBatis>> UserMapper                           │
│                         │   +selectById() : User                             │
│                         │   +selectByEmail() : User                          │
│                         │   +insertUser() : int                              │
│                         │   +updateUser() : int                              │
│                         │                                                   │
│                         ├─ <<MyBatis>> LibraryMapper                        │
│                         │   +insertLibraryItem() : int                        │
│                         │   +selectByUserId() : List<LibraryItem>           │
│                         │   +deleteLibraryItem() : int                       │
│                         │                                                   │
│                         ├─ <<Redis>> SessionManager                         │
│                         │   +storeSession() : void                           │
│                         │   +getSession() : Session                          │
│                         │   +deleteSession() : void                          │
│                         │                                                   │
│                         └─ <<Configuration>> JwtConfig                       │
│                             +jwtTokenProvider() : JwtTokenProvider            │
│                             +passwordEncoder() : PasswordEncoder              │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  API Gateway Droplet  ──deploy───────────┐                                   │
│    (yushan-api-gateway:8080)             │                                   │
│    Docker Container                       │                                   │
│    Spring Cloud Gateway                   │                                   │
│                                          │                                   │
│         └──deploy────→ <<JAR>> ApiGatewayApplication                       │
│                         manifest                                            │
│                                                                             │
│                         ┌─ <<GatewayFilter>> AuthFilter                     │
│                         │   +filter() : GatewayFilter                        │
│                         │   +validateJWT() : boolean                         │
│                         │                                                   │
│                         ├─ <<RouteConfiguration>> GatewayRoutes            │
│                         │   user-service: /api/v1/auth/**                    │
│                         │   user-service: /api/v1/users/**                   │
│                         │   content-service: /api/v1/novels/**               │
│                         │   content-service: /api/v1/chapters/**             │
│                         │   engagement-service: /api/v1/comments/**           │
│                         │   engagement-service: /api/v1/reviews/**           │
│                         │   engagement-service: /api/v1/bookmarks/**         │
│                         │   gamification-service: /api/v1/gamification/**     │
│                         │   analytics-service: /api/v1/analytics/**          │
│                         │                                                   │
│                         ├─ <<LoadBalancer>> SpringCloudLoadBalancer         │
│                         │   +chooseServer() : Server                         │
│                         │   +getServer() : ServiceInstance                   │
│                         │                                                   │
│                         ├─ <<CircuitBreaker>> Resilience4jConfig            │
│                         │   circuit-breaker for user-service                 │
│                         │   circuit-breaker for content-service             │
│                         │   circuit-breaker for engagement-service           │
│                         │                                                   │
│                         └─ <<EurekaClient>> ServiceDiscovery                 │
│                             +getInstance() : ServiceInstance                  │
│                             +getInstances() : List<ServiceInstance>          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  PostgreSQL Database  ──deploy────────┐                                    │
│    (yushan-content-db:5432)            │                                    │
│    Docker Container                     │                                    │
│    PostgreSQL 16                        │                                    │
│                                        │                                    │
│         └──deploy────→ <<Schema>> public                                  │
│                         manifest                                            │
│                         ┌─ <<Table>> novels                                │
│                         │   columns:                                       │
│                         │   - id (PK)                                      │
│                         │   - title                                        │
│                         │   - synopsis                                     │
│                         │   - author_id (FK)                               │
│                         │   - status                                       │
│                         │   - created_at                                   │
│                         │   - updated_at                                  │
│                         │                                                   │
│                         ├─ <<Table>> chapters                              │
│                         │   columns:                                       │
│                         │   - id (PK)                                      │
│                         │   - novel_id (FK)                                │
│                         │   - title                                        │
│                         │   - content                                      │
│                         │   - chapter_order                                │
│                         │   - created_at                                   │
│                         │                                                   │
│                         ├─ <<Table>> genres                               │
│                         │   columns:                                       │
│                         │   - id (PK)                                      │
│                         │   - name                                         │
│                         │   - description                                  │
│                         │                                                   │
│                         ├─ <<Table>> novel_genres                          │
│                         │   columns:                                       │
│                         │   - novel_id (FK)                                │
│                         │   - genre_id (FK)                                │
│                         │                                                   │
│                         ├─ <<Table>> authors                               │
│                         │   columns:                                       │
│                         │   - id (PK)                                      │
│                         │   - user_id (FK)                                 │
│                         │   - pen_name                                     │
│                         │   - bio                                           │
│                         │                                                   │
│                         └─ <<Table>> tags                                 │
│                             columns:                                       │
│                             - id (PK)                                       │
│                             - name                                          │
│                             - created_at                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  Elasticsearch Cluster  ──deploy───────┐                                   │
│    (yushan-elasticsearch:9200)          │                                   │
│    Docker Container                       │                                   │
│    Elasticsearch 8.11                     │                                   │
│                                          │                                   │
│         └──deploy────→ <<Index>> novels                                   │
│                         manifest                                            │
│                         <<Document>> NovelDocument                          │
│                         fields:                                            │
│                         - id                                               │
│                         - title                                            │
│                         - synopsis                                         │
│                         - author                                           │
│                         - genres                                           │
│                         - tags                                             │
│                         - status                                           │
│                         - created_at                                       │
│                                                                             │
│         └──deploy────→ <<Index>> chapters                                  │
│                         manifest                                            │
│                         <<Document>> ChapterDocument                         │
│                         fields:                                            │
│                         - id                                               │
│                         - novel_id                                         │
│                         - title                                            │
│                         - content                                          │
│                         - chapter_order                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  Redis Cache  ──deploy────────────┐                                        │
│    (yushan-redis:6379)             │                                        │
│    Docker Container                 │                                        │
│    Redis 7                           │                                        │
│                                    │                                        │
│         └──deploy────→ <<Cache>> SessionCache                              │
│                         manifest                                            │
│                         Keys: session:{userId}                               │
│                         TTL: 30 minutes                                     │
│                                                                             │
│         └──deploy────→ <<Cache>> NovelMetadataCache                        │
│                         manifest                                            │
│                         Keys: novel:{novelId}                                │
│                         TTL: 5 minutes                                      │
│                                                                             │
│         └──deploy────→ <<Cache>> LeaderboardCache                          │
│                         manifest                                            │
│                         Key: leaderboard:{type}                             │
│                         Data Structure: Sorted Set                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  Kafka Message Broker  ──deploy───────┐                                   │
│    (yushan-kafka:9092)                   │                                   │
│    Docker Container                       │                                   │
│    Apache Kafka with Zookeeper            │                                   │
│                                           │                                   │
│         └──deploy────→ <<Topic>> novel-events                              │
│                         manifest                                            │
│                         <<Event>> NovelCreatedEvent                         │
│                         fields: novelId, title, authorId, timestamp         │
│                                                                             │
│                         <<Event>> ChapterCreatedEvent                        │
│                         fields: chapterId, novelId, title, timestamp        │
│                                                                             │
│                         <<Event>> NovelPublishedEvent                        │
│                         fields: novelId, status, timestamp                   │
│                                                                             │
│         └──deploy────→ <<Topic>> comment-events                            │
│                         manifest                                            │
│                         <<Event>> CommentCreatedEvent                        │
│                         fields: commentId, userId, chapterId, content       │
│                                                                             │
│         └──deploy────→ <<Topic>> review-events                             │
│                         manifest                                            │
│                         <<Event>> ReviewCreatedEvent                         │
│                         fields: reviewId, userId, novelId, rating, content   │
│                                                                             │
│         └──deploy────→ <<Topic>> vote-events                               │
│                         manifest                                            │
│                         <<Event>> VoteCreatedEvent                           │
│                         fields: voteId, userId, targetId, targetType          │
│                                                                             │
│         └──deploy────→ <<Topic>> user.events                               │
│                         manifest                                            │
│                         <<Event>> UserRegisteredEvent                        │
│                         fields: userId, email, username, timestamp           │
│                                                                             │
│                         <<Event>> UserLoggedInEvent                         │
│                         fields: userId, timestamp, ipAddress                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

Legend:
━━━━━ deploy relationship: Node deploys component
┄┄┄┄┄ manifest relationship: Component contains artifacts
```

