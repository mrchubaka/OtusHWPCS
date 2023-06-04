#Домашнее задание №6

##Развернуть HA кластер

####Схема стенда: 8 VM:

```
etcd1
etcd2
etcd3
pg1
pg2
pg3
ha1
ha2
```

####Внутренний интерконнет идет через отдельный интерфейс по сети 192.168.200.8x:

```
3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:74:97:47 brd ff:ff:ff:ff:ff:ff
    altname enp2s5
    inet 192.168.200.85/24 brd 192.168.200.255 scope global noprefixroute ens37
       valid_lft forever preferred_lft forever
    inet6 fe80::2707:7e21:4469:d83e/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
alex@pg3:~/Desktop$ 
```

####Вот таким образом сэмулировал работу DNS:

```
alex@pg3:~/Desktop$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	pg3

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
192.168.200.80	etcd1
192.168.200.81	etcd2
192.168.200.82	etcd3
192.168.200.83	pg1
192.168.200.84	pg2
192.168.200.85	pg3
192.168.200.86	ha1
192.168.200.87	ha2
alex@pg3:~/Desktop$ 
```

####Развернул etcd на трех нодах etcdX:

```
alex@etcd1:~/Desktop$ etcdctl cluster-health
member 9a1f33941721f94d is healthy: got healthy result from http://etcd1:2379
member 9df0146dd9068bd2 is healthy: got healthy result from http://etcd3:2379
member f2aeb69aaf7ffcbf is healthy: got healthy result from http://etcd2:2379
cluster is healthy
alex@etcd1:~/Desktop$ 
```

####Развернул postgres и patroni на трех нодах pgX:

```
alex@pg1:~/Desktop$ sudo patronictl -c /etc/patroni.yml list 
[sudo] password for alex: 
+ Cluster: patroni -------+---------+---------+----+-----------+
| Member | Host           | Role    | State   | TL | Lag in MB |
+--------+----------------+---------+---------+----+-----------+
| pg1    | 192.168.200.83 | Leader  | running |  3 |           |
| pg2    | 192.168.200.84 | Replica | running |  3 |         0 |
| pg3    | 192.168.200.85 | Replica | running |  3 |         0 |
+--------+----------------+---------+---------+----+-----------+
alex@pg1:~/Desktop$ 

alex@pg1:~/Desktop$ sudo systemctl status pgbouncer 
● pgbouncer.service - connection pooler for PostgreSQL
     Loaded: loaded (/lib/systemd/system/pgbouncer.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-06-04 11:40:15 MSK; 32min ago
       Docs: man:pgbouncer(1)
             https://www.pgbouncer.org/
   Main PID: 4587 (pgbouncer)
     Status: "stats: 0 xacts/s, 0 queries/s, in 0 B/s, out 0 B/s, xact 0 μs, query 0 μs, wait 0 μs"
      Tasks: 2 (limit: 2235)
     Memory: 2.8M
        CPU: 3.198s
     CGroup: /system.slice/pgbouncer.service
             └─4587 /usr/sbin/pgbouncer /etc/pgbouncer/pgbouncer.ini

alex@pg2:~/Desktop$ sudo systemctl status pgbouncer
● pgbouncer.service - connection pooler for PostgreSQL
     Loaded: loaded (/lib/systemd/system/pgbouncer.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-06-04 11:50:51 MSK; 22min ago
       Docs: man:pgbouncer(1)
             https://www.pgbouncer.org/
   Main PID: 4019 (pgbouncer)
     Status: "stats: 0 xacts/s, 0 queries/s, in 0 B/s, out 0 B/s, xact 0 μs, query 0 μs, wait 0 μs"
      Tasks: 2 (limit: 2235)
     Memory: 3.7M
        CPU: 446ms
     CGroup: /system.slice/pgbouncer.service
             └─4019 /usr/sbin/pgbouncer /etc/pgbouncer/pgbouncer.ini

alex@pg3:~/Desktop$ sudo systemctl status pgbouncer
● pgbouncer.service - connection pooler for PostgreSQL
     Loaded: loaded (/lib/systemd/system/pgbouncer.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-06-04 11:51:08 MSK; 22min ago
       Docs: man:pgbouncer(1)
             https://www.pgbouncer.org/
   Main PID: 3041 (pgbouncer)
     Status: "stats: 0 xacts/s, 0 queries/s, in 0 B/s, out 0 B/s, xact 0 μs, query 0 μs, wait 0 μs"
      Tasks: 2 (limit: 2235)
     Memory: 3.2M
        CPU: 463ms
     CGroup: /system.slice/pgbouncer.service
             └─3041 /usr/sbin/pgbouncer /etc/pgbouncer/pgbouncer.ini
```

