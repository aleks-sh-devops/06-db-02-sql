# Домашнее задание к занятию "6.2. SQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose манифест.

```
docker container run  \
--name pg_devops -e POSTGRES_PASSWORD=strongpass \
-e POSTGRES_USER=pgusr -p 5434:5432 -v db_1:/var/lib/postgresql/data \
-v db_2:/var/lib/postgresql -d postgres:12


Unable to find image 'postgres:12' locally
12: Pulling from library/postgres
a603fa5e3b41: Pull complete
02d7a77348fd: Pull complete
16b62ca80c8f: Pull complete
fbd795da1fe1: Pull complete
9c68de39d930: Pull complete
2e441a95082c: Pull complete
1c97f440fe14: Pull complete
87a3f78bc5d1: Pull complete
6f5522bdba19: Pull complete
3ffbed8daf3b: Pull complete
fe084ee65e13: Pull complete
3b4e12d98615: Pull complete
f6c5d03edc85: Pull complete
Digest: sha256:30e1c43b089a089dd9a7bab8fbdd5cbd611b8ffb925930d695593b37867a42ab
Status: Downloaded newer image for postgres:12
76c7f1a636d8f42679a4a56b0a0f3f057a06795fb991644a5fca7c4080a910c1
```
```
 docker ps
 
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                    NAMES
22fad2c31dc8   postgres:12   "docker-entrypoint.s…"   4 seconds ago   Up 3 seconds   0.0.0.0:5434->5432/tcp   pg_devops
```
## Задача 2

В БД из задачи 1: 
- создайте пользователя test-admin-user и БД test_db
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже)
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db
- создайте пользователя test-simple-user  
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db

```
docker exec -it musing_goldwasser bash
root@22fad2c31dc8:/# psql -U postgres
psql (12.13 (Debian 12.13-1.pgdg110+1))
Type "help" for help.


postgres=# CREATE DATABASE test_db;
CREATE DATABASE
postgres=# create user "test-admin-user" with encrypted password 'strongadminpass';
CREATE ROLE
postgres-# \c test_db
You are now connected to database "test_db" as user "postgres".
test_db=# create table orders (id serial primary key, наименование varchar, цена integer);
CREATE TABLE
test_db=# create table clients (id serial primary key, "фамилия" varchar, "страна проживания" varchar, заказ integer, foreign key (заказ) references  orders(id));
CREATE TABLE
test_db=# create index strana on clients("страна проживания");
CREATE INDEX
postgres=# grant all on database test_db to "test-admin-user";
GRANT
postgres=# create user "test-simple-user" with encrypted password 'strongsimpleuserpass';
CREATE ROLE
postgres=# grant select, insert, update, delete ON orders to "test-simple-user";
GRANT
postgres=# grant select, insert, update, delete ON clients to "test-simple-user";
GRANT


test_db=# \l
                                     List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |       Access privileges
-----------+----------+----------+------------+------------+--------------------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres                   +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres                   +
           |          |          |            |            | postgres=CTc/postgres
 test_db   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =Tc/postgres                  +
           |          |          |            |            | postgres=CTc/postgres         +
           |          |          |            |            | "test-admin-user"=CTc/postgres
(4 rows)

test_db=# \d orders
                                    Table "public.orders"
    Column    |       Type        | Collation | Nullable |              Default
--------------+-------------------+-----------+----------+------------------------------------
 id           | integer           |           | not null | nextval('orders_id_seq'::regclass)
 наименование | character varying |           |          |
 цена         | integer           |           |          |
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)

test_db=#

test_db=# \d clients
                                       Table "public.clients"
      Column       |       Type        | Collation | Nullable |               Default
-------------------+-------------------+-----------+----------+-------------------------------------
 id                | integer           |           | not null | nextval('clients_id_seq'::regclass)
 фамилия           | character varying |           |          |
 страна проживания | character varying |           |          |
 заказ             | integer           |           |          |
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
    "strana" btree ("страна проживания")
Foreign-key constraints:
    "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)

test_db=# select * from information_schema.table_privileges where table_name = 'orders' or table_name = 'clients';
 grantor  |     grantee      | table_catalog | table_schema | table_name | privilege_type | is_grantable | with_hierarchy
----------+------------------+---------------+--------------+------------+----------------+--------------+----------------
 postgres | postgres         | test_db       | public       | orders     | INSERT         | YES          | NO
 postgres | postgres         | test_db       | public       | orders     | SELECT         | YES          | YES
 postgres | postgres         | test_db       | public       | orders     | UPDATE         | YES          | NO
 postgres | postgres         | test_db       | public       | orders     | DELETE         | YES          | NO
 postgres | postgres         | test_db       | public       | orders     | TRUNCATE       | YES          | NO
 postgres | postgres         | test_db       | public       | orders     | REFERENCES     | YES          | NO
 postgres | postgres         | test_db       | public       | orders     | TRIGGER        | YES          | NO
 postgres | test-simple-user | test_db       | public       | orders     | INSERT         | NO           | NO
 postgres | test-simple-user | test_db       | public       | orders     | SELECT         | NO           | YES
 postgres | test-simple-user | test_db       | public       | orders     | UPDATE         | NO           | NO
 postgres | test-simple-user | test_db       | public       | orders     | DELETE         | NO           | NO
 postgres | postgres         | test_db       | public       | clients    | INSERT         | YES          | NO
 postgres | postgres         | test_db       | public       | clients    | SELECT         | YES          | YES
 postgres | postgres         | test_db       | public       | clients    | UPDATE         | YES          | NO
 postgres | postgres         | test_db       | public       | clients    | DELETE         | YES          | NO
 postgres | postgres         | test_db       | public       | clients    | TRUNCATE       | YES          | NO
 postgres | postgres         | test_db       | public       | clients    | REFERENCES     | YES          | NO
 postgres | postgres         | test_db       | public       | clients    | TRIGGER        | YES          | NO
 postgres | test-simple-user | test_db       | public       | clients    | INSERT         | NO           | NO
 postgres | test-simple-user | test_db       | public       | clients    | SELECT         | NO           | YES
 postgres | test-simple-user | test_db       | public       | clients    | UPDATE         | NO           | NO
 postgres | test-simple-user | test_db       | public       | clients    | DELETE         | NO           | NO
(22 rows)

```


