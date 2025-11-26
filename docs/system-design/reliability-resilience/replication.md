# Replication

Поддержание нескольких копий одних и тех же данных в разных местах (узлах, регионах, ЦОДах) для повышения отказоустойчивости, доступности и скорости доступа.

---

## Цели репликации

- **High Availability (HA)**: при падении одной копии данные доступны с другой.
- **Scalability**: чтения можно направлять на реплики.
- **Durability**: снижает риск потери данных при аппаратных сбоях.
- **Local Latency**: приближение данных к пользователю (edge replication).

---

## Модели репликации

### Синхронная репликация

- Primary ждёт подтверждения от всех реплик перед подтверждением клиенту.
- _Плюсы_: Strong consistency (no data loss).
- _Минусы_: Высокая задержка, низкая доступность при lagging replicas.
- _Примеры_: PostgreSQL synchronous commit, Galera Cluster, Spanner (TrueTime).

### Асинхронная репликация

- Primary отвечает клиенту сразу, а реплики получают данные позже.
- _Плюсы_: Высокая производительность и низкая латентность.
- _Минусы_: Возможна потеря данных при сбое primary.
- _Примеры_: MySQL asynchronous binlog replication, MongoDB secondaries.

### Полусинхронная (Semi-synchronous)

- Primary ждёт подтверждения хотя бы от одной реплики.
- _Компромисс_: повышенная надёжность без большой задержки.
- _Пример_: MySQL semi-sync replication.

---

## Направления репликации

### Master–Slave (Primary–Replica)

- Только один узел принимает записи.
- Реплики только для чтения.
- _Проблемы_: failover требует переизбрания лидера, eventual consistency.

### Multi-Master (Master–Master)

- Запись возможна на нескольких узлах.
- Требует конфликт-резолюции (например, timestamp-based, CRDT).
- _Примеры_: Cassandra, DynamoDB, Active–Active Redis Enterprise.

### Peer-to-Peer

- Каждый узел равноправен.
- Используется в DHT и P2P-сетях (например, Cassandra, Riak).
- Репликация и консенсус выполняются через gossip-протоколы.

---

## Репликация в распределённых системах

| Тип                | Примеры систем                  | Характеристики                               |
| ------------------ | ------------------------------- | -------------------------------------------- |
| Log-based          | PostgreSQL WAL, MySQL binlog    | Репликация через журнал транзакций           |
| State transfer     | MongoDB initial sync, Redis RDB | Полное копирование состояния                 |
| Incremental        | Kafka ISR (in-sync replicas)    | Репликация сообщений по offset               |
| Quorum replication | Cassandra, DynamoDB             | Достижение согласованности через `R + W > N` |

---

## Геораспределённая репликация (Geo-replication)

- Репликация между регионами или датацентрами.
- _Цель_: disaster recovery, latency locality.
- _Проблемы_: сетевые задержки → eventual consistency.

_Решения_:
- Google Spanner — TrueTime (bounded staleness).
- CockroachDB — MVCC + hybrid logical clocks.
- DynamoDB Global Tables — conflict-free replication.

---

## Проблемы и решения

| Проблема              | Описание                             | Решения                                   |
| --------------------- | ------------------------------------ | ----------------------------------------- |
| **Replication Lag**   | Реплики отстают по времени           | Async monitoring, lag metrics, throttling |
| **Inconsistency**     | Разные узлы возвращают разные версии | Conflict resolution, quorum reads/writes  |
| **Split-brain**       | Несколько лидеров                    | Consensus protocols                       |
| **Data loss**         | Отказ primary до синхронизации       | Semi-sync replication, synchronous commit |
| **Network partition** | Потеря связи между регионами         | CAP trade-offs, eventual consistency      |

---

## Итоговое сравнение моделей

