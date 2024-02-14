**<div align="center"><h2>Морозова Юлия, OTUS, группа Postgre-DBA-2023-11</h2></div>**

**<div align=center><h3>Домашнее задание №8 по теме:</h3></div>**
**<div align=center><h3>«Настройка PostgreSQL.»</h3></div>**

***
**<h3>Домашнее задание:
<br>Нагрузочное тестирование и тюнинг PostgreSQL</h3>**

**<h3>Цель:
<br>сделать нагрузочное тестирование PostgreSQL
<br>настроить параметры PostgreSQL для достижения максимальной производительности</h3>**

***

**Выполнение:**

>**1. Развернуть виртуальную машину любым удобным способом**

Для этого задания создала новую ВМ В ЯО с параметрами:
​CPU: 2, RAM: 4 ГБ, Объём дискового пространства: 10 ГБ (SSD):

  ![1_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/8b717521-fcfe-45de-85bc-35c8d5e2405c)

<br/>

>**2. Поставить на неё PostgreSQL 15 любым способом**

Установила PostgreSQL 15й версии скриптом:

``sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15``

  ![2_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/302d207a-7d58-494e-9171-eb3e473556f6)

  ![2_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/30548391-8ad6-4c88-a998-134a8326c840)

<br/>

>**3. настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины**

Перед настройкой кластера принимаю решение сделать тест на производительность с дефольными настройками Postgres, поэтому сначала инициализирую ``pgbench``:

  ![3_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/a8abd872-219b-47d5-b749-33079e81c861)

И провожу тестирование на настройках Postgres по умолчанию командой:

``sudo -u postgres pgbench -c 50 -j 2 -P 10 -T 60 benchmark``

  ![3_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/70ad4f57-d1f9-4bea-b6c7-632b02dfff1b)

<br/>

>**4. Нагрузить кластер через утилиту через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)**



  <br/>

>**5. Написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему**



<br/>
  



***
**<h3> Задание со * :
<br>аналогично протестировать через утилиту https://github.com/Percona-Lab/sysbench-tpcc 
<br>(требует установки https://github.com/akopytov/sysbench)
***

**Выполнение:**


***
