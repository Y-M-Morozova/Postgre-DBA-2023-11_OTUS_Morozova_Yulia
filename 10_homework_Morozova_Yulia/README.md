**<div align="center"><h2>Морозова Юлия, OTUS, группа Postgre-DBA-2023-11</h2></div>**

**<div align=center><h3>Домашнее задание №10 по теме:</h3></div>**
**<div align=center><h3>«Виды и устройство репликации в PostgreSQL. Практика применения.»</h3></div>**

***
**<h3>Домашнее задание:
<br>Репликация</h3>**

**<h3>Цель:
<br>реализовать свой миникластер на 3 ВМ.</h3>**

***

**Выполнение:**

**Подготвительные работы**

- Создаю в ЯО три ВМ:
  </br>otus-db-pg-vm-10-1 [158.160.128.224], otus-db-pg-vm-10-2 [158.160.145.254], otus-db-pg-vm-10-3 [158.160.141.114]:

  ![0_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/a9b1098c-99bc-4f43-887b-5b4617354eae)

- На ВМ устанавливаю Postgres 15й версии и устанавливаю пароль для пользователя postgres: ``\password`` -- otus_replica_test_1234567

  ![0_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/0e0c49b8-cdc4-482e-b0f0-d2328a675c6f)
  
- Устанавливаю уровень репликации и настраиваю прослушивание входящих IP-адресов командами:

  ```sql
    alter system set wal_level to 'logical';
    alter system set listen_addresses to '*';
  ```

    ![0_3](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/7fc4cb65-bbfc-40c1-a00e-0fc06f31d92e)

- В файлах конфигурации ``pg_hba.conf`` прописываю ip-адреса. На 1й ВМ прописываю ip-адреса 2й и 3й виртуальных машин, а на второй - для 1й ВМ и 3й ВМ,  в редакторе : ``nano /etc/postgresql/15/main/pg_hba.conf``:

    ![0_5](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/3864c743-063c-47a3-a6f4-1736cea51152)

    ![0_4](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/a4909b7d-f828-46b3-9cd2-c91be3905871)

- Далее рестартую на всех нодах для применения параметров Postgres :   ``sudo pg_ctlcluster 15 main restart``

<br/>  

***

<br/>

>**1. На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.**

  На 1й ВМ создаю таблицы:

  ```sql
    create table test (id int, txt char(10));
    create table test2 (id int, txt char(10));
  ```

  ![1_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/9cbdad12-7a8e-4a68-8a69-fdbead8f8868)

<br/>

>**2. Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2.**

  На 1й ВМ добавляю тестовые данные в таблицу ``test`` и создаю публикацию этой таблицы:

  ```sql
    insert into test
    select generate_series(1,10) as id,
    md5(random()::text)::char(10) as txt;
    create publication test_pub for table test;
  ```

  ![2_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/b180dbe2-4062-460c-8dd7-ac286fa2d958)

<br/>

>**3. На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение.**

  На 2й ВМ создаю таблицы:

  ```sql
    create table test (id int, txt char(10));
    create table test2 (id int, txt char(10));
  ``` 

  ![3_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/d6cda55b-441b-4ff0-bbcd-00eef0e819ea)

<br/>

>**4. Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test с ВМ №1.**

  На 2й ВМ добавляю тестовые данные в таблицу ``test2`` и создаю публикацию этой таблицы:

  ```sql
    insert into test2
    select generate_series(1,10) as id,
    md5(random()::text)::char(10) as txt;
    create publication test_pub2 for table test2;
  ```

  ![3_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/2f21aa99-d726-494c-9d21-90bad16f4172)

Теперь на 2й ВМ создаю подписку на публикацию таблицы ``test`` 1й ВМ:

```sql
  create subscription test_sub  connection 'host=158.160.128.224 port=5432 user=postgres password=otus_replica_test_1234567 dbname=postgres' publication test_pub with (copy_data = true);
```

  ![3_3](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/4dd5860c-250c-4aab-88f0-f9e076977d48)

  и проверяю командой: 

  ```sql
  select * from pg_stat_subscription \gx
  ```

  ![3_5](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/46808f02-1fc0-46bc-9b6a-575c0f16f891)


А теперь на 1й ВМ подписываюсь на публикацию таблицы ``test2`` 2й ВМ и проверяю:

  ```sql
  create subscription test_sub2 connection 'host=158.160.145.254 port=5432 user=postgres password=otus_replica_test_1234567 dbname=postgres' publication test_pub2 with (copy_data = true);
```

  ![3_4](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/99b2cb4d-b3a7-467b-a882-ec2ecad6f3bc)


Таким образом получаю, что на 1й ВМ  (``otus-db-pg-vm-10-1 [158.160.128.224]``) у меня две таблицы: первая таблица ``test`` ( для записи, оформлена публикация) и ``test2`` (для чтения - это подписка со 2й ВМ) и на 2й ВМ (``otus-db-pg-vm-10-2 [158.160.145.254]``) у меня таблицы ``test2`` для записи(есть публикация) и ``test`` для запросов на чтение(подписка с 1й ВМ).

  ![3_777](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/cfa2fe54-7965-491e-9abe-39888091a3d0)

<br/>

>**5. 3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).**

