# High-Throughput System Design

---

High-Throughput — это не про один сервис.

Это про всю цепочку:

    LB → API → Business logic → Kafka/Hazelcast → DB/Cache → Notifications

---

##  Cache

### Всегда читать их кэша.

- Чтение из Redis/Hazelcast стоит на порядки дешевле по CPU, IO, network и latency, чем чтение из реляционной БД.
- Писать логику актуализации проще, чем бороться с перегрузкой DB.

| Источник             | Латентность | Throughput    |
| -------------------- | ----------- | ------------- |
| Hazelcast near-cache | 0.03–0.2 ms | ~1M ops/s     |
| Redis                | 0.1–1 ms    | ~0.2–1M ops/s |
| Postgres             | 1–10 ms     | ~5k ops/s     |

---

## Kafka

* batch produce (linger.ms 1–5ms)
* compression = lz4/snappy
* acks = 1 или all (зависит от SLA)
* partitions: много (по формуле load / target throughput per partition)

---

## Основные bottlenecks, которые встречаются на практике

1. **Thread pool exhaustion** — все worker-пулы заполнены, новые задачи блокируются или отклоняются → high latency, timeouts, retries.
    - Bounded queue + reject policy: не давать бесконечную очередь (CallerRuns или Abort).
    - Bulkheads: разделить пулы по зависимостям (DB calls vs CPU tasks).
    - Reduce blocking: move blocking IO to separate thread pool (Netty event-loop should not block).
    - Apply backpressure upstream (429 / rate-limit).
    - Tune pool sizes: set core/max according to CPU vs IO-bound type.
    - Use async non-blocking I/O where possible (Netty, reactive stacks).
2. **Connection pool starvation** — все DB/HTTP connections заняты, новые запросы ждут соединения → timeout/queueing.
    - Increase pool size (but only if DB can handle it) — careful.
    - Use separate pools per dependency (bulkhead on connection level).
    - Make calls non-blocking/asynchronous, reduce time per connection.
    - Optimize DB queries to reduce time holding connection.
    - Client-side caching to reduce total number of DB calls.
3. **GC pauses (особенно allocation-heavy код)** — requent allocation → more GC cycles → STW pauses → latency spikes.
    - Reduce allocations: reuse objects, use primitive collections, avoid boxing, reuse buffers.
    - Use off-heap stores for heavy caches (Hazelcast, Chronicle Map).
    - Tune GC: G1/ ZGC / Shenandoah depending on heap and latency needs.
    - Use object pooling cautiously (may add complexity).
    - Profile allocation hotspots and fix code (see §7 profiling tools).
4. **Kafka consumer lag** — consumers не успевают обрабатывать incoming messages → lag grows → eventual OOM or more lag.
    - Scale consumers (more instances / more partitions).
    - Batch processing and bulk commits.
    - Optimize processing function.
    - Rate limit incoming producers or apply backpressure up the chain.
5. **Disk IO saturation** — disk throughput or IOPS exhausted → writes/reads slow → blocking operations.
    - Use SSD / NVMe for IO-heavy components.
    - Tuning fsync policies / batching: e.g., Kafka flush settings.
    - Sharding data across disks/nodes
    - Use write-behind caches and batch writes.
    - Move heavy read workloads to replicas.
6. **DB lock contention, index contention** — транзакции блокируют друг друга (блокируют), или запросы выполняют полное сканирование таблиц из-за отсутствующих/неэффективных индексов.
    - Optimize queries (indexes, rewrite queries).
    - Denormalize / pre-aggregate to avoid expensive joins.
    - Use partitioning in DB (range/list) to localize load.
    - Short transactions and smaller batch sizes.
    - Use optimistic concurrency or application-level locks for hot rows.
    - Move write-heavy work to append-only log and batch-apply to DB.