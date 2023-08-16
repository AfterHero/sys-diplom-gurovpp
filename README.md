
#  Дипломная работа по профессии «Системный администратор»

Содержание
==========
* [Задача](#Задача)
* [Инфраструктура](#Инфраструктура)
    * [Сайт](#Сайт)
    * [Мониторинг](#Мониторинг)
    * [Логи](#Логи)
    * [Сеть](#Сеть)
    * [Резервное копирование](#Резервное-копирование)
* [Выполнение работы](#Выполнение-работы)


---------
## Задача
Ключевая задача — разработать отказоустойчивую инфраструктуру для сайта, включающую мониторинг, сбор логов и резервное копирование основных данных. Инфраструктура должна размещаться в [Yandex Cloud](https://cloud.yandex.com/).

## Инфраструктура
Для развёртки инфраструктуры используйте Terraform и Ansible. 

Параметры виртуальной машины (ВМ) подбирайте по потребностям сервисов, которые будут на ней работать. 

Ознакомьтесь со всеми пунктами из этой секции, не беритесь сразу выполнять задание, не дочитав до конца. Пункты взаимосвязаны и могут влиять друг на друга.

### Сайт
Создайте две ВМ в разных зонах, установите на них сервер nginx, если его там нет. ОС и содержимое ВМ должно быть идентичным, это будут наши веб-сервера.

Используйте набор статичных файлов для сайта. Можно переиспользовать сайт из домашнего задания.

Создайте [Target Group](https://cloud.yandex.com/docs/application-load-balancer/concepts/target-group), включите в неё две созданных ВМ.

Создайте [Backend Group](https://cloud.yandex.com/docs/application-load-balancer/concepts/backend-group), настройте backends на target group, ранее созданную. Настройте healthcheck на корень (/) и порт 80, протокол HTTP.

Создайте [HTTP router](https://cloud.yandex.com/docs/application-load-balancer/concepts/http-router). Путь укажите — /, backend group — созданную ранее.

Создайте [Application load balancer](https://cloud.yandex.com/en/docs/application-load-balancer/) для распределения трафика на веб-сервера, созданные ранее. Укажите HTTP router, созданный ранее, задайте listener тип auto, порт 80.

Протестируйте сайт
`curl -v <публичный IP балансера>:80` 

### Мониторинг
Создайте ВМ, разверните на ней Prometheus. На каждую ВМ из веб-серверов установите Node Exporter и [Nginx Log Exporter](https://github.com/martin-helmich/prometheus-nginxlog-exporter). Настройте Prometheus на сбор метрик с этих exporter.

Создайте ВМ, установите туда Grafana. Настройте её на взаимодействие с ранее развернутым Prometheus. Настройте дешборды с отображением метрик, минимальный набор — Utilization, Saturation, Errors для CPU, RAM, диски, сеть, http_response_count_total, http_response_size_bytes. Добавьте необходимые [tresholds](https://grafana.com/docs/grafana/latest/panels/thresholds/) на соответствующие графики.

### Логи
Cоздайте ВМ, разверните на ней Elasticsearch. Установите filebeat в ВМ к веб-серверам, настройте на отправку access.log, error.log nginx в Elasticsearch.

Создайте ВМ, разверните на ней Kibana, сконфигурируйте соединение с Elasticsearch.

### Сеть
Разверните один VPC. Сервера web, Prometheus, Elasticsearch поместите в приватные подсети. Сервера Grafana, Kibana, application load balancer определите в публичную подсеть.

Настройте [Security Groups](https://cloud.yandex.com/docs/vpc/concepts/security-groups) соответствующих сервисов на входящий трафик только к нужным портам.

Настройте ВМ с публичным адресом, в которой будет открыт только один порт — ssh. Настройте все security groups на разрешение входящего ssh из этой security group. Эта вм будет реализовывать концепцию bastion host. Потом можно будет подключаться по ssh ко всем хостам через этот хост.

### Резервное копирование
Создайте snapshot дисков всех ВМ. Ограничьте время жизни snaphot в неделю. Сами snaphot настройте на ежедневное копирование.



## Выполнение работы

Схема развертываемой инфраструктуры приведена на рисунке 1. (для наглядности на схеме не приведен bastion host, указанный в разделе "Сеть" Задания)  

![Рисунок 1](/img/img1.PNG)  
Рисунок 1.

Создание инфраструктуры в Yandex-облаке произведено в большей части средствами terraform и ansible в автоматизированном режиме, в качестве операционной системы (ОС) для виртуальных машин выбрана Fedora.  

В облаке Yandex Cloud развертываются следующие серверы:  
1. web1.srv, web2.srv - веб-серверы nginx, на которых установлены агенты node_exporter и filebeats, позволяющие передавать информацию мониторинга и содержимом логов на серверы promrtheus и elasticsearch;
2. prometheus.srv - сервер мониторинга prometheus;
3. grafana.srv - сервер визуализации информации, собираемой сервером мониторинга prometheus;
4. elastic.srv - сервер сбора логов, на котором подняты сервисы esasticsearch и logstash, входяшие в стек ELK;
5. kibana.srv - сервер визуализации логов (также из стека ELK);
6. sshgw.srv - (ssh gateway), сервер, позволяющий получить доступ к остальным серверам инфраструктуры по протоколу ssh.

Создание инфрастукткры в terraform осуществлялось путем последовательного добавления блоков кода в конфигурационные файлы с последующей сборкой-разборкой инфраструктуры и отладкой добавляемых блоков кода.

### Сайт
На первом этапе разворачиваем серверы nginx и балансировщик, создаем "Target group", "Backend group", "HTTP router" и "Application load balancer" (ALB). Данным элементам соответствуют блоки кода "Web Server 1", "Web Server 2", "Target group for ALB", "Backend group for ALB", "ALB router", "ALB virtual host", "ALB" в основном конфигурационном файле terraform main.tf, листинг которого доступен по ссылке  

https://github.com/ta7575/sys-diplom/blob/main/config/main.tf  

Листинг файлов ansible-конфигураций веб-серверов web1.srv и web2.srv доступен по ссылкам  

https://github.com/ta7575/sys-diplom/blob/main/config/meta-web1.yml  
и  
https://github.com/ta7575/sys-diplom/blob/main/config/meta-web2.yml  

Также создаем базовые настройки сети - блок кода "Network" в основном файле конфигурации терраформ main.tf, при этом веб-сервера размещаем в разных зонах согласно Заданию.

**Примечание**: для доступа по ssh к серверам используем ключи ed25519, как более безопасные по сравнению с RSA. Для тестирования веб серверов и балансировщика используем тестовые начальные страницы, содержимое которых формируется при развертывании веб-серверов. Устанавливаемые RPM-пакеты для обеспечения доступности при установке размещены на сервисе dropbox.  


Собираем конфигурацию в terraform и тестируем (рисунок 2):  

![Рисунок 2](/img/img2.PNG)  
![Рисунок 2-1](/img/img2-1.PNG)  
Рисунок 2.  

Делая несколько раз запрос на внешний адрес балансировщика с помощью команды curl, наблюдаем ответы от обоих серверов (русунок 2), следовательно балансировщик балансирует http-запросы между веб-серверами.  

Тестируем доступность веб-серверов по ssh (рисунок 3):  

![Рисунок 3](/img/img3.PNG)  
Рисунок 3.  

### Мониторинг

Добавляем сервер prometheus (блок кода "Prometheus Server" в конфигурационном файле main.tf). Листинг файла terraform-конфигурации prometheus доступен по ссылке  

https://github.com/ta7575/sys-diplom/blob/main/prometheus/meta-prometheus.yml  

Файл конфигурации prometheus - по ссылке  

https://github.com/ta7575/sys-diplom/blob/main/prometheus/prometheus.yml  


Добавляем сервер визуализации grafana (блок кода Grafana Server в конфигурационном файле main.tf). Листинг файла terraform-конфигурации grafana доступен по ссылке  

https://github.com/ta7575/sys-diplom/blob/main/grafana/meta-grafana.yml  

Собираем конфигурацию терраформ (рисунок 4).  

![Рисунок 4](/img/img4.PNG)  

Рисунок 4.  

Тестируем prometheus (рисунок 5, 6).  

![Рисунок 5](/img/img5.PNG)  
Рисунок 5.  
` `  

![Рисунок 6](/img/img6.PNG)  
Рисунок 6.  
` `  

Заходим в веб-интерфейс сервера grafana (рисунок 7).  

![Рисунок 7](/img/img7.PNG)  
Рисунок 7.  
` `  

Создаем требуемые дашборды (рисунок 8, 9, 10, 11).  

![Рисунок 8](/img/img8.PNG)  
Рисунок 8.  
` `  

![Рисунок 9](/img/img9.PNG)  
Рисунок 9.  
` `  

![Рисунок 10](/img/img10.PNG)  
Рисунок 10.  
` `  

![Рисунок 11](/img/img11.PNG)  
Рисунок 11.  
` `  

Для того, чтобы не создавать дашборбы при каждом развертывании инфраструктуры в terraform, сохраняем базу данных и настройки grafana в архивы и добавляем в файл terraform-конфигурации сервера grafana (meta-gtafana.yml) код загрузки предварительно сохраненной базы данных и настроек сервера grafana. После чего, при последующем развертывании инфраструктуры средствами терраформ, дашборды восстанавливаются.  


### Логи  

Добавляем в основной файл конфигурации терраформ main.tf блок кода "ElasticSearch Server", отвечающий за развертывыние сервера elasticserch с logstash.  
Также добавляем в main.tf блок кода "Kibana Server", отвечающий за развертывание сервера визуализации логов kibana.  

Листинг файла terraform-конфигурации сервера elasticsearch доступен по ссылке  

https://github.com/ta7575/sys-diplom/blob/main/elastic/meta-elasticsearch.yml  

Листинг файла конфигурации elasticsearch доступен по ссылке  

https://github.com/ta7575/sys-diplom/blob/main/elastic/elasticsearch.yml  

Листинги файлов конфигурации logstash - по ссылкам  

https://github.com/ta7575/sys-diplom/blob/main/elastic/logstash/input.conf  

https://github.com/ta7575/sys-diplom/blob/main/elastic/logstash/filter.conf  

https://github.com/ta7575/sys-diplom/blob/main/elastic/logstash/output.conf  

Кроме того, в файлы terraform-конфигураций веб-серверов (файлы meta-web1.yml, meta-web2.yml) добавлен код установки агента filebeat - поставщика данных для стека ELK.  

Листинг файла конфигурации агента filebeat доступен поссылке  

https://github.com/ta7575/sys-diplom/blob/main/config/filebeat.yml  

Запускаем terraform и собираем инфраструктуру (рисунок 12)  

![Рисунок 12](/img/img12.PNG)  
Рисунок 12.  
` `  
Устанавливаем пароль суперюзера сервера elasticsearch (рисунок 13)  

![Рисунок 13](/img/img13.PNG)  
Рисунок 13.  
` `  
Проверяем работу ноды elasticsearch (рисунок 14)  

![Рисунок 14](/img/img14.PNG)  
Рисунок 14.  
` `  

Создаем токен для подключения сервера визуализации kibana к elasticsearch (рисунок 15)  

![Рисунок 15](/img/img15.PNG)  
Рисунок 15.  
` `  

Подключаемся к серверу kibana по ssh и редактируем файл конфигурации, в котором разрешаем подключение к kibana с других хостов (рисунок 16, 17)  

![Рисунок 16](/img/img16.PNG)  
Рисунок 16.  
` `  

![Рисунок 17](/img/img17.PNG)  
Рисунок 17.  
` `  
Подключаемся к веб-интерфейсу сервера kibana и настраиваем поключение к elasticsearch с помощью предварительно сгенерированного токена (рисунок 18, 19, 20, 21)  

![Рисунок 18](/img/img19.PNG)  
Рисунок 18.  
` `  

![Рисунок 19](/img/img18-1.PNG)  
Рисунок 19.  
` `  

![Рисунок 20](/img/img20.PNG)  
Рисунок 20.  
` `  

![Рисунок 21](/img/img21.PNG)  
Рисунок 21.  
` `  

Проверяем работу kibana (рисунок 22)  

![Рисунок 22](/img/img22-1.PNG)  
Рисунок 22.  
` `  


### Сеть

Создаем виртуальную машину sshgw.srv, доступ на которую будет осуществляться по протоколу ssh, и с которой можно будет получить доступ к остальным серверам инфраструктуры. Для этого добавляем соответсвующий блок кода "Server sshgw" в конфигурационный файл main.tf. Листинг terraform-конфигурации сервера sshgw доступен по ссылке  

https://github.com/ta7575/sys-diplom/blob/main/sshgw/meta-sshgw.yml  


Создаем группы безопасности, конфигурацию которых для наглядности помещаем в отдельный конфигурационный файл sg.tf, листинг которого доступен по ссылке  

https://github.com/ta7575/sys-diplom/blob/main/config/sg.tf  

Всего создаем 4 группы безопасности: sg-balancer - для балансировщика, sg-sshgw - для сервера доступа sshgw, sg-private - для группы серверов, досуп к которым не разрещен из сети Интернет, sg-public - для группы серверов, доступ к которым возможен из сети интернет по определеным портам, соответствующим публикуемым сервисам.

В группу безопасности sg-private включаем сервера web1.srv, web2.srv, prometheus.srv и elastic.srv  
В группу безопасности sg-public - сервера grafana.srv и kibana.srv  

Включение сервера в определенную группу безопасности осуществляется заданием параметра security_group_ids в блоке network_interface, описывающем сетевую конфигурацию каждого сервера в файле конфигурации main.tf  

Собираем инфраструктуру в терраформ и в консоли управления yandex Cloud смотрим информацию о группах безопасности (рис. 23, 24, 25, 26, 27)  

![Рисунок 23](/img/23.PNG)  
Рисунок 23.  
` `  

![Рисунок 24](/img/24.PNG)  
Рисунок 24.  
` `  

![Рисунок 25](/img/25.PNG)  
Рисунок 25.  
` `  

![Рисунок 26](/img/26.PNG)  
Рисунок 26.  
` `  

![Рисунок 27](/img/27.PNG)  
Рисунок 27.  
` `  

### Резервное копирование

На момент написания данной работы использовать для резервного копирования инфраструктуры сервис Cloud Backup не представляется возможным (рисунок 28). Поэтому для резервного копирования виртуальных машин используется создание снапшотов их дисков по расписанию.  

![Рисунок 28](/img/28.PNG)  
Рисунок 28.  
` `  

Создаем файл конфигурации backup.tf, содержащий код расписания, по которому будут создаваться снапшоты дисков всех виртуальных машин, входящих в инфраструктуру. Листинг файла backup.tf доступен по ссылке  

https://github.com/ta7575/sys-diplom/blob/main/config/backup.tf  

Параметр snapshot_count определяет максимальное количество хранимых снапшотов для каждого диска, параметр expression в блоке schedule_policy определяет расписание, по которому создаются снапшоты. В соответствии с Заданием определяем snapshot_count = 7, expression = "0 5 ? * *" (т.е. снапшоты дисков будут создаваться один раз в сутки в 5:00).  

Результат выполнения приведен на рисунках 29, 30)  

![Рисунок 29](/img/29.PNG)  
Рисунок 29.  
` `  

![Рисунок 30](/img/30.PNG)  
Рисунок 30.  
` `  

(Для тестирования создания снапшотов по расписанию выбраны значения snapshot_count = 1, expression = "30 * ? * *")


