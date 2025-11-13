# Caching

_Кэширование_ — целенаправленное хранение копий данных ближе к месту потребления для уменьшения латентности, снижения нагрузки на источники данных и повышения пропускной способности. Ниже — систематизированный, детальный набор принципов, паттернов, практик и инженерных деталей.

---

## Типы кешей по расположению и назначению

1. **Client-side (browser / app)** — HTTP cache, localStorage, in-memory в мобильных клиентах. Быстрый доступ, но слабый контроль инвалидации.
2. **CDN / edge** — статический контент и динамический контент с edge-логикой. Уменьшает latency для геораспределённых пользователей.
3. **Reverse proxy / HTTP cache** — NGINX, Varnish. Кеширование ответов сервиса на уровне HTTP.
4. **Application-level cache (in-process)** — LRU в памяти JVM (Caffeine). Минимальная латентность, но ограничен RAM и не шарится.
5. **Distributed cache (out-of-process)** — Redis, Memcached, Aerospike. Поддерживает шардинг и репликацию.
6. **Database caching / materialized views** — результаты сложных запросов, precomputed aggregates.
7. **Query cache / page cache / object cache** — разные уровни кэширования в зависимости от гранулярности.
8. **Write caches / write-behind** — буферизация записей перед DB.

---

## Cache semantics: consistency & correctness

_Важно при проектировании._

- **Strong consistency** — чтение сразу видит последнюю запись (требует синхронизации или single source of truth).
- **Eventual consistency** — обновление видимо позже; обычно в distributed caches.
- **Stale-while-revalidate** — отдаём старое значение и параллельно обновляем.
- **Monotonic reads** — клиент видит неубывающие версии.

Выбор семантики диктуется требованиями: критичные финансовые данные → сильная консистентность; профили/фото → eventual и stale-while-revalidate.

---

## Основные паттерны использования

### Cache-Aside (Lazy loading)

_Самый распространённый._

_Процесс:_
- Сервис пытается прочитать из кэша.
- При промахе — чтение из DB и запись результата в кэш.
- Запись в DB не трогает кэш напрямую (или инвалидация/обновление отдельно).

_Плюсы:_ простой, контроль над консистентностью.
_Минусы:_ нагрузка на DB при cache cold, race conditions при конкурентных промахах.

_Пример псевдокода:_


    Object get(key) {
        Object v = cache.get(key);
        if (v != null) return v;
        // cache miss
        v = db.read(key);
        cache.set(key, v, TTL);
        return v;
    }

### Read-Through / Write-Through

- **Read-through**: кэш сам загружает данные у источника при промахе (transparent to app).
- **Write-through**: при записи в кэш запись синхронно пишется в DB (согласованность, но высокая задержка).
- **Write-back (write-behind)**: запись в кэш, асинхронная запись в DB позже — повышенная производительность, риск потери данных при крахе.

### Stale-while-revalidate and Early Expiration

- Отдаём устаревший объект и в фоне обновляем. Уменьшает latency и предотвращает thundering herd.
- Требует фонового воркера и контроля версий.

### Cache-aside + Locking / Request Coalescing

Чтобы избежать параллельных хитов DB при одном и том же ключе:

- **Distributed lock** (Redis redlock) вокруг загрузки при промахе.
- **Request coalescing** (singleflight) — первый хост загружает, остальные ждут результата.
- **Promise cache** — хранить в кэше маркер “в процессе загрузки” и клиентам возвращать ожидание.

---

## Eviction и replacement policies

- **LRU (Least Recently Used)** — простая и часто эффективная.
- **LFU (Least Frequently Used)** — лучше если частое повторное обращение важно.
- **ARC (Adaptive Replacement Cache)** — комбинирует LRU и LFU.
- **Random** — полезно при распределённых системах для упрощения.
- **TTL-based** — время жизни вместо эвикшена.

В продакшне часто комбинируют TTL + LRU: TTL ограничивает старение, LRU управляет ограниченной памятью.

---

## Hot keys, hot partitions и mitigation

_Проблема:_ ключи с очень высокой частотой (hot keys) перегружают одну ноду.

_Решения:_
- Key sharding / hashing + virtual nodes.
- Adaptive replication: временно делать дополнительные реплики для hot key.
- Request fan-out to multiple cache shards (read replicas) и consistent hashing по replica set.
- Rate limiting / throttling на уровне приложения.
- Split key: добавить префикс (user_id % N) — увеличивает cardinality и выравнивает нагрузку (но усложняет чтения агрегатов).

