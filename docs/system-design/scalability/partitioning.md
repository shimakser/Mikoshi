# Partitioning

Это процесс деления большой логической системы (или набора данных) на независимые, более мелкие и управляемые части, называемые partitions.

Каждая партиция обслуживается независимо (на уровне данных, сервиса или потока), что снижает нагрузку на отдельные компоненты и улучшает масштабируемость и отказоустойчивость.

## Зачем нужно
- Локализовать нагрузку (isolation по данным, пользователям или типам операций).
- Уменьшить contention на ресурсы.
- Разделить зоны отказа (fault isolation).
- Снизить latency за счёт data locality (например, хранить пользователей EU в EU-region).
- Улучшить параллелизм — разные узлы обслуживают разные партиции.

## Виды Partitioning

### Functional (Vertical) Partitioning

Разделение системы по функциональности — разные сервисы/модули отвечают за разные домены данных.

Разделение системы по функциональности — разные сервисы/модули отвечают за разные домены данных.

_Пример:_
- User Service, Payment Service, Notification Service.
- Разделение таблиц: users, transactions, emails — каждая в своей БД.

_Плюсы:_
- Независимое масштабирование каждого домена.
- Простота командной ответственности (bounded context).

_Минусы:_
- Кросс-доменные запросы → интеграция через API (distributed joins).
- Требует чёткой границы данных (DDD).

### Horizontal Partitioning

Разделение по строкам в пределах одной сущности (table split).
Каждая партиция содержит подмножество записей, но одинаковую схему.

_Пример:_
Таблица orders разбита по region_id или customer_id.
- Partition A → Europe
- Partition B → Asia

_Применяется:_
- в базах данных (PostgreSQL table partitioning, MySQL partitioning),
- в брокерах сообщений (Kafka topic partitions),
- в хранилищах документов (Elasticsearch shards).

### Vertical Data Partitioning

Разделение по столбцам (columns).

_Пример:_
Таблица user разделена на:
- user_core(id, name, email) — часто читаемые данные.
- user_profile(user_id, bio, photo) — редко читаемые.
  _Плюсы:_ уменьшение размера часто используемых данных, ускорение запросов.
  _Минусы:_ необходимость join-ов при выборке полного объекта.

## Где используется Partitioning

| Уровень       | Пример                 | Цель                          |
| ------------- | ---------------------- | ----------------------------- |
| Application   | Микросервисы           | Изоляция бизнес-доменов       |
| Database      | Table partitioning     | Снижение contention и latency |
| Storage       | S3 buckets по регионам | Data locality                 |
| Message queue | Kafka partitions       | Параллелизм обработки         |
| Indexing      | Elasticsearch shards   | Балансировка поиска           |

## Проблемы и trade-offs

| Проблема                | Описание                                           | Возможное решение                         |
| ----------------------- | -------------------------------------------------- | ----------------------------------------- |
| **Uneven load**         | Партиции могут быть неравномерны по объёму или QPS | Adaptive partitioning, rebalancing        |
| **Hot partitions**      | Концентрация трафика на одном диапазоне            | Hash-based keys, dynamic splitting        |
| **Cross-partition ops** | join, group-by, aggregate → сложнее                | Query router, map-reduce, denormalization |
| **Metadata management** | нужно знать, где лежат данные                      | Directory service, consistent hashing     |

## Связь с Sharding и Replication

- **Partitioning** — общее понятие логического деления данных.
- **Sharding** — частный случай horizontal partitioning в распределённой среде.
- **Replication** — копирование одной партиции на несколько узлов для отказоустойчивости.
