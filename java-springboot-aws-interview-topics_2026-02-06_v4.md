# Java Spring Boot AWS Interview Topics - Comprehensive Guide

**Generated:** Tuesday, January 20, 2026  
**Updated:** Thursday, January 29, 2026  
**Enhanced:** Thursday, February 06, 2026  
**Final Update:** Thursday, February 06, 2026 - Added Functional Programming Deep Dive

---

## Java Core

- OOPs concepts (inheritance, polymorphism, encapsulation, abstraction)
- Collections Framework (List, Set, Map, Queue and their implementations)
- Generics
- Exception handling
- Java 8+ features (Lambda, Streams, Optional, Method references)
- Equals/HashCode contract
- Immutability and final keyword
- String, StringBuilder, StringBuffer
- Serialization/Deserialization
- Reflection API
- Annotations and custom annotations
- Garbage Collection types (G1GC, ZGC, etc.)
- JVM architecture and memory areas (heap, stack, metaspace)
- ClassLoaders
- Enum usage and patterns
- Static vs instance members
- Nested and inner classes
- Functional interfaces
- **Record classes (Java 14+)**
- **Sealed classes (Java 17+)**
- **Pattern matching for switch (Java 17+)**
- **Text blocks (Java 15+)**

### Functional Programming Deep Dive

#### Lambda Expressions Fundamentals
- **Lambda syntax and structure**
  - `(parameters) -> expression`
  - `(parameters) -> { statements; }`
  - Type inference for parameters
  - Single parameter without parentheses: `x -> x * 2`
  - No parameters: `() -> System.out.println("Hello")`
  
- **Effectively final variables**
  - Lambdas can only access final or effectively final variables
  - Cannot modify variables from enclosing scope
  
- **Method references**
  - Static method reference: `ClassName::staticMethod`
  - Instance method reference: `instance::instanceMethod`
  - Instance method of arbitrary object: `ClassName::instanceMethod`
  - Constructor reference: `ClassName::new`

#### Functional Interfaces
- **Predicate<T>** - `T -> boolean`
  - `test(T t)` - evaluates condition
  - `and()`, `or()`, `negate()` - combining predicates
  - Use case: filtering collections
  - Example: `Predicate<String> isEmpty = String::isEmpty;`
  
- **Function<T, R>** - `T -> R`
  - `apply(T t)` - transforms input to output
  - `andThen()`, `compose()` - chaining functions
  - Use case: mapping/transforming data
  - Example: `Function<String, Integer> length = String::length;`
  
- **Consumer<T>** - `T -> void`
  - `accept(T t)` - performs operation, returns nothing
  - `andThen()` - chaining consumers
  - Use case: performing actions (printing, saving)
  - Example: `Consumer<String> print = System.out::println;`
  
- **Supplier<T>** - `() -> T`
  - `get()` - provides a value
  - Use case: lazy evaluation, factory methods
  - Example: `Supplier<LocalDateTime> now = LocalDateTime::now;`
  
- **BiFunction<T, U, R>** - `(T, U) -> R`
  - `apply(T t, U u)` - takes two inputs, returns one output
  - Example: `BiFunction<Integer, Integer, Integer> sum = (a, b) -> a + b;`
  
- **BiConsumer<T, U>** - `(T, U) -> void`
  - Use case: forEach on Maps
  - Example: `BiConsumer<String, Integer> print = (k, v) -> System.out.println(k + "=" + v);`
  
- **BiPredicate<T, U>** - `(T, U) -> boolean`
  - Example: `BiPredicate<String, String> equals = String::equals;`
  
- **UnaryOperator<T>** - `T -> T` (extends Function<T, T>)
  - Example: `UnaryOperator<Integer> square = x -> x * x;`
  
- **BinaryOperator<T>** - `(T, T) -> T` (extends BiFunction<T, T, T>)
  - Example: `BinaryOperator<Integer> max = Integer::max;`
  
- **Custom functional interfaces**
  - Creating your own with @FunctionalInterface
  - Single abstract method (SAM) requirement

#### Stream API Mastery

**Stream Creation**
- `Collection.stream()` - from collections
- `Stream.of(T... values)` - from values
- `Arrays.stream(array)` - from arrays
- `Stream.generate(Supplier)` - infinite stream
- `Stream.iterate(seed, UnaryOperator)` - infinite stream with iteration
- `IntStream.range(start, end)` - primitive streams
- `Files.lines(Path)` - from file
- `Stream.builder()` - builder pattern

**Intermediate Operations (Lazy)**
- **filter(Predicate)** - keep elements matching condition
  - `stream.filter(x -> x > 10)`
  - `stream.filter(String::isEmpty)`
  
- **map(Function)** - transform each element
  - `stream.map(String::toUpperCase)`
  - `stream.map(x -> x * 2)`
  
- **flatMap(Function)** - flatten nested structures
  - `stream.flatMap(Collection::stream)` - flatten list of lists
  - `stream.flatMap(str -> Arrays.stream(str.split(" ")))` - split and flatten
  
