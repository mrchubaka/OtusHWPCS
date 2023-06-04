#Домашнее задание №7

##Развернуть Постгрес в миникубе

####Устанавливаем minikube и разворачиваем PostgreSQL 14 через манифест


####Установка minikube
```
alex@alex-otus2:~/Desktop$ wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
--2023-06-04 14:50:45--  https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
Resolving storage.googleapis.com (storage.googleapis.com)... 173.194.221.128, 209.85.233.128, 142.250.150.128, ...
Connecting to storage.googleapis.com (storage.googleapis.com)|173.194.221.128|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 83937631 (80M) [application/octet-stream]
Saving to: ‘minikube-linux-amd64’

minikube-linux-amd64                      100%[===================================================================================>]  80,05M  8,82MB/s    in 9,4s    

2023-06-04 14:50:55 (8,56 MB/s) - ‘minikube-linux-amd64’ saved [83937631/83937631]

alex@alex-otus2:~/Desktop$ chmod +x minikube-linux-amd64
alex@alex-otus2:~/Desktop$ sudo mv minikube-linux-amd64 /usr/local/bin/minikube
alex@alex-otus2:~/Desktop$ minikube version
minikube version: v1.30.1
commit: 08896fd1dc362c097c925146c4a0d0dac715ace0

```


####Создаем манифест файл


```
alex@alex-otus2:~/Desktop$ cd /etc/
alex@alex-otus2:/etc$ sudo nano postgresql.yaml
alex@alex-otus2:/etc$
alex@alex-otus2:/etc$ cat postgresql.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:14
          imagePullPolicy: "IfNotPresent"
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_USER
              value: "postgres"
            - name: POSTGRES_PASSWORD
              value: "password"
          volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: postgres-storage
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  selector:
    app: postgres
  ports:
    - protocol: TCP
      port: 5432
      targetPort: 5432
alex@alex-otus2:/etc$ 


```

####Запуск Minikube

```
alex@etcd1:~/Desktop$ minikube start
😄  minikube v1.30.1 on Ubuntu 22.04
👎  Unable to pick a default driver. Here is what was considered, in preference order:
💡  Alternatively you could install one of these drivers:
    ▪ docker: Not installed: exec: "docker": executable file not found in $PATH
    ▪ kvm2: Not installed: exec: "virsh": executable file not found in $PATH
    ▪ podman: Not installed: exec: "podman": executable file not found in $PATH
    ▪ qemu2: Not installed: exec: "qemu-system-x86_64": executable file not found in $PATH
    ▪ virtualbox: Not installed: unable to find VBoxManage in $PATH

❌  Exiting due to DRV_NOT_DETECTED: No possible driver was detected. Try specifying --driver, or see https://minikube.sigs.k8s.io/docs/start/

alex@etcd1:~/Desktop$ 

```

####Minikube требует драйвера виртуализации для работы.

```
alex@etcd1:~/Desktop$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

####Официальный ключ репозитория Docker GPG:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

####Добавим репозиторий Docker:
```
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

####Обновляем список пакетов в вашем репозитории снова и установим Docker:
```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

####Решаю ошибку прав

```
alex@etcd1:~/Desktop$ minikube start --driver=docker
😄  minikube v1.30.1 on Ubuntu 22.04
✨  Using the docker driver based on user configuration

💣  Exiting due to PROVIDER_DOCKER_NEWGRP: "docker version --format -:" exit status 1: permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/version": dial unix /var/run/docker.sock: connect: permission denied
💡  Suggestion: Add your user to the 'docker' group: 'sudo usermod -aG docker $USER && newgrp docker'
📘  Documentation: https://docs.docker.com/engine/install/linux-postinstall/

alex@etcd1:~/Desktop$ 

alex@etcd1:~/Desktop$ sudo usermod -aG docker $USER
alex@etcd1:~/Desktop$ newgrp docker
alex@etcd1:~/Desktop$ 

```

####Запуск Minikube
```
alex@etcd1:~/Desktop$ minikube start --driver=docker
😄  minikube v1.30.1 on Ubuntu 22.04
✨  Using the docker driver based on user configuration
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.26.3 preload ...
    > preloaded-images-k8s-v18-v1...:  397.02 MiB / 397.02 MiB  100.00% 8.51 Mi
    > gcr.io/k8s-minikube/kicbase...:  373.53 MiB / 373.53 MiB  100.00% 4.41 Mi
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.26.3 on Docker 23.0.2 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
alex@etcd1:~/Desktop$ 

```

####Инициализация контейнера
```
alex@etcd1:~/Desktop$ kubectl get pods
No resources found in default namespace.
alex@etcd1:~/Desktop$ 

alex@etcd1:/etc$ kubectl apply -f postgresql.yaml
deployment.apps/postgres created
service/postgres-service created

alex@etcd1:/etc$ kubectl get pods
NAME                        READY   STATUS              RESTARTS   AGE
postgres-5b555f9dcc-hlv7r   0/1     ContainerCreating   0          5s
alex@etcd1:/etc$ 

```

####Проброс портов из контейнера PostgreSQL, позволяя нам подключиться к базе данных.
```
alex@etcd1:/etc$ kubectl port-forward deployment/postgres 5432:5432
Forwarding from 127.0.0.1:5432 -> 5432
Forwarding from [::1]:5432 -> 5432

```

####Установка клиента
```
alex@alex-otus2:~/Desktop$ sudo apt update && sudo apt upgrade -y && sudo apt install -y postgresql-client-common && sudo apt install postgresql-client -y

```

####Подключение к базе
```
alex@alex-otus2:~/Desktop$ psql -h localhost -U postgres -d postgres
psql (14.8 (Ubuntu 14.8-0ubuntu0.22.04.1))
Type "help" for help.

postgres=# 

```