####Тесты через pg_bench:

```
alex@pg3:~/Desktop$ sudo -u postgres pgbench -p 6432 -i -d otus -h 192.168.200.83

alex@pg3:~/Desktop$ sudo -u postgres pgbench -p 6432 -c 20 -C -T 20 -P 1 -d otus -h 192.168.200.83^C
alex@pg3:~/Desktop$ 

pgbouncer=# SHOW STATS_TOTALS;
 database  | xact_count | query_count | bytes_received | bytes_sent | xact_time | query_time | wait_time 
-----------+------------+-------------+----------------+------------+-----------+------------+-----------
 otus      |       1643 |       11395 |        2277849 |     451834 | 382171269 |  337069416 |    194691
 pgbouncer |          1 |           1 |              0 |          0 |         0 |          0 |         0
(2 rows)

pgbouncer=# 

pgbouncer=# show servers;

type |   user   | database | state  |   addr    | port | local_addr | local_port |      connect_time       |      request_time       | wait | wait_us | close_needed |      ptr       |      link      | remote_pid | tls | application_name 
------+----------+----------+--------+-----------+------+------------+------------+-------------------------+-------------------------+------+---------+--------------+----------------+----------------+------------+-----+------------------
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      56990 | 2023-06-04 12:23:30 MSK | 2023-06-04 12:23:41 MSK |    0 |       0 |            0 | 0x55ada1cf4860 | 0x55ada1cee8b0 |       4823 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      57010 | 2023-06-04 12:23:30 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf4ac0 | 0x55ada1ced5b0 |       4826 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      57018 | 2023-06-04 12:23:31 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf5b60 | 0x55ada1ced810 |       4828 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      56924 | 2023-06-04 12:23:30 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf4d20 | 0x55ada1cee3f0 |       4817 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      57034 | 2023-06-04 12:23:31 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf64e0 | 0x55ada1ceefd0 |       4829 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      56916 | 2023-06-04 12:23:30 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf4600 | 0x55ada1ced0f0 |       4816 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      57014 | 2023-06-04 12:23:30 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf6e60 | 0x55ada1cf0070 |       4827 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      56996 | 2023-06-04 12:23:30 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf6740 | 0x55ada1cef230 |       4825 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      56952 | 2023-06-04 12:23:30 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf6020 | 0x55ada1cee650 |       4819 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      56888 | 2023-06-04 12:23:30 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf5900 | 0x55ada1cee190 |       4813 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      56902 | 2023-06-04 12:23:30 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf6280 | 0x55ada1cef490 |       4815 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      57072 | 2023-06-04 12:23:31 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf56a0 | 0x55ada1cef950 |       4832 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      56938 | 2023-06-04 12:23:30 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf69a0 | 0x55ada1ceed70 |       4818 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      57048 | 2023-06-04 12:23:31 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf6c00 | 0x55ada1cef6f0 |       4830 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      56974 | 2023-06-04 12:23:30 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf5440 | 0x55ada1cedf30 |       4822 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      56900 | 2023-06-04 12:23:30 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf70c0 | 0x55ada1cefbb0 |       4814 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      56966 | 2023-06-04 12:23:30 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf7320 | 0x55ada1ceeb10 |       4821 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      57062 | 2023-06-04 12:23:31 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf5dc0 | 0x55ada1cefe10 |       4831 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      56994 | 2023-06-04 12:23:30 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf51e0 | 0x55ada1cedcd0 |       4824 |     | pgbench
 S    | postgres | otus     | active | 127.0.0.1 | 5432 | 127.0.0.1  |      56960 | 2023-06-04 12:23:30 MSK | 2023-06-04 12:23:42 MSK |    0 |       0 |            0 | 0x55ada1cf4f80 | 0x55ada1ceda70 |       4820 |     | pgbench
(20 rows)
```

####Установил HAProxy на 2х нодах haX:

