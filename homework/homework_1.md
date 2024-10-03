# Домашнее задание 1

## Условие
1. Развернуть ВМ (Linux) с PostgreSQL
2. Залить [Тайские перевозки](https://github.com/aeuge/postgres16book/tree/main/database)
3. Посчитать количество поездок - select count(*) from book.tickets

## Решение
1. Установил postgres в **WSL (Debian)**
```cmd
root@DESKTOP-GDKT3Q2:~# service postgresql start
Starting PostgreSQL 17 database server: main.
root@DESKTOP-GDKT3Q2:~# su postgres
postgres@DESKTOP-GDKT3Q2:/root$ psql
psql (17.0 (Debian 17.0-1.pgdg120+1))
Type "help" for help.

postgres=#
```
2. Залил файл с данными о Тайских перевозках - **(файл 600мб)**
```cmd
wget https://storage.googleapis.com/thaibus/thai_small.tar.gz && tar -xf thai_small.tar.gz && psql < thai.sql
```
```cmd
postgres=# \c thai
You are now connected to database "thai" as user "postgres".
thai=# \dt book.*
            List of relations
 Schema |     Name     | Type  |  Owner
--------+--------------+-------+----------
 book   | bus          | table | postgres
 book   | busroute     | table | postgres
 book   | busstation   | table | postgres
 book   | fam          | table | postgres
 book   | nam          | table | postgres
 book   | ride         | table | postgres
 book   | schedule     | table | postgres
 book   | seat         | table | postgres
 book   | seatcategory | table | postgres
 book   | tickets      | table | postgres
(10 rows)

thai=#
```
3. Подсчитал количество поездок - **5185505**
```cmd
thai=# \c thai
You are now connected to database "thai" as user "postgres".
thai=# select count(*) from book.tickets;
  count
---------
 5185505
(1 row)
```