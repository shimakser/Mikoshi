# Graceful Degradation

Это проектирование системы так, чтобы при ухудшении состояния компонентов она поступательно снижала функциональность или качество сервиса (degrade), но сохраняла доступность и критичные бизнес-процессы.

Когда часть системы выходит из строя или перегружена, система должна продолжать работать в ограниченном режиме, предоставляя базовую функциональность, вместо полного отказа.

---

## Основные принципы и цели

1. **Availability over perfection.** Лучше отдать частичный (или устаревший) ответ, чем упасть полностью.
2. **Predictability / Determinism.** Поведение при деградации должно быть предсказуемым и документированным.
3. **Isolation (Bulkheads).** Деградация одной функции не должна вытеснять ресурсы других.
4. **Fail-fast & inform upstream.** Быстро возвращать понятную ошибку/фолбэк и сигнализировать upstream (429, 503, Retry-After).
5. **Prioritization.** Критичные пути (payments, auth) имеют высший приоритет; неважные функции (recommendations) отключаются первыми.
6. **Observability-first.** Метрики и трассировки нужны, чтобы понять, что и почему деградирует.
7. **Graceful rollback.** Деградация — не «пожарное» выключение, а контролируемая операция с возможностью возврата.

---

## Классификация деградации (виды)

1. **Functional degradation** — отключение нефунциональных возможностей (UI widgets, recommendations, background sync).
2. **Performance degradation** — снижение качества (снижение частоты обновлений, снижение точности, отправка частичных ответов).
3. **Data-quality degradation** — возвращение stale/approximate данных (cached, aggregated).
4. **Geographic degradation** — ограничить функциональность в регионах при сетевых проблемах.
5. **API-level degradation** — возвращать partial responses, stub data, “read-only” mode.
6. **Mode-based degradation** — switch to read-only, batch-mode, degraded throughput.

---

## Паттерны и механики (конкретно применимые)

### Fallback patterns

* **Cache fallback** (stale-while-revalidate) — возвращать последний известный результат, параллельно инициируя обновление.
* **Static fallback** — заранее подготовленные текстовые/JSON-шаблоны.
* **Approximate/aggregate fallback** — вместо детального списка — количество или агрегат.
* **Fallback service** — лёгкий локальный сервис/replica, отвечающий в degraded mode.

### Feature toggles / kill switches

* Управляемые флаги для отключения фич (per-feature, per-tenant, per-region).
* Integrate with CI/CD and runtime config store (etcd/Consul/feature-flag service).

### Prioritized queues & admission control

* **Priority queues** — важные задачи обрабатываются первыми.
* **Admission control** — проверка пропускной способности перед постановкой в очередь; отклонение низкоприоритетных работ при их заполнении.

### Rate limiting + backpressure

Накладывать жесткие лимиты на менее-критичные функции. Комбинировать с backpressure upstream.

### Read-only mode / maintenance mode

Для write-heavy failures — отключение writes, перевод в read-only с объяснением пользователю и retry guidance.

### Circuit Breaker + Bulkhead + Timeouts

CB + bulkhead предотвращают насыщение; в degraded mode CB переводится в open быстрее и fallback включается.

### Degraded UI / progressive enhancement

* UI показывает «умеренно ухудшенное» состояние: skeletons, stripped features, offline mode.
* Progressive loading: essential content first, non-essential later or on demand.

---

## Проектирование: пошаговый рабочий план

1. **Map critical paths** — составить dependency graph: какие компоненты критичны для каких действий пользователя.
2. **Define SLOs per path** — какие p95/p99 и RTO/RPO допустимы на критичных путях.
3. **Classify functions by criticality** — (P0 payments/auth), (P1 core UX), (P2 nice-to-have).
4. **Design fallback behavior for each function** — зафиксируйте, что именно происходит при деградации.
5. **Choose triggers** — какие метрики/события переводят систему в degraded mode (high latency p95/p99, queue depth, error rate, db connections used, region outage).
6. **Implement controls** — feature flags, admission control, rate limiters, CBs, bulkheads.
7. **Instrument heavily** — add metrics, traces, events, centralized logging, alerts & dashboards for degraded states.
8. **Automate** — automate detection → transition → notification → rollback.
9. **Test** — load tests, chaos testing, degrade-mode simulations, end-to-end tests with staging mirrors.
10. **Operationalize** — runbooks, on-call playbooks, customer communication templates.

---

## Как реализуется
- отключаем неважные фичи через конфигурацию без деплоя.
- Fallback значения: вместо данных от внешнего сервиса — временные дефолты или кэш.
- Timeouts и Circuit breakers: не блокировать основной поток из-за зависания зависимых сервисов.
- Priority queues: низкоприоритетные задачи могут быть отложены или отброшены.

---

## Технические реализации — примеры и детали

### API Gateway + деградация

- На уровне edge (границы системы): балансировщик / Ingress может возвращать 503 + Retry-After, если backend перегружен.
- Feature toggle на уровне gateway: шлюз может отключать или перенаправлять вызовы не-критичных эндпоинтов на fallback-сервис.

_Пример логики:_

    Клиент -> API Gateway
    Gateway проверяет:
    - Включена ли фича?
      - Здоров ли downstream сервис?
      - Если нездоров и фича некритична → вернуть кэш / stub / fallback
      - Если критична → быстро вернуть 503 + Retry-After

### Деградация на уровне данных: read-only + stale

- Хранить последний «хороший» снапшот (materialized view) по каждой партиции/таблице.
- Если система не может выполнять записи (например, проблемы с primary DB):
  - переключиться в режим read-only,
  - временно складывать write-операции в DLQ (Dead Letter Queue) или «write-intent log» для последующего восстановления.

### Перенос фоновой работы (background job offload)

Если система нагружена:
- Перевести тяжелые/некритичные операции из синхронных вызовов в асинхронные.
- Запрос → положить в очередь → вернуть клиенту 202 Accepted.
- Обработать позже, когда нагрузка снизится.