```
alex@ha1:~/Desktop$ sudo systemctl status haproxy.service
● haproxy.service - HAProxy Load Balancer
     Loaded: loaded (/lib/systemd/system/haproxy.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-06-04 13:03:38 MSK; 1s ago
       Docs: man:haproxy(1)
             file:/usr/share/doc/haproxy/configuration.txt.gz
   Main PID: 2049 (haproxy)
      Tasks: 3 (limit: 2235)
     Memory: 38.0M
        CPU: 84ms
     CGroup: /system.slice/haproxy.service
             ├─2049 /usr/sbin/haproxy -x /run/haproxy/admin.sock -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock
             └─2052 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock

июн 04 13:03:38 ha1 systemd[1]: Starting HAProxy Load Balancer...
июн 04 13:03:38 ha1 haproxy[2049]: [NOTICE]   (2049) : haproxy version is 2.5.14-1ppa1~jammy
июн 04 13:03:38 ha1 haproxy[2049]: [NOTICE]   (2049) : path to executable is /usr/sbin/haproxy
июн 04 13:03:38 ha1 haproxy[2049]: [WARNING]  (2049) : config : parsing [/etc/haproxy/haproxy.cfg:23] : 'option httplog' not usable with proxy 'postgres_write' (needs 'mode htt>
июн 04 13:03:38 ha1 haproxy[2049]: [WARNING]  (2049) : config : parsing [/etc/haproxy/haproxy.cfg:23] : 'option httplog' not usable with proxy 'postgres_read' (needs 'mode http>
июн 04 13:03:38 ha1 haproxy[2049]: [WARNING]  (2049) : config : proxy 'postgres_read' uses http-check rules without 'option httpchk', so the rules are ignored.
июн 04 13:03:38 ha1 haproxy[2049]: [NOTICE]   (2049) : New worker (2052) forked
июн 04 13:03:38 ha1 haproxy[2049]: [NOTICE]   (2049) : Loading success.
июн 04 13:03:38 ha1 systemd[1]: Started HAProxy Load Balancer.
alex@ha1:~/Desktop$ 

alex@ha2:~/Desktop$ sudo systemctl status haproxy.service
[sudo] password for alex: 
● haproxy.service - HAProxy Load Balancer
     Loaded: loaded (/lib/systemd/system/haproxy.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-06-04 13:43:35 MSK; 1min 25s ago
       Docs: man:haproxy(1)
             file:/usr/share/doc/haproxy/configuration.txt.gz
   Main PID: 1022 (haproxy)
      Tasks: 3 (limit: 2235)
     Memory: 10.9M
        CPU: 150ms
     CGroup: /system.slice/haproxy.service
             ├─1022 /usr/sbin/haproxy -x /run/haproxy/admin.sock -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock
             └─1115 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock

июн 04 13:43:34 ha2 haproxy[1022]: [WARNING]  (1022) : config : proxy 'postgres_read' uses http-check rules without 'option httpchk', so the rules are ignored.
июн 04 13:43:35 ha2 haproxy[1022]: [NOTICE]   (1022) : New worker (1115) forked
июн 04 13:43:35 ha2 haproxy[1022]: [NOTICE]   (1022) : Loading success.
июн 04 13:43:35 ha2 systemd[1]: Started HAProxy Load Balancer.
июн 04 13:43:35 ha2 haproxy[1115]: [WARNING]  (1115) : Server postgres_write/pg1 is DOWN, reason: Layer7 wrong status, code: 503, info: "Service Unavailable", che>
июн 04 13:43:35 ha2 haproxy[1115]: Server postgres_write/pg1 is DOWN, reason: Layer7 wrong status, code: 503, info: "Service Unavailable", check duration: 5ms. 2 >
июн 04 13:43:35 ha2 haproxy[1115]: Server postgres_write/pg1 is DOWN, reason: Layer7 wrong status, code: 503, info: "Service Unavailable", check duration: 5ms. 2 >
июн 04 13:43:36 ha2 haproxy[1115]: [WARNING]  (1115) : Server postgres_write/pg2 is DOWN, reason: Layer7 wrong status, code: 503, info: "Service Unavailable", che>
июн 04 13:43:36 ha2 haproxy[1115]: Server postgres_write/pg2 is DOWN, reason: Layer7 wrong status, code: 503, info: "Service Unavailable", check duration: 9ms. 1 >
июн 04 13:43:36 ha2 haproxy[1115]: Server postgres_write/pg2 is DOWN, reason: Layer7 wrong status, code: 503, info: "Service Unavailable", check duration: 9ms. 1 >
alex@ha2:~/Desktop$ 

```

####Тестирование отказоустойчивости при переключении мастера в patroni:

