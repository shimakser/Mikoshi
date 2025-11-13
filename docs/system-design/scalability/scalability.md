# Scalability

_Как справляться с увеличением нагрузки?_

---

Сервис является масштабируемым, если его производительность увеличивается пропорционально добавляемым ресурсам. Как правило, увеличение производительности означает обслуживание большего количества единиц работы, но это может быть и обработка более крупных единиц работы, например, при росте массивов данных.

Другой способ взглянуть на производительность в сравнении с масштабируемостью:
- Если у вас проблемы с производительностью, то ваша система работает медленно для одного пользователя.
- Если у вас проблемы с масштабируемостью, то ваша система работает быстро для одного пользователя, но медленно при большой нагрузке.

---

**Latency (Латентность)** — это время выполнения некоторого действия или получения некоторого результата.

**Throughput (Пропускная способность)** — это количество таких действий или результатов в единицу времени.

Как правило, вы должны стремиться к максимальной пропускной способности при приемлемой задержке.

---

## Подразделы: 
- [Scaling](docs/system-design/scalability/scaling.md)
- [Partitioning](docs/system-design/scalability/partitioning.md)
- [Sharding](docs/system-design/scalability/sharding.md)
- [Caching](docs/system-design/scalability/caching.md)
- [Data Locality](docs/system-design/scalability/data-locality.md)
- [Load Balancing](docs/system-design/scalability/load-balancing.md)

---

## Scaling vs Partitioning vs Sharding vs Locality

### Краткие определения

- **Scaling (масштабирование)** — процесс увеличения способности системы обрабатывать рост нагрузки (throughput/latency/requests). Включает vertical (scale-up) и horizontal (scale-out).
- **Partitioning (партиционирование)** — логическое разделение пространства ресурсов/данных на независимые части (partitions). Partitioning может быть по функционалу, по колонкам, по строкам (horizontal) и т.д.
- **Sharding (шардинг)** — частный случай horizontal partitioning в распределённой среде: данные делятся по ключу на физические узлы (shards), каждый шард хранит подмножество данных.
- **Data Locality (локальность данных)** — свойство системы, при котором данные и вычисления расположены «близко» друг к другу (время/сеть/топология), чтобы минимизировать задержки и трафик.

Эти понятия пересекаются, но решают разные стороны проблемы масштабируемости и производительности.

### Как они соотносятся логически — иерархия решений

- **Scaling** — верхний уровень: у тебя есть требование «нужно поддержать увеличенную нагрузку». Это цель.
- **Partitioning** — один из приемов достичь масштабирования: разбить систему на изолированные части (логически).
- **Sharding** — конкретная форма partitioning применённая к большим таблицам/ключ-значение: физическое распределение партиций по нодам.
- **Data Locality** — требование/оптимизация, которая влияет на то, как ты проектируешь partitioning/sharding и как реализуешь scaling (чтобы уменьшить сетевые вызовы и стоимость).

Грубо: scaling → partitioning → sharding; а locality — критерий, который оптимизируют при выборе partitioning/sharding и deployment topology.

### Разложение по вопросам: что решает каждый подход

#### Что решает Scaling?

- _Проблема_: увеличивается QPS / объём данных / число пользователей → растёт latency и падает доступность.
- _Решение_: добавить ресурсов (вертикально или горизонтально), увеличить параллелизм, использовать кеши и очереди.
- _Последствия_: может потребовать пересмотра архитектуры; horizontal scaling часто требует partitioning/sharding для эффективности.

#### Что решает Partitioning?

- _Проблема_: одна логическая таблица/сервис становится «слишком большой» или конфликтует по ресурсам (контенция, блокировки).
- _Решение_: разделить данные/функции на части, чтобы уменьшить contention и локализовать операции.
- _Последствия_: уменьшение конкуренции за ресурсы, но усложнение cross-partition операций.

#### Что решает Sharding?

- _Проблема_: partitioning надо реализовать в распределённой среде: нужны физические места хранения и маршрутизация.
- _Решение_: распределить части (shards) по узлам, использовать стратегии хеширования/диапазонов/lookup.
- _Последствия_: масштабирование записей, но добавляются сложности с транзакциями, joins и resharding.

#### Что даёт улучшение Data Locality?

- _Проблема_: высокие задержки и стоимость межузловых операций (latency, egress cost, throttling).
- _Решение_: проектирование так, чтобы чтения/записи и вычисления происходили «рядом» — colocation, geo-partitioning, replica routing.
- _Последствия_: уменьшение latency и сетевых ошибок, но риск горячих точек и сложностей при глобальных транзакциях.

### Точки принятия решения: когда что выбрать

#### Когда начать с vertical scaling?

- Малые или предсказуемые нагрузки.
- Монолитное приложение, где рефакторинг дорогой.
- Если проблема — недостаток CPU / RAM / IO на конкретной ноде и обновление железа дешевле реархитектуры.

#### Когда переходить к horizontal scaling?

