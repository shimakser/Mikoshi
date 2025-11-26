# Failover

Это процесс автоматического или управляемого переключения нагрузки с отказавшего узла (primary/master) на резервный (standby/replica), чтобы сохранить доступность и целостность системы.

Цель — минимизация RTO (Recovery Time Objective) и RPO (Recovery Point Objective) при сбоях.

---

## Ключевые концепции

### Active–Passive

- Active node обрабатывает трафик, Passive node простаивает, но постоянно синхронизируется.
- При сбое активного узла происходит переключение (manual или automatic).
- _Плюсы_: простота, предсказуемость состояния.
- _Минусы_: неиспользуемые ресурсы, длительный failover при ручной активации.

### Active–Active

- Все узлы активны и обслуживают запросы.
- Каждый узел может стать "лидером" для части данных (partitioning/sharding).
- _Плюсы_: высокая утилизация, быстрая реакция на сбой.
- _Минусы_: конфликтующие обновления, необходимость strong conflict resolution (например, CRDT, last-write-wins, vector clocks).

### Standby Types

- **Hot standby** — реплика получает изменения в реальном времени, готова к мгновенному переключению.
- **Warm standby** — реплика синхронизируется периодически, запуск требует времени.
- **Cold standby** — просто резервная машина, восстановление вручную.

---

## Failover Detection & Coordination

Ключевой вызов — **надёжно детектировать сбой и принять решение о переключении** без split-brain.

- **Heartbeat monitoring**: периодические пинги между узлами.
Инструменты: Keepalived, Pacemaker, ZooKeeper, Consul.
- **Quorum-based failover**: решение принимается большинством узлов (majority vote).
- **Fencing (STONITH)**: "Shoot The Other Node In The Head" — исключение старого лидера, чтобы не было двойного лидера.
- **Consensus protocols**:
  - Raft / Paxos**** — выбор лидера и согласование состояния кластера.
  - _Пример_: в etcd или Zookeeper лидер выбирается через Raft, остальные фолловеры догоняют логи.

---

## Failover Scenarios

| Сценарий                 | Пример                        | Сложность                              |
| ------------------------ | ----------------------------- | -------------------------------------- |
| DB failover              | Primary → Replica             | Потеря транзакций при async репликации |
| API failover             | LB перенаправляет трафик      | Session stickiness, кэш прогрев        |
| Distributed job failover | Scheduler перевешивает задачи | Idempotency, job deduplication         |
| Region-level failover    | AWS us-east-1 → eu-west-1     | Cross-region latency, DNS TTL          |

---

## Split-Brain Problem

Происходит, когда оба узла считают себя лидерами.
_Причины_: сетевые задержки, partition, неправильная конфигурация quorum.

_Решения_:
- Quorum majority vote.
- External consensus store (etcd, Consul, Zookeeper).
- Witness node / arbiter для чётного числа реплик.
- STONITH для гарантированного исключения старого лидера.

---

## Failover Architectures

### Manual Failover

- Администратор вручную переключает реплику.
- Минимальные автоматизации.
- _Минус_: длительный downtime.

### Semi-Automatic Failover

- Мониторинг (Prometheus, Orchestrator, Sentinel) фиксирует сбой.
- Реплика промотируется автоматически, но требует ручного вмешательства для DNS/Proxy.

### Full Automatic Failover

- Consensus cluster (Zookeeper/etcd) выполняет выбор лидера.
- DNS или load balancer автоматически перенаправляют трафик.
- _Риск_: split-brain при сетевых проблемах.

---

## Failover Decision & Election Algorithms

| Алгоритм                             | Принцип                                             | Применение                    |
| ------------------------------------ | --------------------------------------------------- | ----------------------------- |
| **Raft**                             | Лидер, лог репликации, heartbeat, election timeout  | etcd, Consul, CockroachDB     |
| **Paxos**                            | Majority agreement через prepare/accept             | Spanner, Chubby               |
| **ZAB (ZooKeeper Atomic Broadcast)** | Лидер упорядочивает операции, follower подтверждают | ZooKeeper                     |
| **Bully Algorithm**                  | Самый “сильный” (ID) узел становится лидером        | Старые распределённые системы |
| **Gossip-based Election**            | Узлы обмениваются статусом, формируют консенсус     | Cassandra, Dynamo             |

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
