#Домашнее задание №1

##Работа с уровнями изоляции транзакции в PostgreSQL

#####Поставить PostgreSQL из пакетов apt install

```
sudo apt update
sudo apt install postgresql postgresql-contrib
```


#####Найти вторым ssh (вторая сессия)
#####Запустить везде psql из под пользователя postgres

```
sudo -i -u postgres
psql
```

#####Выключить auto commit

_Насколько я понял [документацию](https://postgrespro.com/docs/postgresql/14/ecpg-sql-set-autocommit), в PG14 он выключен по умолчанию_
#####Сделать в первой сессии новую таблицу и наполнить ее данными

>>create table persons(id serial, first_name text, second_name text);
>>insert into persons(first_name, second_name) values('ivan', 'ivanov');
>>insert into persons(first_name, second_name) values('petr', 'petrov');
>>commit;

```
postgres=# create table persons(id serial, first_name text, second_name text);
insert into persons(first_name, second_name) values('ivan', 'ivanov');
insert into persons(first_name, second_name) values('petr', 'petrov');
CREATE TABLE
INSERT 0 1
INSERT 0 1
postgres=# commit ;
WARNING:  there is no transaction in progress
COMMIT
postgres=# 
```


#####Посмотреть текущий уровень изоляции: show transaction isolation level


```
postgres=# show transaction_isolation;
 transaction_isolation 
-----------------------
 read committed
(1 row)

```

#####Начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
#####В первой сессии добавить новую запись

>>>insert into persons(first_name, second_name) values('sergey', 'sergeev');
>>сделать select * from persons во второй сессии

Console-1
```
postgres=# begin ;
BEGIN
postgres=*# insert into persons(first_name, second_name) values('sergey', 'sergeev');
INSERT 0 1
postgres=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)

postgres=*# 
```

Console-2
```
postgres=# begin ;
BEGIN
postgres=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)

postgres=*# 

```
#####Видите ли вы новую запись и если да то почему?

_Мы не увидим новую запись, потому что во втором сеансе используется уровень изоляции READ COMMITTED по умолчанию, что означает, что он считывает только данные, зафиксированные до начала транзакции._

#####завершить первую транзакцию - commit;

```
postgres=*# commit ;
COMMIT
postgres=# 

```

#####сделать select * from persons во второй сессии

```
postgres=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)

```

#####видите ли вы новую запись и если да то почему?

_Теперь вы увидите новую запись, потому что первая транзакция была зафиксирована, и данные теперь видны другим транзакциям._

#####завершите транзакцию во второй сессии

```
postgres=*# commit ;
COMMIT
postgres=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)

postgres=# 

```

#####Начать новые но уже repeatable read транзакции - set transaction isolation level repeatable read;

```
postgres=# begin transaction isolation level repeatable read ;
BEGIN
postgres=*# 
postgres=*# show TRANSACTION ISOLATION LEVEL ;
 transaction_isolation 
-----------------------
 repeatable read
(1 row)

```

#####В первой сессии добавить новую запись

>>>insert into persons(first_name, second_name) values('sveta', 'svetova');

#####Cделать select * from persons во второй сессии

```
postgres=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)


```

#####Видите ли вы новую запись и если да то почему?

_Мы не увидим новую запись, потому что второй сеанс теперь использует ПОВТОРЯЕМЫЙ уровень изоляции чтения, что означает, что он будет видеть только данные, которые были зафиксированы до начала транзакции, обеспечивая согласованный моментальный снимок данных._

#####Завершить первую транзакцию - commit;

```
postgres=*# commit ;
COMMIT
```

#####Сделать select * from persons во второй сессии


```
postgres=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)

```

#####Видите ли вы новую запись и если да то почему?

_Мы все равно не увидим новую запись, даже после совершения первой транзакции, потому что ПОВТОРЯЕМЫЙ уровень изоляции ЧТЕНИЯ поддерживает согласованный моментальный снимок данных на протяжении всей транзакции._

#####Завершить вторую транзакцию


```

```

#####Сделать select * from persons во второй сессии


```
postgres=*# commit ;
COMMIT
postgres=# 

```

#####Видите ли вы новую запись и если да то почему?

_Теперь мы увидим новую запись, потому что вторая транзакция была завершена, и данные теперь видны новым транзакциям._