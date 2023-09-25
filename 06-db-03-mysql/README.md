# Домашнее задание к занятию 3. «MySQL»

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/blob/virt-11/additional/README.md).

## Задача 1

Используя Docker, поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

> Docker-compose файл:

<details>
<summary>спойлер</summary>

```
version: "3.9"
services:
  mysql:
    container_name: mysql
    image: mysql:8.0
    restart: always
    tty: true
    environment:
      MYSQL_DATABASE: "test_db"
      MYSQL_ROOT_PASSWORD: "2Netology"
    ports:
      - "3306:3306"
    volumes:
      - dbdata:/var/lib/mysql
      - ".logs:/var/log/mysql"
      - "./mysql.cnf/:/etc/mysql.cnf"
      - "./backup:/backup"
volumes:
  dbdata:
    driver: local
```
</details>

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-03-mysql/test_data) и 
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h`, получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из её вывода версию сервера БД.

> Server version:		8.0.34 MySQL Community Server - GPL

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

> одна запись

<details>
<summary>спойлер с выводом</summary>

```
mysql> SELECT id, title, price FROM orders WHERE price IN (> 300);
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '> 300)' at line 1
mysql> SELECT id, title, price FROM orders WHERE price NOT BETWEEN 0 and 300;
+----+----------------+-------+
| id | title          | price |
+----+----------------+-------+
|  2 | My little pony |   500 |
+----+----------------+-------+
1 row in set (0.00 sec)

mysql> SELECT COUNT(price) FROM orders WHERE price NOT BETWEEN 0 and 300;
+--------------+
| COUNT(price) |
+--------------+
|            1 |
+--------------+
1 row in set (0.00 sec)
```
</details>

В следующих заданиях мы будем продолжать работу с этим контейнером.

## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:

- плагин авторизации mysql_native_password
- срок истечения пароля — 180 дней 
- количество попыток авторизации — 3 
- максимальное количество запросов в час — 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James".

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
    
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES, получите данные по пользователю `test` и 
**приведите в ответе к задаче**.

<details>
<summary> Спойлер с ходом выполнения: </summary>

```
mysql> CREATE USER 'test'@'localhost'
    -> IDENTIFIED WITH mysql_native_password BY 'test-pass'
    -> WITH MAX_CONNECTIONS_PER_HOUR 100
    -> PASSWORD EXPIRE INTERVAL 180 DAY
    -> FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME 2
    -> ATTRIBUTE '{"first_name":"James", "last_name":"Pretty"}';
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT SELECT ON test_db.* TO test@localhost;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> SHOW CREATE USER test@localhost;
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| CREATE USER for test@localhost                                                                                                                                                                                                                                                                                                                                                                                         |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| CREATE USER `test`@`localhost` IDENTIFIED WITH 'mysql_native_password' AS '*62C4834A52EB88A9E3EBA2EFF227C58AD0248317' REQUIRE NONE WITH MAX_CONNECTIONS_PER_HOUR 100 PASSWORD EXPIRE INTERVAL 180 DAY ACCOUNT UNLOCK PASSWORD HISTORY DEFAULT PASSWORD REUSE INTERVAL DEFAULT PASSWORD REQUIRE CURRENT DEFAULT FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME 2 ATTRIBUTE '{"last_name": "Pretty", "first_name": "James"}' |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

</details>

```
mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE USER = 'test';
+------+-----------+------------------------------------------------+
| USER | HOST      | ATTRIBUTE                                      |
+------+-----------+------------------------------------------------+
| test | localhost | {"last_name": "Pretty", "first_name": "James"} |
+------+-----------+------------------------------------------------+
```


## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

> ENGINE=InnoDB

<details>
<summary>спойлер с выводом</summary>

```
mysql> SHOW CREATE TABLE orders\G;
*************************** 1. row ***************************
       Table: orders
Create Table: CREATE TABLE `orders` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(80) NOT NULL,
  `price` int DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
1 row in set (0.00 sec)
```
</details>

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`,
- на `InnoDB`.

```
mysql> SET @@profiling = 1;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> ALTER TABLE orders ENGINE = MyISAM;
Query OK, 5 rows affected (0.05 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> ALTER TABLE orders ENGINE = InnoDB;
Query OK, 5 rows affected (0.07 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SHOW PROFILES;
+----------+------------+------------------------------------+
| Query_ID | Duration   | Query                              |
+----------+------------+------------------------------------+
|       19 | 0.04230900 | ALTER TABLE orders ENGINE = MyISAM |
|       20 | 0.07169550 | ALTER TABLE orders ENGINE = InnoDB |
+----------+------------+------------------------------------+
2 rows in set, 1 warning (0.00 sec)

```

## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):

- скорость IO важнее сохранности данных;
- нужна компрессия таблиц для экономии места на диске;
- размер буффера с незакомиченными транзакциями 1 Мб;
- буффер кеширования 30% от ОЗУ;
- размер файла логов операций 100 Мб.

Приведите в ответе изменённый файл `my.cnf`.

```
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

bind-address=0.0.0.0
server-id=1
log_bin=/var/log/mysql/mybin.log

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

innodb_flush_log_at_trx_commit = 0  # скорость IO важнее сохранности данных;
innodb_file_per_table = ON          # нужна компрессия таблиц для экономии места на диске;
innodb_log_buffer_size = 1M         # размер буффера с незакомиченными транзакциями 1 Мб;
innodb_buffer_pool_size = 2G        # буффер кеширования 30% от ОЗУ;
innodb_log_file_size = 100M         # размер файла логов операций 100 Мб.
```
---

### Как оформить ДЗ

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

