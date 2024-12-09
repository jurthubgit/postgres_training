#Условия задания

Домашнее задание
Работа с уровнями изоляции транзакции в PostgreSQL

Цель:
научиться работать с Google Cloud Platform на уровне Google Compute Engine (IaaS)
научиться управлять уровнем изоляции транзации в PostgreSQL и понимать особенность работы уровней read commited и repeatable read

Описание/Пошаговая инструкция выполнения домашнего задания:
создать новый проект в Яндекс облако или на любых ВМ, докере
далее создать инстанс виртуальной машины с дефолтными параметрами
добавить свой ssh ключ в metadata ВМ
зайти удаленным ssh (первая сессия), не забывайте про ssh-add
поставить PostgreSQL
зайти вторым ssh (вторая сессия)
запустить везде psql из под пользователя postgres
выключить auto commit
сделать

в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;

посмотреть текущий уровень изоляции: show transaction isolation level
начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
сделать select from persons во второй сессии
видите ли вы новую запись и если да то почему?
#Данные не видны, т.к. не зафиксированы. Уровень изоляции READ COMMITED
завершить первую транзакцию - commit;
сделать select from persons во второй сессии
видите ли вы новую запись и если да то почему?
#Данные видны и доступны во втором сеансе и другой транзакции, т.к. зафиксированы.
завершите транзакцию во второй сессии
начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
сделать select* from persons во второй сессии*
видите ли вы новую запись и если да то почему?
#Данные не видны, т.к. не зафиксированы. Уровень изоляции REPEATABLE READ. Во второй транзакции и сеансе видны только
#те данные, которые были зафиксированы до начала транзакции в первом сеансе. 
завершить первую транзакцию - commit;
сделать select from persons во второй сессии
видите ли вы новую запись и если да то почему?
завершить вторую транзакцию
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
#Данные видны, т.к. не зафиксированы. Уровень изоляции REPEATABLE READ. Во второй транзакции и сеансе видны, т.к. чтение происходит после фиксации данных.


-------------------------------------------------------------------------------------------------------------------------------


#Установка postgres 
```
Welcome to Ubuntu 24.04.1 LTS (GNU/Linux 6.8.0-49-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

Expanded Security Maintenance for Applications is not enabled.

3 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

Last login: Sun Dec  8 23:01:33 2024 from fe80::e192:b3c0:93e0:3969%enp0s8
ivan@otus-postgres:~$
ivan@otus-postgres:~$ sudo systemctl enable postgresql
[sudo] password for ivan:
Synchronizing state of postgresql.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable postgresql
ivan@otus-postgres:~$ sudo systemctl start postgresql
ivan@otus-postgres:~$ sudo systemctl status postgresql
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; preset: enabled)
     Active: active (exited) since Fri 2024-12-06 15:35:01 +07; 2 days ago
   Main PID: 1374 (code=exited, status=0/SUCCESS)
        CPU: 7ms

дек 06 15:35:01 otus-postgres systemd[1]: Starting postgresql.service - PostgreSQL RDBMS...
дек 06 15:35:01 otus-postgres systemd[1]: Finished postgresql.service - PostgreSQL RDBMS.
ivan@otus-postgres:~$
ivan@otus-postgres:~$ sudo pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
17  main    5432 online postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
ivan@otus-postgres:~$
ivan@otus-postgres:~$ sudo pg_ctlcluster 17 main status
pg_ctl: server is running (PID: 1309)
/usr/lib/postgresql/17/bin/postgres "-D" "/var/lib/postgresql/17/main" "-c" "config_file=/etc/postgresql/17/main/postgresql.conf"

```




-------------------------------------------------------------------------------------------------------------------------------

# Первый сеанс

```

postgres@otus-postgres:~$ psql
psql (17.2 (Ubuntu 17.2-1.pgdg24.04+1))
Type "help" for help.

postgres=# \set autocommit off;
postgres=#
postgres=# drop table persons;
DROP TABLE
postgres=# select from persons;
ERROR:  relation "persons" does not exist
LINE 1: select from persons;
                    ^
postgres=# begin;
BEGIN
postgres=*#
postgres=*# create table persons(id serial, first_name text, second_name text);
CREATE TABLE
postgres=*#
postgres=*# insert into persons(first_name, second_name) values('ivan', 'ivanov');
INSERT 0 1
postgres=*# insert into persons(first_name, second_name) values('petr', 'petrov');
INSERT 0 1
postgres=*# commit;
COMMIT
postgres=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)
```


```
postgres=#
postgres=# begin;
BEGIN
postgres=*#  insert into persons(first_name, second_name) values('sergey', 'sergeev');
INSERT 0 1
postgres=*# select from persons;
--
(3 rows)

postgres=*# commit;
COMMIT
```
```
postgres=# begin;
BEGIN
postgres=*# set transaction isolation level repeatable read;
SET
postgres=*# show transaction isolation level;
 transaction_isolation
-----------------------
 repeatable read
(1 row)

postgres=*# insert into persons(first_name, second_name) values('sveta', 'svetova');
INSERT 0 1
postgres=*# select from persons;
--
(4 rows)

postgres=*# commit;
COMMIT
postgres=# select from persons;
--
(4 rows)

postgres=# \q
postgres@otus-postgres:~$
```
-------------------------------------------------------------------------------------------------------------------------------

#Второй сеанс
```
postgres@otus-postgres:~$ psql
psql (17.2 (Ubuntu 17.2-1.pgdg24.04+1))
Type "help" for help.

postgres=# \set autocommit off;
postgres=#
postgres=# select from persons;
ERROR:  relation "persons" does not exist
LINE 1: select from persons;
                    ^
postgres=#
postgres=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)

```
```
postgres=#
postgres=# begin;
BEGIN
postgres=*# select from persons;
--
(2 rows)
```
```
postgres=*# select from persons;
--
(3 rows)

postgres=*# end;
COMMIT
```
```
postgres=#
postgres=# select from persons;
--
(3 rows)

postgres=# begin;
BEGIN
postgres=*# select from persons;
--
(3 rows)

postgres=*# select from persons;
--
(4 rows)

postgres=*# commit;
COMMIT
postgres=# select from persons;
--
(4 rows)

postgres=#
postgres=# \q

```
-------------------------------------------------------------------------------------------------------------------------------

# Пояснения

Стандартный уровень изоляции READ COMMITED. Это означает, что запрос видит только те данные, которые были зафиксированы
до начала запроса или же транзакции. Незафиксированные данные не будут доступны для второго сеанса до окончания транзакции.
(commit - фиксация данных и завершение транзакции)

Уровень изоляции REPEATABLE READ подразумевает, что видные только зафиксированные данные перед началом транзакции. Внутри
транзакции измененные данные видны до их фиксации, однако эти изменения не видны и не доступны для других транзакций. 


[Ссылка на документаци по теме "Уровень транзакции"](https://www.postgresql.org/docs/current/sql-set-transaction.html)

The isolation level of a transaction determines what data the transaction can see when other transactions are running concurrently:

READ COMMITTED

    A statement can only see rows committed before it began. This is the default.
REPEATABLE READ

    All statements of the current transaction can only see rows committed before the first query or data-modification statement was executed in this transaction.

B
B
B
B
B
B
B
B
 
