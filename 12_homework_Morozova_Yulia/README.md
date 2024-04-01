**<div align="center"><h2>Морозова Юлия, OTUS, группа Postgre-DBA-2023-11</h2></div>**

**<div align=center><h3>Домашнее задание №12 по теме:</h3></div>**
**<div align=center><h3>«Сбор и использование статистики.»</h3></div>**

***
**<h3>Домашнее задание:
<br>Работа с join'ами, статистикой</h3>**

**<h3>Цель:
<br> - знать и уметь применять различные виды join'ов
<br> - строить и анализировать план выполенения запроса
<br> - оптимизировать запрос
<br> - уметь собирать и анализировать статистику для таблицы.</h3>**

***

**Выполнение:**

**Подготовительные работы**

Для выполнения этого домашнего задания я использую демонстрационная базу данных «Авиаперевозки» : https://postgrespro.ru/education/demodb

1. Сначала скачиваю архив этой БД ``demo`` и распаковываю БД:

``wget https://edu.postgrespro.ru/demo-small.zip`` 
</br>``unzip demo-small.zip``

![0_!](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/a0ff12f7-5c99-4801-a794-83d0e25bd126)

2. Далее, согласно инструкции по установке этой БД, запускаю SQL-скрипт ``demo-small-20170815.sql``(он создает бд ``demo`` и наполняет её данными - фактически, это резервная копия, созданная утилитой ``pg_dump``):

```sql
  \i  demo-small-20170815.sql
```

![0_7](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/67e4e084-b5a4-42e1-b159-ce57c9efec95)

![0_8](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/75c04d9d-6cd0-4014-839c-e6e1be675036)



<br/>  

***

<br/>

>**1. Реализовать прямое соединение двух или более таблиц.**


Это реализую запросом, который показывает название аэропорта, город в котором он находится и количество вылетевших рейсов на дату '2017-08-21':

```sql
  SELECT a.city,
       a.airport_name,
       count(f.status)
FROM bookings.airports a,
     bookings.flights f
WHERE a.airport_code = f.departure_airport
  AND f.scheduled_departure::DATE = '2017-08-21'
GROUP BY city,
         airport_name;
```

![1_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/ebbb7d60-fd22-40c6-a48a-80ae0f6ca130)

<br/>

>**2. Реализовать левостороннее (или правостороннее) соединение двух или более таблиц.**


Для этого беру запрос, который показывает все вылеты(общее количество) из каждого аэропорта за все время:
```sql
SELECT a.airport_name,
       count(f.flight_id)
FROM airports a
LEFT JOIN flights f ON a.airport_code = f.departure_airport
GROUP BY 1
ORDER BY 2 DESC;
```

![2_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/6f8bc8b6-d493-48a9-8caf-c525815a0af1)

<br/>

>**3. Реализовать кросс соединение двух или более таблиц.**

Этот тип соединения покажу на запросе , который показывает все возможные направления перелетов из всех аэропортов:

```sql
SELECT a1.airport_name,
       a2.airport_name
FROM airports a1
CROSS JOIN airports a2
WHERE a1.airport_code <> a2.airport_code
ORDER BY a1.airport_name;
```

![3_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/e46e0698-7703-43aa-bce6-43404c27bc5f)

<br/>

>**4. Реализовать полное соединение двух или более таблиц.**


Полное соединение реализуем запросом, который показывает количество мест в каждой модели самолета:

```sql
SELECT a.model,
       count(s.seat_no)
FROM seats s
FULL JOIN aircrafts a ON a.aircraft_code = s.aircraft_code
GROUP BY 1;
```

![4_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/c542afb8-df5f-4b9d-b5fe-b531880e370d)

<br/>

>**5. Реализовать запрос, в котором будут использованы разные типы соединений.**

Это задание покажу на запросе, который считает количество пассажиров, которые приобрели билеты, но по каким-то причинам не пришли на регистрацию:

```sql
SELECT count(*)
FROM (ticket_flights t
      JOIN flights f ON t.flight_id = f.flight_id)
LEFT JOIN boarding_passes b ON t.ticket_no = b.ticket_no
AND t.flight_id = b.flight_id
WHERE f.actual_departure IS NOT NULL
  AND b.flight_id IS NULL;
```

В БД таких пассажиров нет:

![5_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/7d8883c1-ef7f-4e93-9ad9-166b6d4e80f8)

<br/>

>**6. Сделать комментарии на каждый запрос.**



<br/>

>**7. К работе приложить структуру таблиц, для которых выполнялись соединения.**



<br/>


***
**<h3> Задание со * :**
<br>Придумайте 3 своих метрики на основе показанных представлений, отправьте их через ЛК, а так же поделитесь с коллегами в слаке. 
<br> 
<br>
</h3>

***

**Выполнение:**

1. Метрика: длительность текущих активных транзакций и запросов.

```sql
SELECT datname,
       usename,
       now() - xact_start AS TransactionDuration,
       now() - query_start AS QueryDuration
FROM pg_stat_activity
WHERE state = 'active';
```

![7_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/16dc2675-d73f-45b8-8db3-a6c22f9d67a8)


2. 

***




