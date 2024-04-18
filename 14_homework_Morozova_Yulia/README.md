**<div align="center"><h2>Морозова Юлия, OTUS, группа Postgre-DBA-2023-11</h2></div>**

**<div align=center><h3>Домашнее задание №14 по теме:</h3></div>**
**<div align=center><h3>«Хранимые функции и процедуры.»</h3></div>**

***
**<h3>Домашнее задание:
<br>Триггеры, поддержка заполнения витрин</h3>**

**<h3>Цель:
<br> Научиться разрабатывать DML-триггеры и событийные триггеры.
<br> Создать триггер для поддержки витрины в актуальном состоянии.</h3>**

***
Описание/Пошаговая инструкция выполнения домашнего задания:

Скрипт и развернутое описание задачи – в ЛК (файл hw_triggers.sql) или по ссылке: https://disk.yandex.ru/d/l70AvknAepIJXQ

В БД создана структура, описывающая товары (таблица goods) и продажи (таблица sales).

Есть запрос для генерации отчета – сумма продаж по каждому товару.

БД была денормализована, создана таблица (витрина), структура которой повторяет структуру отчета.

Создать триггер на таблице продаж, для поддержки данных в витрине в актуальном состоянии (вычисляющий при каждой продаже сумму и записывающий её в витрину)

Подсказка: не забыть, что кроме INSERT есть еще UPDATE и DELETE.

***

**Подготовительные работы:**

```sql
-- ДЗ тема: триггеры, поддержка заполнения витрин

DROP SCHEMA IF EXISTS pract_functions CASCADE;
CREATE SCHEMA pract_functions;

SET search_path = pract_functions, publ

-- товары:
CREATE TABLE goods
(
    goods_id    integer PRIMARY KEY,
    good_name   varchar(63) NOT NULL,
    good_price  numeric(12, 2) NOT NULL CHECK (good_price > 0.0)
);
INSERT INTO goods (goods_id, good_name, good_price)
VALUES 	(1, 'Спички хозайственные', .50),
		(2, 'Автомобиль Ferrari FXX K', 185000000.01);

-- Продажи
CREATE TABLE sales
(
    sales_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    good_id     integer REFERENCES goods (goods_id),
    sales_time  timestamp with time zone DEFAULT now(),
    sales_qty   integer CHECK (sales_qty > 0)
);

INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);

-- отчет:
SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;

-- с увеличением объёма данных отчет стал создаваться медленно
-- Принято решение денормализовать БД, создать таблицу
CREATE TABLE good_sum_mart
(
	good_name   varchar(63) NOT NULL,
	sum_sale	numeric(16, 2)NOT NULL
);
```

![0_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/d3dd7539-87da-4732-8798-ec84313fa67c)

<br/>

**Выполнение:**








>**1. Реализовать прямое соединение двух или более таблиц.**




<br/>

>**2. Реализовать левостороннее (или правостороннее) соединение двух или более таблиц.**



<br/>

>**3. Реализовать кросс соединение двух или более таблиц.**



<br/>

>**4. Реализовать полное соединение двух или более таблиц.**


<br/>

>**5. Реализовать запрос, в котором будут использованы разные типы соединений.**



<br/>

***
**<h3> Задание со * :**
<br>Чем такая схема (витрина+триггер) предпочтительнее отчета, создаваемого "по требованию" (кроме производительности)?
<br> Подсказка: В реальной жизни возможны изменения цен. 
</h3>

***

**Выполнение:**

**Метрика: длительность текущих активных транзакций и запросов.**



***





