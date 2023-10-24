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
1. Создать `deployment` с 2 репликами контейнера [ifilyaninitmo/itdt-contained-frontend:master](https://hub.docker.com/repository/docker/ifilyaninitmo/itdt-contained-frontend) и передаются переменные в эти реплики: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.
2. Создать сервис через который будет доступ на эти "поды"
3. Запустить в `minikube` режим проброса портов и подключиться к нашим контейнерам через веб браузер
4. Проверить на странице в веб браузере переменные REACT_APP_USERNAME, REACT_APP_COMPANY_NAME и Container name. Изменяются ли они? Если да то почему?
5. Проверьте логи контейнеров.

### Ход работы
Ниже представлен пошаговый ход работы 

### 1. Запуск minikube
