**<div align="center"><h2>Морозова Юлия, OTUS, группа Postgre-DBA-2023-11</h2></div>**

**<div align=center><h3>Домашнее задание №13 по теме:</h3></div>**
**<div align=center><h3>«Секционирование.»</h3></div>**

***
**<h3>Домашнее задание:
<br>Секционирование таблицы</h3>**

**<h3>Цель:
<br> - научиться секционировать таблицы.
<br> - Секционировать большую таблицу из демо базы flights.</h3>**


***

**Выполнение:**

**Подготовительные работы**

Для выполнения этого домашнего задания я использую демонстрационную базу данных «Авиаперевозки» : https://postgrespro.ru/education/demodb.
</br>(Cтруктуры таблиц, для которых выполнялись соединения в моем домашнем задании так же находятся на сайте: https://edu.postgrespro.ru/demo-20161013.pdf).

1. Сначала скачиваю архив этой БД ``demo`` и распаковываю БД:

``wget https://edu.postgrespro.ru/demo-big.zip`` 
</br>``unzip demo-big.zip``

![0_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/a6d4b958-ad8c-4af9-bcf5-caa54218d273)

2. Далее, согласно инструкции по установке этой БД, запускаю SQL-скрипт ``demo-big-20170815.sql``(он создает бд ``demo`` и наполняет её данными - фактически, это резервная копия, созданная утилитой ``pg_dump``):

```sql
  \i  demo-big-20170815.sql
```

![0_2_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/a15ec3b4-cfdb-4017-9180-2f10e60194a6)

![0_3](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/fd5e6656-6d07-484d-8c4c-deee5d4e8a5c)

<br/>  

***

>**Секционирование таблицы**

1. Для секционирования беру таблицу достаточного большого размера ``boarding_passes``, которая содержит информацию о посадочных талонах пассажиров:

![1_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/f7180259-50db-4579-8a92-267f8f4cbe7b)

Рассмотрю партицирование на основе индетификатора рейса(``flight_id``), партицирование на основе интервалов. 
</br>Всего в таблице 7 925 812 записей, а  значения столбца ``flight_id`` при этом находятся в интервале от 2 до 214867. Буду создавать одну секцию на каждые 10000 рейсов.

```sql
select count (*) from boarding_passes;
select min(flight_id) from boarding_passes; 
select max(flight_id) from boarding_passes;
```

![1_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/d524ad8d-204c-471b-9607-2754a6a4de3c)

<br/>


2. Создаю таблицу ``boarding_passes_part`` с секциами, идентичную по схеме оригинальной ``boarding_passes``:

```sql
CREATE TABLE bookings.boarding_passes_part (
	ticket_no bpchar(13) NOT NULL,
	flight_id int4 NOT NULL,
	boarding_no int4 NOT NULL,
	seat_no varchar(4) NOT NULL,
	CONSTRAINT boarding_passes_p_flight_id_boarding_no_key UNIQUE (flight_id, boarding_no),
	CONSTRAINT boarding_passes_p_flight_id_seat_no_key UNIQUE (flight_id, seat_no),
	CONSTRAINT boarding_passes_p_pkey PRIMARY KEY (ticket_no, flight_id),
	CONSTRAINT boarding_passes_p_ticket_no_fkey FOREIGN KEY (ticket_no,flight_id) REFERENCES bookings.ticket_flights(ticket_no,flight_id)
) PARTITION BY RANGE (flight_id);
```

![1_3](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/a885b390-c2e2-49bd-bf07-a71a46a67145)

<br/>

3. Теперь с помощью ``psql`` создаю секции:

```sql
select 'CREATE TABLE bookings.boarding_passes_part_' || i || ' PARTITION OF bookings.boarding_passes_part FOR VALUES FROM ('|| ps ||') TO ('|| pe ||');'
from (SELECT g as i, g*10000 as ps, (g+1)*10000 as pe FROM generate_series(0, (select max(bp.flight_id) from boarding_passes bp)/10000) g) sec
\gexec
```

скрипт создаст команды на выполнение:

![1_5_5](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/918fc195-5ef5-4cb0-96c0-b4431b311234)

и выполняю:

![1_5](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/3b3d19ae-285a-485d-97bf-12cac183a502)

<br/>

4. Так же создаю секцию по умолчанию, чтобы  избежать ошибок и потерь данных:

```sql
CREATE TABLE bookings.boarding_passes_part_ex PARTITION OF bookings.boarding_passes_part DEFAULT;
```

![1_7](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/41ea4f05-20eb-4a3c-9f3c-ed5e53c78fd1)

<br/>

5. Теперь смотрю и сравниваю планы запросов.
   Сначала беру запрос по исходной таблице, не партицированной ``boarding_passes`` :

   ```sql
	explain analyze
	select * from boarding_passes
	where flight_id > 120000 and flight_id < 175000;
   ```

![17_2_2_nopart](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/5d5974be-04c6-46d0-8742-ed59c37c6dc5)

Теперь этот же запрос выполню на партицированной таблице ``boarding_passes_part``:

   ```sql
	explain analyze
	select * from boarding_passes_part
	where flight_id > 120000 and flight_id < 175000;
   ```

![17_2_2_full](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/1ee71631-7eea-48f1-96a9-81e291ef0491)


Вижу, что на партицированной таблице запрос выполняется быстрее(``Execution Time: 298.652 ms``), и только по тем секциям, где есть соответствующие строки для выборки, а по исходной таблице выполняется дольше (``Execution Time: 345.739 ms``).

А теперь сравниваю планы запросов по партицированной таблице с параметром ``enable_partition_pruning`` (устранение секций — это приём оптимизации запросов, который ускоряет работу с декларативно секционированными таблицами.)
Сначала смотрю с включенным(по умолчанию) параметром план запроса:

```sql
SET enable_partition_pruning = on;
show enable_partition_pruning;
explain analyze
select * from boarding_passes_part
where flight_id > 120000 and flight_id < 125000;
```
![21_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/acf51d94-37a5-4d8a-aa7a-20d9f8dd3db6)

а теперь выключаем устранение секций:

```sql
SET enable_partition_pruning = off;
show enable_partition_pruning;
explain analyze
select * from boarding_passes_part
where flight_id > 120000 and flight_id < 125000;
```

![23_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/262dd86b-c68c-4670-a9ee-1873ec589ac4)

![23_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/cf2457e8-6058-48f8-8824-b2a402286282)

Вижу, что с включенным параметром устранения секций план запроса выгоднее - идет сканирвание только по секции , где находятся соответствующие строки и время выполнения меньше (`` Execution Time: 16.490 ms``), а с выключенным - идет сканирвание по всем строкам и время выполнения больше ( ``Execution Time: 22.123 ms``)

<br/>

***





