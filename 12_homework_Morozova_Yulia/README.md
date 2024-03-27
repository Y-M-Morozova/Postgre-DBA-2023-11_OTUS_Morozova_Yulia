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



<br/>

>**4. Реализовать полное соединение двух или более таблиц.**


<br/>

>**5. Реализовать запрос, в котором будут использованы разные типы соединений.**



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


***




