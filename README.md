# Java Software Engineer Road Map

---

## Оглавление

### Java ecosystem, JVM
1. [Java ecosystem](docs/ecosystem/ecosystem.md)
2. [ClassLoaders](docs/ecosystem/classloaders.md)
3. [Java memory model (JMM), String pool](docs/ecosystem/jmm.md)
4. [Garbage collectors, References](docs/ecosystem/gc.md)

### ООП в Java
1. [ООП](docs/oop/oop.md)
2. [Виды классов](docs/oop/classes.md)
3. [Модификаторы доступа, Ключевые слова](docs/oop/access-modifiers-packages.md)
4. [Класс Object, Контракт `equals`/`hashCode`/`toString`, сравнение объектов](docs/oop/object-compare.md)
5. [Иммутабельность vs мутабельность, защитное копирование](docs/oop/mutable-vs-immutable.md)

### Java Core
1. [Exceptions](docs/core/exceptions.md)
2. [Generics](docs/core/generics.md)
3. [Collections](docs/core/collections.md)
4. [Lambda](docs/core/lambda.md)
5. [Functional interfaces](docs/core/functional-interface.md)
6. [Optional](docs/core/optional.md)
7. [Stream API](docs/core/stream.md)
8. [Files, NIO.2](docs/core/files.md)
9. [Reflection API](docs/core/reflection.md)

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

### Современные фичи Java
1. [GraalVM](docs/modern-java/graal-vm.md)

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
      7. [Views](docs/db/views.md)
    #### NoSQL Databases
      1. [NoSQL, BASE](docs/db/nosql.md)
      2. [MongoDB](docs/db/mongodb.md)
      3. [Apache Casandra](docs/db/apache-casandra.md)

### Security
1. [JWT](docs/security/jwt.md)
2. [OAuth2](docs/security/oauth2.md)
3. [HTTPS](docs/security/https.md)
4. [Security Filter Chain Internals](docs/security/security-filter-chain-internals.md)
5. [CORS](docs/security/cors.md)
6. [CSRF](docs/security/csrf.md) 

### Project assembly automation system
1. [Gradle](docs/project-assembly-automation-system/gradle.md)
2. Maven

### Message brokers
1. [Apache Kafka](docs/message-brokers/kafka.md)
2. [RabbitMQ](docs/message-brokers/rabbitmq.md)

### Caches
1. [LRU cache](docs/cache/lru-cache.md)
1. [Spring Cache](docs/cache/spring-cache.md)
2. [Redis](docs/cache/redis.md)
3. [Hazelcast](docs/cache/hazelcast.md)

### Patterns
1. [GRASP, SOLID](docs/patterns/grasp-solid.md)
2. [Creational patterns](docs/patterns/creational.md)
3. [Structural  patterns](docs/patterns/structural.md)
4. [Behavioral patterns](docs/patterns/behavioral.md)
5. [Distributed Transactions](docs/patterns/distributed-transactions.md)

### Algorithms
1. [Big O](docs/algorithms/big-o.md)
2. [Collections and Data structure](docs/algorithms/collections-and-data-structure.md)
2. [Sorting](docs/algorithms/sotring-alg.md)
3. [Search types](docs/algorithms/search-types.md)

### DevOps
1. [Dockerfile networks](docs/devops/dockerfile-networks.md)
2. [Deployment strategies](docs/devops/deployment-strategies.md)

### Cloud
1. **AWS**<br>
   1.1 [Lambda](docs/cloud/lambda.md)<br>
   1.2 [DynamoDB](docs/cloud/dynamo-db.md)

### Networks
1. [IP, DNS and Networks](docs/networks/ip-dns-networks.md)
2. [Reverse proxy server](docs/networks/reverse-proxy-server.md)
3. [RESTful, gRPC](docs/networks/restful-grpc.md)

### Performance
1. [JVM/Java Performance Optimization](docs/performance/jvm-java-performance-optimization.md)

### System Design
1. [**Scalability**](docs/system-design/scalability/scalability.md)<br>
    1.1 [Scaling](docs/system-design/scalability/scaling.md)<br>
    1.2 [Partitioning](docs/system-design/scalability/partitioning.md)<br>
    1.3 [Sharding](docs/system-design/scalability/sharding.md)<br>
    1.4 [Caching](docs/system-design/scalability/caching.md)<br>
    1.5 [Data Locality](docs/system-design/scalability/data-locality.md)<br>
    1.6 [Load Balancing](docs/system-design/scalability/load-balancing.md)
2. **Reliability & Resilience**<br>
   2.1 [Failover](docs/system-design/reliability-resilience/failover.md)<br>
   2.2 [Replication](docs/system-design/reliability-resilience/replication.md)<br>
   2.3 [Backpressure](docs/system-design/reliability-resilience/backpressure.md)<br>
   2.4 [Rate Limiting](docs/system-design/reliability-resilience/rate-limiting.md)<br>
   2.5 [Circuit Breaker](docs/system-design/reliability-resilience/circuit-breaker.md)<br>
   2.6 [Bulkhead](docs/system-design/reliability-resilience/bulkhead.md)<br>
   2.7 [Graceful Degradation](docs/system-design/reliability-resilience/graceful-degradation.md)<br>
   2.8 [Distributed Coordination](docs/system-design/reliability-resilience/distributed-coordination.md)<br>
   2.9 [Health Checks](docs/system-design/reliability-resilience/health-checks.md)
3. [**Observability**: Metrics, Logs, Tracing](docs/system-design/observability.md)
4. **Fault Tolerance & Recovery**<br>
   4.1 [Failure Models]()<br>
   4.2 [Failure Detection & Recovery Triggers]()<br>
   4.3 [Failover Strategies (Recovery Phase)]()<br>
   4.4 [Data Recovery & Post-Failure Consistency]()<br>
   4.5 [PITR (Point-In-Time Recovery)]()<br>
   4.6 [Backup & Restore Strategies]()<br>
   4.7 [Disaster Recovery (DR)]()<br>
   4.8 [RPO / RTO]()<br>
   4.9 [Self-Healing & Stabilization After Failure]()<br>
5. [High-Throughput System Design](docs/system-design/high-throughput-system-design.md)

### Management и инженерные навыки
1. [Task estimation](docs/management/task-estimation.md)
2. [How to Lead a Team](docs/management/lead-team.md)
3. [Надежность и эксплуатация: SLO/SLA/SLI, постмортемы](docs/management/slo-sla-sli.md)
4. [Безопасная разработка (SSDLC): угрозы и гайдлайны](docs/management/ssdlc.md)
5. [RFC/ADR‑процессы](docs/management/rfc-adr.md)

### Architecture of computer systems
_Source:_ [Carnegie Mellon University Undergraduate Computer Architecture.](https://www.youtube.com/playlist?list=PL5PHm2jkkXmi5CxxI7b3JCL1TWybTDtKq)
1. [Instruction Set Architecture](docs/architecture-of-computer-systems/instruction-set-architecture.md)
2. [Microarchitecture](docs/architecture-of-computer-systems/microarchitecture.md)
3. [Pipelines](docs/architecture-of-computer-systems/pipelines.md)

### Operating Systems
_Source:_ [UC Berkeley CS162 Operating Systems](https://www.youtube.com/playlist?list=PLF2K2xZjNEf97A_uBCwEl61sdxWVP7VWC)
1. [](docs/operating-systems/.md)
