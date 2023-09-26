# Домашнее задание к занятию 4. «PostgreSQL»

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL, используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:

- вывода списка БД,
> \l[+]   [PATTERN]      list databases
- подключения к БД,
> Connection
> \c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}  
>                        connect to new database (currently "postgres")  
> \conninfo              display information about current connection  
> \encoding [ENCODING]   show or set client encoding  
> \password [USERNAME]   securely change the password for a user  
- вывода списка таблиц,
> \dt[S+] [PATTERN]      list tables
- вывода описания содержимого таблиц,
> \d[S+]  NAME           describe table, view, sequence, or index
- выхода из psql.
> \q

## Задача 2

Используя `psql`, создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления, и полученный результат.

```
postgres=# CREATE DATABASE test_database;
CREATE DATABASE

postgres=# \q

root@9c8e31391e7a:/# psql -U admin-user -f /backup/test_dump.sql test_database
SET
SET
SET
SET
SET
 set_config 
------------
 
(1 row)

SET
SET
SET
SET
SET
SET
CREATE TABLE
psql:/backup/test_dump.sql:34: ERROR:  role "postgres" does not exist
CREATE SEQUENCE
psql:/backup/test_dump.sql:49: ERROR:  role "postgres" does not exist
ALTER SEQUENCE
ALTER TABLE
COPY 8
 setval 
--------
      8
(1 row)

ALTER TABLE

root@9c8e31391e7a:/# psql -U admin-user test_database
psql (13.0 (Debian 13.0-1.pgdg100+1))
Type "help" for help.

test_database=# ANALYZE VERBOSE public.orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE
                                                    ^
test_database=# SELECT avg_width FROM pg_stats WHERE tablename='orders';
 avg_width 
-----------
         4
        16
         4
(3 rows)

```

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам как успешному выпускнику курсов DevOps в Нетологии предложили
провести разбиение таблицы на 2: шардировать на orders_1 - price>499 и orders_2 - price<=499.

Предложите SQL-транзакцию для проведения этой операции.

```
test_database=# CREATE TABLE orders_1 (CHECK (price > 499)) INHERITS (orders);
CREATE TABLE
test_database=# INSERT INTO orders_1 SELECT * FROM orders WHERE price > 499;
INSERT 0 3
test_database=# CREATE TABLE orders_2 (CHECK (price <= 499)) INHERITS (orders);
CREATE TABLE
test_database=# INSERT INTO orders_2 SELECT * FROM orders WHERE price <= 499;
INSERT 0 5
test_database=# \dt+
                                List of relations
 Schema |   Name   | Type  |   Owner    | Persistence |    Size    | Description 
--------+----------+-------+------------+-------------+------------+-------------
 public | orders   | table | admin-user | permanent   | 8192 bytes | 
 public | orders_1 | table | admin-user | permanent   | 8192 bytes | 
 public | orders_2 | table | admin-user | permanent   | 8192 bytes | 
(3 rows)
```

Можно ли было изначально исключить ручное разбиение при проектировании таблицы orders?

> Да, можно было изначально сделать orders секционированной таблицей:

```
CREATE TABLE orders (
    id              serial not null,
    title           character varying(80) not null,
    price           integer
) PARTITION BY RANGE (price);

CREATE TABLE orders_less_499 PARTITION OF orders
    FOR VALUES FROM (MINVALUE) TO (500);
ALTER TABLE orders_less_499 ADD PRIMARY KEY (id);

CREATE TABLE orders_more_499 PARTITION OF orders
    FOR VALUES FROM (500) TO (MAXVALUE);
ALTER TABLE orders_more_499 ADD PRIMARY KEY (id);

```
> Примечание: при использовании `FROM` `TO` границы каждого диапазона понимаются как включающие в нижней части и исключающие в верхней части.

## Задача 4

Используя утилиту `pg_dump`, создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