| Критерий     | Синхронная      | Асинхронная          | Полусинхронная  |
| ------------ | --------------- | -------------------- | --------------- |
| Consistency  | Strong          | Eventual             | Compromise      |
| Latency      | Высокая         | Низкая               | Средняя         |
| Data Loss    | Нет             | Возможен             | Минимальный     |
| Availability | Ниже            | Выше                 | Средняя         |
| Примеры      | Spanner, Galera | MySQL async, MongoDB | MySQL semi-sync |

---

## Replication Topologies and Architectures

### 1. Single-Leader Replication (Primary–Replica)

#### Схема

    Clients → [Primary] → Replicas
                        ↖ pull log / binlog

#### Характеристики

- Все записи проходят через одного лидера.
- Реплики получают изменения через журнал репликации (WAL/binlog) — push или pull.
- Чтения можно масштабировать, направляя трафик на реплики.

#### Плюсы

- Простота и предсказуемость порядка операций.
- Strong consistency при синхронной репликации.
- Простая модель failover: promote replica → new leader.

#### Минусы

- Узкое место — один лидер.
- Failover требует лидер-элекции (downtime 1–10 секунд).
- Возможны data loss при асинхронной репликации.

#### Примеры

_PostgreSQL, MySQL, MongoDB (Replica Set), Redis (Sentinel), Kafka controller._

### 2. Multi-Leader Replication (Multi-Primary)

#### Схема

    Clients → [Leader A] ←→ [Leader B] ←→ [Leader C]

#### Характеристики

- Несколько узлов принимают записи.
- Изменения реплицируются между всеми лидерами.
- Каждый узел обслуживает локальные запросы.

#### Проблема

_Конфликтные обновления_: если два лидера изменяют один ключ одновременно.

#### Решения конфликтов

- **Last-write-wins (LWW)** — по timestamp’у.
- **CRDT (Conflict-free Replicated Data Types)** — коммутативные структуры данных (set, counter).
- **Custom merge policies** — бизнес-логика при слиянии.
- **Vector clocks / causal order** — отслеживание причинно-следственной зависимости.

#### Плюсы

- Высокая доступность при локальных сбоях.
- Подходит для geo-distributed систем (региональные лидеры).
- Быстрые локальные записи без межрегиональных RTT.

#### Минусы

- Сложная консистентность (eventual, not strong).
- Репликация конфликтов.
- Трудно обеспечить global ordering.

#### Примеры

_CouchDB, Cassandra, DynamoDB, Active–Active Redis, Git._

### 3. Leaderless Replication

#### Схема

    Clients → write to N replicas directly

#### Принцип

Нет выделенного лидера.
Клиент записывает данные сразу на несколько узлов и читает с нескольких узлов.
Согласованность обеспечивается через кворум:
- Параметры: N — общее число реплик, W — сколько нужно подтвердить запись, R — сколько читаем.
- Для strong consistency: R + W > N.

#### Пример

    N=3, W=2, R=2  →  Strong Consistency

#### Плюсы

- Нет точки отказа (SPOF).
- Высокая доступность при сбоях отдельных узлов.
- Хорошая горизонтальная масштабируемость.

#### Минусы

- Сложная логика клиента (quorum read/write, version reconciliation).
- Конфликты и дубликаты.
- Более высокая латентность при quorum write.

#### Примеры

_Amazon Dynamo, Cassandra, Riak, Voldemort._

### 4. Chain Replication

#### Схема

    Head → Node 2 → Node 3 → Tail

#### Принцип

- Все записи идут в цепочку от "head" к "tail".
- "Head" принимает запись, "tail" подтверждает успешное применение.
- Чтения направляются на "tail" (всегда самая свежая версия).

#### Плюсы

- Упорядоченные записи, strong consistency.
- Высокая пропускная способность для последовательных операций.
- Failover — просто перестроить цепочку.

#### Минусы

- Неравномерная нагрузка (head перегружен на запись).
- Увеличение задержки при длинной цепочке.
- Сложность динамического добавления узлов.

#### Примеры

_Microsoft FaRM, Ceph RADOS, ZooKeeper ZAB protocol._

