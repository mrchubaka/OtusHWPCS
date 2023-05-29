#Домашнее задание №6

##Развернуть HA кластер

#####Поскольку готового образа Patroni для Docker нет, я нашел как это можено сделать:
#####Вот так я создал Docker образ для Patroni:

```
git clone https://github.com/zalando/patroni.git
docker build -t my-patroni .
```

Затем я установил Docker и Docker Compose:

```
sudo apt update
sudo apt install docker.io docker-compose
```

#####Создание файлов docker-compose.yaml и haproxy.cfg:
Порты 23791-23803 были выбраны, поскольку на хосте 2379-2383 уже были заняты. Инициализация падала с ошибкой.

```
alex@alex-otus2:~/Desktop/my_cluster$ cat docker-compose.yaml 
version: '3'
services:
  etcd1:
    image: quay.io/coreos/etcd:v3.4.15
    command: etcd --name etcd1 --listen-client-urls http://0.0.0.0:23791 --advertise-client-urls http://etcd1:23791 --initial-advertise-peer-urls http://etcd1:23801 --listen-peer-urls http://0.0.0.0:23801 --initial-cluster etcd1=http://etcd1:23801,etcd2=http://etcd2:23802,etcd3=http://etcd3:23803 --initial-cluster-token my-etcd-token --initial-cluster-state new
    ports:
      - 23791:2379
      - 23801:2380
  etcd2:
    image: quay.io/coreos/etcd:v3.4.15
    command: etcd --name etcd2 --listen-client-urls http://0.0.0.0:23792 --advertise-client-urls http://etcd2:23792 --initial-advertise-peer-urls http://etcd2:23802 --listen-peer-urls http://0.0.0.0:23802 --initial-cluster etcd1=http://etcd1:23801,etcd2=http://etcd2:23802,etcd3=http://etcd3:23803 --initial-cluster-token my-etcd-token --initial-cluster-state new
    ports:
      - 23792:2379
      - 23802:2380
  etcd3:
    image: quay.io/coreos/etcd:v3.4.15
    command: etcd --name etcd3 --listen-client-urls http://0.0.0.0:23793 --advertise-client-urls http://etcd3:23793 --initial-advertise-peer-urls http://etcd3:23803 --listen-peer-urls http://0.0.0.0:23803 --initial-cluster etcd1=http://etcd1:23801,etcd2=http://etcd2:23802,etcd3=http://etcd3:23803 --initial-cluster-token my-etcd-token --initial-cluster-state new
    ports:
      - 23793:2379
      - 23803:2380
  patroni1:
    image: my-patroni:latest
    command: patroni /etc/patroni/patroni.yml
    volumes:
      - ./patroni1:/etc/patroni
    depends_on:
      - etcd1
      - etcd2
      - etcd3
  patroni2:
    image: my-patroni:latest
    command: patroni /etc/patroni/patroni.yml
    volumes:
      - ./patroni2:/etc/patroni
    depends_on:
      - etcd1
      - etcd2
      - etcd3
  patroni3:
    image: my-patroni:latest
    command: patroni /etc/patroni/patroni.yml
    volumes:
      - ./patroni3:/etc/patroni
    depends_on:
      - etcd1
      - etcd2
      - etcd3
  haproxy:
    image: haproxy
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg
    ports:
      - 5432:5432
      - 5000:5000
alex@alex-otus2:~/Desktop/my_cluster$ cat haproxy.cfg 
global
  maxconn 4096
  tune.ssl.default-dh-param 2048

defaults
  mode tcp
  timeout connect 5000ms
  timeout client 50000ms
  timeout server 50000ms

frontend pg_frontend
  bind *:5432
  default_backend pg_backend

backend pg_backend
  balance roundrobin
  server patroni1 patroni1:5432 check
  server patroni2 patroni2:5432 check
  server patroni3 patroni3:5432 check
alex@alex-otus2:~/Desktop/my_cluster$ 
```

