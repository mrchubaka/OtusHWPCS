#Домашнее задание №4

##Тюнинг Постгреса


###Развернуть Постгрес на ВМ

```
alex@alex-otus:~/Desktop$ sudo apt update
sudo apt install postgresql postgresql-contrib
[sudo] password for alex: 
Hit:1 http://ru.archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://ru.archive.ubuntu.com/ubuntu jammy-updates InRelease [119 kB]
Get:3 http://ru.archive.ubuntu.com/ubuntu jammy-backports InRelease [108 kB]
Get:4 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 DEP-11 Metadata [101 kB]
Get:5 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]    
Get:6 http://ru.archive.ubuntu.com/ubuntu jammy-updates/universe amd64 DEP-11 Metadata [269 kB]
Get:7 http://ru.archive.ubuntu.com/ubuntu jammy-updates/multiverse amd64 DEP-11 Metadata [940 B]
Get:8 http://ru.archive.ubuntu.com/ubuntu jammy-backports/main amd64 DEP-11 Metadata [7 996 B]
Get:9 http://ru.archive.ubuntu.com/ubuntu jammy-backports/universe amd64 DEP-11 Metadata [12,9 kB]
Get:10 http://security.ubuntu.com/ubuntu jammy-security/main amd64 DEP-11 Metadata [41,5 kB]
Get:11 http://security.ubuntu.com/ubuntu jammy-security/universe amd64 DEP-11 Metadata [18,5 kB]
Fetched 789 kB in 1s (603 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
1 package can be upgraded. Run 'apt list --upgradable' to see it.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  libflashrom1 libftdi1-2 libllvm13
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  libcommon-sense-perl libjson-perl libjson-xs-perl libllvm14 libpq5 libtypes-serialiser-perl postgresql-14 postgresql-client-14
  postgresql-client-common postgresql-common sysstat
Suggested packages:
  postgresql-doc postgresql-doc-14 isag
The following NEW packages will be installed:
  libcommon-sense-perl libjson-perl libjson-xs-perl libllvm14 libpq5 libtypes-serialiser-perl postgresql postgresql-14
  postgresql-client-14 postgresql-client-common postgresql-common postgresql-contrib sysstat
0 upgraded, 13 newly installed, 0 to remove and 1 not upgraded.
Need to get 42,4 MB of archives.
After this operation, 161 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 libcommon-sense-perl amd64 3.75-2build1 [21,1 kB]
Get:2 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 libjson-perl all 4.04000-1 [81,8 kB]
Get:3 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 libtypes-serialiser-perl all 1.01-1 [11,6 kB]
Get:4 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 libjson-xs-perl amd64 4.030-1build3 [87,2 kB]
Get:5 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 libllvm14 amd64 1:14.0.0-1ubuntu1 [24,0 MB]
Get:6 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 libpq5 amd64 14.7-0ubuntu0.22.04.1 [141 kB]
Get:7 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 postgresql-client-common all 238 [29,6 kB]
Get:8 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 postgresql-client-14 amd64 14.7-0ubuntu0.22.04.1 [1 220 kB]
Get:9 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 postgresql-common all 238 [169 kB]
Get:10 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 postgresql-14 amd64 14.7-0ubuntu0.22.04.1 [16,1 MB]
Get:11 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 postgresql all 14+238 [3 288 B]                                              
Get:12 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 postgresql-contrib all 14+238 [3 292 B]                                      
Get:13 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 sysstat amd64 12.5.2-2ubuntu0.1 [487 kB]                             
Fetched 42,4 MB in 8s (5 530 kB/s)                                                                                                       
Preconfiguring packages ...
Selecting previously unselected package libcommon-sense-perl:amd64.
(Reading database ... 175314 files and directories currently installed.)
Preparing to unpack .../00-libcommon-sense-perl_3.75-2build1_amd64.deb ...
Unpacking libcommon-sense-perl:amd64 (3.75-2build1) ...
Selecting previously unselected package libjson-perl.
Preparing to unpack .../01-libjson-perl_4.04000-1_all.deb ...
Unpacking libjson-perl (4.04000-1) ...
Selecting previously unselected package libtypes-serialiser-perl.
Preparing to unpack .../02-libtypes-serialiser-perl_1.01-1_all.deb ...
Unpacking libtypes-serialiser-perl (1.01-1) ...
Selecting previously unselected package libjson-xs-perl.
Preparing to unpack .../03-libjson-xs-perl_4.030-1build3_amd64.deb ...
Unpacking libjson-xs-perl (4.030-1build3) ...
Selecting previously unselected package libllvm14:amd64.
Preparing to unpack .../04-libllvm14_1%3a14.0.0-1ubuntu1_amd64.deb ...
Unpacking libllvm14:amd64 (1:14.0.0-1ubuntu1) ...
Selecting previously unselected package libpq5:amd64.
Preparing to unpack .../05-libpq5_14.7-0ubuntu0.22.04.1_amd64.deb ...
Unpacking libpq5:amd64 (14.7-0ubuntu0.22.04.1) ...
Selecting previously unselected package postgresql-client-common.
Preparing to unpack .../06-postgresql-client-common_238_all.deb ...
Unpacking postgresql-client-common (238) ...
Selecting previously unselected package postgresql-client-14.
Preparing to unpack .../07-postgresql-client-14_14.7-0ubuntu0.22.04.1_amd64.deb ...
Unpacking postgresql-client-14 (14.7-0ubuntu0.22.04.1) ...
Selecting previously unselected package postgresql-common.
Preparing to unpack .../08-postgresql-common_238_all.deb ...
Adding 'diversion of /usr/bin/pg_config to /usr/bin/pg_config.libpq-dev by postgresql-common'
Unpacking postgresql-common (238) ...
Selecting previously unselected package postgresql-14.
Preparing to unpack .../09-postgresql-14_14.7-0ubuntu0.22.04.1_amd64.deb ...
Unpacking postgresql-14 (14.7-0ubuntu0.22.04.1) ...
Selecting previously unselected package postgresql.
Preparing to unpack .../10-postgresql_14+238_all.deb ...
Unpacking postgresql (14+238) ...
Selecting previously unselected package postgresql-contrib.
Preparing to unpack .../11-postgresql-contrib_14+238_all.deb ...
Unpacking postgresql-contrib (14+238) ...
Selecting previously unselected package sysstat.
Preparing to unpack .../12-sysstat_12.5.2-2ubuntu0.1_amd64.deb ...
Unpacking sysstat (12.5.2-2ubuntu0.1) ...
Setting up postgresql-client-common (238) ...
Setting up libpq5:amd64 (14.7-0ubuntu0.22.04.1) ...
Setting up libcommon-sense-perl:amd64 (3.75-2build1) ...
Setting up postgresql-client-14 (14.7-0ubuntu0.22.04.1) ...
update-alternatives: using /usr/share/postgresql/14/man/man1/psql.1.gz to provide /usr/share/man/man1/psql.1.gz (psql.1.gz) in auto mode
Setting up libllvm14:amd64 (1:14.0.0-1ubuntu1) ...
Setting up libtypes-serialiser-perl (1.01-1) ...
Setting up libjson-perl (4.04000-1) ...
Setting up sysstat (12.5.2-2ubuntu0.1) ...

Creating config file /etc/default/sysstat with new version
update-alternatives: using /usr/bin/sar.sysstat to provide /usr/bin/sar (sar) in auto mode
Created symlink /etc/systemd/system/sysstat.service.wants/sysstat-collect.timer → /lib/systemd/system/sysstat-collect.timer.
Created symlink /etc/systemd/system/sysstat.service.wants/sysstat-summary.timer → /lib/systemd/system/sysstat-summary.timer.
Created symlink /etc/systemd/system/multi-user.target.wants/sysstat.service → /lib/systemd/system/sysstat.service.
Setting up libjson-xs-perl (4.030-1build3) ...
Setting up postgresql-common (238) ...
Adding user postgres to group ssl-cert

Creating config file /etc/postgresql-common/createcluster.conf with new version
Building PostgreSQL dictionaries from installed myspell/hunspell packages...
  en_us
Removing obsolete dictionary files:
Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /lib/systemd/system/postgresql.service.
Setting up postgresql-14 (14.7-0ubuntu0.22.04.1) ...
Creating new PostgreSQL cluster 14/main ...
/usr/lib/postgresql/14/bin/initdb -D /var/lib/postgresql/14/main --auth-local peer --auth-host scram-sha-256 --no-instructions
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locales
  COLLATE:  en_US.UTF-8
  CTYPE:    en_US.UTF-8
  MESSAGES: en_US.UTF-8
  MONETARY: ru_RU.UTF-8
  NUMERIC:  ru_RU.UTF-8
  TIME:     ru_RU.UTF-8
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

fixing permissions on existing directory /var/lib/postgresql/14/main ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Europe/Moscow
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok
update-alternatives: using /usr/share/postgresql/14/man/man1/postmaster.1.gz to provide /usr/share/man/man1/postmaster.1.gz (postmaster.1.
gz) in auto mode
Setting up postgresql-contrib (14+238) ...
Setting up postgresql (14+238) ...
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for libc-bin (2.35-0ubuntu3.1) ...
alex@alex-otus:~/Desktop$ sudo -i -u postgres 
postgres@alex-otus:~$ psql 
psql (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
Type "help" for help.

postgres=# exit
postgres@alex-otus:~$ 

```