### 5. Cascading Replication

#### Схема

    Primary
    ↓
    Replica A → Replica B → Replica C

#### Принцип

- Реплики получают данные не только от primary, но и от других реплик.
- Снижает нагрузку на primary при большом количестве узлов.

**Применение**

- Подходит для глобальных систем (multi-region read replicas).
- Например, реплика в Европе получает обновления из американской реплики, а не из primary.

#### Минусы

- Увеличение задержки (propagation delay).
- Труднее отслеживать lag и задержки.

#### Примеры

_PostgreSQL cascading replication, MySQL relay slaves._

### 6. Quorum-Based Replication (Consensus-driven)

#### Основная идея

- Все узлы равноправны.
- Изменения коммитятся после подтверждения большинства (> N/2).
- Используется в системах с консенсус-протоколами (Paxos, Raft, Zab).

#### Плюсы

- Strong consistency без выделенного лидера.
- Простое обнаружение отказов через majority quorum.
- Реплики всегда согласованы.

#### Минусы

- Медленно при сетевых задержках.
- Труднее масштабировать при большом N.
- Зависит от стабильного кворума.

#### Примеры

_Raft (etcd, Consul, CockroachDB), Paxos (Google Chubby, Spanner), Zab (ZooKeeper)._

---

## Geo-Distributed Replication Topologies

### Regional Leader (Federated)

- Каждый регион имеет своего лидера.
- Межрегиональная репликация асинхронная.
- Запись идёт в локальный регион, глобальные данные догоняются позже.

### Full Mesh

- Все регионы реплицируют друг с другом.
- Конфликтное разрешение через CRDT.
- Высокая устойчивость, но дорого по трафику.

### Hub-and-Spoke

- Один центральный регион (“hub”), остальные регионы (“spokes”) синхронизируются с ним.
- Упрощает архитектуру, но hub становится SPOF.

---

## Инструменты и практические реализации

| Технология    | Топология                       | Failover механизм               |
| ------------- | ------------------------------- | ------------------------------- |
| PostgreSQL    | Primary–Replica                 | Patroni, etcd, Stolon           |
| MySQL         | Primary–Replica / Multi-primary | Orchestrator, Group Replication |
| Cassandra     | Leaderless (Quorum)             | Gossip + hinted handoff         |
| MongoDB       | Replica Set                     | Automatic election              |
| Redis         | Primary–Replica                 | Sentinel / Cluster              |
| Kafka         | Partition replicas              | Controller election             |
| Etcd / Consul | Raft-based quorum               | Internal election               |

---

## Consistency Models in Replication

### Strong Consistency

После того как операция записи подтверждена, все клиенты читают одно и то же значение — независимо от того, с какого узла они читают.

#### Требует:

- Синхронной репликации или кворумного консенсуса.
- Единого порядка коммитов (global total order).

#### Реализация:

- Single-leader synchronous replication — репликация блокирует запись, пока все реплики не подтвердят применение.
- Consensus protocols (Raft, Paxos, Zab) — кворум подтверждает commit логически.

#### Плюсы:

- Простой ментальный контракт: данные всегда актуальны.
- Упрощает бизнес-логику (например, банковские транзакции).

#### Минусы:

- Высокая латентность (в особенности при межрегиональных записях).
- Возможны write unavailability при падении реплики или потере кворума.

#### Примеры:

- Google Spanner (через TrueTime и Paxos).
- CockroachDB (через Raft).
- Etcd / Consul (через Raft).

### Eventual Consistency

Все узлы в конечном итоге становятся согласованными, если не происходят новые обновления.

В момент времени узлы могут видеть разные данные.

#### Механизм:

- Асинхронная репликация: запись на primary, позже — на replica.
- Репликация по принципу “gossip” — узлы периодически обмениваются состоянием.
- Разрешение конфликтов при слиянии версий (LWW, CRDT, vector clocks).

#### Плюсы:

