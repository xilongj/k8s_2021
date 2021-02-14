## kubectl
### Basic Commands
```buildoutcfg
[root@hdss7-21 ~]# kubectl get namespace
NAME              STATUS   AGE
default           Active   2d
kube-node-lease   Active   2d
kube-public       Active   2d
kube-system       Active   2d
```
```buildoutcfg
[root@hdss7-22 ~]# kubectl get ns
NAME              STATUS   AGE
default           Active   2d
kube-node-lease   Active   2d
kube-public       Active   2d
kube-system       Active   2d
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get all [-n default]
NAME                 READY   STATUS    RESTARTS   AGE
pod/nginx-ds-4s9z8   1/1     Running   0          25h
pod/nginx-ds-pgmrh   1/1     Running   0          25h


NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   192.168.0.1   <none>        443/TCP   2d

NAME                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/nginx-ds   2         2         2       2            2           <none>          22h
```
```buildoutcfg
[root@hdss7-22 ~]# kubectl get all
NAME                 READY   STATUS    RESTARTS   AGE
pod/nginx-ds-4s9z8   1/1     Running   0          25h
pod/nginx-ds-pgmrh   1/1     Running   0          25h


NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   192.168.0.1   <none>        443/TCP   2d

NAME                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/nginx-ds   2         2         2       2            2           <none>          22h
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-1               Healthy   {"health": "true"}
etcd-0               Healthy   {"health": "true"}
etcd-2               Healthy   {"health": "true"}
```
#### create namespace
```buildoutcfg
[root@hdss7-21 ~]# kubectl create namespace app
namespace/app created
[root@hdss7-21 ~]# kubectl get namespace
NAME              STATUS   AGE
app               Active   28s
default           Active   2d
kube-node-lease   Active   2d
kube-public       Active   2d
kube-system       Active   2d
```
#### delete namespace
```buildoutcfg
[root@hdss7-22 ~]# kubectl delete ns app
namespace "app" deleted
[root@hdss7-22 ~]# kubectl get ns
NAME              STATUS   AGE
default           Active   2d
kube-node-lease   Active   2d
kube-public       Active   2d
kube-system       Active   2d
```
#### create deploy(ment)
```buildoutcfg
[root@hdss7-21 ~]# kubectl create deployment nginx-dp --image=harbor.od.com/public/nginx:1.7.9 -n kube-public
deployment.apps/nginx-dp created

[root@hdss7-21 ~]# kubectl get deploy -n kube-public
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
nginx-dp   1/1     1            1           23s

[root@hdss7-21 ~]# kubectl get pods -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE
nginx-dp-656b87bf6d-sgjjd   1/1     Running   0          58s

[root@hdss7-21 ~]# kubectl get deployment
No resources found.

[root@hdss7-21 ~]# kubectl get deployment -n kube-public
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
nginx-dp   1/1     1            1           6m14s
[root@hdss7-21 ~]# kubectl get pods -n kube-public -o wide
NAME                        READY   STATUS    RESTARTS   AGE    IP           NODE                NOMINATED NODE   READINESS GATES
nginx-dp-656b87bf6d-sgjjd   1/1     Running   0          3m3s   172.7.21.3   hdss7-21.host.com   <none>           <none>
```
```buildoutcfg
[root@hdss7-22 ~]# kubectl describe deployment nginx-dp -n kube-public
Name:                   nginx-dp
Namespace:              kube-public
CreationTimestamp:      Sun, 14 Feb 2021 14:08:01 +0800
Labels:                 app=nginx-dp
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx-dp
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx-dp
  Containers:
   nginx:
    Image:        harbor.od.com/public/nginx:1.7.9
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-dp-656b87bf6d (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  8m28s  deployment-controller  Scaled up replica set nginx-dp-656b87bf6d to 1
```
#### enter pod resource
```buildoutcfg
[root@hdss7-21 ~]# kubectl get pods -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE
nginx-dp-656b87bf6d-sgjjd   1/1     Running   0          23m

[root@hdss7-21 ~]# kubectl exec -it nginx-dp-656b87bf6d-sgjjd /bin/bash -n kube-public
root@nginx-dp-656b87bf6d-sgjjd:/# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
8: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:07:15:03 brd ff:ff:ff:ff:ff:ff
    inet 172.7.21.3/24 brd 172.7.21.255 scope global eth0
       valid_lft forever preferred_lft forever
```
```buildoutcfg
[root@hdss7-21 ~]# docker ps -a | grep nginx-dp
8e5868a2e927   84581e99d807                        "nginx -g 'daemon of…"   26 minutes ago   Up 26 minutes             k8s_nginx_nginx-dp-656b87bf6d-sgjjd_kube-public_2cc9d556-045a-4dcf-8eb5-e1f51ce88a19_0
310d4218461a   harbor.od.com/public/pause:latest   "/pause"                 26 minutes ago   Up 26 minutes             k8s_POD_nginx-dp-656b87bf6d-sgjjd_kube-public_2cc9d556-045a-4dcf-8eb5-e1f51ce88a19_0
```
#### delete pod resource
```buildoutcfg
[root@hdss7-21 ~]# kubectl get pods -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE
nginx-dp-656b87bf6d-sgjjd   1/1     Running   0          104m
[root@hdss7-21 ~]# kubectl delete pod nginx-dp-656b87bf6d-sgjjd -n kube-public
pod "nginx-dp-656b87bf6d-sgjjd" deleted
```
```buildoutcfg
[root@hdss7-22 ~]# watch -n 1 'kubectl describe deployment nginx-dp -n kube-public'
Every 1.0s: kubectl describe deployment nginx-dp -n kube-public                                                                                                   Sun Feb 14 15:54:08 2021

Name:                   nginx-dp
Namespace:              kube-public
CreationTimestamp:      Sun, 14 Feb 2021 14:08:01 +0800
Labels:                 app=nginx-dp
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx-dp
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx-dp
  Containers:
   nginx:
    Image:        harbor.od.com/public/nginx:1.7.9
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-dp-656b87bf6d (1/1 replicas created)
Events:          <none>
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get pods -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE
nginx-dp-656b87bf6d-znzjw   1/1     Running   0          53s
```
```buildoutcfg
[root@hdss7-22 ~]# kubectl get pods -n kube-public -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
nginx-dp-656b87bf6d-znzjw   1/1     Running   0          3m36s   172.7.22.3   hdss7-22.host.com   <none>           <none>
```
```buildoutcfg
[root@hdss7-22 ~]# kubectl get pods -n kube-public -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
nginx-dp-656b87bf6d-znzjw   1/1     Running   0          10m   172.7.22.3   hdss7-22.host.com   <none>           <none>

[root@hdss7-22 ~]# kubectl delete pod nginx-dp-656b87bf6d-znzjw -n kube-public
pod "nginx-dp-656b87bf6d-znzjw" deleted
[root@hdss7-22 ~]# kubectl get pods -n kube-public -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
nginx-dp-656b87bf6d-g9krn   1/1     Running   0          15s   172.7.21.3   hdss7-21.host.com   <none>           <none>

[root@hdss7-22 ~]# kubectl delete pod nginx-dp-656b87bf6d-g9krn -n kube-public --force --grace-period=0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "nginx-dp-656b87bf6d-g9krn" force deleted

[root@hdss7-22 ~]# kubectl get pods -n kube-public -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
nginx-dp-656b87bf6d-qw5f5   1/1     Running   0          15s   172.7.22.3   hdss7-22.host.com   <none>           <none>

[root@hdss7-22 ~]# kubectl delete pod nginx-dp-656b87bf6d-qw5f5 -n kube-public --force --grace-period=0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "nginx-dp-656b87bf6d-qw5f5" force deleted
[root@hdss7-22 ~]# kubectl get pods -n kube-public -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
nginx-dp-656b87bf6d-8cdzw   1/1     Running   0          4s    172.7.21.3   hdss7-21.host.com   <none>           <none>
```
#### delete deployment
```buildoutcfg
[root@hdss7-22 ~]# kubectl delete deployment nginx-dp -n kube-public
deployment.extensions "nginx-dp" deleted

[root@hdss7-22 ~]# kubectl get all -n kube-public


No resources found.
```
### Manage Service Resource




## flannel

## coredns

## traefik

## dashboard

## heapster

## k8s maintenance  