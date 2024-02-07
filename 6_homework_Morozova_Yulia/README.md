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

>**1. Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB**

В ЯО создала новую ВМ для этого домашнего задания, но размер диска не сделала 30 ГБ(ЯО не дал возможности создать меньше, думаю, что это не критично):

  ![1_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/46d076d6-1940-41bf-a039-38c5cef40f9d)

<br/>

>**2. Установить на него PostgreSQL 15 с дефолтными настройками**

Подключаюсь к этой ВМ в ЯО по ``ssh`` с помощью putty(ключи public , private генерирую с помощью puttygen) и устанавливаю Postgres 15й версии с настройками по умолчанию командами и далее проверяю:

``sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15``

  ![2_2](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/eb835560-0e8b-49ae-9237-e9b4a8263c54)

<br/>

>**3. Создать БД для тестов: выполнить pgbench -i postgres**

выполняю: ``sudo -u postgres pgbench -i postgres``

  ![3_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/afa29395-577d-4171-9087-6ea90a068b2e)

  <br/>

>**4. Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres**

выполняю: ``sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres``

  ![4_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/0e8c2707-f39f-4a4d-be50-0b9845c94bf8)

  <br/>

>**5. Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла**


Сначала смотрю текущие значения параметров, какие буду менять далее, смотрю запросом:

```sql
    select name, setting, context from pg_settings where name in(
      'max_connections',
      'shared_buffers',
      'effective_cache_size',
      'maintenance_work_mem',
      'checkpoint_completion_target',
      'wal_buffers',
      'default_statistics_target',
      'random_page_cost',
      'effective_io_concurrency',
      'work_mem',
      'min_wal_size',
      'max_wal_size');
```

сейчас они по умолчанию:  
  ![5_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/a78cbf99-1057-447d-97c3-da2b8400bdda)

выставляю их (согласно заданию) скриптом:

  ```sql
    ALTER SYSTEM SET
    max_connections='40';
    ALTER SYSTEM SET
    shared_buffers='1GB';
    ALTER SYSTEM SET
    effective_cache_size='3GB';
    ALTER SYSTEM SET
    maintenance_work_mem='512MB';
    ALTER SYSTEM SET
    checkpoint_completion_target='0.9';
    ALTER SYSTEM SET
    wal_buffers='16MB';
    ALTER SYSTEM SET
    default_statistics_target='500';
    ALTER SYSTEM SET
    random_page_cost='4';
    ALTER SYSTEM SET
    effective_io_concurrency='2';
    ALTER SYSTEM SET
    work_mem='6553kB';
    ALTER SYSTEM SET
    min_wal_size='4GB';
    ALTER SYSTEM SET
    max_wal_size='16GB';
  ```

  ![5_2](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/5e26386e-585b-4e99-aa5b-5e0c95778cbf)

Так как среди измененных есть параметры, требующие рестарта, то сначала выполняю команду: 
<br> ``sudo -u postgres pg_ctlcluster 15 main restart``, потом захожу в postgres и проверяю параметры скриптом:

```sql
    select name, setting, unit, context from pg_settings where name in(
      'max_connections',
      'shared_buffers',
      'effective_cache_size',
      'maintenance_work_mem',
      'checkpoint_completion_target',
      'wal_buffers',
      'default_statistics_target',
      'random_page_cost',
      'effective_io_concurrency',
      'work_mem',
      'min_wal_size',
      'max_wal_size');
```

и все ок:

![5_4](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/0a4e866a-df0d-4751-87aa-8780dfe42ed0)

<br/>
  
>**6. Протестировать заново**

выполняю: ``sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres``

  ![image](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/75d03063-a4ea-4709-b9ad-1ab77d5126cc)

<br/> 

>**7. Что изменилось и почему?**

Наблюдаю значительный прирост производительности: увеличение порядка 25-26% показателя tps(показатель пропускной способности БД - показывает количество транзакций, обработанных базой за одну секунду) и так же вижу снижение latency почти на 20% (время выполнения для каждого оператора).
<br>Этот прирост объясняю тем, что postgres настроен теперь более оптимально и согласно текущей конфигурации оборудования (увеличен буффер разделяемой памяти, доступной серверу, увеличен дисковый кеш запроса, увеличен maintenance_work_mem, увеличено допустимое число параллельных операций ввода/вывода). 

То есть новые настройки позволят нам использовать оба ядра, больше памяти для сортировки, больше дискового кеша для запросов, возможность эффективной параллельной работы.

<br/> 

>**8. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк**

выполняю скриптом:
```sql
  CREATE TABLE test_autovacuum(txt_data text);
  INSERT INTO test_autovacuum(txt_data) SELECT substr(md5(random()::text), 0, 10) FROM generate_series(1,1000000);
```

  ![8_12](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/bb09d985-a57c-4674-a619-8de299c0fd97)

<br/>

>**9. Посмотреть размер файла с таблицей**

выполняю командой:
```sql
  select pg_size_pretty(pg_total_relation_size('test_autovacuum'));
```

  ![9_12](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/e02b30f0-c7a4-4849-a5a0-db4ff8da4346)

<br/>

>**10. 5 раз обновить все строчки и добавить к каждой строчке любой символ**

