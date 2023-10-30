# 2023_2024-introduction_to_distributed_technologies-k4111c-ishutina_e
University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2023/2024
Group: K4111с
Author: Ishutina Yelizaveta
Lab: Lab4
Date of create: 30.10.2023
Date of finished: 16.10.2023

## Лабораторная работа №4 "Сети связи в Minikube, CNI и CoreDNS"
### Описание

Это последняя лабораторная работа в которой вы познакомитесь с сетями связи в Minikube. Особенность Kubernetes заключается в том, что у него одновременно работают `underlay` и `overlay`  сети, а управление может быть организованно различными CNI.

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
Запускается minikube с подключенным плагином [плагин](https://projectcalico.docs.tigera.io/getting-started/kubernetes/minikube) `CNI=calico`, режимом работы `Multi-Node Clusters` и разворачиваются 2 ноды [2 Node](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/) с помощью команды:

```
minikube start --network-plugin=cni --cni=calico --nodes 2 -p multinode-demo
```
Проверить, запустились ли 2 Node можно с помощью команды:
```
kubectl get nodes
```
![]()

Проверить работу CNI Calico, проверяется количество подов с меткой **calico-node**. Их число должно совпадать с количеством нод. Проверяем с помощью следующей команды:
```
kubectl get pods -l k8s-app=calico-node -A
```
![]()

## 2. Пометка нод
В соответствии с заданием помечается каждая нода по географическому расположению

Для назначения IP адресов в Calico необходимо написать манифест для **IPPool** ресурса.

С помощью IPPool можно создать **IP-pool (блок IP-адресов)**, который выделяет IP-адреса только для узлов с определенной **меткой (label)**.

С помощью следующей команды, назначаются метки узлам :

```
kubectl label nodes multinode-demo zone=east  
kubectl label nodes multinode-demo-m02 zone=west
```
Далее из официальной документации Calico берется шаблон манифеста IPPool

```yaml
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
   name: zone-east-ippool
spec:
   cidr: 192.168.0.0/24
   ipipMode: Always
   natOutgoing: true
   nodeSelector: zone == "east"
---
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
   name: zone-west-ippool
spec:
   cidr: 192.168.1.0/24
   ipipMode: Always
   natOutgoing: true
   nodeSelector: zone == "west"
```

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
![]()

Перед созданием IPPools, проверяются созданные по-умолчанию и удаляются: 
```
kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch get ippools -o wide
```
```
kubectl delete ippools default-ipv4-ippool
```

![]()

Теперь можно приступить к созданию IPPools с помощью команды:
```
kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch create -f - < lab4-ippool.yaml
```
![]()

С помощью следующей команды проверяется, что создалось ва pool'а:
```
kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch get ippool -o wide
```
![]()

Заметим, что их [маски подсети (CIDR)](https://ru.wikipedia.org/wiki/Маска_подсети) соответствуют тем, которые указаны в диапазон pool'ов в манифесте. Прочитать про CIDR можно [здесь](https://habr.com/ru/post/351574/).

##3. Deployment и Service
Манифест для развертывания берем из 2 лабораторной работы и заменяем метку на `lab4-frontend`.

[Шаблон манифеста](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/?hl=ru) для сервиса типа Load-Balancer возьмем с официальной документации и также заменим метку на `lab4-frontend`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lab4-service
spec:
  selector:
    app: lab4-frontend
  ports:
    - port: 3000
      targetPort: 3000
  type: LoadBalancer
```

Переходим в папку с .yaml файлом и выполняем команду:
```
kubectl apply -f lab4-deployment.yaml -f lab4-service.yaml
```

Проверяем, что появилось развертывание и сервис: `kubectl get deployments`, `kubectl get services`.

![]()

Проверяем IP созданных Pod'ов: `kubectl get pods -o wide`.

![]()
### Проброс порта
Пробрасываем порт для подключения к сервису через браузер: `kubectl port-forward service/lab4-service 8200:3000`.

Переходим по ссылке: `http://localhost:8200/`.

![]()

### Попингуем?

Пингуем с контейнера `lab4-deployment-84c64d85b4-q2g8q` контейнеру с IP-адресом: `ping 192.168.0.71` с помощью команды:

```
kubectl exec -ti lab4-deployment-84c64d85b4-q2g8q -- sh
```

![]()
Чтобы выйти из контейнера, используем команду `exit`.

Пингуем с контейнера `lab4-deployment-84c64d85b4-rn4hf` контейнеру с IP-адресом: `ping 192.168.1.194` с помощью команды:

```
kubectl exec -ti lab4-deployment-84c64d85b4-rn4hf -- sh
```

![]()

### Диаграмма
Схема организации Node, нарисованная в [draw.io](https://app.diagrams.net/).

![]()

---
## Ошибки (в хронологическом порядке)

---
## Полезные ссылки
1. [Как pod в Kubernetes получает IP-адрес](https://habr.com/ru/company/flant/blog/521406/)
2. [CoreDNS — DNS-сервер для мира cloud native и Service Discovery для Kubernetes](https://habr.com/ru/company/flant/blog/331872/)