#####Запуск контейнеров
```
alex@alex-otus2:~/Desktop/my_cluster$ sudo docker-compose up -d
Creating network "my_cluster_default" with the default driver
Creating my_cluster_etcd2_1   ... done
Creating my_cluster_haproxy_1 ... done
Creating my_cluster_etcd1_1   ... done
Creating my_cluster_etcd3_1   ... done
Creating my_cluster_patroni3_1 ... done
Creating my_cluster_patroni2_1 ... done
Creating my_cluster_patroni1_1 ... done
alex@alex-otus2:~/Desktop/my_cluster$ 
alex@alex-otus2:~/Desktop/my_cluster/patroni1$ sudo docker ps
CONTAINER ID   IMAGE                         COMMAND                  CREATED             STATUS              PORTS                                                                                      NAMES
8414738395c1   my-patroni:latest             "/bin/sh /entrypoint…"   About an hour ago   Up About a minute                                                                                              my_cluster_patroni3_1
d7ecaacea68c   my-patroni:latest             "/bin/sh /entrypoint…"   About an hour ago   Up About a minute                                                                                              my_cluster_patroni2_1
f6a805b212f9   my-patroni:latest             "/bin/sh /entrypoint…"   About an hour ago   Up About a minute                                                                                              my_cluster_patroni1_1
840769525ade   quay.io/coreos/etcd:v3.4.15   "etcd --name etcd1 -…"   About an hour ago   Up About a minute   0.0.0.0:23791->2379/tcp, :::23791->2379/tcp, 0.0.0.0:23801->2380/tcp, :::23801->2380/tcp   my_cluster_etcd1_1
527ee42c012c   quay.io/coreos/etcd:v3.4.15   "etcd --name etcd3 -…"   About an hour ago   Up About a minute   0.0.0.0:23793->2379/tcp, :::23793->2379/tcp, 0.0.0.0:23803->2380/tcp, :::23803->2380/tcp   my_cluster_etcd3_1
e515f7768326   quay.io/coreos/etcd:v3.4.15   "etcd --name etcd2 -…"   About an hour ago   Up About a minute   0.0.0.0:23792->2379/tcp, :::23792->2379/tcp, 0.0.0.0:23802->2380/tcp, :::23802->2380/tcp   my_cluster_etcd2_1
9ae64f7b1d01   haproxy                       "docker-entrypoint.s…"   About an hour ago   Up About a minute   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp, 0.0.0.0:5432->5432/tcp, :::5432->5432/tcp       my_cluster_haproxy_1
alex@alex-otus2:~/Desktop/my_cluster/patroni1$ 
```

###Но вот здесь у меня возникла проблема с инициализацией.
###Не могу разобраться, в чем же ошибся. Прошу помочь.

#####Инициализация кластера Patroni:

```
alex@alex-otus2:~/Desktop/my_cluster/patroni1$ sudo docker exec -it my_cluster_patroni1_1 patronictl -c /etc/patroni/patroni.yml list
2023-05-29 17:35:30,756 - ERROR - Failed to get list of machines from http://my_cluster_etcd3_1:2381/v2: MaxRetryError("HTTPConnectionPool(host='my_cluster_etcd3_1', port=2381): Max retries exceeded with url: /v2/machines (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x7f5ceb13f430>: Failed to establish a new connection: [Errno 111] Connection refused'))")
2023-05-29 17:35:30,757 - ERROR - Failed to get list of machines from http://my_cluster_etcd2_1:2380/v2: MaxRetryError("HTTPConnectionPool(host='my_cluster_etcd2_1', port=2380): Max retries exceeded with url: /v2/machines (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x7f5ceb13f790>: Failed to establish a new connection: [Errno 111] Connection refused'))")
2023-05-29 17:35:30,758 - ERROR - Failed to get list of machines from http://my_cluster_etcd1_1:2379/v2: MaxRetryError("HTTPConnectionPool(host='my_cluster_etcd1_1', port=2379): Max retries exceeded with url: /v2/machines (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x7f5ceb13f940>: Failed to establish a new connection: [Errno 111] Connection refused'))")
^C
Aborted!
alex@alex-otus2:~/Desktop/my_cluster/patroni1$ 

```


```
alex@alex-otus2:~/Desktop/my_cluster/patroni1$ sudo docker exec -it my_cluster_patroni1_1 cat /etc/patroni/patroni.yml
scope: my_cluster
name: my_cluster_patroni1

restapi:
  listen: 0.0.0.0:8008
  connect_address: my_cluster_patroni1_1:8008

etcd:
  hosts: 
    - my_cluster_etcd1_1:2379
    - my_cluster_etcd2_1:2380
    - my_cluster_etcd3_1:2381

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true

  initdb:
  - encoding: UTF8
  - data-checksums

  pg_hba:
  - host replication replicator 127.0.0.1/32 md5
  - host replication replicator my_cluster_patroni1_1/0 md5
  - host replication replicator my_cluster_patroni2_1/0 md5
  - host replication replicator my_cluster_patroni3_1/0 md5
  - host all all 0.0.0.0/0 md5

postgresql:
  listen: 0.0.0.0:5432
  connect_address: patroni1:5432
  data_dir: /data/patroni
  pgpass: /tmp/pgpass
  authentication:
    replication:
      username: replicator
      password: replicatorpass
    superuser:
      username: postgres
      password: secret
  parameters:
    unix_socket_directories: /tmp

tags:
  nofailover: false
  noloadbalance: false
  clonefrom: false
  nosync: false
alex@alex-otus2:~/Desktop/my_cluster/patroni1$ 
```

