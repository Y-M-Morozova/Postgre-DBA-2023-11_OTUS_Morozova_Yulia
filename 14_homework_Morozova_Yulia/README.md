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

- Сначала вставляю и проверяю имеющиеся данные в таблицу витрины:

```sql
insert into good_sum_mart (good_name,sum_sale)
SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;
select * from good_sum_mart;
```

![1_0](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/c4a1dc37-5ae8-4583-b8b1-a7189e7601dc)

- Теперь создаю триггер на вставку записей в таблицу ``sales``:

```sql
CREATE OR REPLACE FUNCTION add_into_sales() RETURNS TRIGGER AS
$BODY$
begin
	if  not exists (select 1 from good_sum_mart GM inner join goods G on G.good_name=GM.good_name where G.goods_id=new.good_id)
	then 
	INSERT into good_sum_mart(good_name,sum_sale) select good_name,0 from goods where goods_id=new.good_id;
	end if;
	update good_sum_mart set sum_sale=(SELECT sum(G.good_price * S.sales_qty) FROM goods G INNER JOIN sales S ON S.good_id = G.goods_id where G.goods_id = new.good_id)
	where good_name = (select GM.good_name from good_sum_mart GM inner join goods G on G.good_name=GM.good_name where G.goods_id=new.good_id);
	RETURN new;   
END;
$BODY$
language plpgsql;

CREATE TRIGGER triger_add_into_sales after INSERT ON sales FOR EACH row EXECUTE PROCEDURE add_into_sales();
```

![2_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/714c5caa-a1c4-40e2-b38c-4f4d2761a262)

<br/>

- Теперь создаю триггер для обновления наименования товара в  таблице ``goods``:

```sql
CREATE OR REPLACE FUNCTION update_name_goods() RETURNS TRIGGER AS
$BODY$
begin
	if  exists (select 1 from good_sum_mart GM inner join goods G on G.good_name=GM.good_name where G.goods_id=old.goods_id)
	then 
	update good_sum_mart set good_name = new.good_name where good_name in (select GM.good_name from good_sum_mart GM inner join goods G on G.good_name=GM.good_name where G.goods_id=old.goods_id);	
	end if;
	RETURN new;   
END;
$BODY$
language plpgsql;

CREATE TRIGGER triger_update_name_goods BEFORE UPDATE ON goods FOR EACH row EXECUTE PROCEDURE update_name_goods();
```

![3_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/902121c4-06cb-4476-b201-d7db0e21f251)

- Теперь создаю триггер для актуализации цен витрины:

```sql
CREATE OR REPLACE FUNCTION update_price_goods() RETURNS TRIGGER AS
$BODY$
begin
	if  exists (select 1 from good_sum_mart GM inner join goods G on G.good_name=GM.good_name where G.goods_id=old.goods_id)
	then 
	update good_sum_mart set sum_sale=(SELECT sum(G.good_price * S.sales_qty) FROM goods G INNER JOIN sales S ON S.good_id = G.goods_id where G.good_name = new.good_name)
	where good_name=new.good_name;
	end if;
	RETURN new;   
END;
$BODY$
language plpgsql;

CREATE TRIGGER triger_update_price_goods AFTER UPDATE ON goods FOR EACH row EXECUTE PROCEDURE update_price_goods();
```

![4_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/a3808f3e-7d42-4840-91e4-1c05f4c65975)

<br/>

**Проверка:**

- Проверяю добавление товара:

```sql
INSERT INTO goods (goods_id, good_name, good_price) VALUES (3, 'стол офисный', 3500);
INSERT INTO sales (good_id, sales_qty) VALUES (3, 2);

select * from goods;
select * from good_sum_mart;
```

![7_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/2261f377-ec62-46bd-88c6-985dbed5978b)

- Проверяю добавление продаж:

```sql
INSERT INTO sales (good_id, sales_qty) VALUES (1, 200), (2, 4), (3, 2);
select * from good_sum_mart;

SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;
```

![7_2!](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/61089627-e50f-4329-acd7-ad32619266e8)






***
**<h3> Задание со * :**
<br>Чем такая схема (витрина+триггер) предпочтительнее отчета, создаваемого "по требованию" (кроме производительности)?
<br> Подсказка: В реальной жизни возможны изменения цен. 
</h3>

***

**Выполнение:**





***