```
alex@pg1:~/Desktop$ patronictl -c /etc/patroni.yml switchover
Current cluster topology
+ Cluster: patroni -------+---------+---------+----+-----------+
| Member | Host           | Role    | State   | TL | Lag in MB |
+--------+----------------+---------+---------+----+-----------+
| pg1    | 192.168.200.83 | Leader  | running |  3 |           |
| pg2    | 192.168.200.84 | Replica | running |  3 |         0 |
| pg3    | 192.168.200.85 | Replica | running |  3 |         0 |
+--------+----------------+---------+---------+----+-----------+
Primary [pg1]: 
Candidate ['pg2', 'pg3'] []: pg3
When should the switchover take place (e.g. 2023-06-04T14:29 )  [now]: 
Are you sure you want to switchover cluster patroni, demoting current leader pg1? [y/N]: y
2023-06-04 13:29:57.23413 Successfully switched over to "pg3"
+ Cluster: patroni -------+---------+---------+----+-----------+
| Member | Host           | Role    | State   | TL | Lag in MB |
+--------+----------------+---------+---------+----+-----------+
| pg1    | 192.168.200.83 | Replica | stopped |    |   unknown |
| pg2    | 192.168.200.84 | Replica | running |  3 |         0 |
| pg3    | 192.168.200.85 | Leader  | running |  3 |           |
+--------+----------------+---------+---------+----+-----------+
alex@pg1:~/Desktop$ patronictl -c /etc/patroni.yml list
+ Cluster: patroni -------+---------+---------+----+-----------+
| Member | Host           | Role    | State   | TL | Lag in MB |
+--------+----------------+---------+---------+----+-----------+
| pg1    | 192.168.200.83 | Replica | stopped |    |   unknown |
| pg2    | 192.168.200.84 | Replica | running |  3 |         0 |
| pg3    | 192.168.200.85 | Leader  | running |  4 |           |
+--------+----------------+---------+---------+----+-----------+
alex@pg1:~/Desktop$ 
alex@pg1:~/Desktop$ patronictl -c /etc/patroni.yml list
+ Cluster: patroni -------+---------+---------+----+-----------+
| Member | Host           | Role    | State   | TL | Lag in MB |
+--------+----------------+---------+---------+----+-----------+
| pg1    | 192.168.200.83 | Replica | running |  4 |         0 |
| pg2    | 192.168.200.84 | Replica | running |  4 |         0 |
| pg3    | 192.168.200.85 | Leader  | running |  4 |           |
+--------+----------------+---------+---------+----+-----------+
alex@pg1:~/Desktop$ 

alex@ha1:~/Desktop$ psql -h localhost -d otus -U postgres -p 5433
Password for user postgres: 
psql (14.8 (Ubuntu 14.8-0ubuntu0.22.04.1))
Type "help" for help.

otus=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 alextest  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 haproxy   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 haproxy2  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 otus      | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 pgbouncer | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(8 rows)

otus=# create database haproxy3;
ERROR:  cannot execute CREATE DATABASE in a read-only transaction
otus=# 
otus=# create database haproxy3;
server closed the connection unexpectedly
	This probably means the server terminated abnormally
	before or while processing the request.
The connection to the server was lost. Attempting reset: Succeeded.
otus=# 
otus=# 
otus=# create database haproxy3;
CREATE DATABASE
otus=# 
```

####Установил KeepAlive

```
alex@ha1:~/Desktop$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:a9:d2:c2 brd ff:ff:ff:ff:ff:ff
    altname enp2s1
    inet 192.168.152.140/24 brd 192.168.152.255 scope global dynamic noprefixroute ens33
       valid_lft 974sec preferred_lft 974sec
    inet6 fe80::62b6:da00:238d:431d/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:a9:d2:cc brd ff:ff:ff:ff:ff:ff
    altname enp2s5
    inet 192.168.200.86/24 brd 192.168.200.255 scope global noprefixroute ens37
       valid_lft forever preferred_lft forever
    inet 192.168.200.88/32 scope global ens37
       valid_lft forever preferred_lft forever
alex@ha1:~/Desktop$ 
```

####Отправил в ребут первую ноду:

```
alex@ha2:~/Desktop$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:a8:e6:32 brd ff:ff:ff:ff:ff:ff
    altname enp2s1
    inet 192.168.152.141/24 brd 192.168.152.255 scope global dynamic noprefixroute ens33
       valid_lft 1463sec preferred_lft 1463sec
    inet6 fe80::cbae:ce5f:b1ef:5a03/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:a8:e6:3c brd ff:ff:ff:ff:ff:ff
    altname enp2s5
    inet 192.168.200.87/24 brd 192.168.200.255 scope global noprefixroute ens37
       valid_lft forever preferred_lft forever
    inet 192.168.200.88/32 scope global ens37
       valid_lft forever preferred_lft forever
alex@ha2:~/Desktop$ 
alex@ha2:~/Desktop$ 
```