- Высокая доступность (availability).
- Минимальная задержка на запись.
- Отлично подходит для систем, где немедленная согласованность не критична (social feed, analytics, caching).

#### Минусы:

- Различные клиенты могут видеть разные состояния (stale reads).
- Сложность в разрешении конфликтов.
- Нет гарантии порядка применения операций.

#### Примеры:

- DynamoDB, Cassandra, Riak, CouchDB.
- Redis Cluster (в режиме async replication).

### Causal Consistency

Если операция B причинно зависит от A, то все узлы увидят A перед B.

#### Механизм:

- Каждая операция сопровождается метаданными причинно-следственной зависимости.
- Используются vector clocks, Lamport timestamps или session tokens.
- Узлы могут применять операции в разном порядке, но при этом сохраняется причинная зависимость.

#### Плюсы:

- Устраняет парадоксы “комментарий раньше поста”, “баланс стал отрицательным до покупки”.
- Сильнее eventual, но дешевле strong consistency.

#### Минусы:

- Хранение и пересылка метаданных (векторные часы растут O(N)).
- Нет глобального порядка, только локальные зависимости.
- Иногда требуется задержка применения для поддержания причинности.

#### Примеры:

- COPS (Causal+ system).
- Cosmos DB (предоставляет уровень "Causal consistency").
- Session consistency в MongoDB частично соответствует causal.

### Read-Your-Writes Consistency (RYW)

Клиент, сделавший запись, **всегда видит результат этой записи** при последующих чтениях — даже если другие клиенты пока не видят.

#### Механизм:

- Client session pinning (на одного leader-а).
- Sticky sessions или session tokens.
- Локальные write buffer’ы с последующей синхронизацией.

#### Плюсы:

- Удобно для UX: пользователь видит свои обновления сразу.
- Простая реализация через sticky sessions или session replication.

#### Минусы:

- Не гарантирует консистентность между клиентами.
- Требует фиксации клиента на одном регионе/реплике (иначе stale reads).

#### Примеры:

- MongoDB (session guarantees).
- Cosmos DB (session consistency).
- DynamoDB “read-your-write” через consistent read flag.

### Monotonic Read Consistency

Клиент никогда не увидит более старую версию данных после того, как увидел новую.

#### Механизм:

- Session pinning: клиент всегда читает с той же реплики, пока не сменит сессию.
- Версионный контроль (timestamps, sequence IDs).

#### Плюсы:

- Гарантирует "поступательное" восприятие данных.
- Исключает аномалии вида “update исчезло”.

#### Минусы:

- Требует стабильного session stickiness.
- Не защищает от временного чтения устаревших данных другими клиентами.

#### Примеры:

- Cassandra read repair + session-level tokens.
- Cosmos DB: monotonic read в рамках уровня “session consistency”.

### Monotonic Write Consistency

Записи от одного клиента применяются в том порядке, в котором они были отправлены.

#### Механизм:

- Write serialization per client ID.
- Локальный буфер или токен с sequence ID.
- Replication log ordering (WAL).

#### Плюсы:

- Устраняет сценарии «сначала удалил, потом создал обратно, но порядок нарушился».
- Гарантия корректного применения изменений в потоках событий.

#### Минусы:

- Требует контроля порядка и idempotency.
- При сетевых задержках возможны блокировки (backpressure).

#### Примеры:

- Kafka log semantics (per partition ordering).
- Raft/Paxos — по определению обеспечивают monotonic writes.
- AWS S3 — обеспечивает per-key write ordering (внутренне через replication log).

### Timeline (or Session) Consistency

Клиент видит данные, согласованные в рамках своей временной сессии, но не обязательно согласованные глобально.

#### Используется в:

- Web-приложениях, где каждый пользователь видит свои обновления, но не других сразу.
- Мобильных клиентах, где возможен offline режим и локальная репликация.

#### Примеры:

- MongoDB session-level read preferences.
- Firebase Realtime Database (offline sync, merge later).
- Cosmos DB “session consistency”.

