## Подготовка cистемы мониторинга и деплой приложения
```
Уже должны быть готовы конфигурации для автоматического создания облачной инфраструктуры и поднятия Kubernetes кластера.
Теперь необходимо подготовить конфигурационные файлы для настройки нашего Kubernetes кластера.

Цель:

Задеплоить в кластер prometheus, grafana, alertmanager, экспортер основных метрик Kubernetes.
Задеплоить тестовое приложение, например, nginx сервер отдающий статическую страницу.
Способ выполнения:

Воспользоваться пакетом kube-prometheus, который уже включает в себя Kubernetes оператор для grafana, prometheus, alertmanager
и node_exporter. Альтернативный вариант - использовать набор helm чартов от bitnami.

Если на первом этапе вы не воспользовались Terraform Cloud, то задеплойте и настройте в кластере atlantis для отслеживания изменений
инфраструктуры. Альтернативный вариант 3 задания: вместо Terraform Cloud или atlantis настройте на автоматический запуск и применение
конфигурации terraform из вашего git-репозитория в выбранной вами CI-CD системе при любом комите в main ветку. Предоставьте скриншоты
работы пайплайна из CI/CD системы.

Ожидаемый результат:

Git репозиторий с конфигурационными файлами для настройки Kubernetes.
Http доступ на 80 порту к web интерфейсу grafana.
Дашборды в grafana отображающие состояние Kubernetes кластера.
Http доступ на 80 порту к тестовому приложению.
```

## Решение:

### 4.1 Деплой в кластер prometheus, grafana, alertmanager, экспортер основных метрик Kubernetes.

Для решения задачи деплоя в кластер prometheus, grafana, alertmanager, экспортер основных метрик Kubernetes воспользуемся решением kube-prometheus.

Склонируем репозиторий

git clone https://github.com/prometheus-operator/kube-prometheus.git

