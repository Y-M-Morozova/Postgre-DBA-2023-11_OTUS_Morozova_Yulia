**<div align="center"><h2>Морозова Юлия, OTUS, группа Postgre-DBA-2023-11</h2></div>**

**<div align=center><h3>Домашнее задание №6 по теме:</h3></div>**
**<div align=center><h3>«Журналы»</h3></div>**

***
**<h3>Домашнее задание:
<br>Работа с журналами</h3>**

**<h3>Цель:
<br>уметь работать с журналами и контрольными точками
<br>уметь настраивать параметры журналов.</h3>**

***

**Выполнение:**

>**1. Настройте выполнение контрольной точки раз в 30 секунд.**

Проверяю текущее значение командой ``checkpoint_timeout``:

```sql
  select name, setting, unit, context from pg_settings where name = 'checkpoint_timeout';
```
  ![1_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/5e71cd77-70b8-4287-817f-d9819f03d938)

Устанавливаю таймаут 30 сек. контрольной точки командой:

```sql
  alter system set checkpoint_timeout to '30s';
```

  ![1_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/cc398446-10ff-4914-a6e3-b56769215f7e)

Так как изменение этого параметра требует обновить (перечитать) конфигурацию (значение поля ``context`` вижу, что``sighup``), то выполняю команду:

```sql
  SELECT pg_reload_conf();
```

  ![1_3](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/b07236eb-46ab-4405-8384-3839da657ac3)

И снова проверяю текущее значение командой ``checkpoint_timeout``, все ок:

```sql
  select name, setting, unit, context from pg_settings where name = 'checkpoint_timeout';
```

  ![1_4](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/7d67094c-f69a-43dc-994e-9a7dbfffc53f)
  
<br/>

>**2. 10 минут c помощью утилиты pgbench подавайте нагрузку.**

Создаю для тестов БД ``wal_test`` командой:

```sql
  CREATE DATABASE wal_test;
```

![2_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/52dc566d-2604-4d45-88b8-3d2e37180292)

Инициализирую ``pgbench`` командой: 

``pgbench -i wal_test;``

  ![2_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/0f49294d-6c3c-4688-a2d1-f08ed9d32476)

Сбрасываю статистику ``bgwriter`` командой:

```sql
  select pg_stat_reset_shared('bgwriter');
```

  ![2_3](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/a7854ea9-8631-40f2-a981-1273eb1c581e)

Смотрю ``lsn`` (позицию в журнале) до нагрузки командой:

```sql
  SELECT pg_current_wal_insert_lsn();
```

  ![2_4](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/c1845d32-c9bb-48a5-83f8-fe5bb3a7ac3c)

Запускаю нагрузочное тестирование согласно заданию на 10 мин командой:

```pgbench -c8 -P 6 -T 600 wal_test```

  ![2_5](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/3c31bf41-322f-4b0a-b9ad-ea21c2cfe6a8)

  ![2_6](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/b3ab90cc-635d-4974-aa54-4652903b4c85)

<br/>

>**3. Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.**

Смотрю ``lsn`` (позицию в журнале) после нагрузки командой:

```sql
  SELECT pg_current_wal_insert_lsn();
```

  ![3_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/946069a2-ddcd-4619-b91a-50aaa0299c9c)

Смотрю какой объем журнальных файлов был сгенерирован за это время командой:

```sql
  SELECT pg_size_pretty('0/20557598'::pg_lsn - '0/25DA440'::pg_lsn);
```

  ![3_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/c1613999-e02a-4d7c-8400-1d224e3b64ba)


  <br/>

Чтобы оценить объем , который в среднем приходится на одну контрольную точку, рассуждаю так - объем журнальных файлов у меня **479 Mb**, а нагрузки было **10 мин**, выполнение контрольной точки должно быть 1 раз в **30 секунд**, то есть объем делю на 20 и получаю, что объем , который приходится в среднем на одну контрольную точку равен:
**479/20 = 22,35 Mb**

<br/>

>**4. Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?**

Cмотрю статистику командой:

```sql
  select * from pg_stat_bgwriter\gx
```

  ![4_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/a7181042-273b-465b-bc3c-90d94e454939)

И согласно стастистике вижу, что в данном интервале времени все контрольные точки выполнялись по расписанию! 
<br>Параметр ``checkpoints_req `` = 0 (количество запрошенных контрольных точек, которые уже были выполнены).
<br>Это говорит о равномерности потока журнальных записей в данном промежутке времени.
  
  <br/>

>**5. Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.**

Для сравнения tps в при работе postgres в режимах синхронного и асинхронного подтверждения транзакций , сначала проверю, что текущий режим у меня синхронный(по умолчанию).
Проверяю командой:

```sql
  show synchronous_commit;
```

  ![5_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/e840fd81-adef-467a-a8ff-aaf542096459)

Далее запускаю нагрузочное тестирование в режиме синхронного подтверждения транзакций на 10 мин командой:

``pgbench -c8 -P 6 -T 600 wal_test``

  ![5_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/5323b30d-14e3-4eb1-965d-50a3c08316ba)

  ![5_2_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/65e38115-98cd-4011-b43a-fb421a049ac7)


Для теста в асинхронном режиме подтверждения транзакций, сначала устанавливаю этот режим, далее обновляю конфигурацию postgres и проверяю текущий режим командами:

```sql
  alter system set synchronous_commit to 'off';
  select pg_reload_conf();
  show synchronous_commit;
```

  ![5_3](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/744a4f9c-81ee-4c2a-83de-098f33df2dd5)

И снова запускаю нагрузочное тестирование, но теперь уже в режиме асинхронного подтверждения транзакций на 10 мин командой:

```pgbench -c8 -P 6 -T 600 wal_test```

  ![5_3_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/c7bb54d3-0b4e-430e-84c2-7cf66004065d)

  ![5_3_3](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/83abbfdc-679b-4a5a-a0b1-fd311ed0936f)

И вижу, что ``tps``(показатель пропускной способности базы данных, который показывает количество транзакций, обработанных базой данных за одну секунду) гораздо больше(примерно в 4,8 раз больше!) при работе Postgres в режиме асинхронного подтверждения транзакций, где **tps=3189.184581** , а в синхронном режиме **tps=660.507523**. Это объясняется тем, что при синхронном режиме сервер ждёт сохранения записей WAL транзакции в постоянном хранилище(диск), прежде чем сообщить клиенту об успешном завершении, а при асинхронном режиме сервер сообщает об успешном завершении сразу, как только транзакция будет завершена логически, прежде чем сгенерированные записи WAL фактически будут записаны на диск. Это может значительно увеличить производительность, но надо помнить о том, что асинхронное подтверждение транзакций приводит к риску потери данных. Существует короткое окно между отчётом о завершении транзакции для клиента и временем, когда транзакция реально подтверждена.

<br/>

>**6. Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите кластер и сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и продолжить работу?**

Сначала создаю каталог для нового кластера ``/var/lib/postgresql/15/test``, делаю владельцем этого каталога пользователя ``postgres`` , проверяю  командами:

``sudo mkdir /var/lib/postgresql/15/test``
<br>``sudo chown -R postgres:postgres /var/lib/postgresql/15/test``
<br>``sudo ls -la /var/lib/postgresql/15/test``

  ![6_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/70e8ae8c-8ffe-4ff2-9742-cddd87390e90)
  

<br/> 






***
**<h3> Задание со * :
<br>Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
<br>Не забыть вывести номер шага цикла</h3>**
***

**Выполнение:**


***

