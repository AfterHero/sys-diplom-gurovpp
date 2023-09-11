 # Задание можно посмотреть по ссылке:
### [Задание Дипломной работы](https://github.com/netology-code/sys-diplom)


# 1. Для развертки инфраструкты используем Terraform:
### [Конфигурационный файл Terraform main.tf](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/diplom/terraform/main.tf)
### [Файл с настройками пользователя meta.txt](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/diplom/terraform/meta.txt)

## Terraform init:
![terraforminit](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/terraform%20init.jpg)

## Terraform validate:
![terraformvalidate](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/terraform%20validate.jpg)

## Terraform apply:
![teraformapply](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/terraform%20apply.jpg)

### Созданная инфраструктура на yandex.cloud:
![yandex.cloud](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/%D0%B4%D0%B0%D1%88%D0%B1%D0%BE%D1%80%D0%B4%20%D0%BA%D0%B0%D1%82%D0%B0%D0%BB%D0%BE%D0%B3%D0%B0.jpg)
![yandex.cloud](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/%D0%B4%D0%B0%D1%88%D0%B1%D0%BE%D1%80%D0%B4%20%D0%B2%D0%B8%D1%80%D1%82%D1%83%D0%B0%D0%BB%D0%BE%D0%BA.jpg)
![yandex.cloud](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/yc%20list.jpg)

### Было созданно 7 виртуальных машин на базе Debian 11, Балансировщик, Target Group, Backend Group, HTTP-router

# 2. Подключаемся к нашей виртуальной машине Bastion-host и через него прокидываем ключи доступа для всех хостов.
### На моем примере я уже заранне согласовал подключения со всеми хостами по ssh заранее, все хосты доступны из под bastion-host
![bastionhost](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/bastionssh.jpg)

# 3. С помощью Ansible устанавливаем и настраиваем нужные нам сервисы на хостах:

## 3.1 Устанавливаем nginx, node-exporter, nginx-logexporter, filebeat
### [Servers-playbook.yml](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/diplom/ansible/servers-playbook.yml)
![serversplaybook](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/serversplaybook.jpg)
![serversplaybook](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/serversplaybook1.jpg)
### Адрес сайта: http://158.160.112.144
### Проверяем доступность сайта (curl -v 158.160.112.144)
![site](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/site.jpg)
![site2](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/site2.jpg)

## 3.2 Устанавливаем prometheus и grafana
### [prometheus-playbook.yml](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/diplom/ansible/prometheus-playbook.yml)
### ![prometheus](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/prometheus.jpg)
### ![prometheustargets](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/prometheustargets.jpg)
### Адрес prometheus: http://51.250.44.251:9090/classic/targets
--------------------------------------------------------------------------------------------------------------------------------------
### [grafana-playbook.yml](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/diplom/ansible/grafana-playbook.yml)
### ![grafana](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/grafana.jpg)
### ![grafanametrics](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/grafanametrics.jpg)
### Адрес grafana: http://51.250.47.218:3000/d/4aBQsjSmz34/servers-metrics?orgId=1&refresh=10s
## 3.3 Устанавливаем elasticsearch и kibana
### [elasticsearch-playbook.yml](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/diplom/ansible/elasticsearch-playbook.yml)
### ![elasticsearch](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/elasticsearch.jpg)
### Запущенный контейнер docker c ELK
### ![elasticdocker](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/dockerelstic.jpg)
---------------------------------------------------------------------------------------------------------------------------------------
### [kibana-playbook.yml](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/diplom/ansible/kibana-playbook.yml)
### ![kibana](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/playbookkibana.jpg)
### Адрес kibana: http://51.250.36.1:5601
### ![elasticsearchlogs](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/elklogs.jpg)
### ![elasticsearchlogs2](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/elklogs2.jpg)