- **distinct()** - remove duplicates
- **sorted()** / **sorted(Comparator)** - sort elements
  - `stream.sorted()`
  - `stream.sorted(Comparator.reverseOrder())`
  - `stream.sorted(Comparator.comparing(User::getName))`
  
- **peek(Consumer)** - perform action without modifying stream (debugging)
- **limit(n)** - take first n elements
- **skip(n)** - skip first n elements
- **takeWhile(Predicate)** - take while condition is true (Java 9+)
- **dropWhile(Predicate)** - skip while condition is true (Java 9+)

**Terminal Operations (Eager)**
- **collect(Collector)** - collect to collection
  - `Collectors.toList()`
  - `Collectors.toSet()`
  - `Collectors.toMap(keyMapper, valueMapper)`
  - `Collectors.groupingBy(classifier)`
  - `Collectors.partitioningBy(predicate)`
  - `Collectors.joining(delimiter)`
  - `Collectors.counting()`
  - `Collectors.summingInt(mapper)`
  - `Collectors.averagingInt(mapper)`
  - `Collectors.maxBy(comparator)`
  - `Collectors.minBy(comparator)`
  
- **forEach(Consumer)** - perform action on each element
- **reduce(BinaryOperator)** - reduce to single value
  - `stream.reduce((a, b) -> a + b)`
  - `stream.reduce(0, Integer::sum)` - with identity
  - `stream.reduce(0, (subtotal, element) -> subtotal + element, Integer::sum)` - parallel version
  
- **count()** - count elements
- **anyMatch(Predicate)** - check if any element matches
- **allMatch(Predicate)** - check if all elements match
- **noneMatch(Predicate)** - check if no elements match
- **findFirst()** - find first element (returns Optional)
- **findAny()** - find any element (returns Optional)
- **min(Comparator)** / **max(Comparator)** - find min/max (returns Optional)
- **toArray()** - convert to array

**Common Patterns**
```java
// Filtering and collecting
List<String> filtered = list.stream()
    .filter(s -> s.length() > 5)
    .collect(Collectors.toList());

// Mapping and collecting
List<Integer> lengths = list.stream()
    .map(String::length)
    .collect(Collectors.toList());

// FlatMap example
List<List<String>> listOfLists = Arrays.asList(
    Arrays.asList("a", "b"),
    Arrays.asList("c", "d")
);
List<String> flattened = listOfLists.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList());

// Grouping
Map<Integer, List<String>> grouped = list.stream()
    .collect(Collectors.groupingBy(String::length));

// Partitioning
Map<Boolean, List<String>> partitioned = list.stream()
    .collect(Collectors.partitioningBy(s -> s.length() > 5));

// Summing
int totalLength = list.stream()
    .mapToInt(String::length)
    .sum();

// Finding max with custom comparator
Optional<String> longest = list.stream()
    .max(Comparator.comparing(String::length));

// Reducing
int sum = IntStream.range(1, 10)
    .reduce(0, Integer::sum);
```

#### Optional Class Deep Dive

**Creating Optional**
- `Optional.of(value)` - creates Optional with non-null value (throws NPE if null)
- `Optional.ofNullable(value)` - creates Optional, handles null safely
- `Optional.empty()` - creates empty Optional

**Checking Presence**
- `isPresent()` - returns true if value present
- `isEmpty()` - returns true if empty (Java 11+)

**Retrieving Values**
- `get()` - returns value, throws NoSuchElementException if empty (avoid!)
- `orElse(defaultValue)` - returns value or default
- `orElseGet(Supplier)` - returns value or calls supplier (lazy)
- `orElseThrow()` - throws NoSuchElementException if empty
- `orElseThrow(Supplier)` - throws custom exception if empty

**Conditional Actions**
- `ifPresent(Consumer)` - executes consumer if value present
- `ifPresentOrElse(Consumer, Runnable)` - executes consumer if present, else runnable (Java 9+)

**Transforming Optional**
- `map(Function)` - transforms value if present
  - `optional.map(String::toUpperCase)`
  - Returns Optional<R>
  
- `flatMap(Function)` - transforms value to Optional if present
  - Use when function returns Optional
  - `optional.flatMap(this::findById)` - avoids Optional<Optional<T>>
  
- `filter(Predicate)` - filters value based on condition
  - `optional.filter(s -> s.length() > 5)`
  - Returns empty Optional if doesn't match

- `or(Supplier<Optional>)` - returns alternative Optional if empty (Java 9+)
- `stream()` - converts to Stream of 0 or 1 element (Java 9+)

**Common Patterns and Best Practices**
```java
// BAD: Using get() without checking
String name = optional.get(); // Dangerous!

// GOOD: Using orElse
String name = optional.orElse("Unknown");

// GOOD: Using orElseGet (lazy evaluation)
String name = optional.orElseGet(() -> fetchDefaultName());

// GOOD: Using orElseThrow with custom exception
String name = optional.orElseThrow(() -> new IllegalStateException("Name not found"));

// Chaining transformations
String result = Optional.ofNullable(user)
    .map(User::getAddress)
    .map(Address::getCity)
    .map(String::toUpperCase)
    .orElse("UNKNOWN");

// Using filter
Optional<User> adult = Optional.ofNullable(user)
    .filter(u -> u.getAge() >= 18);

// FlatMap to avoid nested Optionals
Optional<String> city = Optional.ofNullable(user)
    .flatMap(User::getAddress)  // getAddress returns Optional<Address>
    .map(Address::getCity);

// If-present pattern
optional.ifPresent(value -> System.out.println("Found: " + value));

// If-present-or-else pattern (Java 11+)
optional.ifPresentOrElse(
    value -> System.out.println("Found: " + value),
    () -> System.out.println("Not found")
);
```

**Anti-Patterns to Avoid**
- Don't use `optional.isPresent()` followed by `optional.get()`
  - Use `orElse()`, `orElseGet()`, or `ifPresent()` instead
- Don't use Optional for fields or method parameters
  - Use Optional only for return types
- Don't return null instead of Optional.empty()
- Don't use Optional.of() with potentially null values
  - Use Optional.ofNullable() instead
- Avoid Optional<Collection>
  - Return empty collection instead

#### Practical Lambda Examples

**Sorting with Lambdas**
```java
// Simple sorting
list.sort((a, b) -> a.compareTo(b));
list.sort(String::compareTo);
list.sort(Comparator.naturalOrder());

// Reverse sorting
list.sort(Comparator.reverseOrder());

// Sorting by property
users.sort(Comparator.comparing(User::getName));

// Multi-level sorting
users.sort(Comparator.comparing(User::getAge)
    .thenComparing(User::getName));

// Null-safe sorting
users.sort(Comparator.comparing(User::getName, 
    Comparator.nullsLast(Comparator.naturalOrder())));
```

**Working with Maps**
```java
// forEach on Map
map.forEach((key, value) -> System.out.println(key + "=" + value));

// computeIfAbsent
map.computeIfAbsent(key, k -> new ArrayList<>()).add(value);

// merge
map.merge(key, 1, Integer::sum); // increment counter

// replaceAll
map.replaceAll((k, v) -> v.toUpperCase());
```

**Complex Stream Pipelines**
```java
// Find top 3 most common words
Map<String, Long> topWords = text.stream()
    .flatMap(line -> Arrays.stream(line.split("\\s+")))
    .map(String::toLowerCase)
    .collect(Collectors.groupingBy(
        Function.identity(),
        Collectors.counting()
    ))
    .entrySet().stream()
    .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
    .limit(3)
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        Map.Entry::getValue,
        (e1, e2) -> e1,
        LinkedHashMap::new
    ));

// Group and aggregate
Map<String, Integer> totalByCategory = items.stream()
    .collect(Collectors.groupingBy(
        Item::getCategory,
        Collectors.summingInt(Item::getPrice)
    ));

// Multi-level grouping
Map<String, Map<Integer, List<User>>> grouped = users.stream()
    .collect(Collectors.groupingBy(
        User::getDepartment,
        Collectors.groupingBy(User::getAge)
    ));
```

#### Common Interview Questions on Lambdas/Streams/Optional

**Lambda Questions**
- What is a lambda expression and why is it useful?
- What is a functional interface?
- Can a lambda modify variables from enclosing scope?
- Difference between Predicate, Function, Consumer, and Supplier?
- What are method references and when to use them?

**Stream Questions**
- Difference between intermediate and terminal operations?
- What is lazy evaluation in streams?
- When to use map vs flatMap?
- How does reduce work?
- Difference between findFirst and findAny?
- Can you reuse a stream? (No - streams can only be consumed once)
- Parallel streams - when to use and pitfalls?
- How to convert stream to collection?

**Optional Questions**
- Why use Optional instead of null?
- Difference between orElse and orElseGet?
- When should you NOT use Optional?
- Difference between map and flatMap on Optional?
- How to handle null values safely with Optional?
- Is it good practice to use Optional as method parameter? (No)

**Performance Questions**
- Are streams always better than loops?
- When are parallel streams beneficial?
- Performance implications of boxing/unboxing in streams?
- Why use primitive streams (IntStream, LongStream, DoubleStream)?

---

## Java Concurrency

- Thread lifecycle and creation
- Synchronized keyword and locks
- Volatile keyword
- Executor framework and thread pools
- Concurrent collections (ConcurrentHashMap, CopyOnWriteArrayList)
- CompletableFuture
- Deadlock, race conditions
- Thread-safe patterns
- CountDownLatch, CyclicBarrier, Semaphore
- Atomic classes (AtomicInteger, AtomicReference)
- Fork/Join framework
- ThreadLocal
- Virtual threads (Java 19+)
- **Structured concurrency (Java 21+)**
- **Scoped values (Java 21+)**
- **Thread pinning and monitoring**
- **Lock-free data structures**
- **Memory visibility and ordering**

