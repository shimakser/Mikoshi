# Optional

Это контейнерный класс в Java, появившийся в Java 8, который предназначен для явного представления отсутствующего значения (вместо null) и снижения риска NullPointerException.

Это обёртка вокруг значения типа T, которая либо содержит объект (non-empty), либо пуста (empty).

---

## Создание Optional

- Optional<String> opt1 = Optional.of("hello");     // значение не null
- Optional<String> opt2 = Optional.ofNullable(null); // может быть null
- Optional<String> opt3 = Optional.empty();         // пустой Optional

Optional.of(null) выбросит NullPointerException.
Используй ofNullable() если значение может быть null.

--- 

## Функциональные методы

- map
- flatMap
- filter
- orElse
- orElseGet
- orElseThrow

--- 

## orElse vs orElseGet

- orElse — возвращает value/выполняет логику, даже если значение есть.
- orElseGet — вычисляет значение только если пусто