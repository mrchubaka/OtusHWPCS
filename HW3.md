#Домашнее задание №3

##Настройка дисков для Постгреса

#####Проверьте что кластер запущен через sudo -u postgres pg_lsclusters

```
alex@otus:~$ sudo -u postgres pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
alex@otus:~$ 
```

#####зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым


```
alex@otus:~$ sudo -i -u postgres
[sudo] password for alex: 
postgres@otus:~$ psql
psql (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
Type "help" for help.

postgres=# 
postgres=# 
postgres=#  create table test(c1 text);
CREATE TABLE
postgres=#  insert into test values('1');
INSERT 0 1
postgres=# select * from test ;
 c1 
----
 1
(1 row)

postgres=# \q
postgres@otus:~$ 

```

#####Остановите postgres например через sudo -u postgres pg_ctlcluster 14 main stop

```
alex@otus:~$ sudo -u postgres pg_lsclusters
[sudo] password for alex: 
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
alex@otus:~$ 
alex@otus:~$ sudo -u postgres pg_ctlcluster 14 main stop
Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:
  sudo systemctl stop postgresql@14-main
alex@otus:~$ 
alex@otus:~$ sudo -u postgres pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 down   postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
alex@otus:~$ 
```

#####Проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux

```
alex@otus:~/Desktop$ lsblk 
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
fd0      2:0    1     4K  0 disk 
loop0    7:0    0     4K  1 loop /snap/bare/5
loop1    7:1    0 291,8M  1 loop /snap/code/126
loop2    7:2    0 241,5M  1 loop /snap/firefox/2605
loop3    7:3    0   242M  1 loop /snap/firefox/2645
loop4    7:4    0  63,3M  1 loop /snap/core20/1852
loop5    7:5    0 349,7M  1 loop /snap/gnome-3-38-2004/140
loop6    7:6    0  45,9M  1 loop /snap/snap-store/582
loop7    7:7    0  53,2M  1 loop /snap/snapd/18933
loop8    7:8    0  53,2M  1 loop /snap/snapd/19122
loop9    7:9    0   284K  1 loop /snap/snapd-desktop-integration/14
loop10   7:10   0 302,2M  1 loop /snap/code/128
loop11   7:11   0    73M  1 loop /snap/core22/607
loop12   7:12   0 400,8M  1 loop /snap/gnome-3-38-2004/112
loop13   7:13   0  91,7M  1 loop /snap/gtk-common-themes/1535
loop14   7:14   0  63,3M  1 loop /snap/core20/1879
loop15   7:15   0 460,6M  1 loop /snap/gnome-42-2204/102
loop16   7:16   0    73M  1 loop /snap/core22/617
loop17   7:17   0   452K  1 loop /snap/snapd-desktop-integration/83
loop18   7:18   0 460,6M  1 loop /snap/gnome-42-2204/99
sda      8:0    0    20G  0 disk 
├─sda1   8:1    0     1M  0 part 
├─sda2   8:2    0   513M  0 part /boot/efi
└─sda3   8:3    0  19,5G  0 part /var/snap/firefox/common/host-hunspell
                                 /
sdb      8:16   0    10G  0 disk 
sr0     11:0    1 126,7M  0 rom  /media/alex/CDROM
sr1     11:1    1   3,6G  0 rom  /media/alex/Ubuntu 22.04.1 LTS amd64
alex@otus:~/Desktop$ 
alex@otus:~/Desktop$ 
alex@otus:~/Desktop$ lsblk 
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
fd0      2:0    1     4K  0 disk 
loop0    7:0    0     4K  1 loop /snap/bare/5
loop1    7:1    0 291,8M  1 loop /snap/code/126
loop2    7:2    0 241,5M  1 loop /snap/firefox/2605
loop3    7:3    0   242M  1 loop /snap/firefox/2645
loop4    7:4    0  63,3M  1 loop /snap/core20/1852
loop5    7:5    0 349,7M  1 loop /snap/gnome-3-38-2004/140
loop6    7:6    0  45,9M  1 loop /snap/snap-store/582
loop7    7:7    0  53,2M  1 loop /snap/snapd/18933
loop8    7:8    0  53,2M  1 loop /snap/snapd/19122
loop9    7:9    0   284K  1 loop /snap/snapd-desktop-integration/14
loop10   7:10   0 302,2M  1 loop /snap/code/128
loop11   7:11   0    73M  1 loop /snap/core22/607
loop12   7:12   0 400,8M  1 loop /snap/gnome-3-38-2004/112
loop13   7:13   0  91,7M  1 loop /snap/gtk-common-themes/1535
loop14   7:14   0  63,3M  1 loop /snap/core20/1879
loop15   7:15   0 460,6M  1 loop /snap/gnome-42-2204/102
loop16   7:16   0    73M  1 loop /snap/core22/617
loop17   7:17   0   452K  1 loop /snap/snapd-desktop-integration/83
loop18   7:18   0 460,6M  1 loop /snap/gnome-42-2204/99
sda      8:0    0    20G  0 disk 
├─sda1   8:1    0     1M  0 part 
├─sda2   8:2    0   513M  0 part /boot/efi
└─sda3   8:3    0  19,5G  0 part /var/snap/firefox/common/host-hunspell
                                 /
sdb      8:16   0    10G  0 disk 
sr0     11:0    1 126,7M  0 rom  /media/alex/CDROM
sr1     11:1    1   3,6G  0 rom  /media/alex/Ubuntu 22.04.1 LTS amd64
alex@otus:~/Desktop$ sudo fdisk /dev/sdb 
[sudo] password for alex: 

Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xc0e77b08.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-20971519, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-20971519, default 20971519): 

Created a new partition 1 of type 'Linux' and of size 10 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

alex@otus:~/Desktop$ lsblk 
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
fd0      2:0    1     4K  0 disk 
loop0    7:0    0     4K  1 loop /snap/bare/5
loop1    7:1    0 291,8M  1 loop /snap/code/126
loop2    7:2    0 241,5M  1 loop /snap/firefox/2605
loop3    7:3    0   242M  1 loop /snap/firefox/2645
loop4    7:4    0  63,3M  1 loop /snap/core20/1852
loop5    7:5    0 349,7M  1 loop /snap/gnome-3-38-2004/140
loop6    7:6    0  45,9M  1 loop /snap/snap-store/582
loop7    7:7    0  53,2M  1 loop /snap/snapd/18933
loop8    7:8    0  53,2M  1 loop /snap/snapd/19122
loop9    7:9    0   284K  1 loop /snap/snapd-desktop-integration/14
loop10   7:10   0 302,2M  1 loop /snap/code/128
loop11   7:11   0    73M  1 loop /snap/core22/607
loop12   7:12   0 400,8M  1 loop /snap/gnome-3-38-2004/112
loop13   7:13   0  91,7M  1 loop /snap/gtk-common-themes/1535
loop14   7:14   0  63,3M  1 loop /snap/core20/1879
loop15   7:15   0 460,6M  1 loop /snap/gnome-42-2204/102
loop16   7:16   0    73M  1 loop /snap/core22/617
loop17   7:17   0   452K  1 loop /snap/snapd-desktop-integration/83
loop18   7:18   0 460,6M  1 loop /snap/gnome-42-2204/99
sda      8:0    0    20G  0 disk 
├─sda1   8:1    0     1M  0 part 
├─sda2   8:2    0   513M  0 part /boot/efi
└─sda3   8:3    0  19,5G  0 part /var/snap/firefox/common/host-hunspell
                                 /
sdb      8:16   0    10G  0 disk 
└─sdb1   8:17   0    10G  0 part 
sr0     11:0    1 126,7M  0 rom  /media/alex/CDROM
sr1     11:1    1   3,6G  0 rom  /media/alex/Ubuntu 22.04.1 LTS amd64
alex@otus:~/Desktop$ sudo mkfs.ext4 /dev/sdb
sdb   sdb1  
alex@otus:~/Desktop$ sudo mkfs.ext4 /dev/sdb1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 2621184 4k blocks and 655360 inodes
Filesystem UUID: 08b3623b-9eb5-421b-a737-275542be0e14
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

alex@otus:~/Desktop$ sudo mkdir /mnt/data
alex@otus:~/Desktop$ 
alex@otus:~/Desktop$ sudo mount /dev/sdb1 /mnt/data
alex@otus:~/Desktop$ nano /etc/fstab 
alex@otus:~/Desktop$ sudo nano /etc/fstab 
alex@otus:~/Desktop$ cat /etc/fstab 
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda3 during installation
UUID=f3c97613-5b7c-40b5-b774-487fa1df8c9c /               ext4    errors=remount-ro 0       1
# /boot/efi was on /dev/sda2 during installation
UUID=24F7-6426  /boot/efi       vfat    umask=0077      0       1
/swapfile                                 none            swap    sw              0       0
/dev/fd0        /media/floppy0  auto    rw,user,noauto,exec,utf8 0       0
/dev/sdb1 /mnt/data ext4 defaults 0 0
alex@otus:~/Desktop$ 
```