###Протестировать pg_bench
```
postgres@alex-otus:~$ postgres createdb pgbench
Command 'postgres' not found, did you mean:
  command 'postgrey' from deb postgrey (1.36-5.2)
Try: apt install <deb name>
postgres@alex-otus:~$ psql createdb pgbench
psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  Peer authentication failed for user "pgbench"
postgres@alex-otus:~$ sudo -u postgres createdb pgbench
postgres is not in the sudoers file.  This incident will be reported.
postgres@alex-otus:~$ createdb pgbench
postgres@alex-otus:~$ 
postgres@alex-otus:~$ psql pgbench 
psql (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
Type "help" for help.

pgbench=# \q
postgres@alex-otus:~$ pgbench -i -s 10 pgbench
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
1000000 of 1000000 tuples (100%) done (elapsed 1.11 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 1.76 s (drop tables 0.00 s, create tables 0.00 s, client-side generate 1.14 s, vacuum 0.22 s, primary keys 0.40 s).
postgres@alex-otus:~$ 
postgres@alex-otus:~$ 
postgres@alex-otus:~$ pgbench -c 10 -j 2 -T 60 pgbench
pgbench (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 10
number of threads: 2
duration: 60 s
number of transactions actually processed: 285242
latency average = 2.104 ms
initial connection time = 13.243 ms
tps = 4753.240334 (without initial connection time)
postgres@alex-otus:~$
```

