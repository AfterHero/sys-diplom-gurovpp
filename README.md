# Задание можно посмотреть по ссылке:
## [Задание Дипломной работы](https://github.com/netology-code/sys-diplom)


# 1. Для развертки инфраструкты используем Terraform:
# [Конфигурационный файл Terraform main.tf](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/terraform/main.tf)
# [Файл с настройками пользователя meta.txt](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/terraform/meta.txt)

## Terraform init:
![terraforminit](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/terraform%20init.jpg)

## Terraform validate:
![terrvalvalidate](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/terraform%20validate.jpg)

## Terraform apply:
![terrvalapply](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/terraform%20apply.jpg)

### Созданная инфраструктура на yandex.cloud:
![yandex.cloud](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/%D0%B4%D0%B0%D1%88%D0%B1%D0%BE%D1%80%D0%B4%20%D0%BA%D0%B0%D1%82%D0%B0%D0%BB%D0%BE%D0%B3%D0%B0.jpg)
![yandex.cloud](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/%D0%B4%D0%B0%D1%88%D0%B1%D0%BE%D1%80%D0%B4%20%D0%B2%D0%B8%D1%80%D1%82%D1%83%D0%B0%D0%BB%D0%BE%D0%BA.jpg)
![yandex.cloud](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/yc%20list.jpg)

### Было созданно 7 виртуальных машин на базе Debian 11, Балансировщик, Target Group, Backend Group, HTTP-router

# 2. Подключаемся к нашей виртуальной машине Bastion-host и через него прокидываем ключи доступа для всех хостов.
### На моем примере я уже заранне согласовал подключения со всеми хостами по ssh заранее поэтому токой скрин, все хосты доступны из под bastion-host
![terrvalvalidate](https://github.com/AfterHero/sys-diplom-gurovpp/blob/main/download/bastionssh.jpg)

# 3. С помощью Ansible устанавливаем и настраиваем нужные нам сервисы на хостах:
