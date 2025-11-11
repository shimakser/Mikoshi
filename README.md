# Java Software Engineer Road Map

---

## Оглавление

### Java ecosystem, JVM
1. [Java ecosystem](docs/ecosystem/1-ecosystem.md)
2. [ClassLoaders](docs/ecosystem/2-classloaders.md)
3. [Java memory model (JMM), String pool](docs/ecosystem/3-jmm.md)
4. [Garbage collectors, References](docs/ecosystem/4-gc.md)

### ООП в Java
1. [ООП](docs/oop/1-oop.md)
2. [Виды классов](docs/oop/2-classes.md)
3. [Модификаторы доступа, Ключевые слова](docs/oop/3-access-modifiers-packages.md)
4. [Класс Object, Контракт `equals`/`hashCode`/`toString`, сравнение объектов](docs/oop/4-object-compare.md)
5. [Иммутабельность vs мутабельность, защитное копирование](docs/oop/5-mutable-vs-immutable.md)

### Java Core
1. [Exceptions](docs/core/1-exceptions.md)
2. [Generics](docs/core/2-generics.md)
3. [Collections](docs/core/3-collections.md)
4. [Lambda](docs/core/4-lambda.md)
5. [Functional interfaces](docs/core/5-functional-interface.md)
6. [Optional](docs/core/6-optional.md)
7. [Stream API](docs/core/7-stream.md)
8. [Files, NIO.2]()
9. [Reflection API]()

### Concurrency and Multithreading
1. [Basic concepts](docs/concurrency/basic-concepts.md)
2. [Threads](docs/concurrency/threads.md)
3. [Synchronization](docs/concurrency/synchronized.md)
4. [Concurrency collections](docs/concurrency/concurrency-collections.md)
5. [Volatile & Atomic](docs/concurrency/volatile-atomic.md)
6. [Thread Pool](docs/concurrency/thread-pool.md)
7. [Fork Join Pool](docs/concurrency/fork-join-pool.md)
8. [CountDownLatch, CyclicBarrier](docs/concurrency/countdownlatch-cyclicbarrier.md)
9. [Semaphore](docs/concurrency/semaphore.md)
10. [Virtual threads](docs/concurrency/virtual-threads.md)

### Spring Framework
1. [Spring: Beans, Patterns, Conditions](docs/spring/spring-beans-ioc-di.md)
2. [Spring Boot](docs/spring/spring-boot.md)
3. [Spring Security](docs/spring/spring-security.md)
4. [Spring Cloud](docs/spring/spring-cloud.md)
5. [Spring AOP](docs/spring/spring-aop.md)
6. [Transactions in Spring](docs/spring/transactional.md)

### Tests
[]()

### Databases
1. [CAP-теорема](docs/db/cap.md)
2. [Database normalization](docs/db/normalization.md)
3. [Selecting database](docs/db/selecting-db.md)
4. [Hibernate](docs/db/hibernate.md)
5. [Partitioning, Tuning, Backup/Restore, Introspection](docs/db/partitioning-tuning-backup-restore-introspection.md)
    #### SQL Databases
      1. [Basic SQL commands](docs/db/basic-sql.md)
      2. [ACID, Isolation](docs/db/acid.md)
      3. [Optimizing DB queries](docs/db/optimize-query.md) 
      4. [Indexes](docs/db/indexes.md)
      5. [Optimistic & Pessimistic Locking](docs/db/optimistic-pisimistic-locking.md) 
      6. [MVCC, Vacuum](docs/db/mvcc-vacuum.md) 
    #### NoSQL Databases
      1. [NoSQL, BASE](docs/db/nosql.md)
      2. [MongoDB](docs/db/mongodb.md)
      3. [Apache Casandra](docs/db/apache-casandra.md)

### Security
1. [JWT](docs/security/1-jwt.md)
2. [OAuth2](docs/security/2-oauth2.md)
3. [HTTPS](docs/security/3-https.md)

### Message brokers
1. [Apache Kafka](docs/message-brokers/1-kafka.md)
2. [RabbitMQ](docs/message-brokers/2-rabbitmq.md)

