# JVM/Java Performance Optimization


## CPU/Allocation profiling

### Что такое профилирование в принципе

Профилирование — это сбор объективных данных о том:
* где CPU реально тратит время
* где и сколько памяти реально аллоцируется
* какие потоки реально блокируются

Ключевая мысль:<br>

    Ты не оптимизируешь код — ты оптимизируешь bottleneck, подтверждённый профилем.

### CPU profiling

CPU profiling показывает:
* какие методы выполняются чаще всего
* где процесс реально тратит CPU-время
* где возникают горячие точки (hot methods)

Важно:
* CPU profiling ≠ “какой код часто вызывается”
* CPU profiling = “где реально тратится процессорное время”

Типичные причины CPU bottleneck:
* сложные алгоритмы
* избыточная сериализация
* JSON/XML parsing
* regex
* криптография
* синхронизация (locks)
* spin-wait

### Allocation profiling

Allocation profiling показывает:
* где создаётся больше всего объектов
* какие типы объектов доминируют
* какие аллокации приводят к GC pressure

_Важно:_ Большинство GC-проблем — это allocation-проблемы, а не GC-проблемы.

Типичные источники аллокаций:
* DTO на каждый запрос
* String concatenation
* boxing/unboxing
* Streams API
* JSON parsing
* временные коллекции
* copy-on-write структуры

---

## async-profiler

Профайлер для JVM, который:
- использует perf events / async stack traces
- почти не искажает поведение приложения
- подходит для production-like нагрузок

Он умеет:
- CPU profiling
- Allocation profiling
- Lock profiling
- Wall-clock profiling

Что ты реально смотришь в async-profiler
* hot paths — цепочки методов
* методы, занимающие >10–20% CPU
* массовые allocation sites
* блокировки, которые держатся долго

### Почему async-profiler важен

Классические JVM-профайлеры:
* сильно замедляют приложение
* искажают результаты
* не подходят под нагрузкой

async-profiler:
* можно запускать **в бою**
* можно запускать **под нагрузкой**
* даёт правдивую картину

---

## Java Flight Recorder (JFR) / Java Mission Control (JMC)

### JFR — встроенный в JVM механизм телеметрии:
* события CPU
* события аллокаций
* GC
* блокировки
* потоки
* IO
* compilation
* safepoints

_Особенность:_
* очень низкий overhead
* можно писать continuously

### JMC — GUI-инструмент для анализа JFR-записей.

Он позволяет:
* видеть timeline
* находить correlation (GC ↔ latency)
* анализировать memory pressure
* анализировать thread states

### Когда использовать JFR, а когда async-profiler

| Сценарий                    | Инструмент     |
| --------------------------- | -------------- |
| Глубокий CPU hotspot        | async-profiler |
| Allocation storm            | async-profiler |
| GC pauses / safepoints      | JFR            |
| Общая картина поведения JVM | JFR            |
| Production tracing          | JFR            |

---

## flame graphs

Визуализация:
* стека вызовов
* ширина = количество времени / аллокаций
* высота = глубина стека

Важно:
* верхние блоки ≠ проблема
* ширина = проблема

---

## GC pauses

GC pause — момент, когда:
* application threads останавливаются
* JVM чистит память
* latency скачет

_Важно:_ GC не “плохой”, плохое — слишком много мусора.

---

## allocation-heavy code

Код, который:
* создаёт много временных объектов
* на каждый запрос
* на каждый элемент коллекции

Примеры:
* Streams API
* map/filter/collect
* boxing
* DTO mapping
* JSON parsing

Почему это опасно:
* GC pressure
* рост p99 latency
* throughput падает
* CPU уходит в GC

---

## thread pool starvation

Ситуация, когда:
* все worker threads заняты
* новые задачи стоят в очереди
* latency растёт лавинообразно

Причины:
* блокирующие операции
* sync IO
* long GC pauses
* retry storm

---

## warm connection pools

Заранее открытые и поддерживаемые соединения к внешним системам (DB, HTTP clients), которые переиспользуются для запросов.

Создание соединения = дорого:
* TCP handshake
* TLS handshake
* auth
* negotiation

_Почему важно:_
* при пиковом трафике создание TCP/TLS соединения дорого (handshake, TLS, auth);
* warm pools уменьшают latency и CPU overhead.
* позволяет делать autoscaling без холодного старта.

_Практика:_
* держать достаточный пул соединений (min size) + max size;
* регулярно health-check и keepalive, чтобы conexiones не были закрыты NAT/firewall;
* pre-warm при старте новых инстансов (делать test queries), особенно при autoscale in;
* tune socket options: TCP_NODELAY, TCP keepalive.

_Проблемы:_ слишком большой pool = лишние resource (file descriptors), слишком маленький = connection starvation.
