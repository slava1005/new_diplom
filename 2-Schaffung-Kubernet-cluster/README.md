## Создание Kubernetes кластера
```
На этом этапе необходимо создать Kubernetes кластер на базе предварительно созданной инфраструктуры. Требуется обеспечить доступ к ресурсам из Интернета.

Это можно сделать двумя способами:

Рекомендуемый вариант: самостоятельная установка Kubernetes кластера.
а. При помощи Terraform подготовить как минимум 3 виртуальных машины Compute Cloud для создания Kubernetes-кластера.
Тип виртуальной машины следует выбрать самостоятельно с учётом требовании к производительности и стоимости. Если в дальнейшем поймете,
что необходимо сменить тип инстанса, используйте Terraform для внесения изменений.
б. Подготовить ansible конфигурации, можно воспользоваться, например Kubespray
в. Задеплоить Kubernetes на подготовленные ранее инстансы, в случае нехватки каких-либо ресурсов вы всегда можете создать их при помощи Terraform.
Альтернативный вариант: воспользуйтесь сервисом Yandex Managed Service for Kubernetes
а. С помощью terraform resource для kubernetes создать региональный мастер kubernetes с размещением нод в разных 3 подсетях
б. С помощью terraform resource для kubernetes node group
Ожидаемый результат:

Работоспособный Kubernetes кластер.
В файле ~/.kube/config находятся данные для доступа к кластеру.
Команда kubectl get pods --all-namespaces отрабатывает без ошибок.
```

## Решение:
### На этом этапе создадим Kubernetes кластер на базе предварительно созданной инфраструктуры.

### 2.1. При помощи Terraform подготовим 3 виртуальных машины Compute Cloud для создания Kubernetes-кластера: 
### 1 master-node и 2 worker-node.

Проверим созданные ресурсы с помощью CLI:
```
slava@DESKTOP-QKJU13U:~/new_diplom/2-Schaffung-Kubernet-cluster/config/terraform$ yc vpc network list
+----------------------+------+
|          ID          | NAME |
+----------------------+------+
| enpb0he038virp4qccsa | vpc0 |
+----------------------+------+

slava@DESKTOP-QKJU13U:~/new_diplom/2-Schaffung-Kubernet-cluster/config/terraform$ yc vpc subnet list
+----------------------+----------+----------------------+----------------+---------------+---------------+
|          ID          |   NAME   |      NETWORK ID      | ROUTE TABLE ID |     ZONE      |     RANGE     |
+----------------------+----------+----------------------+----------------+---------------+---------------+
| e2laqsoa69jg2ovi6deg | subnet-b | enpb0he038virp4qccsa |                | ru-central1-b | [10.0.2.0/24] |
| e9bca66l3pckjek8rmv9 | subnet-a | enpb0he038virp4qccsa |                | ru-central1-a | [10.0.1.0/24] |
| fl8220vt7dkst1of7aei | subnet-d | enpb0he038virp4qccsa |                | ru-central1-d | [10.0.3.0/24] |
+----------------------+----------+----------------------+----------------+---------------+---------------+

slava@DESKTOP-QKJU13U:~/new_diplom/2-Schaffung-Kubernet-cluster/config/terraform$ yc compute instance list
+----------------------+----------+---------------+---------+----------------+-------------+
|          ID          |   NAME   |    ZONE ID    | STATUS  |  EXTERNAL IP   | INTERNAL IP |
+----------------------+----------+---------------+---------+----------------+-------------+
| fhm4p0aadf6an7e8vnr6 | worker-1 | ru-central1-a | RUNNING | 84.201.159.138 | 10.0.1.19   |
| fhm8ummvgrpv1c4u0kam | worker-2 | ru-central1-a | RUNNING | 89.169.142.229 | 10.0.1.10   |
| fhmgdo9lh7u4bakip4hv | master-1 | ru-central1-a | RUNNING | 51.250.7.66    | 10.0.1.34   |
+----------------------+----------+---------------+---------+----------------+-------------+

slava@DESKTOP-QKJU13U:~/new_diplom/2-Schaffung-Kubernet-cluster/config/terraform$ yc storage bucket list
+--------------------------+----------------------+----------+-----------------------+---------------------+
|           NAME           |      FOLDER ID       | MAX SIZE | DEFAULT STORAGE CLASS |     CREATED AT      |
+--------------------------+----------------------+----------+-----------------------+---------------------+
| state-storage-11-02-2025 | b1gfurnb189rkavk8tlr |        0 | STANDARD              | 2025-02-11 15:29:03 |
+--------------------------+----------------------+----------+-----------------------+---------------------+

slava@DESKTOP-QKJU13U:~/new_diplom/2-Schaffung-Kubernet-cluster/config/terraform$ yc storage bucket stats --name  state-storage-11-02-2025
name: state-storage-11-02-2025
used_size: "27219"
storage_class_used_sizes:
  - storage_class: STANDARD
    class_size: "27219"
storage_class_counters:
  - storage_class: STANDARD
    counters:
      simple_object_size: "27219"
      simple_object_count: "1"
default_storage_class: STANDARD
anonymous_access_flags:
  read: false
  list: false
  config_read: false
created_at: "2025-02-11T15:29:03.391547Z"
updated_at: "2025-02-11T15:39:58.507834Z"
```
Так же, помимо создание ВМ, сделаем с помощью terraform генерацию файлика hosts.yml с разу в папку с ansible для последующего cоздания kubernetes-кластера 
при помощи ansible-playbook kubsprey. Файл hosts.yml
```
all:
  hosts:
    master-1:
      ansible_host: 51.250.71.6
      ip: 10.0.1.34
      access_ip: 10.0.1.34
      ansible_user: debian
    worker-1:
      ansible_host: 89.169.147.181
      ip: 10.0.1.19
      access_ip: 10.0.1.19
      ansible_user: debian
    worker-2:
      ansible_host: 84.252.129.115
      ip: 10.0.1.10
      access_ip: 10.0.1.10
      ansible_user: debian
  children:
    kube_control_plane:
      hosts:
        master-1:
    kube_node:
      hosts:
        worker-1:
        worker-2:
    etcd:
      hosts:
        master-1:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
```

