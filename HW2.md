#Домашнее задание №2

##Установка и настройка PostgteSQL в контейнере Docker

Поставить Docker Engine

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

#Команда загружает GPG-ключ репозитория Docker, который будет использоваться для проверки подписей пакетов Docker при их установке из репозитория.


```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

#Добавления репозитория Docker в список источников пакетов

```
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```
sudo apt install docker-ce
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
```

Cделать каталог /var/lib/postgres

```
sudo mkdir /var/lib/postgres
sudo chown -R alex:alex /var/lib/postgres
```

Развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres

#Поскольку на хосте у меня уже стоял экземпляр PG, то порт 5432 был занят, поэтому я сделал маппинг на 5433. 


#Посмотрел список не занятых на хосте портов с помощью команды ss -ltnp

```
alex@otus:/var/lib$ sudo ss -ltnp
State        Recv-Q       Send-Q               Local Address:Port                Peer Address:Port       Process                                           
LISTEN       0            4096                 127.0.0.53%lo:53                       0.0.0.0:*           users:(("systemd-resolve",pid=5784,fd=14))       
LISTEN       0            4096                       0.0.0.0:5433                     0.0.0.0:*           users:(("docker-proxy",pid=83040,fd=4))          
LISTEN       0            244                      127.0.0.1:5432                     0.0.0.0:*           users:(("postgres",pid=1035,fd=5))               
LISTEN       0            128                      127.0.0.1:6010                     0.0.0.0:*           users:(("sshd",pid=82417,fd=9))                  
LISTEN       0            511                      127.0.0.1:46803                    0.0.0.0:*           users:(("code",pid=88754,fd=49))                 
LISTEN       0            128                        0.0.0.0:22                       0.0.0.0:*           users:(("sshd",pid=81872,fd=3))                  
LISTEN       0            128                      127.0.0.1:631                      0.0.0.0:*           users:(("cupsd",pid=84366,fd=7))                 
LISTEN       0            4096                          [::]:5433                        [::]:*           users:(("docker-proxy",pid=83045,fd=4))          
LISTEN       0            128                          [::1]:631                         [::]:*           users:(("cupsd",pid=84366,fd=6))                 
LISTEN       0            128                          [::1]:6010                        [::]:*           users:(("sshd",pid=82417,fd=7))                  
LISTEN       0            128                           [::]:22                          [::]:*           users:(("sshd",pid=81872,fd=4))                  
alex@otus:/var/lib$ 
```

```
docker run --name postgres14 -e POSTGRES_PASSWORD=P@ssw0rd -d -p 5433:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
```


Развернуть контейнер с клиентом postgres

```
docker run -it --rm --name pg-client postgres:14 psql -h host.docker.internal -U postgres
```

Подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк


#Контейнер запустится в интерктивном режиме (it) и удалится после выхода (rm).


```
docker run -it --rm --name pg-client postgres:14 psql -h 172.17.0.2 -U postgres
```

```
create table cars(id serial, brand text, model text);
insert into cars(brand, model) values('Toyota', 'Rav4');
insert into cars(brand, model) values('Mazda', 'CX5');
commit; 
```

```
postgres=# select * from cars;
 id | brand  | model 
----+--------+-------
  1 | Toyota | Rav4
  2 | Mazda  | CX5
(2 rows)

postgres=# 
```


Подключится к контейнеру с сервером с ноутбука/компьютера извне

![DBeaver](https://github.com/mrchubaka/OtusHWPCS/blob/main/HW2-12.jpeg)


Удалить контейнер с сервером

```
docker stop postgres14
docker rm postgres14
```

Создать его заново

```
docker run --name postgres14 -e POSTGRES_PASSWORD=P@ssw0rd -d -p 5433:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
```

Подключится снова из контейнера с клиентом к контейнеру с сервером

```
docker run -it --rm --name pg-client postgres:14 psql -h 172.17.0.2 -U postgres
```

Проверить, что данные остались на месте

```
postgres=# select * from cars;
 id | brand  | model 
----+--------+-------
  1 | Toyota | Rav4
  2 | Mazda  | CX5
(2 rows)

postgres=# 
```

