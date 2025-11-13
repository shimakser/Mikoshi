# Distributed Coordination

_Это системы координации узлов, обеспечивающие:_
- Хранение конфигурации.
- Leader election.
- Согласование между distributed компонентами.

---

**Zookeeper**
- Используется в Kafka, Hadoop, HBase.

_Поддерживает:_
- Эфемерные узлы (автоудаление при сбое).
- Watch-механизм для подписки на изменения.
- ZNodes: иерархическая структура, подобная файловой.

---

**etcd**
Основа для Kubernetes (вся конфигурация и состояние API-сервера хранится в etcd).

_Поддерживает:_
- gRPC API.
- TTL, lease, transactional операции.
- Основан на Raft consensus algorithm.

---

**Consul**

_Предоставляет:_
- Service discovery.
- Health checking.
- Key-value store.

Часто используется с Envoy, HashiCorp Nomad.