Таблица orders:
- id (serial primary key)
- наименование (string)
- цена (integer)

Таблица clients:
- id (serial primary key)
- фамилия (string)
- страна проживания (string, index)
- заказ (foreign key orders)

Приведите:
- итоговый список БД после выполнения пунктов выше,
- описание таблиц (describe)
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
- список пользователей с правами над таблицами test_db

## Задача 3

Используя SQL синтаксис - наполните таблицы следующими тестовыми данными:

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

Используя SQL синтаксис:
- вычислите количество записей для каждой таблицы 
- приведите в ответе:
    - запросы 
    - результаты их выполнения.

```
test_db=# insert into orders VALUES (1, 'Шоколад', 10), (2, 'Принтер', 3000), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);
INSERT 0 5

test_db=# SELECT * FROM orders;
 id | наименование | цена
----+--------------+------
  1 | Шоколад      |   10
  2 | Принтер      | 3000
  3 | Книга        |  500
  4 | Монитор      | 7000
  5 | Гитара       | 4000
(5 rows)

test_db=# SELECT * FROM clients;
 id |       фамилия        | страна проживания | заказ
----+----------------------+-------------------+-------
  1 | Иванов Иван Иванович | USA               |
  2 | Петров Петр Петрович | Canada            |
  3 | Иоганн Себастьян Бах | Japan             |
  4 | Ронни Джеймс Дио     | Russia            |
  5 | Ritchie Blackmore    | Russia            |
(5 rows)

Если нужно отдельно количество, то:
test_db=# select count (*) from orders;
 count
-------
     5
(1 row)

test_db=# select count (*) from clients;
 count
-------
     5
(1 row)
```


## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения данных операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.
 
**Подсказк (ошибка в задании)** - используйте директиву `UPDATE`.
```
test_db=# update clients set заказ=(select id from orders where наименование='Книга') where фамилия='Иванов Иван Иванович';
UPDATE 1
test_db=# update clients set заказ=(select id from orders where наименование='Монитор') where фамилия='Петров Петр Петрович';
UPDATE 1
test_db=# update clients set заказ=(select id from orders where наименование='Гитара') where фамилия='Иоганн Себастьян Бах';
UPDATE 1
test_db=# select * from clients where заказ is not null;
 id |       фамилия        | страна проживания | заказ
----+----------------------+-------------------+-------
  1 | Иванов Иван Иванович | USA               |     3
  2 | Петров Петр Петрович | Canada            |     4
  3 | Иоганн Себастьян Бах | Japan             |     5
(3 rows)
```

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните что значат полученные значения.

```
test_db=# explain select * from clients where заказ is not null;
                        QUERY PLAN
-----------------------------------------------------------
 Seq Scan on clients  (cost=0.00..18.10 rows=806 width=72)
   Filter: ("заказ" IS NOT NULL)
(2 rows)

Последовательное сканирование (Seq Scan)  таблицы clients с фильтром заказ не должен быть пустым
```

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).

Остановите контейнер с PostgreSQL (но не удаляйте volumes).

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

```
Бекапимся
docker exec -t pg_devops pg_dump -U postgres test_db -f /var/lib/postgresql/data/dump_tst.sql 

docker exec -t pg_devops ls -lah /var/lib/postgresql/data/
total 140K
drwx------ 19 postgres postgres 4.0K Dec  6 12:20 .
drwxr-xr-x  3 postgres postgres 4.0K Dec  6 09:59 ..
drwx------  6 postgres postgres 4.0K Dec  6 10:35 base
-rw-r--r--  1 root     root     4.3K Dec  6 12:20 dump_tst.sql

Останавливаем контейнер
docker container stop pg_devops

Поднимаем новый контейнер
docker container run  \
--name pg_devops2 -e POSTGRES_PASSWORD=strongpass \
-e POSTGRES_USER=pgusr -p 5434:5432 -v db_1:/var/lib/postgresql/data \
-v db_2:/var/lib/postgresql -d postgres:12

root@gitlab-podman2:/home/usrcon# docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                    NAMES
a174e3573085   postgres:12   "docker-entrypoint.s…"   4 seconds ago   Up 2 seconds   0.0.0.0:5434->5432/tcp   pg_devops2

Проваливаемся в него:
docker exec -it pg_devops2 bash

Поднимаем БД
 psql -U postgres -d test_db -f /var/lib/postgresql/data/dump_tst.sql
 
 Проверяем
 root@a174e3573085:/# psql -U postgres
psql (12.13 (Debian 12.13-1.pgdg110+1))
Type "help" for help.
postgres=# \c test_db
You are now connected to database "test_db" as user "postgres".
test_db=# select * from clients where заказ is not null;
 id |       фамилия        | страна проживания | заказ
----+----------------------+-------------------+-------
  1 | Иванов Иван Иванович | USA               |     3
  2 | Петров Петр Петрович | Canada            |     4
  3 | Иоганн Себастьян Бах | Japan             |     5
(3 rows)

test_db=#
```

Наши записи восстановились. УСПЕХ!