### 2.2. Подготовим ansible конфигурацию для установки kubspray.

Запустим выполнение ansible-playbook на master-node, который скачает kubspray, установит все необходимые для него зависимости 
из файла requirements.txt и скопирует на мастер приватный ключ.
```
slava@DESKTOP-QKJU13U:~/new_diplom/2-Schaffung-Kubernet-cluster/config/ansible$ ansible-playbook -i inventory/hosts.yml site.yml                                                                              
PLAY [Установка pip] ***************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
The authenticity of host '51.250.7.66 (51.250.7.66)' can't be established.
ED25519 key fingerprint is SHA256:XlB+9TFbWZU3hK36UNQp3JCKAMegOjoLJ1VKQjH+nOY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
ok: [master-1]

TASK [Скачиваем файл get-pip.py] ***************************************************************************************************
changed: [master-1]

TASK [Удаляем EXTERNALLY-MANAGED] **************************************************************************************************
changed: [master-1]

TASK [Устанавливаем pip] ***********************************************************************************************************
changed: [master-1]

PLAY [Установка зависимостей из ansible-playbook kubespray] ************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [master-1]

TASK [Выполнение apt update и установка git] ***************************************************************************************
changed: [master-1]

TASK [Клонируем kubespray из репозитория] ******************************************************************************************
changed: [master-1]

TASK [Изменение прав на папку kubspray] ********************************************************************************************
changed: [master-1]

TASK [Установка зависимостей из requirements.txt] **********************************************************************************
changed: [master-1]

TASK [Копирование содержимого папки inventory/sample в папку inventory/mycluster] **************************************************
changed: [master-1]

PLAY [Подготовка master-node к установке kubespray из ansible-playbook] ************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [master-1]

TASK [Копирование на master-node файла hosts.yml] **********************************************************************************
changed: [master-1]

TASK [Копирование на мастер приватного ключа] **************************************************************************************
changed: [master-1]

PLAY RECAP *************************************************************************************************************************
master-1                   : ok=13   changed=10   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Далее зайдем на master-node и запустим ansible-playbook kubspray для установки кластера kubernetes с помощью следующей команды.

```
ansible-playbook -i inventory/mycluster/hosts.yml cluster.yml -b -v -u debian
```

Результат выполнения playbook:

```
PLAY RECAP ***************************************************************************************************************************************************************************************************
master-1                   : ok=642  changed=40   unreachable=0    failed=0    skipped=1096 rescued=0    ignored=5
worker-1                   : ok=445  changed=12   unreachable=0    failed=0    skipped=680  rescued=0    ignored=1
worker-2                   : ok=445  changed=12   unreachable=0    failed=0    skipped=678  rescued=0    ignored=1

