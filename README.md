
# Diplom
Описание хода выполнения Дипломного задания.

Для развертывания необходимой инфраструктуры используется Ansilbe.
Для подключения к облаку необходимо подключиться к провайдеру и получить oAuth_token. Данные облака и ключ указываются в инфраструктурном файле.

Шаг 1: Развертывание инфраструктуры.
Устанавливаем Terraform
Пишем код - он представлен в файле diplom.tf
Переходим в директорию с файлом
#terraform plan - получаем список элементов, которые будут созданы из файла diplom.tf
#terraform apply - запускаем развертывание инфраструктуры

Результат работы:
ВМ
![ВМ](https://github.com/Santa-was-a-skinhead/Diplom/blob/main/img/vm.png)

Сеть
![](https://github.com/Santa-was-a-skinhead/Diplom/blob/main/img/net.png)

Роутер
![](https://github.com/Santa-was-a-skinhead/Diplom/blob/main/img/router.png)

Балансировщик
![](https://github.com/Santa-was-a-skinhead/Diplom/blob/main/img/balans.png)

Целевая группа
![](https://github.com/Santa-was-a-skinhead/Diplom/blob/main/img/target.png)

Группа бэкендов
![](https://github.com/Santa-was-a-skinhead/Diplom/blob/main/img/back.png)

Группы безопасности
![](https://github.com/Santa-was-a-skinhead/Diplom/blob/main/img/sec.png)

Кка видно из скриншотов, все ресурсы созданы согласно условиям задания. Доступ по SSH открыт только из группы безопасности bastion-sg.


Шаг 2: Подготовка подготовка файлов конфигурации

1. Поскольку узлы elstik, web1, web2 не выходят в глобальную сеть, я вручную подготовил конфиги для сервисов:
Zabbix-agent
Logstash
Filebeat

2. Создал статичную страницу, которая будет играть роль сайта.
Index.html

Все сонфиги представлены в папке cfg.

Шаг 3: Подготовка Ansible

1. Устанавливаем Ansible
   sudo apt install ansible
2. Устанавливаем необходимые коллекции и роли
   ansigle-galaxy install geerlingguy.postgresql
   ansigle-galaxy install community.zabbix
   ansigle-galaxy install geerlingguy.kibana
3. Поскольку у нас есть машины без доступа в интернет, из репозитория пакеты не установить. Скачиваем вручную пакеты на локальную машину:
   zabbix-agent2_6.4.0-1+ubuntu22.04_amd64.deb
   nginx_1.22.1-1~jammy_amd64.deb
   filebeat-8.9.2-amd64.deb
   elasticsearch-8.9.1-amd64.deb
4. Вносим изменения в inventory-файл.


   
   
   
   