---

## Spring Core

- Dependency Injection and Inversion of Control
- Bean lifecycle and scopes (singleton, prototype, request, session)
- @Component, @Service, @Repository, @Controller
- @Autowired, @Qualifier, @Primary
- Java-based configuration vs XML
- ApplicationContext
- @Bean and @Configuration
- Profiles (@Profile)
- Property management (@Value, @ConfigurationProperties)
- **BeanPostProcessor and BeanFactoryPostProcessor**
- **@Lazy and lazy initialization strategies**
- **Circular dependency resolution**
- **ApplicationContext hierarchy**
- **Environment abstraction and PropertySource**

---

## Spring Boot

- Auto-configuration
- Starter dependencies
- Application properties/YAML
- Embedded servers (Tomcat, Jetty)
- Actuator (health checks, metrics)
- Spring Boot DevTools
- CommandLineRunner and ApplicationRunner
- @SpringBootApplication annotation
- Custom auto-configuration
- Conditional annotations (@ConditionalOnProperty, @ConditionalOnClass)
- Spring Boot testing slices
- Embedded database configuration (H2)
- Externalized configuration
- Banner customization
- Application events and listeners
- **Creating custom Spring Boot starters**
- **@AutoConfiguration and auto-configuration ordering**
- **Spring Boot 3.x features and migration**
- **AOT compilation and GraalVM native images**
- **Spring Boot Observability (Micrometer Tracing)**
- **Custom Actuator endpoints**
- **Actuator security configuration**
- **HealthIndicator and custom health checks**
- **Application startup performance optimization**

---

## Spring Web/REST API

- @RestController, @RequestMapping, @GetMapping, @PostMapping, etc.
- Request/Response handling (@RequestBody, @ResponseBody, @PathVariable, @RequestParam)
- HTTP methods and status codes
- Exception handling (@ExceptionHandler, @ControllerAdvice)
- Validation (@Valid, @NotNull, @Size, etc.)
- Content negotiation
- CORS configuration
- Interceptors and filters
- Richardson Maturity Model
- Idempotency
- HTTP caching (ETag, Cache-Control)
- Hypermedia controls
- Content compression (gzip)
- Multipart file upload
- Streaming responses
- **ProblemDetail and RFC 7807 error responses**
- **Custom argument resolvers**
- **Custom message converters**
- **Async request processing with @Async**
- **Server-Sent Events (SSE)**
- **WebSocket support**

---

## Spring Data JPA/JDBC

- JPA vs Hibernate
- Entity mapping (@Entity, @Table, @Id, @GeneratedValue)
- Relationships (@OneToOne, @OneToMany, @ManyToOne, @ManyToMany)
- JpaRepository, CrudRepository
- Query methods and @Query
- Pagination and sorting
- Transactions (@Transactional)
- Lazy vs Eager loading
- N+1 problem
- JDBC Template (for plain JDBC)
- **Entity graphs and fetch strategies**
- **Criteria API and Specifications**
- **Query DSL integration**
- **Projections (interface-based and class-based)**
- **@EntityListeners and auditing**
- **Second-level cache (Hibernate)**
- **Custom repository implementations**
- **Batch processing and bulk operations**
- **Database locking strategies**
- **Multi-tenancy with JPA**

---

## Spring Security

- Authentication vs Authorization
- UserDetailsService
- Password encoding (BCrypt)
- JWT tokens
- @EnableWebSecurity
- SecurityFilterChain configuration
- Method-level security (@PreAuthorize, @Secured)
- OAuth2/OpenID Connect
- CSRF protection
- Session management
- SQL injection prevention
- XSS and CSRF attacks
- HTTPS/TLS
- Security headers (CORS, CSP, X-Frame-Options)
- Input validation and sanitization
- Secure coding practices
- OWASP Top 10
- Secrets management best practices
- Certificate management
- **OAuth2 Resource Server configuration**
- **Custom authentication providers**
- **Security filter chain ordering**
- **Rate limiting with Spring Security**
- **Brute force attack prevention**
- **Token refresh strategies**
- **Multi-factor authentication (MFA)**

---

## Spring AOP

- Aspect-Oriented Programming concepts
- @Aspect, @Before, @After, @Around, @AfterReturning, @AfterThrowing
- Pointcut expressions
- JoinPoint
- Use cases (logging, security, transactions)
- **Proxy vs AspectJ weaving**
- **Custom annotations with AOP**
- **Performance monitoring with AOP**
- **Transaction management internals**

---

## Spring Batch

- Job, Step, ItemReader, ItemProcessor, ItemWriter
- Chunk-oriented processing
- Scheduling batch jobs
- **Job parameters and job instances**
- **Step execution context**
- **Partitioning and parallel processing**
- **Skip and retry policies**
- **Job repository and metadata**

---

## Spring Integration/Messaging