Wednesday 12 February 2025  05:42:31 +0000 (0:00:00.113)       0:10:19.746 ****
===============================================================================
kubernetes/kubeadm : Join to cluster if needed ------------------------------------------------------------------------------------------------------------------------------------------------------- 16.95s
kubernetes/control-plane : Kubeadm | Initialize first control plane node ----------------------------------------------------------------------------------------------------------------------------- 16.00s
container-engine/runc : Download_file | Download item ------------------------------------------------------------------------------------------------------------------------------------------------ 12.53s
container-engine/nerdctl : Download_file | Download item --------------------------------------------------------------------------------------------------------------------------------------------- 12.14s
container-engine/containerd : Download_file | Download item ------------------------------------------------------------------------------------------------------------------------------------------ 12.14s
container-engine/crictl : Download_file | Download item ---------------------------------------------------------------------------------------------------------------------------------------------- 11.82s
container-engine/crictl : Extract_file | Unpacking archive -------------------------------------------------------------------------------------------------------------------------------------------- 9.37s
container-engine/nerdctl : Extract_file | Unpacking archive ------------------------------------------------------------------------------------------------------------------------------------------- 8.26s
kubernetes-apps/ansible : Kubernetes Apps | CoreDNS --------------------------------------------------------------------------------------------------------------------------------------------------- 7.88s
download : Download_file | Download item -------------------------------------------------------------------------------------------------------------------------------------------------------------- 5.57s
network_plugin/cni : CNI | Copy cni plugins ----------------------------------------------------------------------------------------------------------------------------------------------------------- 5.56s
etcdctl_etcdutl : Download_file | Download item ------------------------------------------------------------------------------------------------------------------------------------------------------- 5.12s
etcdctl_etcdutl : Extract_file | Unpacking archive ---------------------------------------------------------------------------------------------------------------------------------------------------- 4.92s
download : Download | Download files / images --------------------------------------------------------------------------------------------------------------------------------------------------------- 4.85s
container-engine/containerd : Containerd | Unpack containerd archive ---------------------------------------------------------------------------------------------------------------------------------- 4.69s
network_plugin/calico : Calico | Create calico manifests ---------------------------------------------------------------------------------------------------------------------------------------------- 4.58s
container-engine/runc : Download_file | Create dest directory on node --------------------------------------------------------------------------------------------------------------------------------- 4.25s
container-engine/nerdctl : Download_file | Create dest directory on node ------------------------------------------------------------------------------------------------------------------------------ 4.19s
container-engine/crictl : Download_file | Create dest directory on node ------------------------------------------------------------------------------------------------------------------------------- 4.18s
network_plugin/calico : Start Calico resources -------------------------------------------------------------------------------------------------------------------------------------------------------- 4.17s
```

Для выполнения команд kubectl без sudo скопируем папку .kube в домашнюю дирректорию пользователя и сменим владельца, а также группу владельцев папки с файлами:

```
debian@master-1:~/kubespray$ sudo cp -r /root/.kube ~/
debian@master-1:~/kubespray$ sudo chown -R debian:debian ~/.kube
```

Проверим работоспособность кластера:

```
debian@master-1:~/kubespray$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-69d8557557-hrgtf   1/1     Running   0          4m14s
kube-system   calico-node-dmp8j                          1/1     Running   0          4m48s
kube-system   calico-node-lvbgf                          1/1     Running   0          4m48s
kube-system   calico-node-zrxj2                          1/1     Running   0          4m49s
kube-system   coredns-5c54f84c97-5qgl4                   1/1     Running   0          4m6s
kube-system   coredns-5c54f84c97-rk29d                   1/1     Running   0          3m48s
kube-system   dns-autoscaler-76ddddbbc-hxtpr             1/1     Running   0          4m4s
kube-system   kube-apiserver-master-1                    1/1     Running   0          6m17s
kube-system   kube-controller-manager-master-1           1/1     Running   1          6m20s
kube-system   kube-proxy-9gdkg                           1/1     Running   0          5m27s
kube-system   kube-proxy-hkp92                           1/1     Running   0          5m27s
kube-system   kube-proxy-qmrgc                           1/1     Running   0          5m26s
kube-system   kube-scheduler-master-1                    1/1     Running   1          6m17s
kube-system   nginx-proxy-worker-1                       1/1     Running   0          5m30s
kube-system   nginx-proxy-worker-2                       1/1     Running   0          5m30s
kube-system   nodelocaldns-r2cj8                         1/1     Running   0          3m58s
kube-system   nodelocaldns-tw72v                         1/1     Running   0          3m58s
kube-system   nodelocaldns-vbhzs                         1/1     Running   0          3m58s
```