#####Создание базы на 2млн. записей и тестовый прогон pg_bench.


```
postgres@alex-otus:~$ pgbench -i -s 20 pgbench
dropping old tables...
creating tables...
generating data (client-side)...
2000000 of 2000000 tuples (100%) done (elapsed 2.33 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 3.66 s (drop tables 0.04 s, create tables 0.00 s, client-side generate 2.35 s, vacuum 0.34 s, primary keys 0.93 s).
postgres@alex-otus:~$ 
postgres@alex-otus:~$ 
postgres@alex-otus:~$ psql pgbench 
psql (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
Type "help" for help.

pgbench=# select count(*) from pgbench_accounts ;
  count  
---------
 2000000
(1 row)

pgbench=# 

```

```
postgres@alex-otus:~$ pgbench -c 10 -j 2 -T 60 pgbench
pgbench (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 20
query mode: simple
number of clients: 10
number of threads: 2
duration: 60 s
number of transactions actually processed: 180001
latency average = 3.333 ms
initial connection time = 31.633 ms
tps = 3110.499357 (without initial connection time)
postgres@alex-otus:~$ 
```


###Выставить оптимальные настройки
```
shared_buffers = 2GB # 25% от общего объема оперативной памяти

work_mem = 16MB 

maintenance_work_mem = 128MB # Обычно можно установить значение в 10-20% от общего объема оперативной памяти

effective_cache_size = 4GB # 50% от общего объема оперативной памяти

wal_buffers = -1 # Это автоматически устанавливает размер WAL буферов в 1/32 (около 3%) от размера shared_buffers, или 16MB, в зависимости от того, что меньше.

synchronous_commit = off #уменьшения задержки записи на диск

archive_mode = off #отключить долговременного хранения журналов транзакций (WAL archiving)

```

###Проверить насколько выросла производительность
```
postgres@alex-otus:~$ pgbench -c 10 -j 2 -T 60 pgbench
pgbench (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 20
query mode: simple
number of clients: 10
number of threads: 2
duration: 60 s
number of transactions actually processed: 195646  <<<<<<< прирост
latency average = 3.068 ms <<<<<<< прирост
initial connection time = 29.314 ms
tps = 3259.494106 (without initial connection time) <<<<<<< прирост
```

###Настроить кластер на оптимальную производительность не обращая внимания на стабильность БД
```
synchronous_commit = off #уменьшения задержки записи на диск

archive_mode = off #отключить долговременного хранения журналов транзакций (WAL archiving)

```

###Проверить насколько выросла производительность
```
postgres@alex-otus:~$ pgbench -c 10 -j 2 -T 60 pgbench
pgbench (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 20
query mode: simple
number of clients: 10
number of threads: 2
duration: 60 s
number of transactions actually processed: 204899 <<<<<<< прирост
latency average = 2.929 ms <<<<<<< прирост
initial connection time = 31.557 ms 
tps = 3413.702740 (without initial connection time) <<<<<<< прирост

```