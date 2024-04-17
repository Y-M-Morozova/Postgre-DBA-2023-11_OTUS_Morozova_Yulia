**<div align="center"><h2>Морозова Юлия, OTUS, группа Postgre-DBA-2023-11</h2></div>**

**<div align=center><h3>Проектная работа</h3></div>**
**<div align=center><h3>«тема: Cоздание и тестирование высоконагруженных отказоустойчивых кластеров PostgreSQL 
</br>на базе Patroni из репозитория https://github.com/vitabaks/postgresql_cluster.»</h3></div>**

***

**<h4>Цели проекта:
<br>  - Изучение отказоустойчивого кластера Patroni на базе PostgreSQL, и сервисов: etcd, pgbouncer, vip-manager.
</br> - Изучение принципов работы Ansible и синтаксиса Ansible Playbook
</br> - Развертывание и тестирование кластера PostgreSQL с помощью плейбука ansible: 
</br>postgresql_cluster (https://github.com/vitabaks/postgresql_cluster).</h4>**


***


**<h3>Выполнение</h3>**

**Вступление**

В ходе выполнения данной проектной работы я использую: 
</br>  1. Сценарий (плейбук) ansible — postgresql_cluster: https://github.com/vitabaks/postgresql_cluster. 
</br>  2. демонстрационную базу данных «Авиаперевозки» : https://postgrespro.ru/education/demodb.

Используемые технологии:
- VMware Virtual Platform, OS: Debian 11
- PostgreSQL 14.11, 16.2
- Patroni 3.3.0, etcd 3.5.11, vip-manager 2.3.0

Схема решения - взята с репозитория: https://github.com/vitabaks/postgresql_cluster:

![postgresql_cluster](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/ba6b3b32-b5c9-491b-99b7-47063a5a31a4)

<br/>  

**Выполнение**

1. Разворачиваю кластер согласно инструкции репозитория:

- Установливаю ``Ansible`` на отдельном хосте:

``sudo apt update && sudo apt install -y python3-pip sshpass git``
``pip3 install ansible``

- Клонирую репозиторий:
  
``git clone https://github.com/vitabaks/postgresql_cluster.git``

- Перехожу в каталог ``playbook``:

``cd postgresql_cluster/``

- Редактирую файл инвентаризации ``inventory`` там надо указать IP-адреса хостов кластера и настройки подключения (ansible_user, ansible_ssh_pass или ansible_ssh_private_key_file):
  
``nano inventory``

- Редактирую файл переменных ``vars/main.yml`` - там надо указать ``proxy_env`` - для пакетов загрузки с прокси, ``cluster_vip``-  для клиентского доступа к базам данных в кластере с помощью ``vip-manager``, 
``patroni_cluster_name``, ``postgresql_version``, ``postgresql_data_dir`` with_haproxy_load_balancing 'false' (так как я пока не пользую HAPROXY) , ``dcs_type = "etcd"`` (пока выбираю такой типа dcs) 

``nano vars/main.yml``

- Пробую подключиться к хостам:
  
``ansible all -m ping``

все ок:

![1_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/b3158a80-c274-49be-9946-debd95da4a84)

- Стартую выполнение сценария(``playbook``):
  
``ansible-playbook deploy_pgcluster.yml``

![1_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/8a52593c-e58e-4108-87b5-0d58e1cf2493)

- После установки проверяю состояние кластера:

 `` sudo patronictl -c /etc/patroni/patroni.yml list `` 

![1_4](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/6e07daee-7121-4191-836a-5ceff4ff2df8)

- Далее проверяю ``pgbouncer``:

``systemctl status pgbouncer``
</br>``psql --host=localhost -p 6432 -U postgres pgbouncer``  

![1_5](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/285e0045-390b-47af-85f4-48c3508b6f39)

- Далее проверяю ``vip-manager``:

``systemctl status vip-manager``
</br>``hostname -I``
</br>``ip a``

![1_6](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/cb613d1f-4460-4c7e-98aa-15d7e0192d0c)

2. Тестирую кластер и сервисы  

- Проверяю состояние кластера - вижу , что ``leader`` на 4й ноде, далее создаю там БД ``testdb, таблицу ``t1``, вставляю строку:

 ![111_1](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/12a9614c-47ea-4394-a153-4a86fccfd1f6)
 
- Подключаюсь с помощью  dbeaver по 1-му виртуальному ip-адресу на порт 6432(``pgbouncer``):

![111_2](https://github.com/Y-M-Morozova/Postgre-DBA-2023-11_OTUS_Morozova_Yulia/assets/153178571/92d4e08f-9807-4b0b-95bb-919f032949f5)
  

  
***

<br/>

***