Переходим в папку с kube-prometheus
```
debian@master-1:~$ cd kube-prometheus/
```
Создадим мониторинг стека с использование конфигурации в manifests каталоге:
```
debian@master-1:~$ kubectl apply --server-side -f manifests/setup
debian@master-1:~$ kubectl wait \
 --for condition=Established \
 --all CustomResourceDefinition \
 --namespace=monitoring
debian@master-1:~$ kubectl apply -f manifests/
```
Убедимся, что маниторинг развернулся и работает:
```
debian@master-1:~$ ~/kube-prometheus$ kubectl get all -n monitoring
NAME                                      READY   STATUS    RESTARTS   AGE
pod/alertmanager-main-0                   2/2     Running   0          16h
pod/alertmanager-main-1                   2/2     Running   0          16h
pod/alertmanager-main-2                   2/2     Running   0          16h
pod/blackbox-exporter-9b7bb8d56-kvpk2     3/3     Running   0          16h
pod/grafana-c49b8bf5c-qscp9               1/1     Running   0          16h
pod/kube-state-metrics-5f4b575dd7-pv26h   3/3     Running   0          16h
pod/node-exporter-hmzpf                   2/2     Running   0          16h
pod/node-exporter-nwgld                   2/2     Running   0          16h
pod/node-exporter-rrhf9                   2/2     Running   0          16h
pod/prometheus-adapter-599c88b6c4-dchbh   1/1     Running   0          16h
pod/prometheus-adapter-599c88b6c4-lmnws   1/1     Running   0          16h
pod/prometheus-k8s-0                      2/2     Running   0          16h
pod/prometheus-k8s-1                      2/2     Running   0          16h
pod/prometheus-operator-97cccfbf7-w7vzr   2/2     Running   0          16h

NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/alertmanager-main       ClusterIP   10.233.50.221   <none>        9093/TCP,8080/TCP            16h
service/alertmanager-operated   ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   16h
service/blackbox-exporter       ClusterIP   10.233.28.127   <none>        9115/TCP,19115/TCP           16h
service/grafana                 NodePort    10.233.26.247   <none>        3000:32041/TCP               16h
service/kube-state-metrics      ClusterIP   None            <none>        8443/TCP,9443/TCP            16h
service/node-exporter           ClusterIP   None            <none>        9100/TCP                     16h
service/prometheus-adapter      ClusterIP   10.233.2.9      <none>        443/TCP                      16h
service/prometheus-k8s          ClusterIP   10.233.51.127   <none>        9090/TCP,8080/TCP            16h
service/prometheus-operated     ClusterIP   None            <none>        9090/TCP                     16h
service/prometheus-operator     ClusterIP   None            <none>        8443/TCP                     16h

NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/node-exporter   3         3         3       3            3           kubernetes.io/os=linux   16h

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/blackbox-exporter     1/1     1            1           16h
deployment.apps/grafana               1/1     1            1           16h
deployment.apps/kube-state-metrics    1/1     1            1           16h
deployment.apps/prometheus-adapter    2/2     2            2           16h
deployment.apps/prometheus-operator   1/1     1            1           16h

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/blackbox-exporter-9b7bb8d56     1         1         1       16h
replicaset.apps/grafana-c49b8bf5c               1         1         1       16h
replicaset.apps/kube-state-metrics-5f4b575dd7   1         1         1       16h
replicaset.apps/prometheus-adapter-599c88b6c4   2         2         2       16h
replicaset.apps/prometheus-operator-97cccfbf7   1         1         1       16h

NAME                                 READY   AGE
statefulset.apps/alertmanager-main   3/3     16h
statefulset.apps/prometheus-k8s      2/2     16h
```
Чтобы подключиться снаружи к Grafana необходимо изменить порт с ClusterIP на NodePort
```
debian@master-1:~/kube-prometheus$ cat <<EOF > ~/patch.yml
spec:
  type: NodePort
EOF
debian@master-1:~/kube-prometheus$ kubectl patch svc grafana -n monitoring --patch-file ~/patch.yml
service/grafana patched
```
Проверим изменения
```
debian@master-1:~/kube-prometheus$ kubectl get svc -n monitoring
NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
alertmanager-main       ClusterIP   10.233.50.221   <none>        9093/TCP,8080/TCP            16h
alertmanager-operated   ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   16h
blackbox-exporter       ClusterIP   10.233.28.127   <none>        9115/TCP,19115/TCP           16h
grafana                 NodePort    10.233.26.247   <none>        3000:32041/TCP               16h
kube-state-metrics      ClusterIP   None            <none>        8443/TCP,9443/TCP            16h
node-exporter           ClusterIP   None            <none>        9100/TCP                     16h
prometheus-adapter      ClusterIP   10.233.2.9      <none>        443/TCP                      16h
prometheus-k8s          ClusterIP   10.233.51.127   <none>        9090/TCP,8080/TCP            16h
prometheus-operated     ClusterIP   None            <none>        9090/TCP                     16h
prometheus-operator     ClusterIP   None            <none>        8443/TCP                     16h
```
Проверим Http доступ к web интерфейсу grafana:
![img81111](https://github.com/user-attachments/assets/312c1ce1-b208-49c4-91c6-dcd66674c1fd)

Выведем в grafana Дашборды отображающие состояние Kubernetes кластера:
![img5_4](https://github.com/user-attachments/assets/02b66ddd-7804-4c05-8499-3c4433250af8)

### 4.2 Деплой тестового приложения, например, nginx сервер отдающий статическую страницу.
Создадим папку для приложения и перейдем в нее ```mkdir application && cd application```

Создадим namespace application для приложения

application-ns.yml
```
apiVersion: v1
kind: Namespace
metadata:
  name: application
```
Создадим DeamonSet, чтобы развернуть приложение на все worker-node:

Nginx-DaemonSet.yml
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-deamonset
  namespace: application
spec:
  selector:
    matchLabels:
      app: daemonset
  template:
    metadata:
      labels:
        app: daemonset
    spec:
      containers:
      - name: nginx
        image: slava1005/nginx:v1
```
Создадим Service для приложения с возможностью доступа снаружи кластера к nginx, используя тип порта NodePort.

Nginx-Service.yml
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: application
spec:
  ports:
    - name: nginx
      port: 80
      protocol: TCP
      targetPort: 80
      nodePort: 30000
  selector:
    app: daemonset
  type: NodePort
```
Для развертывания приложения воспользуемся инструментом Kustomize

kustomization.yml
```
namespace: application
resources:
- application-ns.yml
- Nginx-DaemonSet.yml
- Nginx-Service.yml
```
Развернем приложение
```
debian@master-1:~$ kubectl get all -n application
NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-deamonset-9w2ps   1/1     Running   0          3h50m
pod/nginx-deamonset-qrklj   1/1     Running   0          3h50m

NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/nginx-service   NodePort   10.233.43.95   <none>        80:30000/TCP   3h50m

NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/nginx-deamonset   2         2         2       2            2           <none>          3h50m
```
Проверим Http доступ к тестовому приложению:
![img1](https://github.com/user-attachments/assets/62e7cdc8-c188-4f4f-aa6d-a4f13b01fbef)













