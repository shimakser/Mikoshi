# Generics

Generics позволяют:
- Объявлять классы, интерфейсы и методы, зависящие от типа.
- Гарантировать **type safety** во время компиляции.
- Исключить ручное приведение типов и `ClassCastException` во время выполнения.

---

## Wildcards
1. Ковариантность — **extends** — все дочерние, только чтение.
    `class NumberBox<T extends Number> { ... }`

- Контравариантность — **super** — все родительские. Разрешает запись объектов T, но чтение — только как Object.
    `class NumberBox<T super Number> { ... }`

---

## Принцип PECS

_Producer Extends, Consumer Super_

- extends — если объект производит значения (get → read)
- super — если объект потребляет значения (add → write)

---

## Стирание типов (Type Erasure) и влияние на runtime

Generics в Java реализованы через type erasure:
- Все параметры типов стираются на этапе компиляции.
- На уровне байткода остаются только сырые типы (raw types) и приведения.

### Пример

    List<String> list = new ArrayList<>();
    list.add("x");
    String s = list.get(0);
Компилятор превращает это в

    List list = new ArrayList();
    list.add("x");
    String s = (String) list.get(0);

### Последствия
- Нет информации о типах в runtime (type information erased).
- Невозможно сделать new T(), instanceof T, T.class, T[].
- Сравнение типов ограничено (List<String> и List<Integer> выглядят одинаково в runtime).

---

## Ограничения
- Нет new T().
- Нет T.class.
- Нет instanceof T.
- Нельзя использовать generic как тип массива (можно использовать List<?>[], но с ограничениями)..
- Нет рефлексии по T без дополнительного параметра (Class<T> clazz).
- Статические поля не могут быть параметризированы (общие для всех параметров типа).

---

## Обобщённые методы

    public static <T> void print(T value) {
        System.out.println(value);
    }

---

## Итог

| Тема                   | Ключевые идеи                                           |
| ---------------------- | ------------------------------------------------------- |
| **Типизация**          | строгая, статическая, с параметрическим полиморфизмом   |
| **Параметры типов**    | `<T>`, границы `extends`/`super`                        |
| **Инвариантность**     | `List<Integer>` ≠ `List<Number>`                        |
| **Ковариантность**     | `? extends T` — читатель                                |
| **Контравариантность** | `? super T` — писатель                                  |
| **PECS**               | Producer Extends, Consumer Super                        |
| **Стирание**           | Generics существуют только на этапе компиляции          |
| **Type safety**        | проверка типов во время компиляции                      |
| **Ограничения**        | нельзя `new T()`, `static T`, `T.class`, generic arrays |
| **Type tokens**        | способ хранить информацию о типе в runtime              |
