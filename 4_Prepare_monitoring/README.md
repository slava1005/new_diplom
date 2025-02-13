Подготовка cистемы мониторинга и деплой приложения
Решение:
4.1 Деплой в кластер prometheus, grafana, alertmanager, экспортер основных метрик Kubernetes.
Для решения задачи деплоя в кластер prometheus, grafana, alertmanager, экспортер основных метрик Kubernetes воспользуемся решением kube-prometheus.

Склонируем репозиторий

git clone https://github.com/prometheus-operator/kube-prometheus.git
Переходим в папку с kube-prometheus

debian@master-1:~$ cd kube-prometheus/
Создадим мониторинг стека с использование конфигурации в manifests каталоге:

debian@master-1:~$ kubectl apply --server-side -f manifests/setup
debian@master-1:~$ kubectl wait \
 --for condition=Established \
 --all CustomResourceDefinition \
 --namespace=monitoring
debian@master-1:~$ kubectl apply -f manifests/
Убедимся, что маниторинг развернулся и работает:

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

Чтобы подключиться снаружи к Grafana необходимо изменить порт с ClusterIP на NodePort

debian@master-1:~/kube-prometheus$ cat <<EOF > ~/patch.yml
spec:
  type: NodePort
EOF
debian@master-1:~/kube-prometheus$ kubectl patch svc grafana -n monitoring --patch-file ~/patch.yml
service/grafana patched
Проверим изменения

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

Проверим Http доступ к web интерфейсу grafana:

И ТУТ ВСЕ... не могу подключиться 