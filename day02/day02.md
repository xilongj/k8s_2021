## kubectl
### Basic Commands
```buildoutcfg
[root@hdss7-22 ~]# kubectl --help
kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/overview/

Basic Commands (Beginner):
  create         Create a resource from a file or from stdin.
  expose         Take a replication controller, service, deployment or pod and expose it as a new Kubernetes Service
  run            Run a particular image on the cluster
  set            Set specific features on objects

Basic Commands (Intermediate):
  explain        Documentation of resources
  get            Display one or many resources
  edit           Edit a resource on the server
  delete         Delete resources by filenames, stdin, resources and names, or by resources and label selector

Deploy Commands:
  rollout        Manage the rollout of a resource
  scale          Set a new size for a Deployment, ReplicaSet, Replication Controller, or Job
  autoscale      Auto-scale a Deployment, ReplicaSet, or ReplicationController

Cluster Management Commands:
  certificate    Modify certificate resources.
  cluster-info   Display cluster info
  top            Display Resource (CPU/Memory/Storage) usage.
  cordon         Mark node as unschedulable
  uncordon       Mark node as schedulable
  drain          Drain node in preparation for maintenance
  taint          Update the taints on one or more nodes

Troubleshooting and Debugging Commands:
  describe       Show details of a specific resource or group of resources
  logs           Print the logs for a container in a pod
  attach         Attach to a running container
  exec           Execute a command in a container
  port-forward   Forward one or more local ports to a pod
  proxy          Run a proxy to the Kubernetes API server
  cp             Copy files and directories to and from containers.
  auth           Inspect authorization

Advanced Commands:
  diff           Diff live version against would-be applied version
  apply          Apply a configuration to a resource by filename or stdin
  patch          Update field(s) of a resource using strategic merge patch
  replace        Replace a resource by filename or stdin
  wait           Experimental: Wait for a specific condition on one or many resources.
  convert        Convert config files between different API versions
  kustomize      Build a kustomization target from a directory or a remote url.

Settings Commands:
  label          Update the labels on a resource
  annotate       Update the annotations on a resource
  completion     Output shell completion code for the specified shell (bash or zsh)

Other Commands:
  api-resources  Print the supported API resources on the server
  api-versions   Print the supported API versions on the server, in the form of "group/version"
  config         Modify kubeconfig files
  plugin         Provides utilities for interacting with plugins.
  version        Print the client and server version information

Usage:
  kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```
