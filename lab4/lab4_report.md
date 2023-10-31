# 2023_2024-introduction_to_distributed_technologies-k4111c-ishutina_e
University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2023/2024
Group: K4111с
Author: Ishutina Yelizaveta
Lab: Lab4
Date of create: 30.10.2023
Date of finished: 01.11.2023

## Лабораторная работа №4 "Сети связи в Minikube, CNI и CoreDNS"

### Цель работы
Познакомиться с CNI Calico и функцией `IPAM Plugin`, изучить особенности работы CNI и CoreDNS.

### Задачи
- При запуске minikube установите плагин `CNI=calico` и режим работы `Multi-Node Clusters` одновеременно, в рамках данной лабораторной работы вам нужно развернуть 2 ноды.

> Оригинальная инструкция для установки Calico в Minikube [ссылка](https://projectcalico.docs.tigera.io/getting-started/kubernetes/minikube)
> Оригинальная инструкция для включение 2-ух нод в Minikube [ссылка](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/)

- Проверьте работу CNI плагина Calico и количество нод, результаты проверки приложите в отчет.

- Для проверки работы Calico мы попробуем одну из функций под названием `IPAM Plugin`.

- Для проверки режима `IPAM` необходимо для запущеных ранее нод указать `label` по признаку стойки или географического расположения (на ваш выбор).
  
> Оригинальная инструкция для назначения IP адресов в Calico [ссылка](https://projectcalico.docs.tigera.io/networking/assign-ip-addresses-topology)

- После этого вам необходимо разработать манифест для Calico который бы на основе ранее указанных меток назначал бы IP адреса "подам" исходя из пулов IP адресов которые вы указали в манифесте.

- Вам необходимо создать `deployment` с 2 репликами контейнера [ifilyaninitmo/itdt-contained-frontend:master](https://hub.docker.com/repository/docker/ifilyaninitmo/itdt-contained-frontend) и передать переменные в эти реплики: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.

- Создать сервис через который у вас будет доступ на эти "поды". Выбор типа сервиса остается на ваше усмотрение. 

- Запустить в `minikube` режим проброса портов и подключитесь к вашим контейнерам через веб браузер.

- Проверьте на странице в веб браузере переменные `Container name` и `Container IP`. Изменяются ли они? Если да то почему?

- Используя `kubectl exec` зайдите в любой "под" и попробуйте попинговать "поды" используя `FQDN` имя соседенего "пода", результаты пингов необходимо приложить к отчету.

### Ход работы
Ниже представлен пошаговый ход работы 

## 1. Запуск миникуба
Запускается minikube с подключенным плагином `CNI=calico`, режимом работы `Multi-Node Clusters` и разворачиваются 2 ноды с помощью команды:

```
minikube start --network-plugin=cni --cni=calico --nodes 2 -p multinode-demo
```
![](/lab4/image/1.png)

Проверить, запустились ли 2 Node можно с помощью команды:
```
kubectl get nodes
```
![](/lab4/image/22.png)

Проверить работу CNI Calico, проверяется количество подов с меткой **calico-node**. Их число должно совпадать с количеством нод. Проверяем с помощью следующей команды:
```
kubectl get pods -l k8s-app=calico-node -A
```
![](/lab4/image/3.png)

## 2. Пометка нод
В соответствии с заданием помечается каждая нода по географическому расположению

Для назначения IP адресов в Calico необходимо написать манифест для **IPPool** ресурса.

С помощью IPPool можно создать **IP-pool (блок IP-адресов)**, который выделяет IP-адреса только для узлов с определенной **меткой (label)**.

С помощью следующей команды, назначаются метки узлам :

```
kubectl label nodes multinode-demo zone=east  
kubectl label nodes multinode-demo-m02 zone=west
```
Далее из официальной документации Calico берется шаблон манифеста IPPool, прикреплен в директории - [lab4_ippool.yaml](/lab4/manifesty/lab4_ippool.yaml)
В его спеке есть следующие поля:
- `cidr` позволяет определить количество адресов, доступных для конкретно этого IPPool. В нашем случае это `2^8 = 256` адресов.
- `ipipMode` позволяет настроить режим IP-туннелирования.
  Судя по документации, есть два режима:
  - Режим `Always` включает инкапсуляцию пакетов для всего трафика от Calico-хоста к другим Calico-контейнерам и всем VM, имеющим IP в заданном IPPool.
  - Режим `CrossSubnet` включает инкапсуляцию только для того трафика, который ходит между сетями.
Calico рекомендует использовать режим `CrossSubnet` в случае `ipipMode`, так как это уменьшит накладные расходы на инкапсуляцию.
  Но так как у нас работа маленькая, то можно и использовать режим `Always`.
- `natOutgoing` позволяет разрешить подам ходить во внешнюю сеть с помощью `NAT`.
  В данной работе эта настройка не особо играет роли, так как нам не требуется ходить во внешний интернет и наши поды не развернуты на железках.

- `nodeSelector` позволяет определить, какие ноды должны получать адрес из этого пула. В данном конкретном примере все ноды, находящиеся в "нулевой стойке", будут получать IP из этого пула.

Для того, чтобы применить манифест  для IPPool, надо установить **calicoctl**, для этого скачивается [config-файл](https://github.com/projectcalico/calico/blob/master/manifests/calicoctl.yaml) с официального репозитория и выполняется следующая команда: 

```
kubectl create -f calicoctl.yaml
```
![](/lab4/image/4.png)

Перед созданием IPPools, проверяются созданные по-умолчанию и удаляются c помощью следующих команд: 
```
kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch get ippools -o wide
```
```
kubectl delete ippools default-ipv4-ippool
```

Теперь можно приступить к созданию IPPools с помощью команды:
```
kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch create -f - < lab4-ippool.yaml
```
![]()

С помощью следующей команды проверяется, что создалось ва pool'а:
```
kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch get ippool -o wide
```
![](/lab4/image/6.png)

Можно заметить, что их [маски подсети (CIDR)](https://ru.wikipedia.org/wiki/Маска_подсети) соответсвуют тем, что указаны в диапазон pool'ов в манифесте

##3. Deployment и Service
Манифесты для развертывания прикреплены в директории - [lab4-deployment.yaml](/lab4/manifesty/lab4-deployment.yaml), [lab4-service.yaml](lab4/manifesty/lab4-service.yaml)
Далее выполняется команда:
```
kubectl apply -f lab4-deployment.yaml -f lab4-service.yaml
```

Проверяем, что появилось развертывание и сервис: 
```
kubectl get deployments
```
```
kubectl get services
```
![](/lab4/image/7.png)

Проверяем IP созданных Pod'ов: `kubectl get pods -o wide`.

![](/lab4/image/8.png)

### Проброс порта
Пробрасываем порт для подключения к сервису через браузер: 
```
kubectl port-forward service/lab4-service 8200:3000
```
Можно перейти по ссылке: `http://localhost:8200/`.

![](/lab4/image/9.png)

### Ping

Пингуем с контейнера `lab4-deployment-767f5987bb-cxftn` контейнеру с IP-адресом: `ping 192.168.0.78` с помощью команды:

```
kubectl exec -ti lab4-deployment-767f5987bb-cxftn -- sh
```

![](/lab4/image/10.png)

Чтобы выйти из контейнера, используем команду `exit`.

Пингуем с контейнера `lab4-deployment-84c64d85b4-rn4hf` контейнеру с IP-адресом: `ping 192.168.0.79` с помощью команды:

```
kubectl exec -ti lab4-deployment-767f5987bb-5k9jt -- sh
```

![](/lab4/image/11.png)

### Схема
Схема организации Node, нарисованная в [draw.io](https://app.diagrams.net/).

![]()

