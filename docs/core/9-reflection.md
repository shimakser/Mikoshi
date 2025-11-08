# Reflection API

**Пакет:** `java.lang.reflect`

---

## Ключевые классы
- Class<?> — метаинформация о типе
- Field, Method, Constructor
- Modifier — модификаторы доступа
- Parameter — параметры метода (c -parameters компиляцией)

### Примеры

    Class<?> clazz = Person.class;
    Field field = clazz.getDeclaredField("name");
    field.setAccessible(true);
    field.set(person, "Max");
    
    Method method = clazz.getDeclaredMethod("sayHello");
    method.invoke(person);

---

## Возможности
- Получение информации о полях, методах, аннотациях
- Вызов методов и конструкторов в рантайме
- Создание инстансов (clazz.getDeclaredConstructor().newInstance())

---

## Недостатки
- Потеря типобезопасности
- Медленнее обычного вызова
- Может нарушать инкапсуляцию

---

## Используется в
- Spring
- Hibernate
- Jackson
- JUnit
- Lombok