- Message channels
- Message endpoints
- Integration with JMS, AMQP
- Spring for Apache Kafka
- @KafkaListener, KafkaTemplate
- **Error handling in message processing**
- **Message transformation and routing**
- **Poison pill handling**
- **Exactly-once semantics**

---

## Spring WebFlux (Reactive Programming)

- Mono and Flux
- Reactive streams
- WebClient vs RestTemplate
- Backpressure
- Non-blocking I/O
- **Reactive repository support (R2DBC)**
- **Reactive security configuration**
- **Context propagation in reactive streams**
- **Reactive error handling**
- **Schedulers and threading in reactive programming**
- **Cold vs hot publishers**
- **Reactive caching strategies**

---

## Spring Cloud & Distributed Systems

### Service Discovery & Configuration
- **Spring Cloud Config Server**
- **Vault integration for secrets**
- **Eureka service registry**
- **Consul for service discovery**
- **Client-side vs server-side discovery**
- **Config refresh strategies (@RefreshScope)**

### API Gateway
- **Spring Cloud Gateway**
  - Route predicates and filters
  - Rate limiting filters
  - Circuit breaker integration
  - Request/response modification
  - Custom filters
- **Gateway vs Zuul comparison**

### Resilience Patterns
- **Circuit Breaker (Resilience4j)**
  - Circuit states (closed, open, half-open)
  - Fallback methods
  - Metrics and monitoring
- **Retry patterns**
- **Bulkhead pattern**
- **Rate limiter**
- **Time limiter**

### Distributed Tracing
- **Spring Cloud Sleuth**
- **Zipkin integration**
- **Jaeger tracing**
- **OpenTelemetry**
- **Trace context propagation**
- **Sampling strategies**

### Load Balancing
- **Client-side load balancing (Spring Cloud LoadBalancer)**
- **Server-side load balancing (AWS ELB/ALB)**
- **Load balancing algorithms**
- **Sticky sessions**

### Cloud Native Patterns
- **12-Factor App principles (deep dive)**
- **Cloud-native security patterns**
- **Feature flags/toggles**
- **A/B testing strategies**
- **Canary deployments**
- **Blue-green deployments**

---

## Microservices Concepts

- Service discovery (Eureka, Consul)
- API Gateway (Spring Cloud Gateway, Zuul)
- Load balancing (Ribbon, Spring Cloud LoadBalancer)
- Circuit breaker (Resilience4j, Hystrix)
- Distributed tracing (Sleuth, Zipkin)
- Configuration management (Spring Cloud Config)
- Inter-service communication (REST, Feign, gRPC)
- Event-driven architecture (Kafka, RabbitMQ)
- **Domain-Driven Design (DDD) and bounded contexts**
- **Service mesh (Istio, Linkerd)**
- **Sidecar pattern**
- **Ambassador pattern**
- **Anti-corruption layer**
- **Strangler Fig pattern for migration**
- **Backend for Frontend (BFF) pattern**
- **API composition pattern**

---

## Microservice Transaction Management

### Distributed Transaction Patterns

- **Saga Pattern (Choreography-based)**
  - Event-driven coordination between services
  - Each service publishes domain events
  - Services listen and react to events
  - No central coordinator
  - Eventual consistency model
  - Compensating transactions for rollback

- **Saga Pattern (Orchestration-based)**
  - Central orchestrator coordinates the workflow
  - Orchestrator maintains saga state
  - Explicit definition of transaction steps
  - Easier to understand and monitor
  - Better control over complex workflows
  - Frameworks: Axon Framework, Temporal, Camunda

- **Two-Phase Commit (2PC)**
  - Prepare phase and commit phase
  - Strong consistency guarantee
  - Blocking protocol (not recommended for microservices)
  - Single point of failure
  - Performance and scalability issues

- **Try-Confirm/Cancel (TCC)**
  - Three-phase transaction protocol
  - Try: Reserve resources
  - Confirm: Commit reserved resources
  - Cancel: Release reserved resources
  - Better performance than 2PC
  - Requires all services to support TCC pattern

### Supporting Patterns

- **Outbox Pattern**
  - Ensures reliable event publishing
  - Stores events in database table (same transaction)
  - Background processor publishes events
  - At-least-once delivery guarantee
  - Change Data Capture (Debezium) for automatic event publishing
  - Solves dual-write problem

- **Event Sourcing**
  - Store events instead of current state
  - Complete audit trail
  - Can rebuild state from events
  - Temporal queries capability
  - Natural fit with CQRS
  - Event versioning and schema evolution

- **Idempotency**
  - Operations can be safely retried
  - Idempotency keys for duplicate detection
  - Critical for distributed systems
  - Prevents duplicate processing
  - Important for payment and financial transactions

- **Compensation Logic**
  - Semantic rollback transactions
  - Reverse business operations
  - Must be idempotent
  - Should handle partial failures
  - May require manual intervention for critical failures

### Transaction Coordination

- State management in distributed transactions
- Correlation IDs for tracking
- Timeout and retry strategies
- Dead letter queues for failed messages
- Saga execution monitoring and observability
- Handling long-running transactions
- Versioning of events and commands
- Backward compatibility in event schemas

