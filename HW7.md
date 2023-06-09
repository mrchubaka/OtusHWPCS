#Домашнее задание №7

##Разворачиваем и настраиваем БД с большими данными


###Создаем внешнюю таблицу для taxi.csv
```
vehicles=# SET log_duration = on;
SET
vehicles=# 

vehicles=# create foreign table taxi_trips_fdw_2 (
unique_key text, 
taxi_id text, 
trip_start_timestamp TIMESTAMP, 
trip_end_timestamp TIMESTAMP, 
trip_seconds bigint, 
trip_miles numeric, 
pickup_census_tract bigint, 
dropoff_census_tract bigint, 
pickup_community_area bigint, 
dropoff_community_area bigint, 
fare numeric, 
tips numeric, 
tolls numeric, 
extras numeric, 
trip_total numeric, 
payment_type text, 
company text, 
pickup_latitude numeric, 
pickup_longitude numeric, 
pickup_location text, 
dropoff_latitude numeric, 
dropoff_longitude numeric, 
dropoff_location text
)
server pgcsv
options(filename '/etc/taxi_big.csv', format 'csv', header 'true', delimiter ',');
CREATE FOREIGN TABLE
vehicles=# 
vehicles=# select count(*) from taxi_trips_fdw_2;
  count   
----------
 26753683
(1 row)

vehicles=# 


```

###Duration postgresql 14

```
Запрос Count 100655.154 ms

2023-06-08 13:02:51.708 MSK [2448] postgres@vehicles LOG:  duration: 100655.154 ms

```

###Загрузка того же сета в postgres (Greenplum Database) 6.19.0 мз 8 сегментов на 4-х серверах
```
20230607:16:38:39:002068 gpstart:gp61m1:gpadmin-[INFO]:-----------------------------------------------------
20230607:16:38:39:002068 gpstart:gp61m1:gpadmin-[INFO]:-   Successful segment starts                                            = 8
20230607:16:38:39:002068 gpstart:gp61m1:gpadmin-[INFO]:-   Failed segment starts                                                = 0
20230607:16:38:39:002068 gpstart:gp61m1:gpadmin-[INFO]:-   Skipped segment starts (segments are marked down in configuration)   = 0
20230607:16:38:39:002068 gpstart:gp61m1:gpadmin-[INFO]:-----------------------------------------------------
20230607:16:38:39:002068 gpstart:gp61m1:gpadmin-[INFO]:-Successfully started 8 of 8 segment instances
20230607:16:38:39:002068 gpstart:gp61m1:gpadmin-[INFO]:-----------------------------------------------------

Через file_fdw в GP не получилось, поскольку в GP штатно есть только postgres_fdw в итоге использовал COPY.


```

###Duration Greenplum
```
Запрос Count 7701.618 ms

2023-06-08 15:09:06.515789 MSK,"gpadmin","alex",p6005,th615483520,"[local]",,2023-06-08 14:58:54 MSK,0,con15,cmd23,seg-1,,,,,"LOG","00000","duration: 7701.618 ms",,,,,,"select count(*) FROM taxi_trips_fdw_2 ;",0,,"postgres.c",1915,

```


###Использование postgres_fdw в Greenplum

```
CREATE SERVER foreign_postgres_server
    FOREIGN DATA WRAPPER postgres_fdw
    OPTIONS (
        host '192.168.200.80', 
        port '5432', 
        dbname 'vehicles'
    );
	
	
CREATE USER MAPPING FOR CURRENT_USER
    SERVER foreign_postgres_server
    OPTIONS (
        user 'postgres', 
        password 'alex'
    );

alex=# CREATE FOREIGN TABLE taxi_trips_fdw_3 (
    unique_key text,
    taxi_id text,
    trip_start_timestamp TIMESTAMP,
    trip_end_timestamp TIMESTAMP,
    trip_seconds bigint,
    trip_miles numeric,
    pickup_census_tract bigint,
    dropoff_census_tract bigint,
    pickup_community_area bigint,
    dropoff_community_area bigint,
    fare numeric,
    tips numeric,
    tolls numeric,
    extras numeric,
    trip_total numeric,
    payment_type text,
    company text,
    pickup_latitude numeric,
    pickup_longitude numeric,
    pickup_location text,
    dropoff_latitude numeric,
    dropoff_longitude numeric,
    dropoff_location text
)
SERVER foreign_postgres_server
OPTIONS (
    table_name 'taxi_trips_fdw_2'
);

alex=# select count(*) from taxi_trips_fdw_3;
  count
----------
 26753683
(1 row)

alex=#
```

###Duration Greenplum через postgres_fdw 
```

157073.856 ms

2023-06-08 16:44:42.594152 MSK,"gpadmin","alex",p9050,th615483520,"[local]",,2023-06-08 16:41:55 MSK,0,con24,cmd3,seg-1,,,,,"LOG","00000","duration: 157073.856 ms",,,,,,"select count(*) from taxi_trips_fdw_3;",0,,"postgres.c",1915,


```

###Использование pgload в postgres и Duration postgres через pgload 

```

alex@etcd1:/etc$ time pgloader pgload.load 
2023-06-08T14:19:41.012000Z LOG pgloader version "3.6.3~devel"
2023-06-08T14:19:41.016000Z LOG Data errors in '/tmp/pgloader/'
2023-06-08T14:19:41.016000Z LOG Parsing commands from file #P"/etc/pgload.load"
2023-06-08T15:02:15.835095Z LOG report summary reset
                 table name     errors       rows      bytes      total time
---------------------------  ---------  ---------  ---------  --------------
                      fetch          0          0                     0.004s
---------------------------  ---------  ---------  ---------  --------------
"public"."taxi_trips_fdw_4"          0   26753684     1.0 GB      42m34.615s
---------------------------  ---------  ---------  ---------  --------------
            Files Processed          0          1                     0.016s
    COPY Threads Completion          0          2                 42m34.615s
---------------------------  ---------  ---------  ---------  --------------
          Total import time          ✓   26753684     1.0 GB      42m34.631s

real	42m35,322s
user	10m29,076s
sys	35m47,367s
alex@etcd1:/etc$ 


```