---

## Cache invalidation

Инвалидация — самая сложная часть.

1. **Time-based (TTL)** — простое, но приводит к стейлу до истечения TTL.
2. **Explicit invalidation** (delete / update) — при write операциях сервис явно удаляет/обновляет кэш.
3. **Versioning / Cache-busting** — ключ включает версию/etag (e.g., user:123:v42) при schema/semantic changes.
4. **Event-driven invalidation** — подписка на изменения (change data capture, CDC), уведомления через message broker.
5. **Two-step update**: write DB → publish event → consumers invalidate cache. Использовать outbox pattern, чтобы избежать потерянных invalidation.

_Рекомендации:_
- Для критичных данных — explicit invalidation + short TTL.
- Для read-heavy и eventual-consistency-tolerant — longer TTL + background refresh.

---

## Проблемы и контрмеры

### Cache stampede / thundering herd

Когда много клиентов одновременно попадают в промах и бьют по DB.

_Контрмеры:_
- Locking / request coalescing (см. выше).
- Randomized early expiration (jitter): TTL = baseTTL ± random to spread revalidations.
- Leaky bucket / token bucket: ограничивать число одновременных revalidations.
- Serve stale + async refresh (stale-while-revalidate).

### Cache penetration

Запросы к несуществующим ключам (например, userId несуществует) постоянно попадают в DB.

_Решения:_
- Кешировать «не найдено» (negative caching) с коротким TTL.
- Валидация запросов на уровне приложений.

### Cache poisoning / security

- Проверять входные ключи, предотвращать произвольное расширение пространства ключей.
- Не кэшировать конфиденциальные данные без шифрования/контроля доступа.
- Защита от spoofed invalidation events: подписки с аутентификацией/авторизацией.

### Cache coherence при распределённых транзакциях

- Использовать outbox pattern: запись в DB и в outbox в одной транзакции, потом потребитель публикует событие для инвалидации.
- Если необходима строгая согласованность, избегать кэша для критичных операций или применить write-through.

---

## Проектирование распределённого кэша

### Шардирование кэша

- Client-side hashing (ketama/consistent hashing) — клиенты вычисляют shard.
- Proxy/router (twemproxy, Envoy) — маршрутизирует запросы к нужным нодам.
- Coordinatorless (Cassandra-like) — каждая нода умеет принимать запросы в свой диапазон.

### Репликация

- Синхронная репликация даёт согласованность, но замедляет writes.
- Асинхронная повышает throughput, но вводит возможность stale reads.
- Redis Cluster: шардирование + реплики (master/replica), single-key atomic ops.

### Failover и recovery

- Replica promotion (automatic) + re-sync при восстановлении.
- Data persistence (AOF/RDB) для восстановлений при write-behind.
- Throttling миграций и rebalance чтобы не задушить кластер.

---

## Capacity planning и расчёт размера кеша

1. Оценить рабочий набор (working set) — уникальных ключей, активно запрошенных в окне T (например, 1 час).
2. Средний размер одного объекта (bytes).
3. Целевой hit ratio → рассчитать требуемое число объектов в памяти.
4. Учитывать накладные расходы (headers, pointers) — в Redis объект занимает больше, чем payload.

Формула (упрощённо):


    required_memory = working_set_size * avg_object_size * (1 + overhead_factor)
где overhead_factor = 0.2..0.5 в зависимости от реализации.

---

## Best practices

1. Tiered caching: edge CDN → reverse proxy → distributed cache → local in-process cache (multi-level caching) — согласованность управлять через versioning и TTL.
2. Cache negative results (short TTL).
3. Use async refresh + stale-while-revalidate для тяжёлых вычислений.
4. Prefer cache-aside для гибкости; read-through when you want centralised cache loading.
5. Use outbox for invalidation events to keep DB and cache in-sync.
6. Monitor hit ratio and evictions, react with capacity changes or policy tuning.
7. Estimate working set before provisioning; чаще всего undersized cache хуже, чем отсутствие cache.
8. Avoid caching sensitive data unencrypted; apply access controls.

---

## Anti-patterns

- Кеширование всего подряд без измерений.
- Кеширование горячих объектов на одной ноде без репликации.
- Использование одинаковых TTL для всех типов данных.
- Отсутствие negative cache → перегрузка DB.
- Ручное массовое сбрасывание кэша (cache flush) в продакшне.