**<div align="center"><h2>Морозова Юлия, OTUS, группа Postgre-DBA-2023-11</h2></div>**

**<div align=center><h3>Домашнее задание №8 по теме:</h3></div>**
**<div align=center><h3>«Настройка PostgreSQL.»</h3></div>**

***
**<h3>Домашнее задание:
<br>Нагрузочное тестирование и тюнинг PostgreSQL</h3>**

**<h3>Цель:
<br>сделать нагрузочное тестирование PostgreSQL
<br>настроить параметры PostgreSQL для достижения максимальной производительности</h3>**

***

**Выполнение:**

>**1. Развернуть виртуальную машину любым удобным способом**

Для этого задания создала новую ВМ В ЯО с параметрами:
​CPU: 2, RAM: 4 ГБ, Объём дискового пространства: 10 ГБ (SSD):

  ![1_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/8b717521-fcfe-45de-85bc-35c8d5e2405c)

<br/>

>**2. Поставить на неё PostgreSQL 15 любым способом**

Установила PostgreSQL 15й версии скриптом:

``sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15``

  ![2_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/302d207a-7d58-494e-9171-eb3e473556f6)

  ![2_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/30548391-8ad6-4c88-a998-134a8326c840)

<br/>

>**3. настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины**

Перед настройкой кластера принимаю решение сделать тест на производительность с дефольными настройками Postgres, поэтому сначала инициализирую ``pgbench`` в ранее созданную БД ``benchmark``командой:

``sudo -u postgres pgbench -c 50 -j 2 -P 10 -T 60 benchmark``

  ![3_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/a8abd872-219b-47d5-b749-33079e81c861)

И провожу тестирование на настройках Postgres по умолчанию командой:

``sudo -u postgres pgbench -c 50 -j 2 -P 10 -T 60 benchmark``

  ![3_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/70ad4f57-d1f9-4bea-b6c7-632b02dfff1b)

<br/>

Теперь согласно заданию, настраиваю кластер PostgreSQL на максимальную производительность, используя сайт: https://pgconfigurator.cybertec.at/ , где вношу параметры моей ВМ: **CPU: 2, RAM: 4 ГБ, Объём дискового пространства: 10 ГБ (SSD)** , так же принимаю решение поставить ``fsync = off`` и получаю во такой список параметров:

```
-- Connectivity
max_connections = 100
superuser_reserved_connections = 3

-- Memory Settings
shared_buffers = '1024 MB'
work_mem = '32 MB'
maintenance_work_mem = '320 MB'
huge_pages = off
effective_cache_size = '3 GB'
effective_io_concurrency = 100 # concurrent IO only really activated if OS supports posix_fadvise function
random_page_cost = 1.25 # speed of random disk access relative to sequential access (1.0)

-- Replication
wal_level = minimal # consider using at least 'replica'
max_wal_senders = 0
synchronous_commit = off
fsync = off

-- Checkpointing:
checkpoint_timeout = '15 min'
checkpoint_completion_target = 0.9
max_wal_size = '1024 MB'
min_wal_size = '512 MB'

-- WAL writing
wal_compression = off
wal_buffers = -1 # auto-tuned by Postgres till maximum of segment size (16MB by default)


-- Background writer
bgwriter_delay = 200ms
bgwriter_lru_maxpages = 100
bgwriter_lru_multiplier = 2.0
bgwriter_flush_after = 0

-- Parallel queries:
max_worker_processes = 2
max_parallel_workers_per_gather = 1
max_parallel_maintenance_workers = 1
max_parallel_workers = 2
parallel_leader_participation = on
```

Далее я создаю файл ``pgconfigurator.conf`` в каталоге ``/etc/postgresql/15/main/conf.d/``, куда записываю все эти параметры, командой:

``nano /etc/postgresql/15/main/conf.d/pgconfigurator.conf``

далее рестартую сервер(так как есть параметры, которые для применения требуют рестарта сервера, например ``shared_buffers``, все ок:

  ![4_3](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/f3fbbd97-785b-48f2-965d-a53715d6072b)

  ![4_4](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/43b80e68-9975-4cd8-8d71-baf615be6e79)



>**4. Нагрузить кластер через утилиту через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)**

И снова провожу тестирование с помощью ``pgbench``, но уже на новых настройках Postgres  командой:

