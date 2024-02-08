#### *Отчет о выполнении домашнего задания:*


> создайте виртуальную машину c Ubuntu 20.04/22.04 LTS в GCE/ЯО/Virtual Box/докере
* **_Установлен Ubuntu server 22.04.2 на виртуальной машине в VirtualBox_**


> поставьте на нее PostgreSQL 15 через sudo apt
* **_Установлен Postgres 15_**
    * sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-15


> проверьте что кластер запущен через sudo -u postgres pg_lsclusters
* **_sudo -u postgres pg_lsclusters_**
    * Ver Cluster Port Status Owner    Data directory              Log file
    * 15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log


> зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым
* **_sudo -u postgres psql -U postgres_**
    * create table test(c1 text);
        * postgres=# create table test(c1 text);
        * CREATE TABLE
    * insert into test values('1');
        * postgres=# insert into test values('1');
        * INSERT 0 1
    * \q


> остановите postgres например через sudo -u postgres pg_ctlcluster 15 main stop
* **_Остановка Postgres_**
    * sudo -u postgres pg_ctlcluster 15 main stop
    * sudo -u postgres pg_lsclusters
        * Ver Cluster Port Status Owner    Data directory              Log file
        * 15  main    5432 down   postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log


> создайте новый диск к ВМ размером 10GB
* **_Создан новый диск в VirtualBox размером 10Gb_**


> добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
* **_Новый диск подключен к виртуальной машине с PostgreSQL_**


> проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux
* **_Операции с диском произвел по инструкции_**
    * Запись из /etc/fstab
    * /dev/disk/by-uuid/f889ce9f-a583-451d-8b23-cf1a4a2fffdd /mnt/pg_data ext4 defaults 0 2
    * Новый дисковый раздел будет располагаться тут /mnt/pg_data


> перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)
* **_Перезагрузил виртуальною машину и диск примонтирован нормально, после перезагрузки. Проверил, что кластер остановлен_**
    * sudo -u postgres pg_lsclusters
        * Ver Cluster Port Status Owner    Data directory              Log file
        * 15  main    5432 down   postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log


> сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
* **_sudo chown -R postgres:postgres /mnt/pg_data/_**


> перенесите содержимое /var/lib/postgres/14 в /mnt/data - mv /var/lib/postgresql/15/mnt/data
* **_sudo mv /var/lib/postgresql/15/ /mnt/pg_data/_**


> попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
* **_sudo -u postgres pg_ctlcluster 15 main start_**


> напишите получилось или нет и почему
* **_Не получилось_**
    * Error: /var/lib/postgresql/15/main is not accessible or does not exist

> задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/15/main который надо поменять и поменяйте его
* **_Меняем параметр data_directory_**
    * sudo vim /etc/postgresql/15/main/postgresql.conf
        * data_directory = '/mnt/pg_data/15/main'

> напишите что и почему поменяли
* **_Меняем параметр data_directory, чтобы кластер баз данных postgres знал новый путь с данными ($PGDATA)_**
    * sudo vim /etc/postgresql/15/main/postgresql.conf
        * data_directory = '/mnt/pg_data/15/main'

> попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
* **_sudo -u postgres pg_ctlcluster 15 main start_**


> напишите получилось или нет и почему
* **_Получилось_**
    * sudo -u postgres pg_ctlcluster 15 main start
        * Ver Cluster Port Status Owner    Data directory       Log file
        * 15  main    5432 online postgres /mnt/pg_data/15/main /var/log/postgresql/postgresql-15-main.log


> зайдите через через psql и проверьте содержимое ранее созданной таблицы
* **_sudo -u postgres psql -U postgres_**
    * postgres=# select * from test;
        *  c1 
