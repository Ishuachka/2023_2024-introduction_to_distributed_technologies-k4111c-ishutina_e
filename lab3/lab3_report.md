# 2023_2024-introduction_to_distributed_technologies-k4111c-ishutina_e
University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2023/2024
Group: K4111с
Author: Ishutina Yelizaveta
Lab: Lab3
Date of create: 24.10.2023
Date of finished: 16.10.2023

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

Создается `ConfigMap`, файл прикреплен в директории - ConfigMap.yaml
Ключами выступают `react_app_user_name` и `react_app_company_name`, значения `Ishutina` и `ITMO`, соответственно.

![]()

Далее создается `ReplicaSet`, файл прикреплен в директории - replicaSet.yaml

Здесь переменные окружения `REACT_APP_USERNAME` и `REACT_APP_COMPANY_NAME`, значения которых определены в `ConfigMap`. Обращаемся к `front-configmap` по соответствующим ключам.
![]()

Далее создается `Service`, файл прикреплен в директории - lab3_Service.yaml

Поскольку `Ingress` работает с сервисами типа `nodePort` или `LoadBalancer`. Указан явно в сервисе тип. 

![]()

TLS в Ingress Controller: Ingress Controller управляет входящим трафиком в кластер и обеспечивает обработку запросов по определенным правилам. Minikube также может включать Ingress Controller с поддержкой TLS, позволяя обеспечить защищенный доступ к веб-приложениям, используя сертификаты TLS. 

![]()

Secret

Secret содержит небольшое количество конфиденциальных данных, таких как пароль, токен или ключ.

![]()

Проверка, создалось ли:
![]()

Ingress

`Ingress` в Kubernetes - это объект, который управляет входящим сетевым трафиком и обеспечивает доступ к `Services` внутри кластера. `Ingress` предоставляет механизм для настройки маршрутизации внешнего трафика на pod'ы внутри кластера на основе правил, определенных в самом `Ingress`.

Сперва его надо подключить:

![]()

Проверим наличие:

![]()

Теперь переходим по указанной ссылке, предварительно перенаправим трафик командой: `minikube tunnel`.

![]()

![]()

### Схема

![]()

**Выводы:**  
- Написаны манифесты: `ConfigMap`, `ReplicaSet`, `Ingress`, `Service`;
- Сгенерирован `TLS` сертификат;
- Проверка в веб-приложении показала, что при переходе по указанному FQDN используя HTTPS, указан созданный сертификат.






### 2. Далее деплоится данный файл

![]()

### 3. Запуск ingress controller в minikube
С помощью следующей команды будет развернут Ingress Controller Nginx в minikube:

    minikube addons enable ingress

![]()

### 4. Создание сертификата TLS
Ingress Controller Nginx может использовать собственные SSL-сертификаты по умолчанию, но задачей в данной лабораторной работе является сгенерировать свой сертификат и импортировать его в minikube.
 
 Генерируем сертификат (обратите внимание на `CN`, в примере используется hostname `app.itmo.blinov.da`):
   
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=app.itmo.blinov.da

![]()

Добавляем сгененированный сертификат в наш namespace `itmo` в k8s с помощью специального типа `Secret - tls`:

    kubectl create -n itmo secret tls app.itmo.blinov.da-tls --key="tls.key" --cert="tls.crt"

### 4. Запуск развертывания веб-сервиса
С помощью следующей команды запускаем развертывание веб-сервиса:

    kubectl apply -f deployment.yaml

![]()

Проверим, что у нас существует все, что нам нужно:

    kubectl get all -n itmo && kubectl get configmaps -n itmo && kubectl get secrets -n itmo && kubectl get ingress -n itmo


 ![]()

Доступ к развернутому веб-сервису можно получить используя указанный в ingress `hostname`, запустив `minikube tunnel` и перейдя на http://app.itmo.blinov.da, предварительно добавив `DNS` запись в `/etc/hosts` на ip-адрес `minikube`:

 Запустили minikube tunnel:

  ![Alt text](source/image5.png)


 И перейдем на http://app.itmo.blinov.da, должен произойти редирект на HTTPS. Соглашаемся открыть страницу с недоверенным сертификатом и страница развернутого веб-сервиса доступна:

 ![Alt text](source/image6.png)
 
 На странице мы видим заданные в configMap и примененные в deployment значения переменных, а именно `user: blinov.da` и `company: itmo` - они задаются сразу для всех реплик.

 Проверим сертификат:

 ![Alt text](source/image7.png)

 Видим, что используется тот самый выпущенный на 3 этапе сертификат.

 ### 5. Схема
 
 ![Alt text](source/image8.png)
