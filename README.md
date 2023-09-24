
# Diplom
Описание хода выполнения Дипломного задания.

Ресурсы:
Для доступа к Zabbix - добавьте в файл hosts запись: 
158.160.79.1    zabbix.example.com 
Login - Netology 
Pass Eeji6aogaCh6
Dashboard - USE

Kibana - перейдите по ссылке http://158.160.82.75:5601
View error - логи /var/log/ngnix/error.log
View access - логи /var/log/ngnix/access.log

Сам сайт доступен по ссылке - http://51.250.75.72

Для развертывания необходимой инфраструктуры используется Terraform.
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
   [Elasticsearch](https://github.com/Santa-was-a-skinhead/Diplom/blob/main/cfg/1/elasticsearch.yml)
   [Logstash](https://github.com/Santa-was-a-skinhead/Diplom/blob/main/cfg/1/logstash.conf)
   [Filebeat](https://github.com/Santa-was-a-skinhead/Diplom/blob/main/cfg/1/filebeat.yml)

3. Создал статичную страницу, которая будет играть роль сайта. Также создал конфиг виртуального хоста, который слушает порт 8080
Index.html https://github.com/Santa-was-a-skinhead/Diplom/blob/main/cfg/1/index.html
Diplom.conf https://github.com/Santa-was-a-skinhead/Diplom/blob/main/cfg/1/diplom.conf

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
```
[nginx]
nginx1 ansible_host=10.0.129.3 ansible_port=22 ansible_user=ubuntu ansible_ssh_private_key_file=~/braineater/.ssh/braineater ansible_ssh_common_args='-o ProxyCommand="ssh -W %h:%p -q ubuntu@158.160.77.90 -i ~/braineater/.ssh/braineater"'
nginx2 ansible_host=192.168.1.28 ansible_port=22 ansible_user=ubuntu ansible_ssh_private_key_file=~/braineater/.ssh/braineater ansible_ssh_common_args='-o ProxyCommand="ssh -W %h:%p -q ubuntu@158.160.77.90 -i ~/braineater/.ssh/braineater"'
[zabbix_server]
zabbix ansible_host=192.168.1.6 ansible_port=22 ansible_user=ubuntu ansible_ssh_private_key_file=~/braineater/.ssh/braineater zabbix_server_version=6.0 php_enable_webserver=true ansible_ssh_common_args='-o ProxyCommand="ssh -W %h:%p -q ubuntu@158.160.77.90 -i ~/braineater/.ssh/braineater"'
[bastion_host]
bastion ansible_host=158.160.77.90 ansible_port=22 ansible_user=ubuntu ansible_ssh_private_key_file=~/braineater/.ssh/braineater
[ELK]
elastik ansible_host=192.168.1.30 ansible_port=22 ansible_user=ubuntu ansible_ssh_private_key_file=~/braineater/.ssh/braineater ansible_ssh_common_args='-o ProxyCommand="ssh -W %h:%p -q ubuntu@158.160.77.90 -i ~/braineater/.ssh/braineater"'
[kibana_server]
kibana ansible_host=192.168.1.36 ansible_port=22 ansible_user=ubuntu ansible_ssh_private_key_file=~/braineater/.ssh/braineater ansible_ssh_common_args='-o ProxyCommand="ssh -W %h:%p -q ubuntu@158.160.77.90 -i ~/braineater/.ssh/braineater"'
```
5. Разрешаем проброс ключа с локальной машины на bastion-host агентом SSH в файле /etc/ssh_config
   ```
   Host 158.160.77.90
    ForwardAgent yes
   ```
Шаг 4: Установка ПО на машины из плэйбуков Ansible
1. ansible-playbook [zabbix-server.yml](https://github.com/Santa-was-a-skinhead/Diplom/blob/main/playbooks/zabbix-server.yml) - В данном плэбуке выполняется установка Zabbix-server из официальной коллекции. Также выполняется установка всех необходимых компонентов (СУБД, zabbix-web, php, zabbix-agent) и задаютс настройки для корректной работы и передается нужный конфиг.
2. ansible-playbook [elastik.yml](https://github.com/Santa-was-a-skinhead/Diplom/blob/main/playbooks/elastik.yml) - В данном плэйбуке выполняется установка Elastiksearch, Logstash и Filebeat.Также задаются настройки для корректной работы и передается нужный конфиг. Пакеты скачаны из репозитория deb http://elasticrepo.serveradmin.ru
3. ansible-playbook [kibana.yml](https://github.com/Santa-was-a-skinhead/Diplom/blob/main/playbooks/kibana.yaml)  - В данном плэйбуке выполняется установка Kibana из коллекции, а также задаются настройки для корректной работы. Для корректной установки из коллекции я заменил в переменных адрес стандартного репозитория ELK на deb http://elasticrepo.serveradmin.ru
4. ansible-playbook [nginx.yml](https://github.com/Santa-was-a-skinhead/Diplom/blob/main/playbooks/nginx.yml) - В данном плэйбуке выполняется установка nginx, создание виртуального хоста и установка статичной страницы index.html

   Все плэйбуки размещены в папке playbooks

Шаг 5: Настройка мониторинга Zabbix
1. Добавляем хосты в веб-интерфейсе, дожидаемся отчета об их доступности
2. Подключаем к хостам шаблон Linux by Zabbix-agent
3. К хостам с nginx подключаем шаблон Nginx by Zabbix-agent
4. Создаем дэшборды по нужным метрикам.

   
   
   
   
   





