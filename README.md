# 6_4
## 1
Найдите и приведите управляющие команды для:

вывода списка БД -- `\l[+]`  
подключения к БД -- `\c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}`  
вывода списка таблиц -- `\dt[S+] [PATTERN]`  
вывода описания содержимого таблиц -- `\d[S+]  NAME` где name - имя таблицы  
выхода из psql -- `\q`  
## 2
Команда `test_database=# SELECT tablename, attname, avg_width from pg_stats WHERE tablename = 'orders' ORDER BY avg_width DESC LIMIT 1`  
Результат:  
```
tablename | attname | avg_width 
-----------+---------+-----------
 orders    | title   |        16
(1 row)
```
## 3
Создадим 2 наследуемые таблицы с критериями отбора и 2 правила, чтобы при вставке в основную таблицу данные вставлялись в нужную новую таблицу   
```
test_database=# CREATE TABLE orders_more_499 ( CHECK (price > 499)) INHERITS (orders);
CREATE TABLE

test_database=# CREATE TABLE orders_le_499 ( CHECK (price <=499 )) INHERITS (orders);
CREATE TABLE
test_database=# CREATE RULE rule_orders_more_499 AS ON INSERT TO orders WHERE (price > 499) DO INSTEAD INSERT INTO orders_more_499 VALUES (NEW.*);
CREATE RULE
test_database=# CREATE RULE rule_orders_le_499 AS ON INSERT TO orders WHERE (price <= 499) DO INSTEAD INSERT INTO orders_le_499 VALUES (NEW.*);
CREATE RULE
```
Вставил тестовые данные в таюлицу orders. Всё работает!
```
test_database=# select * from orders_more_499;
 id | title | price 
----+-------+-------
  9 | qqqq  |   505
(1 row)

test_database=# select * from orders_le_499;
 id | title | price 
----+-------+-------
 10 | rrrrr |   400
(1 row)

test_database=# select * from orders;
 id |        title         | price 
----+----------------------+-------
  1 | War and peace        |   100
  2 | My little database   |   500
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
  6 | WAL never lies       |   900
  7 | Me and my bash-pet   |   499
  8 | Dbiezdmin            |   501
  9 | qqqq                 |   505
 10 | rrrrr                |   400
(10 rows)

test_database=# 
```
Если бы об этом подумали при создании таблиц, тогда все существующие данные были бы в новых таблицах, а не только новые, как сейчас.
## 4
Добавил бы условие при создании таблицы `CONSTRAINT orders_un UNIQUE (title)`
