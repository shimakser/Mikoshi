# Files, NIO.2

## Основные компоненты

- **Path** – основная абстракция файла/директории (java.nio.file.Path).

        Path path = Paths.get("/home/user/file.txt");

- **Paths** – фабрика для создания Path.
- **Files** – статические методы для операций с файлами и директориями (exists, copy, move, delete, readAllLines и т.д.).
- **FileSystem & FileSystems** – информация о файловой системе, корневые директории, файловые атрибуты.
- **WatchService** – наблюдение за событиями в директориях (CREATE, DELETE, MODIFY).

---

## Работа с файлами

- Проверка существования: Files.exists(path), Files.notExists(path).
- Создание файлов/директорий:

          Files.createFile(path);
          Files.createDirectories(path);

- Чтение и запись:

        List<String> lines = Files.readAllLines(path, StandardCharsets.UTF_8);
        Files.write(path, lines, StandardCharsets.UTF_8, StandardOpenOption.APPEND);

- Копирование/перемещение:

        Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
        Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);

---

## WatchService

- Слежение за изменениями в директории:

        WatchService watchService = FileSystems.getDefault().newWatchService();
        dir.register(watchService, ENTRY_CREATE, ENTRY_DELETE, ENTRY_MODIFY);

- Асинхронная обработка событий.
- _Важно:_ события могут сливаться или теряться, нужен корректный цикл обработки.

---

## Потоковая работа с файлами

- Files.newBufferedReader(path), Files.newBufferedWriter(path) для оптимизированного I/O.
- Files.newInputStream(path) / Files.newOutputStream(path) для потоков.
- Использовать try-with-resources.
