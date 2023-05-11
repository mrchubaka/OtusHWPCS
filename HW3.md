#Домашнее задание №3

##Настройка дисков для Постгреса

#####Когда я все уже сделал, я обнаружил, что нужно было работать с PG15, а я на автомате все сдалал на PG14.
#####Я понимаю принципиальный момент в данном случае и вот каким образом, можно поставить PG15.

1. Загружаем ключ и сохраните в файл.

```
alex@otus:~/Desktop$ wget --quiet -O ~/ACCC4CF8.asc https://www.postgresql.org/media/keys/ACCC4CF8.asc
```

2. Испортируем его в /etc/apt/trusted.gpg.d:

```
alex@otus:~/Desktop$ sudo gpg --no-default-keyring --keyring gnupg-ring:/etc/apt/trusted.gpg.d/postgresql.gpg --import ~/ACCC4CF8.asc
gpg: key 7FCC7D46ACCC4CF8: 2 signatures not checked due to missing keys
gpg: directory '/root/.gnupg' created
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key 7FCC7D46ACCC4CF8: public key "PostgreSQL Debian Repository" imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg: no ultimately trusted keys found

```

3. Назначаем права владения этому файлу пользователю _apt:

```
alex@otus:~/Desktop$ sudo chown _apt:root /etc/apt/trusted.gpg.d/postgresql.gpg
```

4. Добавляем репозиторий PostgreSQL в ваш список репозиториев:

```
alex@otus:~/Desktop$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```

5. Обновляем список доступных пакетов:

```
alex@otus:~/Desktop$ sudo apt update
Hit:1 http://ru.archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://ru.archive.ubuntu.com/ubuntu jammy-updates InRelease [119 kB]              
Get:3 http://ru.archive.ubuntu.com/ubuntu jammy-backports InRelease [108 kB]                                                            
Get:4 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages [577 kB]                                                    
Get:5 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main i386 Packages [390 kB]        
Get:6 http://apt.postgresql.org/pub/repos/apt jammy-pgdg InRelease [116 kB]                          
Get:7 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main Translation-en [166 kB]
Get:8 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 DEP-11 Metadata [101 kB]                      
Get:9 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 c-n-f Metadata [14,4 kB]
Get:10 http://ru.archive.ubuntu.com/ubuntu jammy-updates/restricted amd64 Packages [226 kB]
Get:11 http://ru.archive.ubuntu.com/ubuntu jammy-updates/restricted Translation-en [33,7 kB]
Get:12 http://ru.archive.ubuntu.com/ubuntu jammy-updates/universe amd64 Packages [887 kB]
Get:13 http://ru.archive.ubuntu.com/ubuntu jammy-updates/universe i386 Packages [609 kB]
Get:14 http://ru.archive.ubuntu.com/ubuntu jammy-updates/universe Translation-en [182 kB]
Get:15 http://ru.archive.ubuntu.com/ubuntu jammy-updates/universe amd64 DEP-11 Metadata [268 kB]
Get:16 http://ru.archive.ubuntu.com/ubuntu jammy-updates/universe amd64 c-n-f Metadata [18,8 kB]
Get:17 http://ru.archive.ubuntu.com/ubuntu jammy-updates/multiverse amd64 Packages [34,9 kB]  
Get:18 http://ru.archive.ubuntu.com/ubuntu jammy-updates/multiverse Translation-en [8 080 B]
Get:19 http://ru.archive.ubuntu.com/ubuntu jammy-updates/multiverse amd64 DEP-11 Metadata [940 B]
Get:20 http://ru.archive.ubuntu.com/ubuntu jammy-backports/main amd64 DEP-11 Metadata [8 000 B]
Get:21 http://ru.archive.ubuntu.com/ubuntu jammy-backports/universe amd64 DEP-11 Metadata [12,9 kB]
Get:22 http://apt.postgresql.org/pub/repos/apt jammy-pgdg/main amd64 Packages [262 kB]         
Get:23 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]        
Get:24 http://security.ubuntu.com/ubuntu jammy-security/main i386 Packages [209 kB]
Get:25 http://security.ubuntu.com/ubuntu jammy-security/main amd64 Packages [362 kB]
Get:26 http://security.ubuntu.com/ubuntu jammy-security/main Translation-en [107 kB]
Get:27 http://security.ubuntu.com/ubuntu jammy-security/main amd64 DEP-11 Metadata [41,7 kB]
Get:28 http://security.ubuntu.com/ubuntu jammy-security/main amd64 c-n-f Metadata [9 716 B]
Get:29 http://security.ubuntu.com/ubuntu jammy-security/restricted amd64 Packages [225 kB]
Get:30 http://security.ubuntu.com/ubuntu jammy-security/restricted Translation-en [33,3 kB]
Get:31 http://security.ubuntu.com/ubuntu jammy-security/universe i386 Packages [526 kB]
Get:32 http://security.ubuntu.com/ubuntu jammy-security/universe amd64 Packages [709 kB]
Get:33 http://security.ubuntu.com/ubuntu jammy-security/universe Translation-en [122 kB]
Get:34 http://security.ubuntu.com/ubuntu jammy-security/universe amd64 DEP-11 Metadata [18,5 kB]
Get:35 http://security.ubuntu.com/ubuntu jammy-security/universe amd64 c-n-f Metadata [14,3 kB]
Get:36 http://security.ubuntu.com/ubuntu jammy-security/multiverse amd64 Packages [30,2 kB]
Get:37 http://security.ubuntu.com/ubuntu jammy-security/multiverse Translation-en [5 828 B]
Fetched 6 668 kB in 3s (2 009 kB/s)      
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
290 packages can be upgraded. Run 'apt list --upgradable' to see them.
N: Skipping acquire of configured file 'main/binary-i386/Packages' as repository 'http://apt.postgresql.org/pub/repos/apt jammy-pgdg InRelease' doesn't support architecture 'i386'

```

