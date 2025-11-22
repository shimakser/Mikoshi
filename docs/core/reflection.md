# Reflection API

## Основные возможности

- Доступ к структуре классов во время выполнения.
- Классы:
  - Class<?> — метаинформация класса.
  - Field, Method, Constructor — доступ к полям, методам, конструкторам.
- Получение Class:

         Class<?> clazz = MyClass.class;
         Class<?> clazz2 = Class.forName("com.example.MyClass");
         Object obj = new MyClass();
         clazz = obj.getClass();

---

## Доступ к методам и полям

- Получение методов:

        Method method = clazz.getDeclaredMethod("methodName", String.class);
        method.setAccessible(true);
        Object result = method.invoke(obj, "arg");

- Получение полей:

         Field field = clazz.getDeclaredField("fieldName");
         field.setAccessible(true);
         field.set(obj, value);
         Object val = field.get(obj);

- Получение конструкторов:

         Constructor<MyClass> ctor = MyClass.class.getDeclaredConstructor(String.class);
         ctor.setAccessible(true);
         MyClass instance = ctor.newInstance("arg");

---

## Доступ к приватным элементам

- setAccessible(true) позволяет обойти модификаторы доступа.
- Использовать осторожно, т.к. нарушает инкапсуляцию и может ломаться при JDK модулях.

---

## Инспекция типов

- `clazz.getSuperclass()` — суперкласс.
- `clazz.getInterfaces()` — реализованные интерфейсы.
- `clazz.isAssignableFrom(otherClass)` — проверка совместимости типов.

---

## Аннотации

- Получение аннотаций:

        MyAnnotation ann = clazz.getAnnotation(MyAnnotation.class);
        Method m = clazz.getDeclaredMethod("method");
        MyAnnotation methodAnn = m.getAnnotation(MyAnnotation.class);

- Можно применять для runtime-проверок, DI, сериализации.