### Consistency Models

- Strong consistency vs eventual consistency
- CAP theorem implications
- BASE (Basically Available, Soft state, Eventual consistency)
- Conflict resolution strategies
- Version vectors and timestamps
- Last-write-wins (LWW)
- Read-your-writes consistency
- Causal consistency

### Failure Handling

- Partial failure scenarios
- Network partition handling
- Service unavailability
- Compensating transaction failures
- Circuit breakers in saga coordination
- Bulkhead pattern for isolation
- Graceful degradation
- Manual intervention workflows

### AWS Integration for Sagas

- AWS Step Functions for orchestration
- EventBridge for choreography
- SQS for reliable messaging
- DynamoDB for saga state storage
- Lambda for saga steps
- X-Ray for distributed tracing

### Best Practices

- Design for failure and idempotency
- Monitor saga execution and completion rates
- Implement proper logging and correlation IDs
- Use timeouts at every step
- Plan for compensating transactions upfront
- Version all events and commands
- Test failure scenarios thoroughly
- Document saga workflows clearly
- Implement alerting for stuck sagas
- Keep sagas as simple as possible

### Common Interview Questions

- When to use Saga vs 2PC?
- How to ensure exactly-once processing?
- Handling compensating transaction failures
- Choosing between choreography vs orchestration
- Data consistency across services
- Idempotency implementation strategies
- Event ordering guarantees
- Handling duplicate events/messages

---

## AWS Services

### Compute
- EC2
- ECS
- EKS
- Lambda
- Elastic Beanstalk
- **Fargate**
- **EC2 Auto Scaling**
- **Lambda cold starts and optimization**

### Storage
- S3
- EBS
- EFS
- **S3 storage classes and lifecycle policies**
- **S3 event notifications**
- **Cross-region replication**

### Database
- RDS (PostgreSQL, MySQL)
- DynamoDB
- Aurora
- **RDS read replicas**
- **Aurora Serverless**
- **DynamoDB global tables**
- **DynamoDB streams**
- **Database backup strategies**

### Networking
- VPC
- Subnets
- Security Groups
- Route53
- CloudFront
- ELB/ALB/NLB/CLB
- **VPC peering**
- **Transit Gateway**
- **PrivateLink**
- **NAT Gateway vs NAT Instance**
- **Network ACLs**

### Messaging
- SQS (standard vs FIFO)
- SNS (pub/sub)
- EventBridge
- **SQS visibility timeout**
- **SQS dead letter queues**
- **EventBridge rules and patterns**
- **SNS message filtering**

### Caching
- ElastiCache (Redis, Memcached)
- **Redis vs Memcached comparison**
- **ElastiCache cluster modes**

### Monitoring
- CloudWatch
- X-Ray
- **CloudWatch Logs Insights**
- **CloudWatch custom metrics**
- **CloudWatch alarms**
- **X-Ray service maps**

### Security
- IAM
- Secrets Manager
- Parameter Store
- KMS
- WAF
- **IAM roles vs users**
- **IAM policies and permissions**
- **Cognito for user management**
- **Certificate Manager (ACM)**
- **GuardDuty**
- **Security Hub**

### CI/CD
- CodePipeline
- CodeBuild
- CodeDeploy
- **CodeCommit**
- **Deployment strategies (in-place, blue/green)**

### Additional Services
- ECR (Container Registry)
- SAM, Step Functions (Serverless)
- API Gateway
- AWS Backup
- Cost Explorer
- **CloudFormation and infrastructure as code**
- **AWS CDK (Cloud Development Kit)**
- **Organizations and multi-account strategies**
- **Cost optimization strategies**
- **Well-Architected Framework pillars**

---

## AWS SDK for Java

- S3 client usage
- DynamoDB integration
- SQS/SNS integration
- Credentials management (IAM roles, profiles)
- AWS SDK v2
- **Async clients and CompletableFuture**
- **Pagination handling**
- **Error handling and retries**
- **Request signing**

---

## Database Concepts

- SQL fundamentals (joins, aggregations, subqueries)
- Indexes and performance optimization
- ACID properties
- Normalization
- Connection pooling (HikariCP)
- Database migrations (Flyway, Liquibase)
- NoSQL basics (DynamoDB, MongoDB)
- Transactions isolation levels
- Optimistic vs pessimistic locking
- Database connection pooling configuration
- Read replicas
- Multi-tenancy patterns
- Stored procedures and functions
- Database triggers
- Full-text search
- **Database sharding and partitioning**
- **Replication strategies (synchronous vs asynchronous)**
- **Database failover and high availability**
- **Query execution plans**
- **Database monitoring and profiling**
- **NoSQL data modeling patterns**
- **Denormalization strategies**
- **CQRS (Command Query Responsibility Segregation)**

---

## Message Queues & Event Streaming

