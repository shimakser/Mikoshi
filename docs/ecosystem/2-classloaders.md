# Classloaders

---

Это часть JVM, которая загружает .class-файлы (байткод) в память при запуске программы. Он превращает файл на диске в объект Class.

## Основные виды
1. Bootstrap Class Loader
   - _Загружает:_ базовые классы Java (из rt.jar, если Java 8, или java.base в Java 9+), например java.lang.String, java.util.List.
   - Не написан на Java — встроен в JVM (на C++).
   - _Источник:_ JAVA_HOME/lib.
2. Extension Class Loader (сейчас называется Platform Class Loader в Java 9+)
   - _Загружает:_ расширенные библиотеки из папки lib/ext (или модули платформы в Java 9+).
   - _Источник:_ JAVA_HOME/lib/ext.
3. Application (System) Class Loader
   - _Загружает:_ классы из твоего classpath (т.е. твой target/classes, библиотеки из lib/, build/classes и т.д.).
   - Это то, что ты используешь по умолчанию.
4. Custom Class Loader
   - Это твой собственный класс, расширяющий ClassLoader.
   - _Используется для:_
     - загрузки классов из нестандартных мест (например, из БД, интернета);
     - изоляции плагинов;
     - hot-reload классов;
     - sandbox исполнения (например, при создании своей JVM на базе JVM).
