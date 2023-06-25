#Домашнее задание №8

##Вариант 2 
###Introducing pg_auto_failover: Open source extension for automated

###Кластер состоит из 3-м машин: pgm для монитора, pg1 как мастер и pg2 как реплика

###Устанавливаем pg_auto_failover и переменные окружения на каждой ноде
```
sudo apt-get update
sudo apt-get install postgresql-14 -y
sudo apt-get install postgresql-14-auto-failover -y

sudo su - postgres 
mkdir -p 14/autofail
export PGDATA="/var/lib/postgresql/14/autofail"

postgres@pgm:~$ vi .profile
export PGDATA="/var/lib/postgresql/14/autofail"
```

###Прописать расширение pgautofailover 
```
В postgresql.conf прописать значение shared_preload_libraries = 'pgautofailover'.
```

###Созджать монитор:
```
postgres@pgm:~$ pg_autoctl create monitor --auth trust --ssl-mode require --ssl-self-signed --pgport 6000 --hostname pgm
13:26:22 5874 INFO  Using --ssl-self-signed: pg_autoctl will create self-signed certificates, allowing for encrypted network traffic
13:26:22 5874 WARN  Self-signed certificates provide protection against eavesdropping; this setup does NOT protect against Man-In-The-Middle attacks nor Impersonation attacks.
13:26:22 5874 WARN  See https://www.postgresql.org/docs/current/libpq-ssl.html for details
13:26:22 5874 WARN  Failed to find pg_ctl command in your PATH
13:26:22 5874 INFO   /usr/bin/openssl req -new -x509 -days 365 -nodes -text -out /var/lib/postgresql/14/autofail/server.crt -keyout /var/lib/postgresql/14/autofail/server.key -subj "/CN=pgm"
13:26:23 5874 INFO  Started pg_autoctl postgres service with pid 5882
13:26:23 5874 INFO  Started pg_autoctl monitor-init service with pid 5883
13:26:23 5882 INFO   /usr/bin/pg_autoctl do service postgres --pgdata /var/lib/postgresql/14/autofail -v
13:26:23 5887 INFO   /usr/lib/postgresql/14/bin/postgres -D /var/lib/postgresql/14/autofail -p 6000 -h *
13:26:23 5883 INFO  The user "autoctl" already exists, skipping.
13:26:23 5882 INFO  Postgres is now serving PGDATA "/var/lib/postgresql/14/autofail" on port 6000 with pid 5887
13:26:23 5883 INFO  The database "pg_auto_failover" already exists, skipping.
13:26:23 5883 WARN  NOTICE:  extension "pgautofailover" already exists, skipping
13:26:23 5883 INFO  Granting connection privileges on 192.168.200.0/24
13:26:23 5883 INFO  Your pg_auto_failover monitor instance is now ready on port 6000.
13:26:23 5883 INFO  Monitor has been successfully initialized.
13:26:23 5874 WARN  pg_autoctl service monitor-init exited with exit status 0
13:26:23 5882 INFO  Postgres controller service received signal SIGTERM, terminating
13:26:23 5882 INFO  Stopping pg_autoctl postgres service
13:26:23 5882 INFO  /usr/lib/postgresql/14/bin/pg_ctl --pgdata /var/lib/postgresql/14/autofail --wait stop --mode fast
13:26:23 5874 INFO  Stop pg_autoctl
postgres@pgm:~$ 

```

###Прописать пользователя и пароль ему
```
postgres@pgm:~$ psql -c "CREATE USER autoctl_node"
postgres@pgm:~$ psql -c "ALTER USER autoctl_node PASSWORD 'qweasd98'"

postgres@postgres-backup:~$ psql -c "alter user autoctl_node password 'qweasd98'"

```

###Прописать сервис
```
postgres@pgm:~$ pg_autoctl show systemd
13:27:49 5917 INFO  HINT: to complete a systemd integration, run the following commands (as root):
13:27:49 5917 INFO  pg_autoctl -q show systemd --pgdata "/var/lib/postgresql/14/autofail" | tee /etc/systemd/system/pgautofailover.service
13:27:49 5917 INFO  systemctl daemon-reload
13:27:49 5917 INFO  systemctl enable pgautofailover
13:27:49 5917 INFO  systemctl start pgautofailover
[Unit]
Description = pg_auto_failover

[Service]
WorkingDirectory = /var/lib/postgresql
Environment = 'PGDATA=/var/lib/postgresql/14/autofail'
User = postgres
ExecStart = /usr/bin/pg_autoctl run
Restart = always
StartLimitBurst = 0
ExecReload = /usr/bin/pg_autoctl reload

[Install]
WantedBy = multi-user.target
postgres@pgm:~$
```