- Apache Kafka concepts (topics, partitions, consumer groups, offsets)
- RabbitMQ (exchanges, queues, bindings)
- AWS SQS (standard vs FIFO)
- AWS SNS (pub/sub)
- Message ordering and idempotency
- Dead letter queues
- **Kafka Connect and stream processing**
- **Kafka Streams API**
- **Exactly-once semantics in Kafka**
- **Kafka partitioning strategies**
- **Message serialization (Avro, Protobuf)**
- **Event schema registry**
- **Compacted topics in Kafka**

---

## Caching

- Cache-aside pattern
- Write-through, write-behind
- Cache eviction policies (LRU, LFU)
- Spring Cache abstraction (@Cacheable, @CacheEvict, @CachePut)
- Redis data structures
- Distributed caching challenges
- **Cache warming strategies**
- **Cache invalidation patterns**
- **Multi-level caching**
- **Cache coherence in distributed systems**
- **Redis pub/sub**
- **Redis transactions and pipelining**
- **Cache stampede prevention**

---

## API Design & Documentation

- Swagger/OpenAPI (SpringDoc, Springfox)
- API versioning strategies (URI, header, parameter)
- HATEOAS
- GraphQL with Spring
- Rate limiting and throttling
- API authentication (API keys, OAuth2)
- Pagination best practices
- ETags and caching headers
- **API design best practices**
- **Deprecation strategies**
- **Backward compatibility**
- **API governance**
- **Contract-first vs code-first**
- **gRPC and Protocol Buffers**
- **AsyncAPI for async APIs**

---

## Testing

- JUnit 5
- Mockito
- @SpringBootTest, @WebMvcTest, @DataJpaTest
- MockMvc for API testing
- Integration testing
- Test coverage
- TestContainers
- **Mutation testing (PIT)**
- **Contract testing (Pact, Spring Cloud Contract)**
- **Performance testing (JMeter, Gatling)**
- **Chaos engineering principles**
- **Test data management**
- **Mocking external services (WireMock)**
- **BDD with Cucumber**

---

## Design Patterns

- Singleton, Factory, Builder
- Strategy, Observer, Decorator
- MVC pattern
- DAO/Repository pattern
- DTO pattern
- **CQRS pattern**
- **Event Sourcing pattern**
- **Circuit Breaker pattern**
- **Retry pattern**
- **Throttling pattern**
- **Materialized View pattern**
- **Cache-Aside pattern**
- **Gateway Aggregation pattern**

---

## DevOps & Deployment

- Docker (Dockerfile, docker-compose)
- Kubernetes basics
- CI/CD pipelines
- Maven/Gradle build tools
- Application monitoring and logging
- Log aggregation (ELK stack, CloudWatch Logs)
- **Container orchestration best practices**
- **Kubernetes deployments, services, ingress**
- **Kubernetes ConfigMaps and Secrets**
- **Helm charts**
- **ArgoCD and GitOps**
- **Infrastructure as Code (Terraform, CloudFormation)**
- **Multi-stage Docker builds**
- **Container security scanning**

---

## Build & Dependency Management

- Maven lifecycle phases
- Gradle tasks and plugins
- Dependency scope (compile, runtime, test, provided)
- BOM (Bill of Materials)
- Multi-module projects
- Artifact repositories (Nexus, Artifactory)
- **Dependency conflict resolution**
- **Gradle build cache**
- **Incremental builds**
- **Build reproducibility**

---

## System Design & Architecture

- RESTful API design principles
- Scalability patterns
- Caching strategies
- Database sharding and replication
- CAP theorem
- Eventual consistency
- Rate limiting
- API versioning
- **Designing high-throughput systems**
- **Database partitioning strategies**
- **Multi-region architectures**
- **Disaster recovery planning**
- **SLA/SLO/SLI definitions**
- **Error budgets**
- **Load shedding and backpressure**

---

## Monitoring & Observability

- Application metrics
- Logging best practices (SLF4J, Logback)
- Distributed tracing
- Health checks
- Alerting
- MDC (Mapped Diagnostic Context) for correlation IDs
- Structured logging (JSON logs)
- Log levels and when to use them
- Prometheus metrics
- Grafana dashboards
- APM tools (New Relic, Datadog, Dynatrace)
- **OpenTelemetry adoption**
- **Custom metrics and instrumentation**
- **Log aggregation at scale**
- **Alerting best practices**
- **On-call and incident management**
- **Service Level Objectives (SLOs)**

---

## Performance & Optimization

- JVM tuning
- Memory management and garbage collection
- Connection pooling
- Caching (Redis, Caffeine)
- Database query optimization
- Async processing
- **JVM tuning for containers**
- **Memory leak detection and prevention**
- **Profiling tools (JProfiler, YourKit, Async Profiler)**
- **HikariCP connection pool tuning**
- **Query optimization techniques**
- **N+1 query problem solutions**
- **Lazy loading optimization**
- **HTTP/2 and connection multiplexing**

---

## Resilience & Fault Tolerance

