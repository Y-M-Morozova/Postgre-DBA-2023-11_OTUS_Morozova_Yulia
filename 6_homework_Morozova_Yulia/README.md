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



<br/>

>**3. Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.**



  <br/>

>**4. Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?**



  <br/>

>**5. Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.**



<br/>
  
>**6. Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите кластер и сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и продолжить работу?**



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

