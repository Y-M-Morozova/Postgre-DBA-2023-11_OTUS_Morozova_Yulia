**<div align="center"><h2>Морозова Юлия, OTUS, группа Postgre-DBA-2023-11</h2></div>**

**<div align=center><h3>Домашнее задание №11 по теме:</h3></div>**
**<div align=center><h3>«Виды индексов. Работа с индексами и оптимизация запросов»</h3></div>**

***
**<h3>Домашнее задание:
<br>Работа с индексами</h3>**

**<h3>Цель:
<br>создать индексы;
<br>научиться использовать различные виды индексов;
<br>прочитать вывод команду explain;
<br>оптимизировать скорость работы запросов.</h3>**

***

**Выполнение:**

**Подготовительные работы**

***

- Создаю тестовую таблицу ``test`` скриптом:

```sql
create table test as
select 
    generate_series as id
  , generate_series::text || (random() * 10)::text as txt
  , (array['first', 'second', 'third', 'four', 'fif'])[floor(random() * 5 + 1)] as array
  , random() > 0.5 as random_boolean
from generate_series(1, 100000);
```

![0_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/34004f5a-d1de-498a-bb29-10003b021024)

- Собираю статистику по таблице ``test``:

```sql
    analyze test;
    select attname, correlation from pg_stats where tablename = 'test';
```

![0_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/eca810e0-90fa-486b-8401-dc863240304d)

<br/>  

***

<br/>

>**1. Создать индекс к какой-либо из таблиц вашей БД**

- Сначала смотрю план выполнения запроса без индекса по полю ``id``:
```sql
    explain (analyze,buffers) select id from test;
```

![1_1_super2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/e2467507-6443-46d5-a8d7-6c20f5751924)

- Ожидаемо вижу, что в данном случае у меня последовательное сканирование по таблице(``seq scan``)
- Cтоимость получения первой строки = 0
- ``shared_hit`` (кол-во страниц на диске/в памяти), которые необходимо прочитать = 818 (``relpages``)
- Cтоимость получения всех строк=shared_hit(818)*seq_page_cost(1)+rows(100000)*cpu_tuple_cost(0.01)=1818
- Ширина полученного массива данных (width) = 4
- Время получения (``actual_time``) = первой строки (0.008)...всех строк(8.125)
- Строк отдано (``rows``) = 100000

<br/>

- Теперь создаю индекс по полю ``id``:

```sql
    create index "idx_id" on test ( id );
```

![1_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/f671e962-a3c3-49ab-8f76-626d58572fea)

<br/>

>**2. Прислать текстом результат команды explain, в которой используется данный индекс**

- Теперь смотрю план выполнения запроса с созданным индексом по полю ``id``:
```sql
    explain (analyze,buffers) select id from test where id = 7;
```

![2_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/60ef049c-4433-43cf-b09f-c2ddd4cab779)

```
test=# explain (analyze,buffers) select id from test where id = 7;
                                                    QUERY PLAN
------------------------------------------------------------------------------------------------------------------
 Index Only Scan using idx_id on test  (cost=0.29..4.31 rows=1 width=4) (actual time=0.022..0.023 rows=1 loops=1)
   Index Cond: (id = 7)
   Heap Fetches: 0
   Buffers: shared hit=1 read=2
 Planning:
   Buffers: shared hit=15 read=1 dirtied=2
 Planning Time: 0.140 ms
 Execution Time: 0.034 ms
(8 rows)
```

<br/>

>**3. Реализовать индекс для полнотекстового поиска**



<br/>

>**4. Реализовать индекс на часть таблицы или индекс на поле с функцией**



<br/>

>**5. Создать индекс на несколько полей**



<br/>

>**6. Написать комментарии к каждому из индексов**



<br/>

>**7. Описать что и как делали и с какими проблемами столкнулись**



<br/>

***
**<h3> Задание со * :**
<br>Реализовать горячее реплицирование для высокой доступности на 4ВМ. 
</h3>

***

**Выполнение:**


***