теперь на 3й ВМ (``otus-db-pg-vm-10-3 [158.160.141.114]``) создаю таблицы и оформляю подписки на таблицы ``test`` 1й ВМ и ``test2`` 2й ВМ:

  ```sql
    create table test (id int, txt char(10));
    create table test2 (id int, txt char(10));
    create subscription test_3_sub2 connection 'host=158.160.145.254 port=5432 user=postgres password=otus_replica_test_1234567 dbname=postgres' publication test_pub2 with (copy_data = true);
    create subscription test_3_sub connection 'host=158.160.128.224 port=5432 user=postgres password=otus_replica_test_1234567 dbname=postgres' publication test_pub with (copy_data = true);
  ``` 

все ок:

  ![4_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/7337b164-65d6-4330-8027-9d84c3d45209)

  ![4_3](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/d6676156-ce1c-4c7f-85f6-2e916c342643)

  ![4_4](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/0a4c3680-e715-4305-8fb3-a99175cf071d)

<br/>

***
**<h3> Задание со * :**
<br>Реализовать горячее реплицирование для высокой доступности на 4ВМ. 
<br>Источником должна выступать ВМ №3. 
<br>Написать с какими проблемами столкнулись. 
</h3>

***

**Выполнение:**

Для выполнения этого задания я буду использовать опять свои ВМ в ЯО, теперь беру четыре ВМ: 
</br>otus-db-pg-vm-10-1 [158.160.128.224], otus-db-pg-vm-10-2 [158.160.145.254], otus-db-pg-vm-10-3 [158.160.141.114]:

  ![6_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/b3c5dfc8-d48a-4b25-96e3-be88e4e17663)

  - Сначала настраиваю  мастер (``otus-db-pg-vm-10-1 [158.160.128.224]``).
  В файле конфигурации ``pg_hba.conf`` прописываю ip-адреса реплик в редакторе : ``nano /etc/postgresql/15/main/pg_hba.conf``:

  ``host    replication             postgres        158.160.143.60/32       scram-sha-256``
  </br>``host    replication             postgres        158.160.129.0/32       scram-sha-256``
  </br>``host    replication             postgres        158.160.149.161/32       scram-sha-256``

  ![6_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/f8c1e15b-05c1-43ac-8d89-02690d3605d9)

  - Устанавливаю уроверь репликации и настраиваю прослушивание входящих IP-адресов командами:

   ```sql
     alter system set wal_level to 'replica';
     alter system set listen_addresses to '*';
     alter system set hot_standby to 'on';
   ```

  ![6_3](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/84174d39-d576-4b1f-af12-e7050033b856)

  - Далее рестартую  для применения параметров Postgres :   ``sudo pg_ctlcluster 15 main restart`` 

  ![6_4](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/518be4ef-f0e2-4a66-bb10-b533fa16a411)

  - Создаю для тестов таблицу:

  ```sql
    create table test_hot as
    select generate_series(1,12) as id,
    md5(random()::text)::char(10) as txt
  ```
  ![6_5](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/3037efc1-9a8a-415f-a161-96e514104bc8)

  - Теперь настраиваю реплики.
    
  На всех трех нодах(репликах) в файле конфигурации ``pg_hba.conf`` прописываю ip-адрес мастера:

  ``host    replication             postgres        158.160.147.25/32       scram-sha-256``

  - и далее устанавливаю уровень репликации и настраиваю прослушивание входящих IP-адресов командами:

   ```sql
     alter system set wal_level to 'replica';
     alter system set listen_addresses to '*';
     alter system set hot_standby to 'on';
   ```

- рестартую неа всех 3х нодах репликах postgres, проверяю, все ок:

![7_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/93d3361a-9840-446b-aa63-7538ccf5cdaf)

- На всех трех нодах(репликах) останавливаю postgres, архивирую и потом удаляю каталоги с данными, восстанавливаю данные с мастера скриптом:

``sudo pg_ctlcluster 15 main stop``
<br>``sudo pg_ctlcluster 15 main status``
<br>``cd /var/lib/postgresql/15``
<br>``sudo tar -cvzf main_backup-`date +%s`.tgz main``
<br>``sudo rm -rf /var/lib/postgresql/15/main``
<br>``ls -la``
<br>``sudo mkdir main``
<br>``sudo chmod go-rwx main``
<br>``sudo chown -R postgres:postgres /var/lib/postgresql/15/main``
<br>``ls -la``
<br>``sudo pg_basebackup -P -R -X stream -c fast -h 158.160.128.211 -U postgres -D ./main``

  ![8_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/784008b0-a263-4692-ad11-8dc0ab6e1a08)

- все ок, теперь стартую postgres , проверяю реплику:

``sudo pg_ctlcluster 15 main start``

  ![8_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/0dd8b22a-797a-4b06-9a69-113ec94edceb)

  ![8_3](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/29c18e03-e647-42a8-acb0-2a5520467a3c)

- проверяю мастер:

```sql
select * from pg_stat_replication \gx
select * from pg_current_wal_lsn();
```

  ![8_4_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/195ba86e-7445-46c9-94a7-0dc7943d62e7)

  ![8_4_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/8d191d5c-29ee-4c1c-869b-a6949a958678)

  ![8_4_3](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/9c603f7b-7ac1-4a29-be70-9ba54277b819)

- проверяю реплику:

```sql
select * from pg_stat_wal_receiver \gx
select pg_last_wal_receive_lsn();
select pg_last_wal_replay_lsn();  
```

  ![8_5](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/9c54d950-58f9-4afc-bfa4-a1e324763ff3)


Таким образом у меня горячее реплицирование для высокой доступности на 4ВМ:

  ![bnjuj](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/9ff7cf1d-ec5c-4fdc-8540-aafa69882188)

 
<br>``  ``
<br>``  ``
<br>``  ``


