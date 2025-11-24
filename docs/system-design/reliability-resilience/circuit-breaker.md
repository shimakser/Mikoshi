# Circuit Breaker

Шаблон устойчивости (resilience pattern), который предотвращает попытки обращения к внешним зависимостям (например, микросервисам, базам данных, API), если они временно недоступны или работают нестабильно.

---

## Определения

**Downstream service** — «тот, к кому ты ходишь». Downstream — это компонент, которому ты отправляешь запрос.

**Upstream service** — «тот, кто ходит к тебе». Upstream — это компонент, который вызывает тебя, т.е. твой клиент.

---

## Проблемы, которые решает
1. **Cascading Failures** (каскадные отказы)<br>
   Если один сервис зависает или падает, все клиенты продолжают слать к нему запросы → растет очередь → перегружается не только он, но и они.
2. **Resource Exhaustion**<br>
   Повторные попытки обращения к медленно работающему сервису могут занять все потоки, ухудшив производительность.
3. **Latency Increase**<br>
   Таймауты на каждой попытке создают цепочку задержек и сбоев по всей архитектуре.

---

## Принцип работы
1. Closed (замкнут)
    - Сервис работает стабильно, все запросы проходят.
      Если начинается рост ошибок, увеличивается счётчик отказов.
    - Переход в Open, если число/доля ошибок превышает порог.
2. Open (разомкнут)
    - Запросы немедленно отклоняются — нет попыток обратиться к зависимому сервису.
    - Обычно здесь используется fallback (например, кэш, заглушка, сообщение «сервис недоступен»).
    - Через заданное время (reset timeout) — переход в Half-Open.
3. Half-Open (полуоткрыт)
    - Тестовая фаза: ограниченное количество запросов пропускается.
    - Если они успешны — переход обратно в Closed.
    - Если ошибки продолжаются — снова в Open.

---

## Fallback (Degradation Strategy)

Fallback — необязательная часть CB, но очень распространённая.

Варианты:
* возврат кэша,
* возврат stale данных (кэш с истекшим TTL),
* возврат заранее подготовленной заглушки,
* graceful degradation: «данные временно недоступны».

_Важно:_ fallback — часть общей resilience-архитектуры, не Circuit Breaker сам по себе.

---

## Circuit Breaker vs Retry vs Rate Limiter
- Circuit Breaker — Прерывает поток запросов к сломанной зависимости.
- Retry — Повторяет запрос после ошибки.
- Rate Limiter — Контролирует скорость запросов (предотвращает DoS).

---

## Реализация Circuit Breaker в практике

### Netflix Hystrix

    HystrixCommand<String> command =
        new HystrixCommand<String>(HystrixCommandGroupKey.Factory.asKey("Group")) {
            @Override
            protected String run() throws Exception {
                return callExternalService();
            }
            
            @Override
            protected String getFallback() {
                return "fallback";
            }
    };
    
    String result = command.execute();

Hystrix автоматически:
* считает метрики,
* открывает CB при errorRate > threshold,
* делает fallback,
* пробует восстановиться.


### Resilience4j

Resilience4j удобнее Hystrix:
* основан на функциональном стиле,
* лёгкий, без больших зависимостей,
  * отдельно модули: CB, retry, rate limiter, bulkhead, time limiter.


      CircuitBreakerConfig config = CircuitBreakerConfig.custom()
          .failureRateThreshold(50)
          .slowCallRateThreshold(50)
          .slowCallDurationThreshold(Duration.ofMillis(300))
          .waitDurationInOpenState(Duration.ofSeconds(10))
          .permittedNumberOfCallsInHalfOpenState(5)
          .minimumNumberOfCalls(50)
          .slidingWindowType(SlidingWindowType.TIME_BASED)
          .slidingWindowSize(10)
          .build();
    
      CircuitBreaker circuitBreaker = CircuitBreaker.of("myService", config);
    
      Supplier<String> decorated = CircuitBreaker.decorateSupplier(
          circuitBreaker,
          () -> callExternalService()
      );
    
      Try<String> result = Try.ofSupplier(decorated)
          .recover(ex -> "fallback");

Особенности Resilience4j:
* есть события (listeners) для метрик,
* хорошо интегрируется в Micrometer + Prometheus,
* гибкие политики Half-Open,
* интеграция с Spring Boot.

---

## Best practices

1. Ставить timeout перед Circuit Breaker
CB не может сработать, если запрос “висит” бесконечно.<br>
`timeout → retry (ограниченный) → CB → fallback`
2. Не ставить retry перед CB без ограничения.
3. Считать slow calls, а не только ошибки. Медленный сервис так же опасен, как упавший.
