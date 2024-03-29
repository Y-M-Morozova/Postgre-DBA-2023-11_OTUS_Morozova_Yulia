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

- Здесь вижу, что используется ``Index Only Scan`` - очень быстрая операция, индекс включает все необходимые для выборки поля( в моем случае это только ``id`` и дополнительный доступ к таблице не требуется), отсюда вижу весомое уменьшение стоимости запроса и времени выполнения.

<br/>

>**3. Реализовать индекс для полнотекстового поиска**

- так как полнотекстный поиск обычно применяют по полям типа text и json, то сначала оценю поиск по полю ``array``:

```sql
  explain (ANALYZE, BUFFERS) select * from test where test.array like '%third%';
```
- ожидаемо, вижу, что в данном случае у меня последовательное сканирование по таблице(``seq scan``) и высокая стоимость запроса:

![3_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/10cdf705-4464-411c-90a5-e08e66612a0e)

- Далее я для индекса создаю столбец типа ``tsvector`` , заполняю данными и создаю индекс на поле с типом ``ts_vector``:

```sql
    alter table test add column array_tsvector tsvector;
    update test set array_tsvector = to_tsvector('english', test.array);
    create index idx_array_tsvector ON test USING GIN (array_tsvector);
```

![3_3](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/d524626c-7ad1-43ba-abca-d74c79b83392)


- И снова смотрю план запроса:
  
```sql
    explain (ANALYZE, BUFFERS) select * from index_test where array_tsvector @@ to_tsquery('third');
```

![3_4](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/74ea81e6-b21e-43a4-8557-c9c849bb87d2)

- теперь вижу, что выполняется сканирование уже по битовой карте, так же видно, что количество блоков, считанных из кэша  = 6 для узла с индексом и кол-во блоков из кеша  = 1037 для построения индекса по битовой карте. 
- Время выполнения уменьшилось в  ~ 2 раза! 
<br/>

>**4. Реализовать индекс на часть таблицы или индекс на поле с функцией**

- Создаю частичный индекс с условием ``id``<100:

```sql
    create index "idx_id_less" on test( id ) where id < 100;
```

![4_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/826faefc-75d1-4154-b0da-4f09c40e9c57)

- сравниваю разницу в размерах индексов:
  
```sql
    select pg_size_pretty(pg_total_relation_size('idx_id')); 
    select pg_size_pretty(pg_total_relation_size('idx_id_less')); 
```

![4_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/03703db5-602a-4efe-bf42-8d6b0577e6c4)

- Да, разница в размерах индексов ощутима.

<br/>

>**5. Создать индекс на несколько полей**

- Сначала смотрю план запроса с условием по двум полям, далее создаю индекс по двум полям и снова анализирую план запроса:

```sql
    explain (analyze) select * from test where id<1000 and random_boolean = TRUE ;
    create index "idx_id_random_boolean" on test( id,random_boolean );
    explain (analyze) select * from test where id<1000 and random_boolean = TRUE ;
```

![5_5](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/41019c6e-ab9d-4b8e-bd4d-eab916b9ea6e)

- Вижу, что в первом плане - запрос с высокой стоимостью (ожидаемо, так как в данном случае у меня последовательное сканирование по таблице - ``seq scan``), а после создания индекса вижу, что стоимость запроса существенно ниже и время выполнения меньше, так как план запроса - с индексаным сканированием (``Index Scan``).

<br/>

>**Описать что и как делали и с какими проблемами столкнулись**

В команде ``EXPLAIN`` пользовала опции  (``analyze,buffers``) для более информативного вывода плана запроса, но помню, что c опцией ``analyze`` реально выполняется оператор, что может повлечь изменение данных (например в командах ``update``, ``insert`` и тп). То есть использовать в продуктивных и тп базах - недопустимо.
Так же при создании таблицы некорректно назвала один из столцбов - ``array`` - так как это тестовая среда , но хотела посмотреть - как отреагирует postgres на использование служебного зарезервированного слова в имени сотлбца. Создать таблицу с таким столбцом дал, но вот в операторах селекта стал давать ошибку - вышла из ситуации - указывая столбец с именем таблицы ``test.array``. Так же понимаю, что в продуктивных средах такое не допустимо. Здесь оставила для теста.

<br/>

![7_7](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/99da1955-28bd-4221-88c4-2afb1d7b5650)

***