- Retry mechanisms
- Timeout configuration
- Bulkhead pattern
- Graceful degradation
- Health checks and readiness probes
- Blue-green deployment
- Canary releases
- Rolling updates
- **Chaos engineering practices**
- **Fault injection testing**
- **Disaster recovery strategies**
- **Multi-region failover**
- **Circuit breaker configuration**

---

## Data Formats

- JSON (Jackson, Gson)
- XML (JAXB)
- Protocol Buffers
- Avro
- CSV processing
- **Schema evolution strategies**
- **Binary vs text formats**
- **Compression techniques**

---

## HTTP Clients

- RestTemplate
- WebClient (reactive)
- Feign client
- Apache HttpClient
- OkHttp
- **HTTP/2 support**
- **Connection pooling configuration**
- **Timeout and retry strategies**
- **SSL/TLS configuration**

---

## Scheduling

- @Scheduled annotation
- Cron expressions
- Quartz scheduler
- AWS EventBridge for scheduled tasks
- **Distributed scheduling challenges**
- **ShedLock for distributed locks**
- **Leader election in distributed systems**

---

## Internationalization (i18n)

- MessageSource
- Locale handling
- Resource bundles
- **Timezone handling**
- **Currency formatting**
- **Right-to-left (RTL) support**

---

## Version Control & Collaboration

- Git branching strategies (GitFlow, trunk-based)
- Pull request process
- Code review practices
- Merge vs rebase
- **Git hooks and automation**
- **Semantic versioning**
- **Changelog management**

---

## Common Interview Scenarios

- How to handle large file uploads
- Implementing pagination
- Bulk operations optimization
- Rate limiting implementation
- Implementing soft delete
- Audit logging
- Multi-tenancy
- Handling timezone conversions
- File processing (CSV, Excel)
- **Designing a high-throughput API**
- **Implementing distributed locks**
- **Cache invalidation strategies**
- **Handling duplicate requests**
- **Implementing search functionality**
- **Real-time notifications**
- **Batch processing optimization**

---

## Troubleshooting & Debugging

- Thread dumps analysis
- Heap dumps analysis
- OutOfMemoryError handling
- Performance profiling (JProfiler, VisualVM)
- Common exceptions and fixes
- Debugging production issues
- **Remote debugging**
- **Analyzing GC logs**
- **Memory leak investigation**
- **CPU profiling**
- **Connection leak detection**
- **Slow query identification**

---

## Software Development Principles

- SOLID principles
- DRY, KISS, YAGNI
- 12-Factor App methodology
- Clean code practices
- Code smells and refactoring
- **Hexagonal architecture**
- **Clean architecture**
- **Event-driven architecture principles**
- **Microservices design principles**

---

## Documentation

- JavaDoc
- README best practices
- Architecture decision records (ADRs)
- **API documentation strategies**
- **Runbook documentation**
- **System diagrams (C4 model)**

---

## Advanced Senior-Level Topics

### Distributed Systems
- **Consistency models (strong, eventual, causal)**
- **Consensus algorithms (Paxos, Raft)**
- **Vector clocks and logical time**
- **Gossip protocols**
- **Distributed locks and leader election**
- **Split-brain problem**

### Performance at Scale
- **Horizontal vs vertical scaling**
- **Database read/write splitting**
- **Connection pool sizing**
- **Bulk operations optimization**
- **CDN strategies**
- **Edge computing**

### Security Deep Dive
- **Zero-trust architecture**
- **Security boundaries in microservices**
- **API security best practices**
- **Threat modeling**
- **Security scanning and SAST/DAST**

### Cost Optimization
- **AWS cost optimization strategies**
- **Reserved instances vs spot instances**
- **Resource rightsizing**
- **Cost allocation tags**
- **FinOps practices**

### Multi-tenancy
- **Database per tenant vs shared database**
- **Schema per tenant**
- **Row-level security**
- **Tenant isolation strategies**
- **Performance isolation**

---

## Interview Preparation Tips

1. **Prioritize based on job description** - Focus on topics that align with the specific role
2. **Build practical examples** - Create a small project demonstrating key concepts
3. **Practice explaining concepts** - Be able to explain topics clearly and concisely
4. **Prepare real-world scenarios** - Have examples from your experience ready
5. **Stay current** - Review latest Spring Boot and AWS updates
6. **System design practice** - Be ready to design scalable, secure systems
7. **Code on whiteboard/paper** - Practice writing code without IDE assistance
8. **Deep dive into architecture** - Understand trade-offs in architectural decisions
9. **Know your resume** - Be ready to discuss every project and technology listed
10. **Prepare questions** - Have thoughtful questions about the role and team

---

## Senior-Level Focus Areas

For senior positions, interviewers will focus more on:
- **Architectural decisions and trade-offs**
- **System design at scale**
- **Leading technical initiatives**
- **Mentoring and code review practices**
- **Production incident handling**
- **Cost and performance optimization**
- **Cross-team collaboration**
- **Technical strategy and roadmap**

---

**Good luck with your interview!**