сначала обновляю 5 раз все строки, для этого запускаю 5 раз команду:

```sql
  UPDATE test_autovacuum SET txt_data = md5(random()::text);
```
  
  ![9_2](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/69ca2d38-e899-43f2-b4e9-091a9834711a)

а теперь добавляю к каждой строке любой символ командой:

```sql
  UPDATE test_autovacuum SET txt_data = txt_data || substr(md5(random()::text), 1, 1);
```

  ![9_3](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/36a38725-6669-4952-855b-dfed2006509f)

<br/>

>**11. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум**

Смотрю количество мертвых строк в таблице и время последнего автовакуума командой:

```sql
  SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_tables WHERE relname = 'test_autovacuum';
```

  ![10_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/568f4f29-8e43-45d4-935a-da7c5e89edea)

<br/>

>**12. Подождать некоторое время, проверяя, пришел ли автовакуум**

В моем случае ждать не пришлось, автовакуум сработал пока я выполняла задание, поэтому количество мертвых строк у меня = 0, и время последнего автовакуума было ранее, чем я выполняла запрос.

<br/>

>**13. 5 раз обновить все строчки и добавить к каждой строчке любой символ**

снова выполняю:

сначала обновляю 5 раз все строки, для этого запускаю 5 раз команду:

```sql
  UPDATE test_autovacuum SET txt_data = md5(random()::text);
```
  
 а теперь добавляю к каждой строке любой символ командой:

```sql
  UPDATE test_autovacuum SET txt_data = txt_data || substr(md5(random()::text), 1, 1);
```

  ![13_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/f1fec07f-d36d-4bba-ae01-fe8a1e8761f1)

<br/>

и смотрю - в этот раз автовакуум еще не успел отработать и вижу мертвые строки!

  ![13_3](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/b3852925-d556-471f-bb13-170591f955ea)

<br/>

>**14. Посмотреть размер файла с таблицей**

выполняю командой:
```sql
  select pg_size_pretty(pg_total_relation_size('test_autovacuum'));
```

  ![14_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/0be00b11-61fc-488e-b9af-49624c89e091)

<br/>

>**15. Отключить Автовакуум на конкретной таблице**

выполняю командой:
```sql
  alter table test_autovacuum set (autovacuum_enabled = off);
```

![15_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/6af271fd-8813-4d9f-a25c-7b5b2bd50cca)

<br/>

>**16. 10 раз обновить все строчки и добавить к каждой строчке любой символ**

снова выполняю:

сначала обновляю 10 раз все строки, для этого запускаю 10 раз команду:

```sql
  UPDATE test_autovacuum SET txt_data = md5(random()::text);
```
  
 а теперь добавляю к каждой строке любой символ командой:

```sql
  UPDATE test_autovacuum SET txt_data = txt_data || substr(md5(random()::text), 1, 1);
```

  ![16_3](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/23ce3619-f818-4cfa-9def-fcb862c9a66b)


<br/>

>**17. Посмотреть размер файла с таблицей**

выполняю командой:
```sql
  select pg_size_pretty(pg_total_relation_size('test_autovacuum'));
```

  ![17_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/b9da3795-3e4a-4441-a724-ebd7964a73a7)

<br/>

>**18. Объясните полученный результат**

Наблюдаю, что при включенном автовакууме "мертвые" строки удаляются, а при выключенном - нет. Так же наблюдаю прирост размера таблицы, особенно заметен при выключенном автовакууме - так как при включенном автовакууме , полагаю, что новые строки добавляются частично в "дыры"(автовакуум удаляет мертвые строки). Так же показательно видно, что при включенном автовакууме "мертвые" строки удаляются, а размер таблицы не уменьшается! Так как для этого(уменьшить размер таблицы) необходимо выполнить ``VACUUM FULL``! 

Так как мне это интересно попробовать, то делаю это(``VACUUM FULL``) и да, вижу, что размер таблицы уменьшается ! Но опять же помним - что ``VACUUM FULL`` на продуктовых базах надо делать осторожно - так как при этом мы получим сжатую таблицу, но по факту новую версию файла таблицы без неиспользуемого пространства. Это минимизирует размер таблицы, однако может занять много времени. Кроме того, для этого требуется больше места на диске для записи новой копии таблицы до завершения операции + наложит блокировку на таблицу).

  ![17_2](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/2d201bef-b38e-48fc-a153-901446324138)

<br/>

>**19. Не забудьте включить автовакуум)**

выполняю командой:
```sql
  alter table test_autovacuum set (autovacuum_enabled = on);
```

  ![19_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/6f44d93f-aed2-4f7b-bdd2-7075380e6ca2)

<br/>  


***
**<h3> Задание со * :
<br>Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
<br>Не забыть вывести номер шага цикла</h3>**
***

**Выполнение:**

текст процедуры:

```sql
  DO
  $do$
  DECLARE
     i int;
  BEGIN
     FOR i IN 1..10 LOOP
 	  UPDATE test_autovacuum SET txt_data = md5(random()::text);
      	  raise info 'номер шага цикла: %', i;
     END LOOP;
  END
  $do$;
```

  ![20_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/5ef80c2e-7ffe-46d0-9a7a-95dbcbed325c)

***

