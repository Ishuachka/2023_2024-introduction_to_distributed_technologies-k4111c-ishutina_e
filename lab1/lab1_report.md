# 2023_2024-introduction_to_distributed_technologies-k4111c-ishutina_e
University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2023/2024
Group: K4111с
Author: Ishutina Yelizaveta
Lab: Lab1
Date of create: 05.09.2023
Date of finished: 

## Лабораторная работа №1 "Установка Docker и Minikube, мой первый манифест."

### Цель работы
Ознакомиться с инструментами Minikube и Docker, развернуть свой первый "под".

### Задачи
1. Установить Docker, minikube
2. Развернуть minikube cluster
3. Создать под Vault и получить доступ к контейнеру
4. Найти сгенерированный корневой токен, чтобы получить доступ к Vault.

### Ход работы
Изначально был установлен Docker Desktop для Windows и настроен WSL, в котором был установлен Minikube
После установки был развернут minikube cluster с помощью команды minikube start:
```bash
minikube start
```
![](start.png)

Далее выполняются команды, которые добавляют на нашей локальной машине в список образов образ нужного ПО - Vault.
```bash
docker pull vault:1.13.3
docker images
```
Теперь посмотрим список образов с помощью следующих команд:
```bash
#просмотреть в minikube список всех образов
minikube image ls
```
![](images.png)

В конце мы видим, что есть нужный нам образ vault:1.13.3

Следующим шагом было написание манифеста для развертывания "пода" (наименьший объект рабочей нагрузки)  HashiCorp Vault, и при этом прокинуть внутрь порт 8200.
Манифест (vault.yaml) прилигается в папке lab1.
Манифест составлялся в соответствии с object model Kubernetes.
Обязательные поля:
- `apiVersion` - используемая версия API;
- `kind` - тип описываемого объекта;
- `metadata` - метаданные;
- `spec` - конфигурация объекта;

Далее запускается описанный под с помощью команды:
```bash
minikube kubectl -- apply -f vault.yaml
```
![](launch.png)


