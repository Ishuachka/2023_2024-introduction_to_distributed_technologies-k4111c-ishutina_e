# 2023_2024-introduction_to_distributed_technologies-k4111c-ishutina_e
University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2023/2024
Group: K4111с
Author: Ishutina Yelizaveta
Lab: Lab3
Date of create: 24.10.2023
Date of finished: 28.10.2023

## Лабораторная работа №3 "Сертификаты и "секреты" в Minikube, безопасное хранение данных."

### Цель работы
Познакомиться с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube. 

### Задачи
- Вам необходимо создать `configMap` с переменными: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.

- Вам необходимо создать `replicaSet` с 2 репликами контейнера [ifilyaninitmo/itdt-contained-frontend:master](https://hub.docker.com/repository/docker/ifilyaninitmo/itdt-contained-frontend) и используя ранее созданный `configMap` передать переменные `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME` .

- Включить `minikube addons enable ingress` и сгенерировать TLS сертификат, импортировать сертификат в minikube. 

- Создать ingress в minikube, где указан ранее импортированный сертификат, FQDN по которому вы будете заходить и имя сервиса который вы создали ранее.

> Если вы делаете эту работу на Windows/macOS для доступа к ingress вам необходимо использовать команду `minikube tunnel` к созданному ingress. 
> Если вы делаете эту работу на Windows/macOS для доступа к ingress вам необходимо в hosts добавить ip address localhost и ваш FQDN. Если установлен Linux, то нужно указывать minikube ip.

- В `hosts` пропишите FQDN и IP адрес вашего ingress и попробуйте перейти в браузере по FQDN имени. 

- Войдите в веб приложение по вашему FQDN используя HTTPS и проверьте наличие сертификата.

> Обычно в браузере это маленький замочек рядом с FQDN сайта, нажмите на него и сделайте скриншот с информацией.

### Ход работы
Ниже представлен пошаговый ход работы 

### 1. Создается манифеста
Первым делом запускается minikube start как в прошлых лабораторных работах
```
minikube start
```
![](/lab3/image/10.png)

Далее создается манифест `lab3-deployment`, прикреплен в директории - ![lab3-deployment](/lab3/manifest/lab3-deployment.yaml), там создаются ConfigMap, Deployment с ReplicaSet и LoadBalancer для доступа к подам. 
В ConfigMap ключами выступают `react_app_user_name` и `react_app_company_name`, значения `Ishutina` и `ITMO`, соответственно.
Поскольку `Ingress` работает с сервисами типа `nodePort` или `LoadBalancer`. Указан явно в сервисе тип. 

Выполняются следующие команды:
```
kubectl apply -f lab3-deployment.yaml
```
![](/lab3/image/20.png)
```
kubectl get configmap
```
```
kubectl get deployment
```
```
kubectl get service
```
```
kubectl get pod
```
![](/lab3/image/30.png)

Теперь создается TLS сертификат. 
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=lab3ishutina.com"
```
![](/lab3/image/40.png)
Secret содержит небольшое количество конфиденциальных данных, таких как пароль, токен или ключ.
```
kubectl create secret tls tls-secret --cert=tls.crt --key=tls.key
```
![](/lab3/image/50.png)

Проверяем, создалось ли:
```
kubectl get secret
```
![](/lab3/image/110.png)

Ingress

`Ingress` в Kubernetes - это объект, который управляет входящим сетевым трафиком и обеспечивает доступ к `Services` внутри кластера. `Ingress` предоставляет механизм для настройки маршрутизации внешнего трафика на pod'ы внутри кластера на основе правил, определенных в самом `Ingress`.

Сперва его надо подключить:
```
minikube addons enable ingress
```

![](/lab3/image/60.png)

Теперь создается ресурс Ingress. Файл прикреплен в директории - ![lab3-ingress](/lab3/manifest/lab3-ingress.yaml)
```
kubectl apply -f lab3_Ingress.yaml
```
![](/lab3/image/70.png)

Проверка статуса:
```
kubectl get ingress
```
![](/lab3/image/80.png)

Далее, чтоб перейти в сервис, нужно вписать в hosts IP и FQNX:
```
minikube ip
```
```
sudo nano /etc/hosts
```
![](/lab3/image/90.png)

Перенаправляем трафик командой: 
```
minikube tunnel
```
![](/lab3/image/100.png)

Переходим в сервис по доменному имени :

![](/lab3/image/photo_2023-11-01_22-47-42.jpg)

Проверям сертификат

![](/lab3/image/photo_2023-11-01_22-48-12.jpg)

### Схема


![](/lab3/image/200.png)
