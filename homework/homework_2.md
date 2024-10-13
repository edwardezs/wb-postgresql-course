# Домашнее задание 2

## Условие
1. Открыть консоль и зайти по ssh на ВМ
2. Открыть вторую консоль и также зайти по ssh на ту же ВМ
3. Запустить везде psql из под пользователя postgres
4. Сделать в первой сессии новую таблицу и наполнить ее данными
5. Посмотреть текущий уровень изоляции
6. Начать новую транзакцию в обеих сессиях с дефолтным (не меняя) уровнем изоляции
7. В первой сессии добавить новую запись
8. Сделать запрос на выбор всех записей во второй сессии
9. Видите ли вы новую запись и если да то почему?
10. Завершить транзакцию в первом окне
11. Сделать запрос на выбор всех записей второй сессии
12. Видите ли вы новую запись и если да то почему?
13. Завершите транзакцию во второй сессии
14. Начать новые транзакции, но уже на уровне repeatable read в ОБЕИХ сессиях
15. В первой сессии добавить новую запись
16. Сделать запрос на выбор всех записей во второй сессии
17. Видите ли вы новую запись и если да то почему?
18. Завершить транзакцию в первом окне
19. Сделать запрос во выбор всех записей второй сессии
20. Видите ли вы новую запись и если да то почему?

## Решение
1. Зашел в WSL (Debian) - **первый сеанс**
```cmd
root@DESKTOP-GDKT3Q2:~#
```
2. Зашел в WSL (Debian) - **второй сеанс**
```cmd
root@DESKTOP-GDKT3Q2:~#
```
3. Запустил в обоих сеансах psql из под пользователя postgres
```cmd
root@DESKTOP-GDKT3Q2:~# service postgresql start // первый сеанс
Starting PostgreSQL 17 database server: main.
root@DESKTOP-GDKT3Q2:~# su postgres
postgres@DESKTOP-GDKT3Q2:/root$ psql
psql (17.0 (Debian 17.0-1.pgdg120+1))
Type "help" for help.

postgres=#
```
```cmd
root@DESKTOP-GDKT3Q2:~# su postgres // второй сеанс
postgres@DESKTOP-GDKT3Q2:/root$ psql
psql (17.0 (Debian 17.0-1.pgdg120+1))
Type "help" for help.

postgres=#
```
4. Создал в первой сессии таблицу **test_table** в схеме **test_schema** в базе **test** и заполнил ее данными
```cmd
test=# INSERT INTO test_schema.test_table (name) VALUES
('ed'), ('phil'), ('erik'), ('svyat'), ('zahar');
INSERT 0 5
test=# SELECT * FROM test_schema.test_table;
 id | name
----+-------
  1 | ed
  2 | phil
  3 | erik
  4 | svyat
  5 | zahar
(5 rows)
```
5. Посмотрел текущий уровень изоляции транзакций - **read commited**
```cmd
test=# SHOW default_transaction_isolation;
 default_transaction_isolation
-------------------------------
 read committed
(1 row)
``` 
6. Начал новые транзакции в обеих сессиях
```cmd
test=# BEGIN; // первый сеанс
BEGIN
test=*#
```
```cmd
postgres=# \c test // второй сеанс
You are now connected to database "test" as user "postgres".
test=# BEGIN;
BEGIN
```
7. В первой сессии добавил новую запись
```cmd
test=*# INSERT INTO test_schema.test_table (name) VALUES ('ivan');
INSERT 0 1
```
8. Сделал запрос на выбор всех записей во второй сессии
```cmd
test=*# SELECT * FROM test_schema.test_table;
 id | name
----+-------
  1 | ed
  2 | phil
  3 | erik
  4 | svyat
  5 | zahar
(5 rows)
```
9. Новую запись 'ivan' **не видно**, так как при дефолтном уровне изоляции (read commited) видны только **зафиксированные** изменения, а в первом сеансе мы еще не завершили транзакцию
10. Завершил транзакцию в первом окне
```cmd
test=*#COMMIT;
COMMIT
```
11. Выполнил повторный запрос на выбор всех записей во втором сеансе
```
test=*# SELECT * FROM test_schema.test_table;
 id | name
----+-------
  1 | ed
  2 | phil
  3 | erik
  4 | svyat
  5 | zahar
  6 | ivan
(6 rows)
```
12. **Увидел** новую запись, так как мы завершили первую транзакцию в п. 11 и мы работаем на уровне изоляции **read commited**
13. Завершил транзакцию во второй сессии
```cmd
 test=*# COMMIT;
COMMIT
```
14. Начал новые транзакции на уровне **repeatable read** в обоих сеансах
```cmd
test=# BEGIN ISOLATION LEVEL REPEATABLE READ; // первый сеанс
BEGIN
```
```cmd
test=# BEGIN ISOLATION LEVEL REPEATABLE READ; // второй сеанс
BEGIN
```
15. В первой сессии добавил новую запись
```cmd
test=*# INSERT INTO test_schema.test_table (name) VALUES ('vanya');
INSERT 0 1
```
16. Выполнил запрос на выбор всех записей таблицы **test_table** во второй сессии
```cmd
test=*#SELECT * FROM test_schema.test_table;
 id | name
----+-------
  1 | ed
  2 | phil
  3 | erik
  4 | svyat
  5 | zahar
  6 | ivan
(6 rows)
```
17. Новую запись 'vanya' **не видно**, так как при уровне изоляции **repeatable read** мы работаем со снимком данных и даже при завершении транзакции в первом сеансе мы ее не увидим в рамках текущей. Этот уровень считается более строгим по сравнению с **read commited**
18. Завершил транзакцию в первом окне
```cmd
test=*# COMMIT;
COMMIT
```
19. Выполнил повторный запрос на выборку всех данных во второй сессии
```cmd
test=*# SELECT * FROM test_schema.test_table;
 id | name
----+-------
  1 | ed
  2 | phil
  3 | erik
  4 | svyat
  5 | zahar
  6 | ivan
(6 rows)
```
20. Новую запись **не видно**. Объяснение в п. 17 :)
 