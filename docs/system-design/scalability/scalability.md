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

## Подразделы
1. [Scaling](docs/system-design/scalability/scaling.md)
2. [Partitioning](docs/system-design/scalability/partitioning.md)
3. [Sharding](docs/system-design/scalability/sharding.md)
4. [Caching](docs/system-design/scalability/caching.md)
5. [Data Locality](docs/system-design/scalability/data-locality.md)
6. [Load Balancing](docs/system-design/scalability/load-balancing.md)

---

## Scaling vs Partitioning vs Sharding vs Locality

### Краткие определения

- **Scaling (** — процесс увеличения способности системы обрабатывать рост нагрузки (throughput/latency/requests). Включает vertical (scale-up) и horizontal (scale-out).
- **Partitioning** — логическое разделение пространства ресурсов/данных на независимые части (partitions). Partitioning может быть по функционалу, по колонкам, по строкам (horizontal) и т.д.
- **Sharding** — частный случай horizontal partitioning в распределённой среде: данные делятся по ключу на физические узлы (shards), каждый шард хранит подмножество данных.
- **Data Locality** — свойство системы, при котором данные и вычисления расположены «близко» друг к другу (время/сеть/топология), чтобы минимизировать задержки и трафик.

Эти понятия пересекаются, но решают разные стороны проблемы масштабируемости и производительности.

### Как они соотносятся логически — иерархия решений

- **Scaling** — верхний уровень: у тебя есть требование «нужно поддержать увеличенную нагрузку». Это цель.
- **Partitioning** — один из приемов достичь масштабирования: разбить систему на изолированные части (логически).
- **Sharding** — конкретная форма partitioning применённая к большим таблицам/ключ-значение: физическое распределение партиций по нодам.
- **Data Locality** — требование/оптимизация, которая влияет на то, как ты проектируешь partitioning/sharding и как реализуешь scaling (чтобы уменьшить сетевые вызовы и стоимость).

_Грубо:_ scaling → partitioning → sharding; а locality — критерий, который оптимизируют при выборе partitioning/sharding и deployment topology.

---

## Что решает каждый подход

### Что решает Scaling

- _Проблема_: увеличивается QPS / объём данных / число пользователей → растёт latency и падает доступность.
- _Решение_: добавить ресурсов (вертикально или горизонтально), увеличить параллелизм, использовать кеши и очереди.
- _Последствия_: может потребовать пересмотра архитектуры; horizontal scaling часто требует partitioning/sharding для эффективности.

### Что решает Partitioning

- _Проблема_: одна логическая таблица/сервис становится «слишком большой» или конфликтует по ресурсам (контенция, блокировки).
- _Решение_: разделить данные/функции на части, чтобы уменьшить contention и локализовать операции.
- _Последствия_: уменьшение конкуренции за ресурсы, но усложнение cross-partition операций.

### Что решает Sharding

- _Проблема_: partitioning надо реализовать в распределённой среде: нужны физические места хранения и маршрутизация.
- _Решение_: распределить части (shards) по узлам, использовать стратегии хеширования/диапазонов/lookup.
- _Последствия_: масштабирование записей, но добавляются сложности с транзакциями, joins и resharding.

### Что даёт улучшение Data Locality

- _Проблема_: высокие задержки и стоимость межузловых операций (latency, egress cost, throttling).
- _Решение_: проектирование так, чтобы чтения/записи и вычисления происходили «рядом» — colocation, geo-partitioning, replica routing.
- _Последствия_: уменьшение latency и сетевых ошибок, но риск горячих точек и сложностей при глобальных транзакциях.

---

## Когда что выбрать

### Когда делать partitioning?

- Горизонтальный рост данных → table size, index bloat, long-running scans.
- Высокая конкуренция lock contention на уровне записей/таблиц.
- Разные access patterns по подмножествам данных (например, region-based).

### Когда выбирать sharding (и какую стратегию)?

- Write-heavy нагрузка, большой dataset, требует распределённого хранения.
- Стратегия:
  - Hash-based: если нужен равномерный load; плохо подходит для range queries.
  - Range-based: если важны range queries и locality; риск hotspot.
  - Directory-based: если нужна максимальная гибкость и динамическое перемещение — но нужен lookup service.

### Когда оптимизировать Data Locality?

- Latency-sensitive приложения (игры, realtime, finance).
- Геораспределённые пользователи и дорогой egress.
- Большие ML/analytics jobs (выносить compute к данным).

---

## Metrics и наблюдаемость: что измерять для каждой стратегии

- **Scaling**: CPU, memory, network I/O, RPS, latency p50/p95/p99, error rate.
- **Partitioning**: per-partition latency, QPS, size on disk, lock contention, number of cross-partition queries.
- **Sharding**: per-shard throughput, shard skew (stddev of QPS/size), rebalancing events, resharding time.
- **Locality**: local hit ratio, cross-node request %, average hop count, egress cost, remote read latency.

---

## Короткое резюме

* **Scaling** — определить bottleneck, stateless-first, bounded buffers, autoscaling.
* **Partitioning** — равномерность, выбор правильного ключа, hotspot контроль.
* **Sharding** — hash/range/directory, минимизация cross-shard, support resharding.
* **Caching** — TTL, eviction, consistency, stampede protection, измерять hit ratio.
* **Data Locality** — co-location, вычисления рядом с данными, geo-aware архитектура.
* **Load Balancing** — health-checks, выбор алгоритма, распределение по locality.
