# Домашнее задание к занятию 2. «SQL»

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/blob/virt-11/additional/README.md).

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.

```bash
version: "3.9"
services:
  postgres:
    container_name: postgres
    image: postgres:13.3
    restart: always
    environment:
      POSTGRES_DB: "test_db"
      POSTGRES_USER: "admin-user"
      POSTGRES_PASSWORD: "2Netology"
      PGDATA: "/var/lib/postgresql/pgdata"
    ports:
      - "5432:5432"
    volumes:
      - "./docker-entrypoint-initdb.d:/docker-entrypoint-initdb.d"
      - "./test_db-data:/var/lib/postgresql/pgdata"
      - "./backup:/backup"
```

Файл action.sql в docker-entrypoint-initdb.d с учётом последующих заданий:

~~~~sql
CREATE ROLE "test-admin-user" LOGIN PASSWORD '2Netology';

CREATE ROLE "test-simple-user" LOGIN PASSWORD '2Netology';

CREATE TABLE IF NOT EXISTS orders (
    "id"                 serial PRIMARY KEY,
    "наименование"       VARCHAR(200) NOT NULL,
    "цена"               INTEGER NOT NULL
);

CREATE TABLE IF NOT EXISTS clients (
    "id"                 serial PRIMARY KEY,
    "фамилия"            VARCHAR(200) NOT NULL,
    "страна проживания"  VARCHAR(200) NOT NULL,
    "заказ"              INTEGER,
    FOREIGN KEY (заказ) REFERENCES orders (id)
);

GRANT ALL ON orders, clients TO "test-admin-user";

GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE orders, clients TO "test-simple-user";

INSERT INTO
    orders (наименование, цена)
VALUES
    ('Шоколад',	'10'),
    ('Принтер',	'3000'),
    ('Книга',	'500'),
    ('Монитор',	'7000'),
    ('Гитара',	'4000');

INSERT INTO
    clients (фамилия, "страна проживания")
VALUES
    ('Иванов Иван Иванович', 'USA'),
    ('Петров Петр Петрович', 'Canada'),
    ('Иоганн Себастьян Бах', 'Japan'),
    ('Ронни Джеймс Дио',     'Russia'),
    ('Ritchie Blackmore',    'Russia');

UPDATE clients SET "заказ" = (SELECT id FROM orders WHERE "наименование"='Книга') WHERE "фамилия"='Иванов Иван Иванович';
UPDATE clients SET "заказ" = (SELECT id FROM orders WHERE "наименование"='Монитор') WHERE "фамилия"='Петров Петр Петрович';
UPDATE clients SET "заказ" = (SELECT id FROM orders WHERE "наименование"='Гитара') WHERE "фамилия"='Иоганн Себастьян Бах';
~~~~

## Задача 2

В БД из задачи 1: 

- создайте пользователя test-admin-user и БД test_db;
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже);
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db;
- создайте пользователя test-simple-user;
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE этих таблиц БД test_db.

Таблица orders:

- id (serial primary key);
- наименование (string);
- цена (integer).

Таблица clients:

- id (serial primary key);
- фамилия (string);
- страна проживания (string, index);
- заказ (foreign key orders).

Приведите:

- итоговый список БД после выполнения пунктов выше;

```
test_db=# \l
                                      List of databases
   Name    |   Owner    | Encoding |  Collate   |   Ctype    |       Access privileges       
-----------+------------+----------+------------+------------+-------------------------------
 postgres  | admin-user | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | admin-user | UTF8     | en_US.utf8 | en_US.utf8 | =c/"admin-user"              +
           |            |          |            |            | "admin-user"=CTc/"admin-user"
 template1 | admin-user | UTF8     | en_US.utf8 | en_US.utf8 | =c/"admin-user"              +
           |            |          |            |            | "admin-user"=CTc/"admin-user"
 test_db   | admin-user | UTF8     | en_US.utf8 | en_US.utf8 | 
(4 rows)
```

- описание таблиц (describe);

```
test_db=# \d clients
                                         Table "public.clients"
      Column       |          Type          | Collation | Nullable |               Default               
-------------------+------------------------+-----------+----------+-------------------------------------
 id                | integer                |           | not null | nextval('clients_id_seq'::regclass)
 фамилия           | character varying(200) |           | not null | 
 страна проживания | character varying(200) |           | not null | 
 заказ             | integer                |           |          | 
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
Foreign-key constraints:
    "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)
```
```
test_db=# \d orders
                                       Table "public.orders"
    Column    |          Type          | Collation | Nullable |              Default               
--------------+------------------------+-----------+----------+------------------------------------
 id           | integer                |           | not null | nextval('orders_id_seq'::regclass)
 наименование | character varying(200) |           | not null | 
 цена         | integer                |           | not null | 
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)
```

- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;