6. Устанавливаем PostgreSQL 15:

```
alex@otus:~/Desktop$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
alex@otus:~/Desktop$ sudo apt install postgresql-15
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following package was automatically installed and is no longer required:
  systemd-hwe-hwdb
Use 'sudo apt autoremove' to remove it.
The following additional packages will be installed:
  libcommon-sense-perl libjson-perl libjson-xs-perl libllvm14 libpq5 libtypes-serialiser-perl postgresql-client-15 postgresql-client-common postgresql-common sysstat
Suggested packages:
  postgresql-doc-15 isag
The following NEW packages will be installed:
  libcommon-sense-perl libjson-perl libjson-xs-perl libllvm14 libpq5 libtypes-serialiser-perl postgresql-15 postgresql-client-15 postgresql-client-common postgresql-common sysstat
0 upgraded, 11 newly installed, 0 to remove and 290 not upgraded.
Need to get 43,8 MB of archives.
After this operation, 174 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 libjson-perl all 4.04000-1 [81,8 kB]
Get:2 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 libcommon-sense-perl amd64 3.75-2build1 [21,1 kB]
Get:3 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 libtypes-serialiser-perl all 1.01-1 [11,6 kB]
Get:4 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 libjson-xs-perl amd64 4.030-1build3 [87,2 kB]
Get:5 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 libllvm14 amd64 1:14.0.0-1ubuntu1 [24,0 MB]
Get:6 http://apt.postgresql.org/pub/repos/apt jammy-pgdg/main amd64 postgresql-client-common all 248.pgdg22.04+1 [92,7 kB]
Get:7 http://apt.postgresql.org/pub/repos/apt jammy-pgdg/main amd64 postgresql-common all 248.pgdg22.04+1 [237 kB]
Get:8 http://apt.postgresql.org/pub/repos/apt jammy-pgdg/main amd64 libpq5 amd64 15.2-1.pgdg22.04+1 [183 kB]
Get:9 http://apt.postgresql.org/pub/repos/apt jammy-pgdg/main amd64 postgresql-client-15 amd64 15.2-1.pgdg22.04+1 [1 680 kB]
Get:10 http://apt.postgresql.org/pub/repos/apt jammy-pgdg/main amd64 postgresql-15 amd64 15.2-1.pgdg22.04+1 [16,9 MB]
Get:11 http://ru.archive.ubuntu.com/ubuntu jammy-updates/main amd64 sysstat amd64 12.5.2-2ubuntu0.1 [487 kB]
Fetched 43,8 MB in 4s (12,3 MB/s)                                       
Preconfiguring packages ...
Selecting previously unselected package libjson-perl.
(Reading database ... 174832 files and directories currently installed.)
Preparing to unpack .../00-libjson-perl_4.04000-1_all.deb ...
Unpacking libjson-perl (4.04000-1) ...
Selecting previously unselected package postgresql-client-common.
Preparing to unpack .../01-postgresql-client-common_248.pgdg22.04+1_all.deb ...
Unpacking postgresql-client-common (248.pgdg22.04+1) ...
Selecting previously unselected package postgresql-common.
Preparing to unpack .../02-postgresql-common_248.pgdg22.04+1_all.deb ...
Adding 'diversion of /usr/bin/pg_config to /usr/bin/pg_config.libpq-dev by postgresql-common'
Unpacking postgresql-common (248.pgdg22.04+1) ...
Selecting previously unselected package libcommon-sense-perl:amd64.
Preparing to unpack .../03-libcommon-sense-perl_3.75-2build1_amd64.deb ...
Unpacking libcommon-sense-perl:amd64 (3.75-2build1) ...
Selecting previously unselected package libtypes-serialiser-perl.
Preparing to unpack .../04-libtypes-serialiser-perl_1.01-1_all.deb ...
Unpacking libtypes-serialiser-perl (1.01-1) ...
Selecting previously unselected package libjson-xs-perl.
Preparing to unpack .../05-libjson-xs-perl_4.030-1build3_amd64.deb ...
Unpacking libjson-xs-perl (4.030-1build3) ...
Selecting previously unselected package libllvm14:amd64.
Preparing to unpack .../06-libllvm14_1%3a14.0.0-1ubuntu1_amd64.deb ...
Unpacking libllvm14:amd64 (1:14.0.0-1ubuntu1) ...
Selecting previously unselected package libpq5:amd64.
Preparing to unpack .../07-libpq5_15.2-1.pgdg22.04+1_amd64.deb ...
Unpacking libpq5:amd64 (15.2-1.pgdg22.04+1) ...
Selecting previously unselected package postgresql-client-15.
Preparing to unpack .../08-postgresql-client-15_15.2-1.pgdg22.04+1_amd64.deb ...
Unpacking postgresql-client-15 (15.2-1.pgdg22.04+1) ...
Selecting previously unselected package postgresql-15.
Preparing to unpack .../09-postgresql-15_15.2-1.pgdg22.04+1_amd64.deb ...
Unpacking postgresql-15 (15.2-1.pgdg22.04+1) ...
Selecting previously unselected package sysstat.
Preparing to unpack .../10-sysstat_12.5.2-2ubuntu0.1_amd64.deb ...
Unpacking sysstat (12.5.2-2ubuntu0.1) ...
Setting up postgresql-client-common (248.pgdg22.04+1) ...
Setting up libpq5:amd64 (15.2-1.pgdg22.04+1) ...
Setting up libcommon-sense-perl:amd64 (3.75-2build1) ...
Setting up postgresql-client-15 (15.2-1.pgdg22.04+1) ...
update-alternatives: using /usr/share/postgresql/15/man/man1/psql.1.gz to provide /usr/share/man/man1/psql.1.gz (psql.1.gz) in auto mode
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
Setting up postgresql-common (248.pgdg22.04+1) ...
Adding user postgres to group ssl-cert

Creating config file /etc/postgresql-common/createcluster.conf with new version
Building PostgreSQL dictionaries from installed myspell/hunspell packages...
  en_us
Removing obsolete dictionary files:
'/etc/apt/trusted.gpg.d/apt.postgresql.org.gpg' -> '/usr/share/postgresql-common/pgdg/apt.postgresql.org.gpg'
Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /lib/systemd/system/postgresql.service.
Setting up postgresql-15 (15.2-1.pgdg22.04+1) ...
Creating new PostgreSQL cluster 15/main ...
/usr/lib/postgresql/15/bin/initdb -D /var/lib/postgresql/15/main --auth-local peer --auth-host scram-sha-256 --no-instructions
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with this locale configuration:
  provider:    libc
  LC_COLLATE:  en_US.UTF-8
  LC_CTYPE:    en_US.UTF-8
  LC_MESSAGES: en_US.UTF-8
  LC_MONETARY: ru_RU.UTF-8
  LC_NUMERIC:  ru_RU.UTF-8
  LC_TIME:     ru_RU.UTF-8
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

fixing permissions on existing directory /var/lib/postgresql/15/main ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Europe/Moscow
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok
update-alternatives: using /usr/share/postgresql/15/man/man1/postmaster.1.gz to provide /usr/share/man/man1/postmaster.1.gz (postmaster.1.gz) in auto mode
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for libc-bin (2.35-0ubuntu3.1) ...
alex@otus:~/Desktop$ 
```

7. Проверяем.

```
alex@otus:~/Desktop$ 
alex@otus:~/Desktop$ sudo -u postgres psql -c "SELECT version();"
could not change directory to "/home/alex/Desktop": Permission denied
                                                              version                                                              
-----------------------------------------------------------------------------------------------------------------------------------
 PostgreSQL 15.2 (Ubuntu 15.2-1.pgdg22.04+1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0, 64-bit
(1 row)

alex@otus:~/Desktop$ 

```

========================================================================================================================================================================
========================================================================================================================================================================

#####Далее я все делал на базе 14 версии.

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



#####Перенесите содержимое /var/lib/postgres/14 в /mnt/data - 
/var/lib/postgresql/14 /mnt/data


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

######Затем я примонтировал новый диск в к новой VM и изменил пути к данным, как это было сделано в предыдущем задании.
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
```

######Вот только тут я не удалил данные, а просто переименовал папку, что по смыслу в данном контексте тоже самое.

```
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