- Вертикальный предел или непропорциональная стоимость scale-up.
- Требуется высокая доступность и fault-tolerance.
- Начиная рефакторинг под микросервисы / контейнеризацию.

#### Когда делать partitioning?

- Горизонтальный рост данных → table size, index bloat, long-running scans.
- Высокая конкуренция lock contention на уровне записей/таблиц.
- Разные access patterns по подмножествам данных (например, region-based).

#### Когда выбирать sharding (и какую стратегию)?

- Write-heavy нагрузка, большой dataset, требует распределённого хранения.
- Стратегия:
  - Hash-based: если нужен равномерный load; плохо подходит для range queries.
  - Range-based: если важны range queries и locality; риск hotspot.
  - Directory-based: если нужна максимальная гибкость и динамическое перемещение — но нужен lookup service.

#### Когда оптимизировать Data Locality?

- Latency-sensitive приложения (игры, realtime, finance).
- Геораспределённые пользователи и дорогой egress.
- Большие ML/analytics jobs (выносить compute к данным).

### Глубокие компромиссы (trade-offs) — практические нюансы

#### Consistency vs Availability vs Locality

- Если ты помещаешь копии данных ближе к клиентам (replication + locality), то поддержание сильной консистентности потребует межрегиональных round-trips (повышает latency) либо сложных протоколов (Paxos/Spanner).
- _Выбор_: пожертвовать latency (для strong consistency) либо согласованностью (выбрать eventual, read-repair, conflict-free replicated data types — CRDTs).

#### Hotspots vs Locality

- Локализация по бизнес-объекту (например, шардинг по customer_id) улучшает locality, но может вызвать hotspot, если многие пользователи работают с одним customer.
- _Митигейт_: adaptive rebalancing, split-key techniques, replicate hot keys.

#### Cross-partition transactions

- Partitioning/sharding усложняют ACID. Решения:
  - 2PC: дорогой и медленный.
  - Saga / Eventual compsation: более масштабируемо, но нужен переосмысленный бизнес-лог.
  - Single-shard design: проектировать, чтобы транзакции оставались в рамках одного shard, если возможно.

#### Operational overhead

- Sharding требует сервисов метаданных (routing), мониторинга per-shard, автоматического rebalancer’а и инструментов миграции.
- Data locality требует topology-aware schedulers и deployment pipeline, что увеличивает devops-комплексность.

### Конкретные архитектурные шаблоны и mapping

#### Малый рост → vertical + caching

- Монолит, Redis in-process + CDN. Мало изменений в логике — быстро.

#### Большой read-heavy рост → replication + caching + read replicas

- Репликация для scaling reads, кэширование hot objects, CDN для statics.

#### Большой write-heavy рост → sharding + partition-aware application

- Hash/range sharding, partition-aware routing, single-shard transactions, background rebalancer.

#### Геораспределённые пользователи → geo-partitioning + locality-aware routing

- Данные/шарды привязаны к регионам, read-local strategies, cross-region sync по необходимости.

#### Analytics / ML → move compute to data

- HDFS/S3 co-located compute (Spark), avoid moving TBs across regions.

### Metrics и наблюдаемость: что измерять для каждой стратегии

- **Scaling**: CPU, memory, network I/O, RPS, latency p50/p95/p99, error rate.
- **Partitioning**: per-partition latency, QPS, size on disk, lock contention, number of cross-partition queries.
- **Sharding**: per-shard throughput, shard skew (stddev of QPS/size), rebalancing events, resharding time.
- **Locality**: local hit ratio, cross-node request %, average hop count, egress cost, remote read latency.

_Важно_: для принятия решений нужны метрики со стратификацией по shard/region/service.

### Пошаговая методика принятия решения (checklist)

1. Собрать факты: текущее QPS, growth rate, p95/p99 latency, working set size, distribution of requests по ключам.
2. Определить bottleneck: CPU / IO / DB locks / network / memory.
3. Оценить quick wins: кэширование, индексирование, query optimization.
4. Решить масштабирование: vertical short-term; horizontal long-term.
5. Если horizontal → определить partitioning axis: по access pattern, по entity ownership, по geo.
6. Выбрать sharding strategy: hash (balance) vs range (locality) vs directory (flexibility).
7. Планировать locality: где должны находиться данные иcompute; реплики по region.
8. Проектировать миграцию: безопасный resharding, rolling migrations, dual-write / outbox.
9. Внедрить наблюдаемость: per-shard metrics, alerts на skew/hotspot/evictions.
10. Тестировать: load tests (cold start, resharding), chaos tests, disaster recovery drills.

### Частые ошибки и anti-patterns

- Шардирование «наугад» без анализа access patterns → быстрый рост technical debt.
- Игнорирование data locality при геораспределении → высокие egress-расходы и latency.
- Попытки ACID across shards без переосмысления бизнес-процессов → сложные 2PC-реализации.
- Решение «мы всё решим later» — оставляет монолитную БД у которой лимит масштабирования.


