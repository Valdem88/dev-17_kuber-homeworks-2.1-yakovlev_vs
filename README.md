# Домашнее задание к занятию «Хранение в K8s. Часть 1» dev-17_kuber-homeworks-2.1-yakovlev_vs
2.1 Хранение в K8s. Часть 1

### Цель задания

В тестовой среде Kubernetes нужно обеспечить обмен файлами между контейнерам пода и доступ к логам ноды.

------

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным GitHub-репозиторием.

------

### Дополнительные материалы для выполнения задания

1. [Инструкция по установке MicroK8S](https://microk8s.io/docs/getting-started).
2. [Описание Volumes](https://kubernetes.io/docs/concepts/storage/volumes/).
3. [Описание Multitool](https://github.com/wbitt/Network-MultiTool).

------

### Задание 1 

**Что нужно сделать**

Создать Deployment приложения, состоящего из двух контейнеров и обменивающихся данными.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Сделать так, чтобы busybox писал каждые пять секунд в некий файл в общей директории.
3. Обеспечить возможность чтения файла контейнером multitool.
4. Продемонстрировать, что multitool может читать файл, который периодически обновляется.
5. Предоставить манифесты Deployment в решении, а также скриншоты или вывод команды из п. 4.

#### Решение 

- [Deployment-volume.yaml](file/Deployment-volume.yaml)

Запускаем deployment, проверяем что pod поднялся и контейнеры работают. 
```bash
$ kubectl apply -f file/Deployment-volume.yaml 
deployment.apps/volume-deployment created

$ kubectl get pod
NAME                                 READY   STATUS     RESTARTS   AGE
myapp-pod-76f4f9959f-fr8kg           0/1     Unknown    3          21d
myapp-pod-76f4f9959f-k2cjw           0/1     Evicted    0          78m
myapp-pod-76f4f9959f-bshnq           0/1     Init:0/1   0          78m
volume-deployment-6fff9cd7b4-wsrgv   2/2     Running    0          9s

$ kubectl describe pod volume-deployment-6fff9cd7b4-wsrgv
Name:         volume-deployment-6fff9cd7b4-wsrgv
Namespace:    default
Priority:     0
Node:         microk8s/192.168.1.88
Start Time:   Fri, 10 Mar 2023 15:33:40 +0300
Labels:       app=main2
              pod-template-hash=6fff9cd7b4
Annotations:  cni.projectcalico.org/containerID: e8c938a93f37942c2cf91059e8dd83c9365157d24882c4ca8f0735e9f2f82f4f
              cni.projectcalico.org/podIP: 10.1.128.206/32
              cni.projectcalico.org/podIPs: 10.1.128.206/32
Status:       Running
IP:           10.1.128.206
IPs:
  IP:           10.1.128.206
Controlled By:  ReplicaSet/volume-deployment-6fff9cd7b4
Containers:
  busybox:
    Container ID:  containerd://393c7675410e7ce7df2e7365e5263ffcc352ffacb690f682ed9a0eed57a0a897
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:c118f538365369207c12e5794c3cbfb7b042d950af590ae6c287ede74f29b7d4
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      while true; do echo Success! >> /tmp/cache/success.txt; sleep 5; done
    State:          Running
      Started:      Fri, 10 Mar 2023 15:33:42 +0300
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /tmp/cache from my-vol (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-5rcwp (ro)
  network-multitool:
    Container ID:   containerd://acd9b352c09ca264f220438c9a88d8ff688cee9eec1e7431d41876ed0ad6dac8
    Image:          wbitt/network-multitool
    Image ID:       docker.io/wbitt/network-multitool@sha256:82a5ea955024390d6b438ce22ccc75c98b481bf00e57c13e9a9cc1458eb92652
    Ports:          80/TCP, 443/TCP
    Host Ports:     0/TCP, 0/TCP
    State:          Running
      Started:      Fri, 10 Mar 2023 15:33:44 +0300
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     10m
      memory:  20Mi
    Requests:
      cpu:     1m
      memory:  20Mi
    Environment:
      HTTP_PORT:   80
      HTTPS_PORT:  443
    Mounts:
      /static from my-vol (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-5rcwp (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  my-vol:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  kube-api-access-5rcwp:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```

Busybox пишет в файл /tmp/cache/success.txt каждые 5 секунд. Для network-multitool файл должен находится по пути `/static`. Заходим в network-multitool и проверяем.

```bash
$ kubectl exec volume-deployment-6fff9cd7b4-wsrgv -c network-multitool -it -- sh

/ # ls -l
total 80
drwxr-xr-x    1 root     root          4096 Dec 19  2021 bin
drwx------    2 root     root          4096 Dec 19  2021 certs
drwxr-xr-x    5 root     root           360 Mar 10 12:33 dev
drwxr-xr-x    1 root     root          4096 Dec 19  2021 docker
drwxr-xr-x    1 root     root          4096 Mar 10 12:33 etc
drwxr-xr-x    2 root     root          4096 Nov 24  2021 home
drwxr-xr-x    1 root     root          4096 Dec 19  2021 lib
drwxr-xr-x    5 root     root          4096 Nov 24  2021 media
drwxr-xr-x    2 root     root          4096 Nov 24  2021 mnt
drwxr-xr-x    2 root     root          4096 Nov 24  2021 opt
dr-xr-xr-x  308 root     root             0 Mar 10 12:33 proc
drwx------    1 root     root          4096 Mar 10 12:38 root
drwxr-xr-x    1 root     root          4096 Mar 10 12:33 run
drwxr-xr-x    1 root     root          4096 Dec 19  2021 sbin
drwxr-xr-x    2 root     root          4096 Nov 24  2021 srv
drwxrwxrwx    2 root     root          4096 Mar 10 12:33 static
dr-xr-xr-x   13 root     root             0 Mar 10 12:33 sys
drwxrwxrwt    2 root     root          4096 Nov 24  2021 tmp
drwxr-xr-x    1 root     root          4096 Dec 19  2021 usr
drwxr-xr-x    1 root     root          4096 Dec 19  2021 var

/ # cat static/success.txt
Success!
Success!
Success!
Success!
Success!
Success!
Success!
Success!
Success!
Success!
Success!
Success!
Success!
Success!
Success!
Success!
Success!
```

------

### Задание 2

**Что нужно сделать**

Создать DaemonSet приложения, которое может прочитать логи ноды.

1. Создать DaemonSet приложения, состоящего из multitool.
2. Обеспечить возможность чтения файла `/var/log/syslog` кластера MicroK8S.
3. Продемонстрировать возможность чтения файла изнутри пода.
4. Предоставить манифесты Deployment, а также скриншоты или вывод команды из п. 2.

#### Решение

- [DaemonSet-volume.yaml](file/DaemonSet-volume.yaml)

```bash
$ kubectl apply -f file/DaemonSet-volume.yaml 
daemonset.apps/volume-daemonset configured

$ kubectl get pod
NAME                         READY   STATUS     RESTARTS   AGE
myapp-pod-76f4f9959f-fr8kg   0/1     Unknown    3          22d
myapp-pod-76f4f9959f-k2cjw   0/1     Evicted    0          25h
myapp-pod-76f4f9959f-bshnq   0/1     Init:0/1   1          25h
volume-daemonset-kjvvn       1/1     Running    0          17s
```
Проверяем возможность чтения файла изнутри пода

```bash
$ kubectl exec volume-daemonset-kjvvn -c network-multitool -it -- sh

/ # ls -l output/
total 13816
-rw-r--r--    1 root     root             0 Mar 10 11:13 alternatives.log
-rw-r--r--    1 root     root         40504 Feb 14 17:27 alternatives.log.1
-rw-r-----    1 root     adm            429 Mar 11 12:02 apport.log
-rw-r-----    1 root     adm            429 Mar 10 11:14 apport.log.1
-rw-r-----    1 root     adm            235 Feb 24 11:30 apport.log.2.gz
-rw-r-----    1 root     adm            235 Feb 22 17:40 apport.log.3.gz
-rw-r-----    1 root     adm            238 Feb 21 17:47 apport.log.4.gz
-rw-r-----    1 root     adm            238 Feb 16 18:21 apport.log.5.gz
-rw-r-----    1 root     adm            236 Feb 15 15:58 apport.log.6.gz
-rw-r-----    1 root     adm            393 Feb 14 15:49 apport.log.7.gz
drwxr-xr-x    2 root     root          4096 Mar 10 11:13 apt
-rw-r-----    1 107      adm           6905 Mar 11 12:30 auth.log
-rw-r-----    1 107      adm          11946 Mar 10 11:13 auth.log.1
-rw-r-----    1 107      adm           7060 Feb 21 17:47 auth.log.2.gz
-rw-r--r--    1 root     root         64549 Aug  9  2022 bootstrap.log
-rw-rw----    1 root     43               0 Mar 10 11:13 btmp
-rw-rw----    1 root     43            1536 Feb 10 20:55 btmp.1
drwxr-xr-x    3 root     root          4096 Feb 11 14:56 calico
-rw-r-----    1 root     adm          31710 Mar 11 12:02 cloud-init-output.log
-rw-r--r--    1 107      adm         850752 Mar 11 12:02 cloud-init.log
drwxr-xr-x    2 root     root         12288 Mar 11 13:00 containers
drwxr-xr-x    2 root     root          4096 Mar 11 12:02 cups
drwxr-xr-x    2 root     root          4096 Aug  3  2022 dist-upgrade
-rw-r-----    1 root     adm          50936 Mar 11 12:02 dmesg
-rw-r-----    1 root     adm          50850 Mar 10 11:13 dmesg.0
-rw-r-----    1 root     adm          15434 Feb 24 11:30 dmesg.1.gz
-rw-r-----    1 root     adm          15410 Feb 22 17:40 dmesg.2.gz
-rw-r-----    1 root     adm          15270 Feb 21 17:47 dmesg.3.gz
-rw-r-----    1 root     adm          15151 Feb 16 18:21 dmesg.4.gz
-rw-r--r--    1 root     root             0 Mar 10 11:13 dpkg.log
-rw-r--r--    1 root     root       1205055 Feb 22 18:18 dpkg.log.1
-rw-r--r--    1 root     root      18713248 Feb 11 15:10 faillog
-rw-r--r--    1 root     root         10487 Feb 10 20:13 fontconfig.log
drwx--x--x    2 root     135           4096 Feb 10 20:17 gdm3
-rw-r--r--    1 root     root          1300 Mar 11 12:02 gpu-manager.log
drwxr-xr-x    3 root     root          4096 Feb 10 20:11 hp
drwxr-x---    4 root     adm           4096 Feb 10 19:58 installer
drwxr-sr-x    3 root     nginx         4096 Feb 10 19:59 journal
-rw-r-----    1 107      adm          83851 Mar 11 13:00 kern.log
-rw-r-----    1 107      adm         211795 Mar 10 11:13 kern.log.1
-rw-r-----    1 107      adm         115506 Feb 21 17:47 kern.log.2.gz
drwxr-xr-x    2 111      117           4096 Feb 10 20:07 landscape
-rw-rw-r--    1 root     43       170758388 Mar 11 12:21 lastlog
drwxr-xr-x    2 root     root          4096 Jul 14  2022 openvpn
drwxr-xr-x   12 root     root          4096 Mar 11 13:01 pods
drwx------    2 root     root          4096 Aug  9  2022 private
drwx------    2 ntp      root          4096 Aug 31  2022 speech-dispatcher
drwxr-x---    2 root     root          4096 Oct  4 23:04 sssd
-rw-r-----    1 107      adm        2954203 Mar 11 13:01 syslog
-rw-r-----    1 107      adm        7250363 Mar 10 11:13 syslog.1
-rw-r-----    1 107      adm         814429 Feb 21 17:47 syslog.2.gz
-rw-r--r--    1 root     root           460 Mar 11 12:15 ubuntu-advantage-timer.log
-rw-r--r--    1 root     root          1840 Feb 24 12:02 ubuntu-advantage-timer.log.1
-rw-r--r--    1 root     root             0 Mar 10 11:13 ubuntu-advantage.log
-rw-r--r--    1 root     root          1665 Feb 21 19:19 ubuntu-advantage.log.1
-rw-r-----    1 107      adm              0 Feb 21 17:47 ufw.log
-rw-r-----    1 107      adm          51947 Feb 14 19:43 ufw.log.1
drwxr-x---    2 root     adm           4096 Mar 10 11:13 unattended-upgrades
-rw-rw-r--    1 root     43           16512 Mar 11 12:21 wtmp


/ # cat /output/syslog | grep "Created slice User"
Mar 10 14:13:48 microk8s systemd[1206]: Created slice User Application Slice.
Mar 10 14:13:48 microk8s systemd[1206]: Created slice User Background Tasks Slice.
Mar 10 14:13:48 microk8s systemd[1206]: Created slice User Core Session Slice.
Mar 10 15:01:39 microk8s systemd[1]: Created slice User Slice of UID 0.
Mar 10 15:01:39 microk8s systemd[61383]: Created slice User Application Slice.
Mar 10 15:01:39 microk8s systemd[61383]: Created slice User Background Tasks Slice.
Mar 10 15:01:39 microk8s systemd[61383]: Created slice User Core Session Slice.
Mar 11 15:02:07 microk8s kernel: [    7.599154] systemd[1]: Created slice User and Session Slice.
Mar 11 15:02:13 microk8s systemd[1]: Created slice User Slice of UID 130.
Mar 11 15:02:14 microk8s systemd[1214]: Created slice User Application Slice.
Mar 11 15:02:14 microk8s systemd[1214]: Created slice User Background Tasks Slice.
Mar 11 15:02:14 microk8s systemd[1214]: Created slice User Core Session Slice.
Mar 11 15:21:25 microk8s systemd[1]: Created slice User Slice of UID 0.
Mar 11 15:21:25 microk8s systemd[26673]: Created slice User Application Slice.
Mar 11 15:21:25 microk8s systemd[26673]: Created slice User Background Tasks Slice.
Mar 11 15:21:25 microk8s systemd[26673]: Created slice User Core Session Slice.
```


------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------