#####Перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)

```
alex@otus:~$ ls -al /mnt/data/
total 24
drwxr-xr-x 3 root root  4096 мая 10 14:34 .
drwxr-xr-x 3 root root  4096 мая 10 13:45 ..
drwx------ 2 root root 16384 мая 10 13:38 lost+found
-rw-r--r-- 1 root root     0 мая 10 14:34 test1
alex@otus:~$
```

#####Сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/

```
alex@otus:~$ ls -al /mnt/data/
total 24
drwxr-xr-x 3 root root  4096 мая 10 14:34 .
drwxr-xr-x 3 root root  4096 мая 10 13:45 ..
drwx------ 2 root root 16384 мая 10 13:38 lost+found
alex@otus:~$ 
alex@otus:~$ sudo chown -R postgres:postgres /mnt/data/
alex@otus:~$
alex@otus:~$ ls -al /mnt/data/
total 24
drwxr-xr-x 3 postgres postgres  4096 мая 10 14:40 .
drwxr-xr-x 3 root     root      4096 мая 10 13:45 ..
drwx------ 2 postgres postgres 16384 мая 10 13:38 lost+found
alex@otus:~$ 

```



#####Перенесите содержимое /var/lib/postgres/14 в /mnt/data - mv /var/lib/postgresql/14 /mnt/data


