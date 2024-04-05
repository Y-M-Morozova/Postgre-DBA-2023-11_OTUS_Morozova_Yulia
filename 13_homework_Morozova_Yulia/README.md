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

<br/>

4. Так же создаю секцию по умолчанию, чтобы  избежать ошибок и потерь данных:

```sql
CREATE TABLE bookings.boarding_passes_part_ex PARTITION OF bookings.boarding_passes_part DEFAULT;
```


<br/>

>**5.**



<br/>


***
**<h3> Задание со * :**
<br>Придумайте 3 своих метрики на основе показанных представлений, отправьте их через ЛК, а так же поделитесь с коллегами в слаке. 
<br> 
<br>
</h3>

***

**Выполнение:**


***





