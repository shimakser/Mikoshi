# Basic SQL commands

**DDL** — CREATE, ALTER, DROP

**DML** — SELECT, INSERT, UPDATE, DELETE

**DCL** — GRANT, REVOKE, DENY

**TCL** — BEGIN TRANSACTION, COMMIT TRANSACTION, ROLLBACK TRANSACTION, SAVE TRANSACTION

---

### CREATE

- CREATE DATABASE — Создает БД.
- CREATE INDEX — Создает индекс (позволяет дублировать значения).
- CREATE OR REPLACE VIEW — Обновление представления.
- CREATE TABLE — Создает новую таблицу.
- CREATE PROCEDURE — Создает хранимую процедуру.
- CREATE UNIQUE INDEX — Создает уникальный индекс (без повторяющихся значений).
- CREATE VIEW — Создает представление на основе результирующего набора инструкции SELECT.

### INSERT

- INSERT INTO — Вставка новых строк.
- INSERT INTO — SELECT Копирует данные из одной таблицы в другую.

### SELECT

- SELECT — Выбор данных.
- SELECT DISTINCT — Выбирает только разные значения.
- SELECT INTO — Копирует данные из одной таблицы в новую таблицу.
- SELECT TOP — Задает количество записей, возвращаемых в результирующем наборе.

### DROP

- DROP COLUMN — Удаляет столбец.
- DROP CONSTRAINT — Удаляет UNIQUE, PRIMARY KEY, FOREIGN KEY, или ограничение CHECK.
- DROP DATABASE — Удаляет БД.
- DROP DEFAULT — Удаляет ограничение по умолчанию.
- DROP INDEX — Удаление индекса.
- DROP TABLE — Удаляет таблицу.
- DROP VIEW — Удаление представления.

### ALTER

- ALTER TABLE — Добавляет, удаляет или изменяет столбцы.
- ALTER COLUMN — Изменяет тип данных столбца.

### ADD

- ADD COLUMN — Добавляет столбец.
- ADD CONSTRAINT — Добавляет ограничение.

### JOIN

- JOIN — Для объединения таблиц.
- INNER JOIN — Возвращает строки, имеющие совпадающие значения в обеих таблицах.
- LEFT JOIN — Возвращает все строки из левой таблицы и соответствующие строки из правой таблицы.
- FULL OUTER JOIN — Возвращает все строки при наличии совпадения в левой или правой таблице.
- RIGHT JOIN — Возвращает все строки из правой таблицы и соответствующие строки из левой таблицы.
- OUTER JOIN — Возвращает все строки при наличии совпадения в левой или правой таблице.
- CROSS JOIN — подразумевает сбор сразу всех комбинаций элементов из нескольких таблиц. `SELECT * FROM table-1 CROSS JOIN table-2`.
- SELF JOIN — полезны в тех случаях, когда необходимо выполнить фильтрацию контента внутри одной таблицы. `SELECT * FROM products JOIN products ON table.product=table.brand`.

### UNION

Команда UNION объединяет данные из нескольких таблиц в одну при выборке.
`SELECT column_name(s) FROM table1
UNION
SELECT column_name(s) FROM table2;`

### GROUP

позволяет группировать результаты при выборке из базы данных.
`SELECT * FROM таблица WHERE условие GROUP BY поле`

### HAVING

WHERE вводит условие на отдельные строки; HAVING вводит условие на агрегаты, т.е. результаты отбора, когда из нескольких строк получается один результат, такой как счет, среднее значение, min, max или сумма.
`SELECT Singer, SUM(Sale) FROM Artists
GROUP BY Singer
HAVING SUM(Sale) > 2000000`

_Агрегатные функции:_ AVG, SUM, MIN, MAX, COUNT.

### VIEW

VIEW (Представление) — объект базы данных, являющийся результатом выполнения запроса к базе данных, определенного с помощью оператора SELECT, в момент обращения к представлению.

Дает возможность гибкой настройки прав доступа к данным за счет того, что права даются не на таблицу, а на представление. Это очень удобно в случае если пользователю нужно дать права на отдельные строки таблицы или возможность получения не самих данных, а результата каких-то действий над ними.

`CREATE VIEW v AS SELECT a.id, b.id FROM a,b;`
