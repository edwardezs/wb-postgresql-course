# Домашнее задание 4

## Условие
1. Создать таблицу accounts(id integer, amount numeric)
2. Добавить несколько записей и подключившись через 2 терминала добиться ситуации взаимоблокировки (deadlock)
3. Посмотреть логи и убедиться, что информация о дедлоке туда попала

## Решение
1. Создал таблицу **accounts**
```cmd
test=# CREATE TABLE test_schema.accounts(id integer, amount numeric);
CREATE TABLE
test=# \dt test_schema.*
              List of relations
   Schema    |    Name    | Type  |  Owner
-------------+------------+-------+----------
 test_schema | accounts   | table | postgres
 test_schema | test_table | table | postgres
(2 rows)
```
2. Добавил несколько записей. Далее добился взаимоблокировки, начав две транзакции, по одной в каждом сеансе. Одна блокирует первую строку таблицы и пытается заблокировать вторую, другая транзакция - наоборот. В результате, обе транзакции ждут друг друга
```cmd
test=# INSERT INTO test_schema.accounts (id, amount) VALUES (1, 10), (2, 20), (3, 30);
INSERT 0 3
test=# SELECT * FROM test_schema.accounts;
 id | amount
----+--------
  1 |     10
  2 |     20
  3 |     30
(3 rows)
```
```cmd
test=# BEGIN; // первый сеанс
BEGIN
test=*# SELECT * FROM test_schema.accounts WHERE id=1 FOR UPDATE;
 id | amount
----+--------
  1 |     10
(1 row)

test=*# SELECT * FROM test_schema.accounts WHERE id=2 FOR UPDATE;
 id | amount
----+--------
  2 |     20
(1 row)
```
```cmd
test=# BEGIN; // второй сеанс
BEGIN
test=*# SELECT * FROM test_schema.accounts WHERE id=2 FOR UPDATE;
 id | amount
----+--------
  2 |     20
(1 row)

test=*# SELECT * FROM test_schema.accounts WHERE id=1 FOR UPDATE;
ERROR:  deadlock detected
DETAIL:  Process 58 waits for ShareLock on transaction 920; blocked by process 53.
Process 53 waits for ShareLock on transaction 921; blocked by process 58.
HINT:  See server log for query details.
CONTEXT:  while locking tuple (0,1) in relation "accounts"
```
3. Просмотрел логи и убедился, что соответствующая информация появилась
```cmd
root@DESKTOP-GDKT3Q2:/var/log/postgresql# cat  postgresql-17-main.log

2024-10-13 18:30:02.671 MSK [58] postgres@test ERROR:  deadlock detected // фрагмент логов
2024-10-13 18:30:02.671 MSK [58] postgres@test DETAIL:  Process 58 waits for ShareLock on transaction 920; blocked by process 53.
        Process 53 waits for ShareLock on transaction 921; blocked by process 58.
        Process 58: SELECT * FROM test_schema.accounts WHERE id=1 FOR UPDATE;
        Process 53: SELECT * FROM test_schema.accounts WHERE id=2 FOR UPDATE;
2024-10-13 18:30:02.671 MSK [58] postgres@test HINT:  See server log for query details.
2024-10-13 18:30:02.671 MSK [58] postgres@test CONTEXT:  while locking tuple (0,1) in relation "accounts"
```