***
### Common Commands
```buildoutcfg
    1  kubectl get namespace
    2  kubectl get ns
    3  kubectl get all -n default
    4  kubectl create ns app01
    5  kubectl delete namespace app01
    6  kubectl create deployment web-dp --image=nginx -n kube-public
    7  kubectl get deploy -n kube-public
    8  kubectl get deploy -n kube-public -o wide
    9  kubectl describe deployment web-dp -n kube-public
   10  kubectl get pods -n kube-public
   11  kubectl exec -it web-dp-7955d7c58b-p65fc bash -n kube-public
   12  kubectl delete pod web-dp-7955d7c58b-p65fc -n kube-public
   13  kubectl delete deploy web-np -n kube-public
   14  kubectl expose deployment web-dp --port=80 -n kube-public
   15  kubectl scale deployment web-dp --replicas=2 -n kube-public
   16  ipvsadm -Ln
   17  kubectl describe svc web-dp -n kube-public
```
***
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
#### Create Namespace
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
#### Delete Namespace
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
#### Create Deploy(ment)
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
#### Enter pod Resource
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
#### Delete pod Resource
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
#### Delete Deployment
```buildoutcfg
[root@hdss7-22 ~]# kubectl delete deployment nginx-dp -n kube-public
deployment.extensions "nginx-dp" deleted

[root@hdss7-22 ~]# kubectl get all -n kube-public


No resources found.
```
#### Manage Service Resource
```buildoutcfg
[root@hdss7-21 ~]# kubectl create deployment nginx-dp --image=harbor.od.com/public/nginx:1.7.9 -n kube-public
deployment.apps/nginx-dp created
[root@hdss7-21 ~]# kubectl get all -n kube-public
NAME                            READY   STATUS    RESTARTS   AGE
pod/nginx-dp-656b87bf6d-65rzf   1/1     Running   0          14s




NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-dp   1/1     1            1           14s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-dp-656b87bf6d   1         1         1       14s




[root@hdss7-21 ~]# kubectl get pods -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE
nginx-dp-656b87bf6d-65rzf   1/1     Running   0          44s

[root@hdss7-21 ~]# kubectl get pods -n kube-public -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
nginx-dp-656b87bf6d-65rzf   1/1     Running   0          2m24s   172.7.22.3   hdss7-22.host.com   <none>           <none>
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl expose deployment nginx-dp --port=80 -n kube-public
service/nginx-dp exposed

[root@hdss7-21 ~]# kubectl get all -n kube-public
NAME                            READY   STATUS    RESTARTS   AGE
pod/nginx-dp-656b87bf6d-65rzf   1/1     Running   0          3m48s


NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/nginx-dp   ClusterIP   192.168.98.18   <none>        80/TCP    42s


NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-dp   1/1     1            1           3m48s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-dp-656b87bf6d   1         1         1       3m48s




[root@hdss7-21 ~]# kubectl get all -n kube-public -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
pod/nginx-dp-656b87bf6d-65rzf   1/1     Running   0          4m35s   172.7.22.3   hdss7-22.host.com   <none>           <none>


NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service/nginx-dp   ClusterIP   192.168.98.18   <none>        80/TCP    89s   app=nginx-dp


NAME                       READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES                             SELECTOR
deployment.apps/nginx-dp   1/1     1            1           4m35s   nginx        harbor.od.com/public/nginx:1.7.9   app=nginx-dp

NAME                                  DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES                             SELECTOR
replicaset.apps/nginx-dp-656b87bf6d   1         1         1       4m35s   nginx        harbor.od.com/public/nginx:1.7.9   app=nginx-dp,pod-template-hash=656b87bf6d




```
```buildoutcfg
[root@hdss7-22 ~]# kubectl get pods -n kube-public -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
nginx-dp-656b87bf6d-65rzf   1/1     Running   0          8m23s   172.7.22.3   hdss7-22.host.com   <none>           <none>
[root@hdss7-22 ~]# kubectl get all -n kube-public -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
pod/nginx-dp-656b87bf6d-65rzf   1/1     Running   0          8m35s   172.7.22.3   hdss7-22.host.com   <none>           <none>


NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE     SELECTOR
service/nginx-dp   ClusterIP   192.168.98.18   <none>        80/TCP    5m29s   app=nginx-dp


NAME                       READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES                             SELECTOR
deployment.apps/nginx-dp   1/1     1            1           8m35s   nginx        harbor.od.com/public/nginx:1.7.9   app=nginx-dp

NAME                                  DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES                             SELECTOR
replicaset.apps/nginx-dp-656b87bf6d   1         1         1       8m35s   nginx        harbor.od.com/public/nginx:1.7.9   app=nginx-dp,pod-template-hash=656b87bf6d




[root@hdss7-22 ~]# curl 192.168.98.18
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
```buildoutcfg
[root@hdss7-22 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.1:443 nq
  -> 10.4.7.21:6443               Masq    1      0          0
  -> 10.4.7.22:6443               Masq    1      0          0
TCP  192.168.98.18:80 nq
  -> 172.7.22.3:80                Masq    1      0          2
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get pods -n kube-public -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
nginx-dp-656b87bf6d-65rzf   1/1     Running   0          11m   172.7.22.3   hdss7-22.host.com   <none>           <none>

[root@hdss7-21 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.1:443 nq
  -> 10.4.7.21:6443               Masq    1      0          0
  -> 10.4.7.22:6443               Masq    1      0          0
TCP  192.168.98.18:80 nq
  -> 172.7.22.3:80                Masq    1      0          0

[root@hdss7-21 ~]# kubectl scale deployment nginx-dp --replicas=4 -n kube-public
deployment.extensions/nginx-dp scaled

[root@hdss7-21 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.1:443 nq
  -> 10.4.7.21:6443               Masq    1      0          0
  -> 10.4.7.22:6443               Masq    1      0          0
TCP  192.168.98.18:80 nq
  -> 172.7.21.3:80                Masq    1      0          0
  -> 172.7.21.4:80                Masq    1      0          0
  -> 172.7.22.3:80                Masq    1      0          0
  -> 172.7.22.4:80                Masq    1      0          0

[root@hdss7-21 ~]# kubectl scale deployment nginx-dp --replicas=2 -n kube-public
deployment.extensions/nginx-dp scaled

[root@hdss7-21 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.1:443 nq
  -> 10.4.7.21:6443               Masq    1      0          0
  -> 10.4.7.22:6443               Masq    1      0          0
TCP  192.168.98.18:80 nq
  -> 172.7.22.3:80                Masq    1      0          0
  -> 172.7.22.4:80                Masq    1      0          0
```
#### Check Service
```buildoutcfg
[root@hdss7-21 ~]# kubectl describe svc nginx-dp -n kube-public
Name:              nginx-dp
Namespace:         kube-public
Labels:            app=nginx-dp
Annotations:       <none>
Selector:          app=nginx-dp
Type:              ClusterIP
IP:                192.168.98.18
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         172.7.22.3:80
Session Affinity:  None
Events:            <none>
```
```buildoutcfg
[root@hdss7-21 ~]# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:5d:ef:37 brd ff:ff:ff:ff:ff:ff
    inet 10.4.7.21/24 brd 10.4.7.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:85:71:1f:22 brd ff:ff:ff:ff:ff:ff
    inet 172.7.21.1/24 brd 172.7.21.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:85ff:fe71:1f22/64 scope link
       valid_lft forever preferred_lft forever
4: dummy0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 4e:84:69:94:60:48 brd ff:ff:ff:ff:ff:ff
5: kube-ipvs0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default
    link/ether b2:09:60:f1:b4:f8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.1/32 brd 192.168.0.1 scope global kube-ipvs0
       valid_lft forever preferred_lft forever
    inet 192.168.98.18/32 brd 192.168.98.18 scope global kube-ipvs0
       valid_lft forever preferred_lft forever
7: veth1737324@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 66:48:3b:4d:bc:75 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::6448:3bff:fe4d:bc75/64 scope link
       valid_lft forever preferred_lft forever
```
```buildoutcfg
[root@hdss7-22 ~]# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:3f:a3:f9 brd ff:ff:ff:ff:ff:ff
    inet 10.4.7.22/24 brd 10.4.7.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:c0:0b:32:f5 brd ff:ff:ff:ff:ff:ff
    inet 172.7.22.1/24 brd 172.7.22.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:c0ff:fe0b:32f5/64 scope link
       valid_lft forever preferred_lft forever
4: dummy0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 3e:20:a4:74:d6:12 brd ff:ff:ff:ff:ff:ff
5: kube-ipvs0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default
    link/ether 66:02:3f:c8:e9:09 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.1/32 brd 192.168.0.1 scope global kube-ipvs0
       valid_lft forever preferred_lft forever
    inet 192.168.98.18/32 brd 192.168.98.18 scope global kube-ipvs0
       valid_lft forever preferred_lft forever
7: veth8a922e8@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether b6:2c:29:e6:15:a5 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::b42c:29ff:fee6:15a5/64 scope link
       valid_lft forever preferred_lft forever
13: veth4b6244c@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether c6:05:23:7a:8d:87 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::c405:23ff:fe7a:8d87/64 scope link
       valid_lft forever preferred_lft forever
```
```buildoutcfg
# cannot expose DaemonSet
[root@hdss7-21 ~]# kubectl get daemonset -n default
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
nginx-ds   2         2         2       2            2           <none>          25h
[root@hdss7-21 ~]# kubectl expose daemonset nginx-ds --port=443
error: cannot expose a DaemonSet.extensions
```
***
```buildoutcfg
[root@hdss7-21 ~]# kubectl get pods -o wide -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
nginx-dp-656b87bf6d-65rzf   1/1     Running   0          4h22m   172.7.22.3   hdss7-22.host.com   <none>           <none>

[root@hdss7-21 ~]# kubectl get pods nginx-dp-656b87bf6d-65rzf -o yaml -n kube-public
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2021-02-14T08:15:36Z"
  generateName: nginx-dp-656b87bf6d-
  labels:
    app: nginx-dp
    pod-template-hash: 656b87bf6d
  name: nginx-dp-656b87bf6d-65rzf
  namespace: kube-public
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: nginx-dp-656b87bf6d
    uid: 42c27651-014f-44ae-b05a-c6057fe020d1
  resourceVersion: "166073"
  selfLink: /api/v1/namespaces/kube-public/pods/nginx-dp-656b87bf6d-65rzf
  uid: 161ef1ae-fb45-4685-bf82-890e002dc071
spec:
  containers:
  - image: harbor.od.com/public/nginx:1.7.9
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-h8qnx
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: hdss7-22.host.com
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-h8qnx
    secret:
      defaultMode: 420
      secretName: default-token-h8qnx
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2021-02-14T08:15:36Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2021-02-14T08:15:37Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2021-02-14T08:15:37Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2021-02-14T08:15:36Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://e072ae42c2635fbf7de6dd2062091f1215040ba244413b48d6f6dd169b871075
    image: harbor.od.com/public/nginx:1.7.9
    imageID: docker-pullable://harbor.od.com/public/nginx@sha256:b1f5935eb2e9e2ae89c0b3e2e148c19068d91ca502e857052f14db230443e4c2
    lastState: {}
    name: nginx
    ready: true
    restartCount: 0
    state:
      running:
        startedAt: "2021-02-14T08:15:37Z"
  hostIP: 10.4.7.22
  phase: Running
  podIP: 172.7.22.3
  qosClass: BestEffort
  startTime: "2021-02-14T08:15:36Z"
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get service -n kube-public
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx-dp   ClusterIP   192.168.98.18   <none>        80/TCP    4h24m
web-dp     ClusterIP   192.168.62.62   <none>        80/TCP    3h59m
[root@hdss7-21 ~]# kubectl get svc -n kube-public
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx-dp   ClusterIP   192.168.98.18   <none>        80/TCP    4h24m
web-dp     ClusterIP   192.168.62.62   <none>        80/TCP    3h59m
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get svc nginx-dp -o yaml -n kube-public
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2021-02-14T08:18:42Z"
  labels:
    app: nginx-dp
  name: nginx-dp
  namespace: kube-public
  resourceVersion: "166341"
  selfLink: /api/v1/namespaces/kube-public/services/nginx-dp
  uid: c96223b5-928f-45f8-b0de-b8a41895ff8e
spec:
  clusterIP: 192.168.98.18
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx-dp
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}

[root@hdss7-21 ~]# kubectl get svc web-dp -o yaml -n kube-public
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2021-02-14T08:43:35Z"
  labels:
    app: web-dp
  name: web-dp
  namespace: kube-public
  resourceVersion: "168636"
  selfLink: /api/v1/namespaces/kube-public/services/web-dp
  uid: c85ddc5e-b291-410c-bfaa-007f119c8f9b
spec:
  clusterIP: 192.168.62.62
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: web-dp
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl explain service.metadata
```
```buildoutcfg
[root@hdss7-21 ~]# cat nginx-ds-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-ds
  labels:
    app: nginx-ds
  namespace: default
spec:
  type: ClusterIP
  selector:
    app: nginx-ds
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    
[root@hdss7-21 ~]# kubectl create -f nginx-ds-svc.yaml
service/nginx-ds created

[root@hdss7-21 ~]# kubectl get svc -n default
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   192.168.0.1     <none>        443/TCP   2d10h
nginx-ds     ClusterIP   192.168.38.70   <none>        80/TCP    25s

[root@hdss7-21 ~]# kubectl get svc nginx-ds -o yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2021-02-14T15:48:27Z"
  labels:
    app: nginx-ds
  name: nginx-ds
  namespace: default
  resourceVersion: "205156"
  selfLink: /api/v1/namespaces/default/services/nginx-ds
  uid: 3ff232b6-2eb0-4f35-8424-51b54a6a45f9
spec:
  clusterIP: 192.168.38.70
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx-ds
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```
```buildoutcfg
# offline edit (recommend)
[root@hdss7-21 ~]# kubectl get svc -n default
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   192.168.0.1     <none>        443/TCP   2d10h
nginx-ds     ClusterIP   192.168.38.70   <none>        80/TCP    5m19s

[root@hdss7-21 ~]# kubectl apply -f nginx-ds-svc.yaml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
service/nginx-ds configured

[root@hdss7-21 ~]# kubectl delete svc nginx-ds
service "nginx-ds" deleted

[root@hdss7-21 ~]# kubectl apply -f nginx-ds-svc.yaml
service/nginx-ds created

[root@hdss7-21 ~]# kubectl get svc -n default
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   192.168.0.1      <none>        443/TCP   2d10h
nginx-ds     ClusterIP   192.168.124.67   <none>        80/TCP    4s
```
```buildoutcfg
# online edit
[root@hdss7-21 ~]# kubectl get svc -n default
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   192.168.0.1      <none>        443/TCP   2d10h
nginx-ds     ClusterIP   192.168.124.67   <none>        80/TCP    2m32s
[root@hdss7-21 ~]# kubectl edit svc nginx-ds
service/nginx-ds edited
[root@hdss7-21 ~]# kubectl get svc -n default
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   192.168.0.1      <none>        443/TCP   2d10h
nginx-ds     ClusterIP   192.168.124.67   <none>        180/TCP   2m49s
```
#### Delete Service
```buildoutcfg
[root@hdss7-21 ~]# kubectl delete svc nginx-ds
service "nginx-ds" deleted

[root@hdss7-21 ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   192.168.0.1   <none>        443/TCP   2d11h
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get svc -n kube-public
NAME     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
web-dp   ClusterIP   192.168.62.62   <none>        80/TCP    7h28m

[root@hdss7-21 ~]# kubectl delete svc web-dp -n kube-public
service "web-dp" deleted

[root@hdss7-21 ~]# kubectl get svc -n kube-public
No resources found.
```
```buildoutcfg
# delete running pods
[root@hdss7-21 ~]# kubectl get pods -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE
nginx-dp-656b87bf6d-65rzf   1/1     Running   0          8h
[root@hdss7-21 ~]# kubectl delete pod nginx-dp-656b87bf6d-65rzf -n kube-public
pod "nginx-dp-656b87bf6d-65rzf" deleted
[root@hdss7-21 ~]# kubectl get pods -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE
nginx-dp-656b87bf6d-s84bf   1/1     Running   0          13s

[root@hdss7-21 ~]# kubectl get deployment -n kube-public
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
nginx-dp   1/1     1            1           8h
[root@hdss7-21 ~]# kubectl delete deployment nginx-dp -n kube-public
deployment.extensions "nginx-dp" deleted
[root@hdss7-21 ~]# kubectl get pods -n kube-public
No resources found.
```
***

## flannel
### CNI - the Container Network Interface
#### What is CNI?
CNI (Container Network Interface), a Cloud Native Computing Foundation project, consists of a specification and libraries for writing plugins to configure network interfaces in Linux containers, along with a number of supported plugins. CNI concerns itself only with network connectivity of containers and removing allocated resources when the container is deleted. Because of this focus, CNI has a wide range of support and the specification is simple to implement.
  * [Container Networking on Github](https://github.com/containernetworking/cni)
  * [Understanding CNI (Container Networking Interface)](https://www.dasblinkenlichten.com/understanding-cni-container-networking-interface/)
***
```buildoutcfg
[root@hdss7-21 ~]# kubectl create deployment nginx-ds --image=harbor.od.com/public/nginx:1.7.9 -n kube-public
deployment.apps/nginx-ds created
[root@hdss7-21 ~]# kubectl get pods -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE
nginx-ds-7bc4d86467-jv4pb   1/1     Running   0          10s

[root@hdss7-21 ~]# kubectl get pods -o wide -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
nginx-ds-7bc4d86467-jv4pb   1/1     Running   0          39s   172.7.22.2   hdss7-22.host.com   <none>           <none>

[root@hdss7-21 ~]# kubectl scale deployment nginx-ds --replicas=2 -n kube-public
deployment.extensions/nginx-ds scaled

[root@hdss7-21 ~]# kubectl get pods -o wide -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
nginx-ds-7bc4d86467-cd8wz   1/1     Running   0          3s    172.7.21.2   hdss7-21.host.com   <none>           <none>
nginx-ds-7bc4d86467-jv4pb   1/1     Running   0          82s   172.7.22.2   hdss7-22.host.com   <none>           <none>
```
```buildoutcfg
# hdss7-21
[root@hdss7-21 ~]# kubectl get pods -o wide -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE    IP           NODE                NOMINATED NODE   READINESS GATES
nginx-ds-7bc4d86467-cd8wz   1/1     Running   0          110s   172.7.21.2   hdss7-21.host.com   <none>           <none>
nginx-ds-7bc4d86467-jv4pb   1/1     Running   0          3m9s   172.7.22.2   hdss7-22.host.com   <none>           <none>
[root@hdss7-21 ~]# ping -c 1 172.7.21.2
PING 172.7.21.2 (172.7.21.2) 56(84) bytes of data.
64 bytes from 172.7.21.2: icmp_seq=1 ttl=64 time=0.145 ms

--- 172.7.21.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.145/0.145/0.145/0.000 ms
[root@hdss7-21 ~]# ping -c 1 172.7.22.2
PING 172.7.22.2 (172.7.22.2) 56(84) bytes of data.

--- 172.7.22.2 ping statistics ---
1 packets transmitted, 0 received, 100% packet loss, time 0ms

[root@hdss7-21 ~]# ip a s docker0
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:85:71:1f:22 brd ff:ff:ff:ff:ff:ff
    inet 172.7.21.1/24 brd 172.7.21.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:85ff:fe71:1f22/64 scope link
       valid_lft forever preferred_lft forever
```
````buildoutcfg
# hdss7-22
[root@hdss7-22 ~]# kubectl get pods -o wide -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
nginx-ds-7bc4d86467-cd8wz   1/1     Running   0          2m59s   172.7.21.2   hdss7-21.host.com   <none>           <none>
nginx-ds-7bc4d86467-jv4pb   1/1     Running   0          4m18s   172.7.22.2   hdss7-22.host.com   <none>           <none>
[root@hdss7-22 ~]# ping -c 1 172.7.21.2
PING 172.7.21.2 (172.7.21.2) 56(84) bytes of data.

--- 172.7.21.2 ping statistics ---
1 packets transmitted, 0 received, 100% packet loss, time 0ms

[root@hdss7-22 ~]# ping -c 1 172.7.22.2
PING 172.7.22.2 (172.7.22.2) 56(84) bytes of data.
64 bytes from 172.7.22.2: icmp_seq=1 ttl=64 time=0.074 ms

--- 172.7.22.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.074/0.074/0.074/0.000 ms

[root@hdss7-22 ~]# ip a s docker0
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:c0:0b:32:f5 brd ff:ff:ff:ff:ff:ff
    inet 172.7.22.1/24 brd 172.7.22.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:c0ff:fe0b:32f5/64 scope link
       valid_lft forever preferred_lft forever
````
```buildoutcfg
[root@hdss7-21 ~]# docker ps -a
CONTAINER ID   IMAGE                               COMMAND                  CREATED         STATUS         PORTS     NAMES
74353f9950d1   84581e99d807                        "nginx -g 'daemon of…"   6 minutes ago   Up 6 minutes             k8s_nginx_nginx-ds-7bc4d86467-cd8wz_kube-public_8e2c7c14-922a-4cc4-af65-f1f360d64d33_0
dcab32c048b0   harbor.od.com/public/pause:latest   "/pause"                 6 minutes ago   Up 6 minutes             k8s_POD_nginx-ds-7bc4d86467-cd8wz_kube-public_8e2c7c14-922a-4cc4-af65-f1f360d64d33_0
[root@hdss7-21 ~]# docker exec -it 74353f9950d1 bash
root@nginx-ds-7bc4d86467-cd8wz:/# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
26: eth0@if27: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:07:15:02 brd ff:ff:ff:ff:ff:ff
    inet 172.7.21.2/24 brd 172.7.21.255 scope global eth0
       valid_lft forever preferred_lft forever

root@nginx-ds-7bc4d86467-cd8wz:/# ping -c 1 172.7.22.2
PING 172.7.22.2 (172.7.22.2): 48 data bytes
--- 172.7.22.2 ping statistics ---
1 packets transmitted, 0 packets received, 100% packet loss
```
***
#### [Common CNI network plugins](https://www.cni.dev/plugins)
* [Flannel](https://github.com/coreos/flannel/releases)
* [Calico](https://github.com/projectcalico/calico)
* [Canal](https://github.com/projectcalico/canal)
...
***

Hostname | Role | IP
:-----: | :-----: | :-----:
hdss7-21.host.com  | fannel | 10.4.7.21
hdss7-22.host.com  | fannel | 10.4.7.22

#### [hdss7-21]
```buildoutcfg
# download flannel
[root@hdss7-21 src]# wget https://github.com/coreos/flannel/releases/download/v0.12.0/flannel-v0.12.0-linux-amd64.tar.gz

# decompressing files
[root@hdss7-21 src]# mkdir -p /opt/flannel-v0.12.0
[root@hdss7-21 src]# tar xvf flannel-v0.12.0-linux-amd64.tar.gz -C /opt/flannel-v0.12.0/
flanneld
mk-docker-opts.sh
README.md

# create soft link
[root@hdss7-21 ~]# ln -s /opt/flannel-v0.12.0/ /opt/flannel
```
```buildoutcfg
[root@hdss7-21 ~]# cd /opt/flannel
[root@hdss7-21 flannel]# mkdir certs
[root@hdss7-21 certs]# scp hdss7-200:/opt/certs/ca.pem .
[root@hdss7-21 certs]# scp hdss7-200:/opt/certs/client.pem .
[root@hdss7-21 certs]# scp hdss7-200:/opt/certs/client-key.pem .                                                           
```
```buildoutcfg
[root@hdss7-21 ~]# tree -L 2 /opt/
/opt/
├── containerd
│   ├── bin
│   └── lib
├── etcd -> /opt/etcd-v3.1.20/
├── etcd-v3.1.20
│   ├── certs
│   ├── Documentation
│   ├── etcd
│   ├── etcdctl
│   ├── etcd-server-startup.sh
│   ├── README-etcdctl.md
│   ├── README.md
│   └── READMEv2-etcdctl.md
├── flannel -> /opt/flannel-v0.12.0/
├── flannel-v0.12.0
│   ├── certs
│   ├── flanneld
│   ├── mk-docker-opts.sh
│   └── README.md
├── kubernetes -> /opt/kubernetes-v1.15.2/
├── kubernetes-v1.15.2
│   ├── addons
│   ├── LICENSES
│   └── server
└── src
    ├── etcd-v3.1.20-linux-amd64.tar.gz
    ├── flannel-v0.12.0-linux-amd64.tar.gz
    └── kubernetes-server-linux-amd64.tar.gz

15 directories, 13 files
```
```buildoutcfg
[root@hdss7-21 flannel]# cat subnet.env
FLANNEL_NETWORK=172.7.0.0/16
FLANNEL_SUBNET=172.7.21.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=false
```
```buildoutcfg
[root@hdss7-21 flannel]# cat flanneld.sh
#!/bin/sh
./flanneld \
  --public-ip=10.4.7.21 \
  --etcd-endpoints=https://10.4.7.12:2379,https://10.4.7.21:2379,https://10.4.7.22:2379 \
  --etcd-keyfile=./certs/client-key.pem \
  --etcd-certfile=./certs/client.pem \
  --etcd-cafile=./certs/ca.pem \
  --iface=eth0 \
  --subnet-file=./subnet.env \
  --healthz-port=2401
```
```buildoutcfg
[root@hdss7-21 flannel]# chmod +x flanneld.sh

[root@hdss7-21 flannel]# ls -l
total 34448
drwxr-xr-x 2 root root       60 Feb 15 11:17 certs
-rwxr-xr-x 1 root root 35253112 Mar 13  2020 flanneld
-rwxr-xr-x 1 root root      323 Feb 15 11:26 flanneld.sh
-rwxr-xr-x 1 root root     2139 May 29  2019 mk-docker-opts.sh
-rw-r--r-- 1 root root     4300 May 29  2019 README.md
-rw-r--r-- 1 root root       96 Feb 15 11:23 subnet.env
```
```buildoutcfg
[root@hdss7-21 flannel]# mkdir -p /data/logs/flanneld
```
#### Add host-gw
```buildoutcfg
[root@hdss7-21 ~]# cd /opt/etcd
[root@hdss7-21 etcd]# ./etcdctl member list
988139385f78284: name=etcd-server-7-22 peerURLs=https://10.4.7.22:2380 clientURLs=http://127.0.0.1:2379,https://10.4.7.22:2379 isLeader=false
5a0ef2a004fc4349: name=etcd-server-7-21 peerURLs=https://10.4.7.21:2380 clientURLs=http://127.0.0.1:2379,https://10.4.7.21:2379 isLeader=true
f4a0cb0a765574a8: name=etcd-server-7-12 peerURLs=https://10.4.7.12:2380 clientURLs=http://127.0.0.1:2379,https://10.4.7.12:2379 isLeader=false

[root@hdss7-21 etcd]# ./etcdctl set /coreos.com/network/config '{"Network": "172.7.0.0/16", "Backend": {"Type": "host-gw"}}'
{"Network": "172.7.0.0/16", "Backend": {"Type": "host-gw"}}

[root@hdss7-21 etcd]# ./etcdctl get /coreos.com/network/config
{"Network": "172.7.0.0/16", "Backend": {"Type": "host-gw"}}
```
```buildoutcfg
[root@hdss7-21 ~]# cat /etc/supervisord.d/flanneld.ini
[program:flanneld-7-21]
command=/opt/flannel/flanneld.sh                             ; the program (relative uses PATH, can take args)
numprocs=1                                                   ; number of processes copies to start (def 1)
directory=/opt/flannel                                       ; directory to cwd to before exec (def no cwd)
autostart=true                                               ; start at supervisord start (default: true)
autorestart=true                                             ; retstart at unexpected quit (default: true)
startsecs=30                           ; number of secs prog must stay running (def. 1)
startretries=3                       ; max # of serial start failures (default 3)
exitcodes=0,2                        ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                      ; signal used to kill process (default TERM)
stopwaitsecs=10                      ; max num secs to wait b4 SIGKILL (default 10)
user=root                                                    ; setuid to this UNIX account to run the program
redirect_stderr=true                                         ; redirect proc stderr to stdout (default false)
stderr_logfile=/data/logs/flanneld/flanneld.stdout.log       ; stderr log path, NONE for none; default AUTO
stderr_logfile_maxbytes=64MB                                 ; max # logfile bytes b4 rotation (default 50MB)
stderr_logfile_backups=4                                     ; # of stderr logfile backups (default 10)
stderr_capture_maxbytes=1MB                                  ; number of bytes in 'capturemode' (default 0)
stderr_events_enabled=false                                  ; emit events on stderr writes (default false)
```
```buildoutcfg
[root@hdss7-21 ~]# supervisorctl update
flanneld-7-21: added process group

[root@hdss7-21 ~]# supervisorctl status
etcd-server-7-21                 RUNNING   pid 12063, uptime 3 days, 0:31:09
flanneld-7-21                    RUNNING   pid 47167, uptime 0:00:46
kube-apiserver-7-21              RUNNING   pid 38810, uptime 2 days, 22:35:15
kube-controller-manager-7-21     RUNNING   pid 37008, uptime 1 day, 19:55:31
kube-kubelet-7-21                RUNNING   pid 73965, uptime 2 days, 10:47:25
kube-proxy-7-21                  RUNNING   pid 27587, uptime 1 day, 20:13:13
kube-scheduler-7-21              RUNNING   pid 37007, uptime 1 day, 19:55:31
```
```buildoutcfg
# error
[root@hdss7-21 flanneld]# supervisorctl start flanneld-7-21
flanneld-7-21: ERROR (spawn error)

[root@hdss7-21 flanneld]# netstat -lntup | grep flannel
tcp6       0      0 :::2401                 :::*                    LISTEN      47168/./flanneld

[root@hdss7-21 flanneld]# kill -9 47168
[root@hdss7-21 flanneld]# ps -ef | grep flannel
root      52289 116228  0 11:50 pts/0    00:00:00 grep --color=auto flannel

[root@hdss7-21 ~]# supervisorctl start flanneld-7-21
flanneld-7-21: started
[root@hdss7-21 ~]# supervisorctl status
etcd-server-7-21                 RUNNING   pid 12063, uptime 3 days, 0:38:10
flanneld-7-21                    RUNNING   pid 52394, uptime 0:00:40
kube-apiserver-7-21              RUNNING   pid 38810, uptime 2 days, 22:42:16
kube-controller-manager-7-21     RUNNING   pid 37008, uptime 1 day, 20:02:32
kube-kubelet-7-21                RUNNING   pid 73965, uptime 2 days, 10:54:26
kube-proxy-7-21                  RUNNING   pid 27587, uptime 1 day, 20:20:14
kube-scheduler-7-21              RUNNING   pid 37007, uptime 1 day, 20:02:32
```
#### [hdss7-22]










## coredns

## traefik

## dashboard

## heapster

## k8s maintenance  