###Создадим и добавим сервер в кластер
```
postgres@pg1:~$ pg_createcluster -u postgres -p 5432 -d /var/lib/postgresql/14/alexdb --start-conf=manual -l /var/log/postgresql/alexdb.log 14 alexdb
Configuring already existing cluster (configuration: /etc/postgresql/14/alexdb, data: /var/lib/postgresql/14/alexdb, owner: 129:137)
Ver Cluster Port Status Owner    Data directory                Log file
14  alexdb  5432 down   postgres /var/lib/postgresql/14/alexdb /var/log/postgresql/alexdb.log
postgres@pg1:~$ pg_dropcluster 14 alexdb
postgres@pg1:~$ pg_createcluster -u postgres -p 5432 -d /var/lib/postgresql/14/alexdb --start-conf=manual -l /var/log/postgresql/alexdb.log 14 alexdb
Creating new PostgreSQL cluster 14/alexdb ...
/usr/lib/postgresql/14/bin/initdb -D /var/lib/postgresql/14/alexdb --auth-local peer --auth-host scram-sha-256 --no-instructions
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

fixing permissions on existing directory /var/lib/postgresql/14/alexdb ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Europe/Moscow
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok
Ver Cluster Port Status Owner    Data directory                Log file
14  alexdb  5432 down   postgres /var/lib/postgresql/14/alexdb /var/log/postgresql/alexdb.log
postgres@pg1:~$ 
postgres@pg1:~$ 
postgres@pg1:~$ 
postgres@pg1:~$ pg_autoctl create postgres --hostname pg1 --name pg1 --auth trust --ssl-self-signed --monitor 'postgres://autoctl_node:qweasd98@pgm:6000/pg_auto_failover?sslmode=require'
postgres@pg1:~$ 

```

###Проверим на ноде монитора pgm
```
postgres@pgm:~$ pg_autoctl show state
Name |  Node |  Host:Port |       TLI: LSN |   Connection |      Reported State |      Assigned State
-----+-------+------------+----------------+--------------+---------------------+--------------------
 pg1 |     1 |   pg1:5432 |   1: 0/1749BD0 | read-write   |              single |              single

postgres@pgm:~$ 

```


###Добавим вторую ноду PG2 в кластер
```
alex@pg2:~/Desktop$ sudo su - postgres 
[sudo] password for alex: 
postgres@pg2:~$ export PGDATA="/var/lib/postgresql/14/alexdb"
postgres@pg2:~$ nano .profile
postgres@pg2:~$ 
postgres@pg2:~$ pg_autoctl create postgres --hostname pg2 --name pg2 --auth trust --ssl-self-signed --monitor 'postgres://autoctl_node:qweasd98@pgm:6000/pg_auto_failover?sslmode=require'

```


###Проверим на ноде монитора pgm
```
postgres@pgm:~$ pg_autoctl show state
Name |  Node |  Host:Port |       TLI: LSN |   Connection |      Reported State |      Assigned State
-----+-------+------------+----------------+--------------+---------------------+--------------------
 pg1 |     1 |   pg1:5432 |   1: 0/1749BD0 | read-write   |             primary |             primary 
 pg2 |     2 |   pg2:5432 |   1: 0/1749BD0 | read-only    |           secondary |           secondary
```

###Выполняю failover:
```
postgres@pgm:~$ psql -p 6000 postgres://autoctl_node@localhost:6000/pg_auto_failover
psql (14.8 (Ubuntu 14.8-0ubuntu0.22.04.1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

pg_auto_failover=> select pgautofailover.perform_failover();

```
###Проверим на ноде монитора pgm
```
postgres@pgm:~$ pg_autoctl show state
Name |  Node |  Host:Port |       TLI: LSN |   Connection |      Reported State |      Assigned State
-----+-------+------------+----------------+--------------+---------------------+--------------------
 pg1 |     1 |   pg1:5432 |   1: 0/1749BD0 | read-write   |           secondary |           secondary
 pg2 |     2 |   pg2:5432 |   1: 0/1749BD0 | read-only    |             primary |             primary 
```


###Убрать ноду из кластера
```
postgres@pg2:~$ pg_autoctl drop node --destroy

```

###Проверим на ноде монитора pgm
```
postgres@pgm:~$ pg_autoctl show state
Name |  Node |  Host:Port |       TLI: LSN |   Connection |      Reported State |      Assigned State
-----+-------+------------+----------------+--------------+---------------------+--------------------
 pg1 |     1 |   pg1:5432 |   1: 0/1749BD0 | read-write   |              single |              single

postgres@pgm:~$ 

```