### Caches
1. [LRU cache](docs/cache/lru-cache.md)
1. [Spring Cache](docs/cache/spring-cache.md)
2. [Redis](docs/cache/redis.md)
3. [Hazelcast](docs/cache/hazelcast.md)

### Patterns
1. [GRASP, SOLID](docs/patterns/1-grasp-solid.md)
2. [Creational patterns](docs/patterns/2-creational.md)
3. [Structural  patterns](docs/patterns/3-structural.md)
4. [Behavioral patterns](docs/patterns/4-behavioral.md)
5. [Microservices patterns](docs/patterns/5-microservices-patterns.md)

### Algorithms
1. [Big O](docs/algorithms/big-o.md)
2. [Collections and Data structure](docs/algorithms/collections-and-data-structure.md)
2. [Sorting](docs/algorithms/sotring-alg.md)
3. [Search types](docs/algorithms/search-types.md)

### DevOps & Cloud
1. [Dockerfile networks](docs/devops-cloud/dockerfile-networks.md)

### Networks
1. [IP, DNS and Networks](docs/networks/ip-dns-networks.md)
2. [Reverse proxy server](docs/networks/reverse-proxy-server.md)
3. [RESTful, gRPC](docs/networks/restful-grpc.md)

### System Design
1. [**Scalability**](docs/system-design/scalability/scalability.md)

    1.1 [Scaling](docs/system-design/scalability/scaling.md)
    
    1.2 [Partitioning](docs/system-design/scalability/partitioning.md)
    
    1.3 [Sharding](docs/system-design/scalability/sharding.md)
        
    1.4 [Caching](docs/system-design/scalability/caching.md)
        
    1.5 [Data Locality](docs/system-design/scalability/data-locality.md)
        
    1.6 [Load Balancing](docs/system-design/scalability/load-balancing.md)
2. [**Reliability & Resilience**](docs/system-design/reliability-resilience/reliability-resilience.md)

   1.1 [Failover & Replication](docs/system-design/reliability-resilience/failover-replication.md)

   1.2 [Backpressure](docs/system-design/reliability-resilience/backpressure.md)

   1.3 [Rate Limiting](docs/system-design/reliability-resilience/rate-limiting.md)

   1.4 [Circuit Breaker](docs/system-design/reliability-resilience/circuit-breaker.md)

   1.5 [Graceful Degradation](docs/system-design/reliability-resilience/graceful-degradation.md)

   1.6 [Distributed Coordination](docs/system-design/reliability-resilience/distributed-coordination.md)

   1.7 [Health Checks](docs/system-design/reliability-resilience/health-checks.md)
3. [Observability: Metrics, Logs, Tracing](docs/system-design/observability.md)
4. **Fault Tolerance & Recovery**

   1.1 [Leader Election](docs/system-design/fault-tolerance-and-recovery/leader-election.md)

   1.2 [Data Consistency](docs/system-design/fault-tolerance-and-recovery/data-consistency.md)

   1.3 [Distributed Transactions](docs/system-design/fault-tolerance-and-recovery/distributed-transactions.md)

   1.4 [PITR / Backup / Restore](docs/system-design/fault-tolerance-and-recovery/pitr-backup-restore.md)


### Management
1. [Task estimation](docs/management/task-estimation.md)
2. [How to Lead a Team](docs/management/lead-team.md)

### Architecture of computer systems
_Source:_ [Carnegie Mellon University Undergraduate Computer Architecture.](https://www.youtube.com/playlist?list=PL5PHm2jkkXmi5CxxI7b3JCL1TWybTDtKq)
1. [Instruction Set Architecture](docs/architecture-of-computer-systems/instruction-set-architecture.md)
2. [Microarchitecture](docs/architecture-of-computer-systems/microarchitecture.md)
3. [Pipelines](docs/architecture-of-computer-systems/pipelines.md)

### Operating Systems
_Source:_ [UC Berkeley CS162 Operating Systems](https://www.youtube.com/playlist?list=PLF2K2xZjNEf97A_uBCwEl61sdxWVP7VWC)
1. [](docs/operating-systems/.md)