```
test_db=# SELECT 
    grantee, table_name, privilege_type 
FROM 
    information_schema.table_privileges 
WHERE 
    grantee in ('test-admin-user','test-simple-user')
    and table_name in ('clients','orders')
order by 
    grantee;
```
- список пользователей с правами над таблицами test_db.

```
     grantee      | table_name | privilege_type 
------------------+------------+----------------
 test-admin-user  | orders     | INSERT
 test-admin-user  | orders     | SELECT
 test-admin-user  | orders     | UPDATE
 test-admin-user  | orders     | DELETE
 test-admin-user  | orders     | TRUNCATE
 test-admin-user  | orders     | REFERENCES
 test-admin-user  | orders     | TRIGGER
 test-admin-user  | clients    | INSERT
 test-admin-user  | clients    | SELECT
 test-admin-user  | clients    | UPDATE
 test-admin-user  | clients    | DELETE
 test-admin-user  | clients    | TRUNCATE
 test-admin-user  | clients    | REFERENCES
 test-admin-user  | clients    | TRIGGER
 test-simple-user | clients    | INSERT
 test-simple-user | orders     | INSERT
 test-simple-user | orders     | SELECT
 test-simple-user | orders     | UPDATE
 test-simple-user | orders     | DELETE
 test-simple-user | clients    | SELECT
 test-simple-user | clients    | UPDATE
 test-simple-user | clients    | DELETE
(22 rows)
```


## Задача 3

Используя SQL-синтаксис, наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL-синтаксис:
- вычислите количество записей для каждой таблицы.

Приведите в ответе:

    - запросы,
    - результаты их выполнения.

> Таблица наполнена через action.sql смонтированный в docker-entrypoint-initdb.d

```
test_db=# select count(1) from orders;
 count 
-------
     5
(1 row)

test_db=# select count(1) from clients;
 count 
-------
     5
(1 row)

```

## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys, свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения этих операций.

> Заказы заполнены через action.sql смонтированный в docker-entrypoint-initdb.d

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.

```
test_db=# SELECT * FROM clients WHERE заказ IS NOT NULL;
 id |       фамилия        | страна проживания | заказ 
----+----------------------+-------------------+-------
  1 | Иванов Иван Иванович | USA               |     3
  2 | Петров Петр Петрович | Canada            |     4
  3 | Иоганн Себастьян Бах | Japan             |     5
(3 rows)

``` 

Подсказка: используйте директиву `UPDATE`.

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните, что значат полученные значения.

```
test_db=# EXPLAIN SELECT * FROM clients WHERE заказ IS NOT NULL;
                        QUERY PLAN                         
-----------------------------------------------------------
 Seq Scan on clients  (cost=0.00..10.90 rows=90 width=844)
   Filter: ("заказ" IS NOT NULL)
(2 rows)
```

> Числа, перечисленные в скобках (слева направо), имеют следующий смысл:

>  * Приблизительная стоимость запуска. Это время, которое проходит, прежде чем начнётся этап вывода данных.  
>  * Приблизительная общая стоимость. Она вычисляется в предположении, что узел плана выполняется до конца, то есть возвращает все доступные строки.  
>  * Ожидаемое число строк, которое должен вывести этот узел плана.  
>  * Ожидаемый средний размер строк, выводимых этим узлом плана (в байтах).  

> Результат EXPLAIN в данном случае не совпадает (как минимум по строкам) т.к. выводит только предполагаемый результат, команда ANALYZE выведет фактическое время выполнения и другую статистику
```
test_db=# EXPLAIN ANALYZE SELECT * FROM clients WHERE заказ IS NOT NULL;
                                             QUERY PLAN                                              
-----------------------------------------------------------------------------------------------------
 Seq Scan on clients  (cost=0.00..10.90 rows=90 width=844) (actual time=0.012..0.013 rows=3 loops=1)
   Filter: ("заказ" IS NOT NULL)
   Rows Removed by Filter: 2
 Planning Time: 0.051 ms
 Execution Time: 0.026 ms
(5 rows)

```
## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

```bash
wolin@wolinubuntu:~/netology/06-db-02-sql$ docker exec -it postgres bash

root@6d11af5b8868:/# pg_dump -U admin-user -Fc test_db > ./backup/1test_db_backup.psql
root@6d11af5b8868:/# exit
```

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

```bash
 wolin@wolinubuntu:~/netology/06-db-02-sql$ docker stop postgres
```

Поднимите новый пустой контейнер с PostgreSQL.

```bash
wolin@wolinubuntu:~/netology/06-db-02-sql$ docker run --rm -d -e POSTGRES_USER=admin-user -e POSTGRES_PASSWORD=2Netology -e POSTGRES_DB=test_db -v ./backup:/backup --name postgres2 postgres:13.3
```

Восстановите БД test_db в новом контейнере.

```bash
wolin@wolinubuntu:~/netology/06-db-02-sql$ docker exec -it postgres2 bash

root@119a68dc6b92:/# pg_restore -O -U admin-user -d test_db /backup/1test_db_backup.psql
```

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