```
alex@otus:~$ sudo mv /var/lib/postgresql/14 /mnt/data
alex@otus:~$ 
alex@otus:~$ ls -al /mnt/data/
total 28
drwxr-xr-x 4 postgres postgres  4096 мая 10 14:44 .
drwxr-xr-x 3 root     root      4096 мая 10 13:45 ..
drwxr-xr-x 3 postgres postgres  4096 апр 29 12:03 14
drwx------ 2 postgres postgres 16384 мая 10 13:38 lost+found
alex@otus:~$ 

```

#####Попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start

```
alex@otus:~$ sudo -u postgres pg_ctlcluster 14 main start
Error: /var/lib/postgresql/14/main is not accessible or does not exist
alex@otus:~$
```

#####Напишите получилось или нет и почему
#####Задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/14/main который надо поменять и поменяйте его
#####Напишите что и почему поменяли
#####Попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start
#####Напишите получилось или нет и почему

######Ошибка возникает потому, что после переноса содержимого директории /var/lib/postgresql/14 в /mnt/data, кластер PostgreSQL не может найти свои файлы данных в старой директории.

######Для исправления этой проблемы необходимо указать PostgreSQL использовать новый путь к файлам данных. Для этого нужно отредактировать файл /etc/postgresql/14/main/postgresql.conf и изменить параметр data_directory на новый путь к директории с данными, т.е. на /mnt/data/14/main.

#####Зайдите через через psql и проверьте содержимое ранее созданной таблицы

```
alex@otus:~$ sudo -i -u postgres
postgres@otus:~$ 
postgres@otus:~$ psql 
psql (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
Type "help" for help.

postgres=# 
postgres=# select * from test ;
 c1 
----
 1
(1 row)

postgres=# 
```

#####Задание со звездочкой *: не удаляя существующий GCE инстанс/ЯО сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.

######Сначала я остановил PG на основной системе и выключил VM. Затем файлы "нового диска" в папку новой VM и затем добавил их через добавление существующего диска. Весь процесс в VMWare Workstation ниже на картинках. 