``sudo -u postgres pgbench -c 50 -j 2 -P 10 -T 60 benchmark``

  ![5_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/7d47b59b-97ca-42b9-8ad7-b5e4b01690df)

<br/>

>**5. Написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему**

В ходе тестов вижу заметное увеличение производительности в ~3.1 раза!!!! Ранее ``tps``(показатель пропускной способности базы данных, который показывает количество транзакций, обработанных базой данных за одну секунду) был ``tps = 754.801471 ``, а теперь ``tps = 2414.908549 ``! Как писала выше, изменили много дефолтных параметров кластера PostgreSQL, но так как в задании не оговаривалось - какой тип приложения будет работать с БД - брала mixed, на продуктовой БД - думаю, надо уже более прицельно рассматривать параметры ``shared_buffers``, ``shared_buffers``, настройки ``WAL``  -  в зависимости от типа приложения - WEB, OLTP, DWH и тп.
Так же обращаю внимание, что в данном тесте - в силу задания, как было озвучено: **не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины** - я выставила достаточно серьезные параметры: **synchronous_commit = off**, **fsync = off**, что на продуктовых средах делать не стоит, так как я помню, что именно отключение этих параметров дает возможность заметного увеличения быстродействия, но именно ценой вопроса надежности работы кластера в случае сбоев и возможности потери транзакционных данных. Так как я помню, что В режиме асинхронного подтверждения(**synchronous_commit = off**) сервер сообщает об успешном завершении сразу, как только транзакция будет завершена логически, прежде чем сгенерированные записи WAL фактически будут записаны на диск. Это может значительно увеличить производительность при выполнении небольших транзакций, но существует короткое окно между отчётом о завершении транзакции для клиента и временем, когда транзакция реально подтверждена (т. е. гарантируется, что она не будет потеряна в случае краха сервера). Таким образом, асинхронное подтверждение транзакций не должно использоваться, если клиент будет выполнять внешние действия, опираясь на предположение, что транзакция будет сохранена.То есть 
 асинхронное подтверждение транзакций приводит к риску потери данных.Риск потери данных при использовании асинхронного подтверждения транзакций — это не риск повреждения данных. Если случился крах СУБД, она будет восстановлена путём воспроизведения WAL до последней записи, которая была записана на диск. Таким образом, будет восстановлено целостное состояние СУБД, но любые транзакции, которые ещё не были сохранены на диск, в этом состоянии не будут отражены. Чистый эффект будет заключаться в потере нескольких последних транзакций. Поскольку транзакции воспроизводятся в том же порядке, в котором подтверждались, воспроизведение не нарушает целостность. (Это выдержка с сайта , сохранила целоком в том числе и на будущее для себя - https://postgrespro.ru/docs/postgresql/15/wal-async-commit).
А параметр **fsync = off** может привести к краху всего кластера, так как настройка fsync касается всего сервера и может изменить поведение всех транзакций. Она выключает всю логику внутри PostgreSQL, которая пытается синхронизировать запись отдельных порций в базу данных и, таким образом, крах системы (обусловленный отказом аппаратного обеспечения или операционной системы, который не является сбоем самой СУБД PostgreSQL ) может в результате привести к повреждению состояния базы данных.
Отключение fsync часто даёт выигрыш в скорости, но это может привести к неисправимой порче данных в случае отключения питания или сбоя системы. Поэтому отключать fsync рекомендуется, только если вы легко сможете восстановить всю базу из внешнего источника. (https://postgrespro.ru/docs/postgresql/15/runtime-config-wal)

<br/>

***
**<h3> Задание со * :**
<br>аналогично протестировать через утилиту https://github.com/Percona-Lab/sysbench-tpcc 
<br>(требует установки https://github.com/akopytov/sysbench)
</h3>

***

**Выполнение:**

1. Устанавливаю ``sysbench`` командами:

``$ curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash``
<br>``$ sudo apt -y install sysbench``

  ![6_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/7c59b9d2-4280-4abf-bb02-d077c6251919)

<br/>

2. Клонирую скрипты sysbench-tpcc командой:
 
``git clone https://github.com/Percona-Lab/sysbench-tpcc``

  ![6_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/097d7bd2-733e-46e1-b7d5-0e14c18b878f)

<br/>


***