### Bounded Staleness Consistency

Чтения могут быть устаревшими, но только в пределах гарантированного окна времени (Δt) или числа версий (k).

#### Механизм:

- Асинхронная репликация с контролем lag.
- Комбинация heartbeat + LSN (log sequence number).
- TTL для репликационного лага.

#### Плюсы:

- Компромисс между strong и eventual consistency.
- Предсказуемость stale window.

#### Минусы:

- Более сложная координация между регионами.
- Возможен рост задержки при глобальной нагрузке.

#### Примеры:

- Google Spanner “bounded staleness” через TrueTime.
- Cosmos DB поддерживает bounded staleness как отдельный consistency level.

---

### Trade-offs Between Consistency Models

| Модель            | Консистентность            | Доступность | Латентность | Типичные системы     |
| ----------------- | -------------------------- | ----------- | ----------- | -------------------- |
| Strong            | Сильная                    | Низкая      | Высокая     | Spanner, CockroachDB |
| Bounded Staleness | Контролируемая устарелость | Средняя     | Средняя     | Cosmos DB            |
| Causal            | Причинная                  | Средняя     | Средняя     | COPS, Cosmos DB      |
| Read-your-writes  | Локальная для клиента      | Средняя     | Средняя     | MongoDB, DynamoDB    |
| Eventual          | Слабая                     | Высокая     | Низкая      | Cassandra, DynamoDB  |

---

### Consistency in Practice

| Система         | Модель                                                | Механизм                          |
| --------------- | ----------------------------------------------------- | --------------------------------- |
| **PostgreSQL**  | Strong                                                | Синхронная репликация (WAL flush) |
| **MySQL**       | Eventual / Semi-sync                                  | Binlog replication                |
| **MongoDB**     | Tunable (Strong–Eventual)                             | Write concern / Read concern      |
| **Cassandra**   | Tunable consistency                                   | Quorum (R+W > N)                  |
| **DynamoDB**    | Tunable (Strong / Eventual)                           | Consistent read flag              |
| **Spanner**     | Global Strong                                         | TrueTime + Paxos                  |
| **CockroachDB** | Strong                                                | Raft per range                    |
| **Cosmos DB**   | 5 levels (Strong, Bounded, Causal, Session, Eventual) | Configurable API-level            |

---

## ISR (in-sync replicas)

ISR = набор реплик, которые находятся “в синхроне” с лидером.

Для каждого partition есть:
- Leader replica
- Followers
- ISR subset — те из followers, которые догнали лидера по log end offset.

_Пример:_

    Partition P1:
        Leader = Broker 1
        Followers = Broker 2,3
        ISR = {1,2,3}

Если Broker 3 начинает отставать → он выпадает из ISR: `ISR = {1,2}`

### Зачем ISR?

ISR обеспечивает strong consistency модели Kafka.

Запись считается committed только:
- если replica = leader
- и запись replicated в majority из ISR (по acks=all)

_То есть:_ ISR определяет, кто может участвовать в commit quorum.

### Как работает ISR под капотом

#### Шаги

1. Лидер получает сообщение → пишет в свой лог.
2. Лидер делает append в followers через FETCH RPC.
3. Когда каждый follower догнал leader offset → он считается in-sync.
4. Лидер ведёт ISR-list локально и в ZooKeeper/KRaft.
5. Commit происходит только если:
   * acks=all
   * запись replicated на все реплики в ISR

#### Ключевые параметры

* replica.lag.time.max.ms — максимум, сколько реплика может не отвечать
* min.insync.replicas — минимальное количество ISR для commit
* unclean.leader.election.enable — разрешать ли лидеров вне ISR (обычно NO)

### Что происходит при сбоях

#### Сбой follower-а

* follower отстаёт → удаляется из ISR
* но cluster продолжает работать

#### Сбой лидера

Kafka выбирает нового лидера среди реплик в ISR.

Если ISR пустой → leader election невозможен (если unclean=false).