![Disk moving](https://github.com/mrchubaka/OtusHWPCS/blob/main/hw3.png)

```
alex@otus-2:~/Desktop$ sudo systemctl stop postgresql@14-main
alex@otus-2:~/Desktop$ 
alex@otus-2:~/Desktop$ sudo systemctl status postgresql@14-main
× postgresql@14-main.service - PostgreSQL Cluster 14-main
     Loaded: loaded (/lib/systemd/system/postgresql@.service; enabled-runtime; vendor preset: enabled)
     Active: failed (Result: exit-code) since Wed 2023-05-10 17:53:08 MSK; 19s ago
    Process: 5166 ExecStart=/usr/bin/pg_ctlcluster --skip-systemctl-redirect 14-main start (code=exited, status=0/SUCCESS)
    Process: 5700 ExecStop=/usr/bin/pg_ctlcluster --skip-systemctl-redirect -m fast 14-main stop (code=exited, status=2)
   Main PID: 5171 (code=exited, status=0/SUCCESS)
        CPU: 264ms

мая 10 17:45:08 otus-2 systemd[1]: Starting PostgreSQL Cluster 14-main...
мая 10 17:45:10 otus-2 systemd[1]: Started PostgreSQL Cluster 14-main.
мая 10 17:53:08 otus-2 postgresql@14-main[5700]: Cluster is not running.
мая 10 17:53:08 otus-2 systemd[1]: postgresql@14-main.service: Control process exited, code=exited, status=2/INVALIDARGUMENT
мая 10 17:53:08 otus-2 systemd[1]: postgresql@14-main.service: Failed with result 'exit-code'.
alex@otus-2:~/Desktop$

alex@otus-2:/var/lib/postgresql$ sudo mv 14/ 14_bk

alex@otus-2:~/Desktop$ lsblk 
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
fd0      2:0    1     4K  0 disk 
loop0    7:0    0 400,8M  1 loop /snap/gnome-3-38-2004/112
loop1    7:1    0  91,7M  1 loop /snap/gtk-common-themes/1535
loop2    7:2    0 163,3M  1 loop /snap/firefox/1635
loop3    7:3    0     4K  1 loop /snap/bare/5
loop4    7:4    0    62M  1 loop /snap/core20/1587
loop5    7:5    0  45,9M  1 loop /snap/snap-store/582
loop6    7:6    0    47M  1 loop /snap/snapd/16292
loop7    7:7    0   284K  1 loop /snap/snapd-desktop-integration/14
sda      8:0    0    20G  0 disk 
├─sda1   8:1    0     1M  0 part 
├─sda2   8:2    0   513M  0 part /boot/efi
└─sda3   8:3    0  19,5G  0 part /
sdb      8:16   0    10G  0 disk 
└─sdb1   8:17   0    10G  0 part 
sr0     11:0    1 126,7M  0 rom  /media/alex/CDROM
sr1     11:1    1   3,6G  0 rom  /media/alex/Ubuntu 22.04.1 LTS amd64
alex@otus-2:~/Desktop$ ls -al /mnt/
total 12
drwxr-xr-x  3 root root 4096 мая 10 17:09 .
drwxr-xr-x 20 root root 4096 мая 10 16:57 ..
drwxr-xr-x  2 root root 4096 мая 10 17:09 data
alex@otus-2:~/Desktop$ \
> 
alex@otus-2:~/Desktop$ sudo mount -t ext4 /dev/sdb1 /mnt/data
[sudo] password for alex: 
alex@otus-2:~/Desktop$ 
alex@otus-2:~/Desktop$ 
alex@otus-2:~/Desktop$ ls -al /mnt/data/
total 28
drwxr-xr-x 4  128  136  4096 мая 10 14:44 .
drwxr-xr-x 3 root root  4096 мая 10 17:09 ..
drwxr-xr-x 3  128  136  4096 апр 29 12:03 14
drwx------ 2  128  136 16384 мая 10 13:38 lost+found
alex@otus-2:~/Desktop$

alex@otus-2:/var/lib/postgresql$ sudo systemctl start postgresql@14-main
alex@otus-2:/var/lib/postgresql$ 
alex@otus-2:/var/lib/postgresql$ sudo systemctl status postgresql@14-main
● postgresql@14-main.service - PostgreSQL Cluster 14-main
     Loaded: loaded (/lib/systemd/system/postgresql@.service; enabled-runtime; vendor preset: enabled)
     Active: active (running) since Wed 2023-05-10 18:14:49 MSK; 5s ago
    Process: 5978 ExecStart=/usr/bin/pg_ctlcluster --skip-systemctl-redirect 14-main start (code=exited, status=0/SUCCESS)
   Main PID: 5983 (postgres)
      Tasks: 7 (limit: 4575)
     Memory: 18.0M
        CPU: 138ms
     CGroup: /system.slice/system-postgresql.slice/postgresql@14-main.service
             ├─5983 /usr/lib/postgresql/14/bin/postgres -D /mnt/data/14/main -c config_file=/etc/postgresql/14/main/postgresql.conf
             ├─5985 "postgres: 14/main: checkpointer " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ">
             ├─5986 "postgres: 14/main: background writer " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "">
             ├─5987 "postgres: 14/main: walwriter " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ">
             ├─5988 "postgres: 14/main: autovacuum launcher " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" >
             ├─5989 "postgres: 14/main: stats collector " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ">
             └─5990 "postgres: 14/main: logical replication launcher " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" >

мая 10 18:14:47 otus-2 systemd[1]: Starting PostgreSQL Cluster 14-main...
мая 10 18:14:49 otus-2 systemd[1]: Started PostgreSQL Cluster 14-main.
alex@otus-2:/var/lib/postgresql$ 
alex@otus-2:/var/lib/postgresql$ sudo -i -u postgres
postgres@otus-2:~$ psql
psql (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
Type "help" for help.

postgres=# select * from test ;
 c1 
----
 1
(1 row)

postgres=# 

```
