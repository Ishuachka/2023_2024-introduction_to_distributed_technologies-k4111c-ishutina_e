# 2023_2024-introduction_to_distributed_technologies-k4111c-ishutina_e
University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2023/2024
Group: K4111с
Author: Ishutina Yelizaveta
Lab: Lab2
Date of create: 11.10.2023
Date of finished: .10.2023


## Лабораторная работа №2 "Развертывание веб сервиса в Minikube, доступ к веб интерфейсу сервиса. Мониторинг сервиса."

### Цель работы


### Задачи

### Ход работы
Ниже представлен пошаговый ход работы 

### 1. Запуск minikube
Первым делом, запускается minikube, как в первой лабораторной работе. 


### 2. Создание манифеста
Создется манифест `deployment` (прикреплен в директории). В нем описана конфигурация развертывания веб-сервиса. файл содержит в себе создание 3-х объектов k8s: deployment, service и ingress. Deployment содержит информацию об использованном образе, количестве реплик, порте и заданных переменных веб-сервиса, на основе чего создает 2 реплики pod'а с веб-сервисом. Объект service управляет доступом в данные pod'ы и позволяет передавать в них запросы и получать данные. Ingress описывает конфигурацию, с которой внешние запросы к кластеру будут приниматься и отправляться запросы на service, в частности это то, с каким доменным именем, путем, портом и протоколом будут обработаны запросы.

с 2 репликами контейнера [ifilyaninitmo/itdt-contained-frontend:master](https://hub.docker.com/repository/docker/ifilyaninitmo/itdt-contained-frontend) и передаются переменные в эти реплики: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.
Создать сервис через который у вас будет доступ на эти "поды". Выбор типа сервиса остается на ваше усмотрение.

### 3. 

### 4.

### 5.

