# Bulkhead

Это паттерн устойчивости, который изолирует ресурсы внутри сервиса так, чтобы сбой одной части системы не утопил весь сервис.
Название взято из кораблестроения: водонепроницаемые переборки — если одна секция пробита, корабль не тонет.

В распределённых системах булкхед — это жёсткая изоляция ресурса, чаще всего:
* потоков,
* очередей,
* пулов подключений,
* процессорного времени,
* лимита одновременных запросов.

Bulkhead = «каждый downstream использует свой выделенный резерв ресурсов».

---

## Какую проблему решает

_Проблема:_ если одна зависимость начинает «тормозить» или виснуть, все её вызовы занимают общие ресурсы (например, общий thread pool), и весь сервис перестаёт обрабатывать другие запросы.

### Пример без Bulkhead
Service A:
- вызывает Payment API,
- вызывает Profile API.

Payment API завис → все потоки заняты ожиданием → сервис A перестаёт отвечать → падает весь функционал.

### Пример с Bulkhead
Service A:
- Payment pool — 10 потоков.
- Profile pool — 10 потоков.

Payment завис, заняв свои 10 потоков, но 10 потоков для Profile остаются свободны.
Сервис деградирует частично, но не падает полностью.

---

## Основные типы Bulkhead

### Thread-based bulkhead

_Самая распространённая форма:_
- отдельные thread pools для разных типов операций,
- отдельные executor'ы для разных downstream.

_В Java:_
- Executors.newFixedThreadPool(),
- ScheduledThreadPoolExecutor,
- ForkJoinPool (сложнее – нужен fine-grained контроль).

_В Web-сервисах:_
- Tomcat NIO + async Servlet API + ограниченные worker pools.

### Semaphore-based bulkhead

Ограничение числа параллельных запросов:
- без выделения потоков,
- но с ограничением concurrency на уровне access.

Если предел достигнут → запрос сразу отклоняется или уходит в fallback.

Используется в Resilience4j Bulkhead (SemaphoreBulkhead).

### Connection pool bulkhead

Выделенные пулы подключений к разным downstream:
- отдельный JDBC pool к каждой базе,
- отдельный HTTP-пул к каждому внешнему API.

Это часто недооценивается, но на практике connection pool — один из самых важных bulkhead-механизмов.

### Queue-based bulkhead

На входе стоит очередь с фиксированной длиной.

Если очередь полна:
- запрос отклоняется,
- либо возвращается ошибка «server busy».

Цель — не допустить неконтролируемого роста очередей.

---

## Зачем Bulkhead, если есть Circuit Breaker

| Паттерн             | Что делает                                                                     |
| ------------------- | ------------------------------------------------------------------------------ |
| **Circuit Breaker** | Останавливает обращения к больной зависимости                                  |
| **Bulkhead**        | Гарантирует, что если зависимость умерла, она «не утопит» другие части сервиса |

Обычно они работают вместе:
- Bulkhead не даёт одному downstream сожрать все ресурсы.
- Circuit Breaker быстро открывает цепь, чтобы не ждать таймаутов.

---

## Техническая реализация Bulkhead в Java

### Несколько ExecutorService

    ExecutorService paymentPool = Executors.newFixedThreadPool(10);
    ExecutorService profilePool = Executors.newFixedThreadPool(10);

### Собственный HTTP-клиент под каждый downstream

    HttpClient paymentClient = HttpClient.newBuilder()
        .executor(paymentPool)
        .build();
    
    HttpClient profileClient = HttpClient.newBuilder()
        .executor(profilePool)
        .build();

### Отдельные timeout'ы и connection pool'ы

    OkHttpClient payment = new OkHttpClient.Builder()
        .connectionPool(new ConnectionPool(5, 5, TimeUnit.MINUTES))
        .build();

---

## Bulkhead в Resilience4j

Есть два варианта:
* SemaphoreBulkhead — ограничивает concurrency.
* ThreadPoolBulkhead — выделенный thread pool + очередь.


    BulkheadConfig config = BulkheadConfig.custom()
        .maxConcurrentCalls(10)
        .maxWaitDuration(Duration.ofMillis(100))
        .build();

или

    ThreadPoolBulkheadConfig config = ThreadPoolBulkheadConfig.custom()
        .coreThreadPoolSize(10)
        .maxThreadPoolSize(20)
        .queueCapacity(50)
        .build();