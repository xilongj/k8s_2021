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
    1  kubectl get ns (namespace)
    2  kubectl get all -n default
    3  kubectl create ns app01
    4  kubectl delete namespace app01
    5  kubectl create deployment web-dp --image=nginx -n kube-public
    6  kubectl get deploy -n kube-public
    7  kubectl get deploy -o wide -n kube-public
    8  kubectl describe deployment web-dp -n kube-public
    9  kubectl get pods -n kube-public
   10  kubectl exec -it web-dp-7955d7c58b-p65fc bash -n kube-public
   11  kubectl delete pod web-dp-7955d7c58b-p65fc -n kube-public
   12  kubectl delete deploy(ment) web-np -n kube-public
   13  kubectl expose deployment web-dp --port=80 -n kube-public
   14  kubectl scale deployment web-dp --replicas=2 -n kube-public
   15  ipvsadm -Ln
   16  kubectl describe svc web-dp -n kube-public
```
***
```buildoutcfg
[root@hdss7-21 ~]# kubectl get ns (namespace)
NAME              STATUS   AGE
default           Active   2d
kube-node-lease   Active   2d
kube-public       Active   2d
kube-system       Active   2d
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get all [-n default]
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

[root@hdss7-21 ~]# kubectl get deployment -n kube-public
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
[root@hdss7-21 ~]# kubectl get pods -o wide -n kube-public
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
[root@hdss7-22 ~]# kubectl get pods -n -o wide kube-public
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
nginx-dp-656b87bf6d-znzjw   1/1     Running   0          3m36s   172.7.22.3   hdss7-22.host.com   <none>           <none>
```
```buildoutcfg
[root@hdss7-22 ~]# kubectl delete pod nginx-dp-656b87bf6d-znzjw -n kube-public
pod "nginx-dp-656b87bf6d-znzjw" deleted
[root@hdss7-22 ~]# kubectl get pods -o wide -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
nginx-dp-656b87bf6d-g9krn   1/1     Running   0          15s   172.7.21.3   hdss7-21.host.com   <none>           <none>

[root@hdss7-22 ~]# kubectl delete pod nginx-dp-656b87bf6d-g9krn -n kube-public --force --grace-period=0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "nginx-dp-656b87bf6d-g9krn" force deleted
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


[root@hdss7-21 ~]# kubectl get pods -o wide -n kube-public
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


[root@hdss7-21 ~]# kubectl get all -o wide -n kube-public
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
[root@hdss7-22 ~]# kubectl get pods -o wide -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
nginx-dp-656b87bf6d-65rzf   1/1     Running   0          8m23s   172.7.22.3   hdss7-22.host.com   <none>           <none>
[root@hdss7-22 ~]# kubectl get all -o wide -n kube-public
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
[root@hdss7-21 ~]# kubectl get pods -o wide -n kube-public
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
#### [hdss7-21]
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
# 100% packet loss
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
#### [hdss7-22]
````buildoutcfg
[root@hdss7-22 ~]# kubectl get pods -o wide -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
nginx-ds-7bc4d86467-cd8wz   1/1     Running   0          2m59s   172.7.21.2   hdss7-21.host.com   <none>           <none>
nginx-ds-7bc4d86467-jv4pb   1/1     Running   0          4m18s   172.7.22.2   hdss7-22.host.com   <none>           <none>
# 100% packet loss
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
***

#### [Common CNI network plugins](https://www.cni.dev/plugins)
* [Flannel](https://github.com/coreos/flannel/releases)
* [Calico](https://github.com/projectcalico/calico)
* [Canal](https://github.com/projectcalico/canal) <br>
......
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
#### ADD host-gw 
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
startsecs=30                                                 ; number of secs prog must stay running (def. 1)
startretries=3                                               ; max # of serial start failures (default 3)
exitcodes=0,2                                                ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                              ; signal used to kill process (default TERM)
stopwaitsecs=10                                              ; max num secs to wait b4 SIGKILL (default 10)
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
```buildoutcfg
# download flannel
[root@hdss7-22 ~]# cd /opt/src/
[root@hdss7-22 src]# wget wget https://github.com/coreos/flannel/releases/download/v0.12.0/flannel-v0.12.0-linux-amd64.tar.gz

# decompressing files
[root@hdss7-22 src]# mkdir -p /opt/flannel-v0.12.0
[root@hdss7-22 src]# tar xvf flannel-v0.12.0-linux-amd64.tar.gz -C /opt/flannel-v0.12.0/
flanneld
mk-docker-opts.sh
README.md

# create soft link
[root@hdss7-22 src]# ln -s /opt/flannel-v0.12.0/ /opt/flannel
```
```buildoutcfg
[root@hdss7-22 ~]# cd /opt/flannel
[root@hdss7-22 flannel]# mkdir certs
[root@hdss7-22 flannel]# cd certs/
[root@hdss7-22 certs]# scp hdss7-200:/opt/certs/ca.pem .
[root@hdss7-22 certs]# scp hdss7-200:/opt/certs/client.pem .
[root@hdss7-22 certs]# scp hdss7-200:/opt/certs/client-key.pem .
```
```buildoutcfg
[root@hdss7-22 ~]# tree -L 2 /opt/
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
[root@hdss7-22 ~]# cd /opt/flannel
[root@hdss7-22 flannel]# vim subnet.env

[root@hdss7-22 flannel]# cat subnet.env
FLANNEL_NETWORK=172.7.0.0/16
FLANNEL_SUBNET=172.7.22.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=false
```
```buildoutcfg
[root@hdss7-22 flannel]# cat flanneld.sh
#!/bin/sh
./flanneld \
  --public-ip=10.4.7.22 \
  --etcd-endpoints=https://10.4.7.12:2379,https://10.4.7.21:2379,https://10.4.7.22:2379 \
  --etcd-keyfile=./certs/client-key.pem \
  --etcd-certfile=./certs/client.pem \
  --etcd-cafile=./certs/ca.pem \
  --iface=eth0 \
  --subnet-file=./subnet.env \
  --healthz-port=2401
```
```buildoutcfg
[root@hdss7-22 flannel]# chmod +x flanneld.sh
[root@hdss7-22 flannel]# ls -l
total 34448
drwxr-xr-x 2 root root       60 Feb 15 11:17 certs
-rwxr-xr-x 1 root root 35253112 Mar 13  2020 flanneld
-rwxr-xr-x 1 root root      323 Feb 15 11:26 flanneld.sh
-rwxr-xr-x 1 root root     2139 May 29  2019 mk-docker-opts.sh
-rw-r--r-- 1 root root     4300 May 29  2019 README.md
-rw-r--r-- 1 root root       96 Feb 15 11:23 subnet.env
```
```buildoutcfg
[root@hdss7-22 flannel]# mkdir -p /data/logs/flanneld
```
```buildoutcfg
[root@hdss7-22 ~]# cat /etc/supervisord.d/flanneld.ini
[program:flanneld-7-22]
command=/opt/flannel/flanneld.sh                             ; the program (relative uses PATH, can take args)
numprocs=1                                                   ; number of processes copies to start (def 1)
directory=/opt/flannel                                       ; directory to cwd to before exec (def no cwd)
autostart=true                                               ; start at supervisord start (default: true)
autorestart=true                                             ; retstart at unexpected quit (default: true)
startsecs=30                                                 ; number of secs prog must stay running (def. 1)
startretries=3                                               ; max # of serial start failures (default 3)
exitcodes=0,2                                                ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                              ; signal used to kill process (default TERM)
stopwaitsecs=10                                              ; max num secs to wait b4 SIGKILL (default 10)
user=root                                                    ; setuid to this UNIX account to run the program
redirect_stderr=true                                         ; redirect proc stderr to stdout (default false)
stderr_logfile=/data/logs/flanneld/flanneld.stdout.log       ; stderr log path, NONE for none; default AUTO
stderr_logfile_maxbytes=64MB                                 ; max # logfile bytes b4 rotation (default 50MB)
stderr_logfile_backups=4                                     ; # of stderr logfile backups (default 10)
stderr_capture_maxbytes=1MB                                  ; number of bytes in 'capturemode' (default 0)
stderr_events_enabled=false                                  ; emit events on stderr writes (default false)
```
```buildoutcfg
[root@hdss7-22 ~]# supervisorctl update
flanneld-7-22: added process group

[root@hdss7-22 ~]# supervisorctl status
etcd-server-7-22                 RUNNING   pid 8461, uptime 3 days, 5:32:52
flanneld-7-22                    STARTING
kube-apiserver-7-22              RUNNING   pid 14815, uptime 3 days, 3:31:43
kube-controller-manager-7-22     RUNNING   pid 35002, uptime 2 days, 0:59:34
kube-kubelet-7-22                RUNNING   pid 50018, uptime 2 days, 15:57:35
kube-proxy-7-22                  RUNNING   pid 23280, uptime 2 days, 1:21:26
kube-scheduler-7-22              RUNNING   pid 34991, uptime 2 days, 0:59:34
```
#### Verify Flannel Network
```buildoutcfg
[root@hdss7-22 ~]# kubectl get pods -o wide -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE    IP           NODE                NOMINATED NODE   READINESS GATES
nginx-ds-7bc4d86467-cd8wz   1/1     Running   0          3h3m   172.7.21.2   hdss7-21.host.com   <none>           <none>
nginx-ds-7bc4d86467-jv4pb   1/1     Running   0          3h4m   172.7.22.2   hdss7-22.host.com   <none>           <none>

[root@hdss7-22 ~]# ping -c 1 172.7.22.2
PING 172.7.22.2 (172.7.22.2) 56(84) bytes of data.
64 bytes from 172.7.22.2: icmp_seq=1 ttl=64 time=0.039 ms

--- 172.7.22.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.039/0.039/0.039/0.000 ms

[root@hdss7-22 ~]# ping -c 1 172.7.21.2
PING 172.7.21.2 (172.7.21.2) 56(84) bytes of data.
64 bytes from 172.7.21.2: icmp_seq=1 ttl=63 time=0.253 ms

--- 172.7.21.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.253/0.253/0.253/0.000 ms
```
```buildoutcfg
# hdss7-21.host.com
[root@hdss7-21 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.4.7.254      0.0.0.0         UG    100    0        0 eth0
10.4.7.0        0.0.0.0         255.255.255.0   U     100    0        0 eth0
172.7.21.0      0.0.0.0         255.255.255.0   U     0      0        0 docker0
172.7.22.0      10.4.7.22       255.255.255.0   UG    0      0        0 eth0

# hdss7-22.host.com
[root@hdss7-22 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.4.7.254      0.0.0.0         UG    100    0        0 eth0
10.4.7.0        0.0.0.0         255.255.255.0   U     100    0        0 eth0
172.7.21.0      10.4.7.21       255.255.255.0   UG    0      0        0 eth0
172.7.22.0      0.0.0.0         255.255.255.0   U     0      0        0 docker0
```
#### Flannel Networking models
```buildoutcfg
# flannel Host GW
[root@hdss7-21 etcd]# ./etcdctl set /coreos.com/network/config '{"Network": "172.7.0.0/16", "Backend": {"Type": "host-gw"}}'
# add route
[root@hdss7-21 ~]# route add -net 172.7.22.0/24 gw 10.4.7.22 dev eth0
[root@hdss7-22 ~]# route add -net 172.7.21.0/24 gw 10.4.7.21 dev eth0
# del route
[root@hdss7-22 ~]# route del -net 172.7.21.0/24 gw 10.4.7.21
[root@hdss7-21 ~]# route del -net 172.7.22.0/24 gw 10.4.7.22

# flannel VxLAN
[root@hdss7-21 etcd]# ./etcdctl set /coreos.com/network/config '{"Network": "172.7.0.0/16", "Backend": {"Type": "VxLAN"}}'

# flannel Directrouting
[root@hdss7-21 etcd]# ./etcdctl set /coreos.com/network/config '{"Network": "172.7.0.0/16", "Backend": {"Type": "VxLAN", "Directrouting": "true"}}'
```
```buildoutcfg
 https://github.com/coreos/flannel/blob/master/Documentation/backends.md
 https://mvallim.github.io/kubernetes-under-the-hood/documentation/kube-flannel.html
 https://github.com/mvallim/kubernetes-under-the-hood/blob/master/documentation/networking.md
```
***
```buildoutcfg
[root@hdss7-21 ~]# kubectl get deployment -o wide -n kube-public
NAME       READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES                             SELECTOR
nginx-ds   2/2     2            2           6h57m   nginx        harbor.od.com/public/nginx:1.7.9   app=nginx-ds
[root@hdss7-21 ~]# kubectl get pods -o wide -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
nginx-ds-7bc4d86467-cb8c9   1/1     Running   0          23m   172.7.22.2   hdss7-22.host.com   <none>           <none>
nginx-ds-7bc4d86467-h6rfk   1/1     Running   0          23m   172.7.21.2   hdss7-21.host.com   <none>           <none>

# Run CentOS (curl)
[root@hdss7-21 ~]# docker run -it --rm --name lnx01 harbor.od.com/public/centos:8.3.2011 bash
[root@02f08bcfdbda /]# ip a s
40: eth0@if41: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:07:15:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.7.21.3/24 brd 172.7.21.255 scope global eth0
       valid_lft forever preferred_lft forever

[root@02f08bcfdbda /]# curl -sIL -w "%{http_code}\n" -o /dev/null 172.7.21.2
200
[root@02f08bcfdbda /]# curl -sIL -w "%{http_code}\n" -o /dev/null 172.7.22.2
200

# Output (physical machine IP)
[root@hdss7-22 ~]# kubectl logs -f nginx-ds-7bc4d86467-cb8c9 -n kube-public
10.4.7.21 - - [15/Feb/2021:09:17:13 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.61.1" "-"
10.4.7.21 - - [15/Feb/2021:09:18:52 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.61.1" "-"

# masquerade
-A POSTROUTING -s 172.7.22.0/24 ! -o docker0 -j MASQUERADE
```
```buildoutcfg
# enable iptables [hdss7-21]
[root@hdss7-21 ~]# yum -y install iptables-services
[root@hdss7-21 ~]# rpm -qa iptables
iptables-1.4.21-35.el7.x86_64
[root@hdss7-21 ~]# systemctl start iptables
[root@hdss7-21 ~]# systemctl enable iptables

# enable iptables [hdss7-22]
[root@hdss7-22 ~]# yum -y install iptables-services
[root@hdss7-22 ~]# rpm -qa iptables
iptables-1.4.21-35.el7.x86_64
[root@hdss7-22 ~]# systemctl start iptables
[root@hdss7-22 ~]# systemctl enable iptables
```
#### [hdss7-21]
```buildoutcfg
[root@hdss7-21 ~]# iptables-save | grep -i postrouting
:POSTROUTING ACCEPT [72:4338]
:KUBE-POSTROUTING - [0:0]
-A POSTROUTING -m comment --comment "kubernetes postrouting rules" -j KUBE-POSTROUTING
-A POSTROUTING -s 172.7.21.0/24 ! -o docker0 -j MASQUERADE
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -m mark --mark 0x4000/0x4000 -j MASQUERADE

# delete rule (-A POSTROUTING -s 172.7.21.0/24 ! -o docker0 -j MASQUERADE)
[root@hdss7-21 ~]# iptables -t nat -D POSTROUTING -s 172.7.21.0/24 ! -o docker0 -j MASQUERADE
[root@hdss7-21 ~]# iptables-save | grep -i postrouting
:POSTROUTING ACCEPT [2:126]
:KUBE-POSTROUTING - [0:0]
-A POSTROUTING -m comment --comment "kubernetes postrouting rules" -j KUBE-POSTROUTING
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -m mark --mark 0x4000/0x4000 -j MASQUERADE

# add new rule (-A POSTROUTING -s 172.7.21.0/24 ! -d 172.7.0.0/16 ! -o docker0 -j MASQUERADE)
[root@hdss7-21 ~]# iptables -t nat -I POSTROUTING -s 172.7.21.0/24 ! -d 172.7.0.0/16 ! -o docker0 -j MASQUERADE
[root@hdss7-21 ~]# iptables-save | grep -i postrouting
:POSTROUTING ACCEPT [61:3666]
:KUBE-POSTROUTING - [0:0]
-A POSTROUTING -s 172.7.21.0/24 ! -d 172.7.0.0/16 ! -o docker0 -j MASQUERADE
-A POSTROUTING -m comment --comment "kubernetes postrouting rules" -j KUBE-POSTROUTING
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -m mark --mark 0x4000/0x4000 -j MASQUERADE
# save rules
[root@hdss7-21 ~]# iptables-save > /etc/sysconfig/iptables
```
```buildoutcfg
[root@hdss7-21 ~]# iptables-save | grep -i reject
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
[root@hdss7-21 ~]# iptables -t filter -D INPUT -j REJECT --reject-with icmp-host-prohibited
[root@hdss7-21 ~]# iptables -t filter -D FORWARD -j REJECT --reject-with icmp-host-prohibited
[root@hdss7-21 ~]# iptables-save > /etc/sysconfig/iptables
```
#### [hdss7-22]
```buildoutcfg
[root@hdss7-22 ~]# iptables-save | grep -i postrouting
:POSTROUTING ACCEPT [14:852]
:KUBE-POSTROUTING - [0:0]
-A POSTROUTING -m comment --comment "kubernetes postrouting rules" -j KUBE-POSTROUTING
-A POSTROUTING -s 172.7.22.0/24 ! -o docker0 -j MASQUERADE
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -m mark --mark 0x4000/0x4000 -j MASQUERADE
```
```buildoutcfg
[root@hdss7-22 ~]# iptables -t nat -D POSTROUTING -s 172.7.22.0/24 ! -o docker0 -j MASQUERADE

[root@hdss7-22 ~]# iptables -t nat -I POSTROUTING -s 172.7.22.0/24 ! -d 172.7.0.0/16 ! -o docker0 -j MASQUERADE

[root@hdss7-22 ~]# iptables-save > /etc/sysconfig/iptables
[root@hdss7-22 ~]# iptables-save | grep -i reject
```
#### Note
```buildoutcfg
1: iptables规则各主机略有不同, 其它运算节点上执行时注意修改;
2: 优化SNAT规则, 各运算节点间各Pod间的网络通信不再出网;
3: 10.4.7.21主机上, 来源是172.7.21.0/24段的docker的IP, 目标IP不是172.7.0.0/16段,网络发包不从docker0桥设备出站,才进行SNAT转换;
```
```buildoutcfg
[root@hdss7-21 ~]# docker run -it --rm --name lnx01 harbor.od.com/public/centos:8.3.2011 bash
[root@6504cdb4a9c3 /]# ip a s
42: eth0@if43: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:07:15:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.7.21.3/24 brd 172.7.21.255 scope global eth0
       valid_lft forever preferred_lft forever
[root@6504cdb4a9c3 /]# curl -sIL -w "%{http_code}\n" -o /dev/null 172.7.22.2
200

[root@hdss7-22 ~]# docker run -it --rm --name lnx01 harbor.od.com/public/centos:8.3.2011 bash
[root@70c07e187cbd /]# ip a s
28: eth0@if29: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:07:16:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.7.22.3/24 brd 172.7.22.255 scope global eth0
       valid_lft forever preferred_lft forever
[root@70c07e187cbd /]# curl -sIL -w "%{http_code}\n" -o /dev/null 172.7.21.2
200
```
#### Verify IP Change
```buildoutcfg
[root@hdss7-22 ~]# kubectl logs -f nginx-ds-7bc4d86467-cb8c9 -n kube-public
10.4.7.21 - - [15/Feb/2021:09:18:52 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.61.1" "-"
172.7.21.3 - - [15/Feb/2021:16:19:29 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.61.1" "-"

[root@hdss7-21 ~]# kubectl logs -f nginx-ds-7bc4d86467-h6rfk -n kube-public
10.4.7.22 - - [15/Feb/2021:16:22:53 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.61.1" "-"
172.7.22.3 - - [15/Feb/2021:16:29:46 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.61.1" "-"
```
***

## [CoreDNS](https://github.com/coredns/coredns)
### CoreDNS: DNS and Service Discovery
#### [What is CoreDNS?](https://coredns.io/)
CoreDNS is a DNS server. It is written in Go. It can be used in a multitude of environments because of its flexibility. CoreDNS is licensed under the Apache License Version 2, and completely open source.
#### CoreDNS Plugins
CoreDNS chains plugins. Each plugin performs a DNS function, such as Kubernetes service discovery, prometheus metrics, rewriting queries, or just serving from zone files. And many more.
#### Service Discovery
CoreDNS integrates with Kubernetes via the Kubernetes plugin, or with etcd with the etcd plugin. All major cloud providers have plugins too: Microsoft Azure DNS, CGP Cloud DNS and AWS Route53.
***

####[hdss7-200]
```buildoutcfg
[root@hdss7-200 ~]# vim /etc/nginx/conf.d/k8s-yaml.od.com.conf
[root@hdss7-200 ~]# cat /etc/nginx/conf.d/k8s-yaml.od.com.conf
server {
    listen       80;
    server_name  k8s-yaml.od.com;

    location / {
        autoindex on;
        default_type text/plain;
        root /data/k8s-yaml;
    }
}
```
```buildoutcfg
[root@hdss7-200 ~]# mkdir -p /data/k8s-yaml

[root@hdss7-200 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

[root@hdss7-200 ~]# nginx -s reload

[root@hdss7-200 ~]# ls -l /data/k8s-yaml/
total 0
[root@hdss7-200 ~]# mkdir /data/k8s-yaml/coredns
```
#### [hdss7-11]
```buildoutcfg
# add k8s-yaml A record
[root@hdss7-11 ~]# cat /var/named/od.com.zone
$ORIGIN od.com.
$TTL 600        ; 10 minutes
@       IN SOA dns.od.com. dnsadmin.od.com. (
                        2021021003      ; serial
                        10800           ; refresh (3 hours)
                        900             ; retry (15 minutes)
                        604800          ; expire (1 week)
                        86400           ; minimum (1 day)
                        )
                        NS      dns.od.com.
$TTL 60 ; 1 minute
dns             A       10.4.7.11
harbor          A       10.4.7.200
k8s-yaml        A       10.4.7.200
```
```buildoutcfg
[root@hdss7-11 ~]# systemctl restart named
[root@hdss7-11 ~]#
[root@hdss7-11 ~]# dig -t A k8s-yaml.od.com @10.4.7.11 +short
10.4.7.200
```
```buildoutcfg
# disable network card
chrome: http://k8s-yaml.od.com/

[root@hdss7-200 ~]# curl -sIL -w "%{http_code}\n" -o /dev/null http://k8s-yaml.od.com/
200
```
#### [hdss7-200]
```buildoutcfg
[root@hdss7-200 ~]# cd /data/k8s-yaml/coredns/
[root@hdss7-200 coredns]# docker pull coredns/coredns:1.6.1
[root@hdss7-200 coredns]# docker images | grep coredns
coredns/coredns                 1.6.1     c0f6e815079e   18 months ago   42.2MB
```
```buildoutcfg
[root@hdss7-200 coredns]# docker tag c0f6e815079e harbor.od.com/public/coredns:1.6.1

[root@hdss7-200 coredns]# docker images | grep coredns
coredns/coredns                 1.6.1     c0f6e815079e   18 months ago   42.2MB
harbor.od.com/public/coredns    1.6.1     c0f6e815079e   18 months ago   42.2MB
```
```buildoutcfg
[root@hdss7-200 coredns]# docker login harbor.od.com
Login Succeeded
[root@hdss7-200 coredns]# docker push harbor.od.com/public/coredns:1.6.1
The push refers to repository [harbor.od.com/public/coredns]
da1ec456edc8: Pushed
225df95e717c: Pushed

1.6.1: digest: sha256:c7bf0ce4123212c87db74050d4cbab77d8f7e0b49c041e894a35ef15827cf938 size: 739
```
### [CoreDNS Resource](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns/coredns)
#### [hdss7-200]
```buildoutcfg
[root@hdss7-200 ~]# ls -l /data/k8s-yaml/coredns/
total 16
-rw-r--r-- 1 root root  319 Feb 16 13:07 configmap.yaml
-rw-r--r-- 1 root root 1290 Feb 16 13:33 deployment.yaml
-rw-r--r-- 1 root root  954 Feb 16 13:07 rbac.yaml
-rw-r--r-- 1 root root  387 Feb 16 13:11 svc.yaml
```
```buildoutcfg
# rbac.yaml
[root@hdss7-200 coredns]# cat rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: coredns
  namespace: kube-system
  labels:
      kubernetes.io/cluster-service: "true"
      addonmanager.kubernetes.io/mode: Reconcile
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
    addonmanager.kubernetes.io/mode: Reconcile
  name: system:coredns
rules:
- apiGroups:
  - ""
  resources:
  - endpoints
  - services
  - pods
  - namespaces
  verbs:
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
    addonmanager.kubernetes.io/mode: EnsureExists
  name: system:coredns
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:coredns
subjects:
- kind: ServiceAccount
  name: coredns
  namespace: kube-system
```
```buildoutcfg
# configmap.yaml
[root@hdss7-200 coredns]# cat configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        log
        health
        ready
        kubernetes cluster.local 192.168.0.0/16
        forward . 10.4.7.11
        cache 30
        loop
        reload
        loadbalance
       }
```
```buildoutcfg
# deployment.yaml
[root@hdss7-200 coredns]# cat deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coredns
  namespace: kube-system
  labels:
    k8s-app: coredns
    kubernetes.io/name: "CoreDNS"
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: coredns
  template:
    metadata:
      labels:
        k8s-app: coredns
    spec:
      priorityClassName: system-cluster-critical
      serviceAccountName: coredns
      containers:
      - name: coredns
        image: harbor.od.com/public/coredns:1.6.1
        args:
        - -conf
        - /etc/coredns/Corefile
        volumeMounts:
        - name: config-volume
          mountPath: /etc/coredns
        ports:
        - containerPort: 53
          name: dns
          protocol: UDP
        - containerPort: 53
          name: dns-tcp
          protocol: TCP
        - containerPort: 9153
          name: metrics
          protocol: TCP
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 5
      dnsPolicy: Default
      volumes:
        - name: config-volume
          configMap:
            name: coredns
            items:
            - key: Corefile
              path: Corefile
```
```buildoutcfg
# svc.yaml
[root@hdss7-200 coredns]# cat svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: coredns
  namespace: kube-system
  labels:
    k8s-app: coredns
    kubernetes.io/cluster-service: "true"
    kubernetes.io/name: "CoreDNS"
spec:
  selector:
    k8s-app: coredns
  clusterIP: 192.168.0.2
  ports:
  - name: dns
    port: 53
    protocol: UDP
  - name: dns-tcp
    port: 53
  - name: metrics
    port: 9153
    protocol: TCP
```
#### [hdss7-21]
```buildoutcfg
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/coredns/rbac.yaml
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/coredns/configmap.yaml
configmap/coredns created
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/coredns/deployment.yaml
deployment.apps/coredns created
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/coredns/svc.yaml
service/coredns created
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get all -n kube-system
NAME                           READY   STATUS    RESTARTS   AGE
pod/coredns-6c87bf5d98-7mfvb   1/1     Running   0          4m42s


NAME              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
service/coredns   ClusterIP   192.168.0.2   <none>        53/UDP,53/TCP,9153/TCP   80m


NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   1/1     1            1           4m42s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-6c87bf5d98   1         1         1       4m42s



[root@hdss7-21 ~]# kubectl get pods -n kube-system
NAME                       READY   STATUS    RESTARTS   AGE
coredns-6c87bf5d98-7mfvb   1/1     Running   0          4m52s
```
#### Cluster DNS
```buildoutcfg
[root@hdss7-21 ~]# grep 'cluster-dns' /opt/kubernetes/server/bin/kubelet.sh
  --cluster-dns 192.168.0.2 \
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get all -o wide -n kube-system
NAME                           READY   STATUS    RESTARTS   AGE    IP           NODE                NOMINATED NODE   READINESS GATES
pod/coredns-6c87bf5d98-7mfvb   1/1     Running   0          9m9s   172.7.22.3   hdss7-22.host.com   <none>           <none>


NAME              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE   SELECTOR
service/coredns   ClusterIP   192.168.0.2   <none>        53/UDP,53/TCP,9153/TCP   84m   k8s-app=coredns


NAME                      READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS   IMAGES                               SELECTOR
deployment.apps/coredns   1/1     1            1           9m9s   coredns      harbor.od.com/public/coredns:1.6.1   k8s-app=coredns

NAME                                 DESIRED   CURRENT   READY   AGE    CONTAINERS   IMAGES                               SELECTOR
replicaset.apps/coredns-6c87bf5d98   1         1         1       9m9s   coredns      harbor.od.com/public/coredns:1.6.1   k8s-app=coredns,pod-template-hash=6c87bf5d98



[root@hdss7-21 ~]# cat /opt/kubernetes/server/bin/kubelet.sh
#!/bin/sh
./kubelet \
  --anonymous-auth=false \
  --cgroup-driver systemd \
  --cluster-dns 192.168.0.2 \
  --cluster-domain cluster.local \
  --runtime-cgroups=/systemd/system.slice \
  --kubelet-cgroups=/systemd/system.slice \
  --fail-swap-on="false" \
  --client-ca-file ./certs/ca.pem \
  --tls-cert-file ./certs/kubelet.pem \
  --tls-private-key-file ./certs/kubelet-key.pem \
  --hostname-override hdss7-21.host.com \
  --image-gc-high-threshold 20 \
  --image-gc-low-threshold 10 \
  --kubeconfig ./conf/kubelet.kubeconfig \
  --log-dir /data/logs/kubernetes/kube-kubelet \
  --pod-infra-container-image harbor.od.com/public/pause:latest \
  --root-dir /data/kubelet
```
```buildoutcfg
[root@hdss7-21 ~]# dig -t A www.baidu.com @192.168.0.2 +short
www.a.shifen.com.
110.242.68.4
110.242.68.3
[root@hdss7-21 ~]# dig -t A hdss7-21.host.com @192.168.0.2 +short
10.4.7.21
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get svc -o wide
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE    SELECTOR
kubernetes   ClusterIP   192.168.0.1   <none>        443/TCP   4d2h   <none>
[root@hdss7-21 ~]# kubectl get svc -o wide -n kube-system
NAME      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE   SELECTOR
coredns   ClusterIP   192.168.0.2   <none>        53/UDP,53/TCP,9153/TCP   92m   k8s-app=coredns
[root@hdss7-21 ~]# kubectl get svc -o wide -n kube-public
No resources found.

[root@hdss7-21 ~]# kubectl get pods -o wide -n kube-public
No resources found.
[root@hdss7-21 ~]# kubectl get pods -o wide -n kube-system
NAME                       READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
coredns-6c87bf5d98-7mfvb   1/1     Running   0          24m   172.7.22.3   hdss7-22.host.com   <none>           <none>
```
#### Verify CoreDNS
```buildoutcfg
[root@hdss7-21 ~]# kubectl get pods -o wide -n kube-public
No resources found.
[root@hdss7-21 ~]# kubectl create deployment nginx-dp --image=harbor.od.com/public/nginx:1.7.9 -n kube-public
deployment.apps/nginx-dp created
[root@hdss7-21 ~]# kubectl get pods -o wide -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
nginx-dp-656b87bf6d-gbvpm   1/1     Running   0          8s    172.7.21.2   hdss7-21.host.com   <none>           <none>
[root@hdss7-21 ~]# kubectl get svc -o wide -n kube-public
No resources found.
[root@hdss7-21 ~]# kubectl expose deployment nginx-dp --port=80 -n kube-public
service/nginx-dp exposed
[root@hdss7-21 ~]# kubectl get svc -o wide -n kube-public
NAME       TYPE        CLUSTER-IP        EXTERNAL-IP   PORT(S)   AGE   SELECTOR
nginx-dp   ClusterIP   192.168.150.147   <none>        80/TCP    2s    app=nginx-dp
```
#### FQDN
```buildoutcfg
[root@hdss7-21 ~]# dig -t A nginx-dp.kube-public.svc.cluster.local. @192.168.0.2 +short
192.168.150.147
```
```buildoutcfg
[root@hdss7-21 ~]# dig -t A nginx-dp.kube-public.svc.cluster.local. @192.168.0.2

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.3 <<>> -t A nginx-dp.kube-public.svc.cluster.local. @192.168.0.2
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43952
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;nginx-dp.kube-public.svc.cluster.local.        IN A

;; ANSWER SECTION:
nginx-dp.kube-public.svc.cluster.local. 5 IN A  192.168.150.147

;; Query time: 0 msec
;; SERVER: 192.168.0.2#53(192.168.0.2)
;; WHEN: Tue Feb 16 15:35:07 CST 2021
;; MSG SIZE  rcvd: 121

```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get pods -o wide -n kube-public
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
nginx-dp-656b87bf6d-gbvpm   1/1     Running   0          12m   172.7.21.2   hdss7-21.host.com   <none>           <none>
[root@hdss7-21 ~]# kubectl exec -it nginx-dp-656b87bf6d-gbvpm bash -n kube-public
root@nginx-dp-656b87bf6d-gbvpm:/# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
50: eth0@if51: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:07:15:02 brd ff:ff:ff:ff:ff:ff
    inet 172.7.21.2/24 brd 172.7.21.255 scope global eth0
       valid_lft forever preferred_lft forever
root@nginx-dp-656b87bf6d-gbvpm:/# ping -c 2 nginx-dp.kube-public
PING nginx-dp.kube-public.svc.cluster.local (192.168.150.147): 48 data bytes
56 bytes from 192.168.150.147: icmp_seq=0 ttl=64 time=0.032 ms
56 bytes from 192.168.150.147: icmp_seq=1 ttl=64 time=0.065 ms
--- nginx-dp.kube-public.svc.cluster.local ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.032/0.049/0.065/0.000 ms
root@nginx-dp-656b87bf6d-gbvpm:/# exit
exit
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get svc -o wide -n kube-public
NAME       TYPE        CLUSTER-IP        EXTERNAL-IP   PORT(S)   AGE   SELECTOR
nginx-dp   ClusterIP   192.168.150.147   <none>        80/TCP    28m   app=nginx-dp
[root@hdss7-21 ~]# curl -sIL -w "%{http_code}\n" -o /dev/null 192.168.150.147
200
[root@hdss7-21 ~]# curl nginx-dp.kube-public.svc.cluster.local.
curl: (6) Could not resolve host: nginx-dp.kube-public.svc.cluster.local.; Unknown error
```
***
## [traefik](https://github.com/traefik/traefik)
### What is traefik?
Traefik (pronounced traffic) is a modern HTTP reverse proxy and load balancer that makes deploying microservices easy. Traefik integrates with your existing infrastructure components (Docker, Swarm mode, Kubernetes, Marathon, Consul, Etcd, Rancher, Amazon ECS, ...) and configures itself automatically and dynamically. Pointing Traefik at your orchestrator should be the only configuration step you need.
### What is Ingress?
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

#### [hdss7-200]
```buildoutcfg
[root@hdss7-200 ~]# docker pull traefik:1.7.2-alpine

[root@hdss7-200 ~]# docker images | grep traefik
traefik                         1.7.2-alpine   add5fac61ae5   2 years ago     72.4MB

[root@hdss7-200 ~]# docker tag add5fac61ae5 harbor.od.com/public/traefik:1.7.2

[root@hdss7-200 ~]# docker images | grep traefik
traefik                         1.7.2-alpine   add5fac61ae5   2 years ago     72.4MB
harbor.od.com/public/traefik    1.7.2          add5fac61ae5   2 years ago     72.4MB
[root@hdss7-200 ~]# docker push harbor.od.com/public/traefik:1.7.2
The push refers to repository [harbor.od.com/public/traefik]
a02beb48577f: Pushed
ca22117205f4: Pushed
3563c211d861: Pushed
df64d3292fd6: Pushed
1.7.2: digest: sha256:6115155b261707b642341b065cd3fac2b546559ba035d0262650b3b3bbdd10ea size: 1157
```
### [Traefik Resource](https://github.com/traefik/traefik/tree/v1.7/examples/k8s)
#### [hdss7-200]
```buildoutcfg
[root@hdss7-200 ~]# mkdir -p /data/k8s-yaml/traefik && cd /data/k8s-yaml/traefik
[root@hdss7-200 traefik]# vim rbac.yaml
```
```buildoutcfg
[root@hdss7-200 ~]# ls -l /data/k8s-yaml/traefik/
total 16
-rw-r--r-- 1 root root 1099 Feb 16 18:38 ds.yaml
-rw-r--r-- 1 root root  330 Feb 16 18:41 ingress.yaml
-rw-r--r-- 1 root root  800 Feb 16 18:29 rbac.yaml
-rw-r--r-- 1 root root  269 Feb 16 18:40 svc.yaml
```
```buildoutcfg
# create rbac.yaml
[root@hdss7-200 traefik]# cat rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: kube-system
```
```buildoutcfg
# create ds.yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: traefik-ingress
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress
spec:
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress
        name: traefik-ingress
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      containers:
      - image: harbor.od.com/public/traefik:1.7.2
        name: traefik-ingress
        ports:
        - name: controller
          containerPort: 80
          hostPort: 81
        - name: admin-web
          containerPort: 8080
        securityContext:
          capabilities:
            drop:
            - ALL
            add:
            - NET_BIND_SERVICE
        args:
        - --api
        - --kubernetes
        - --logLevel=INFO
        - --insecureskipverify=true
        - --kubernetes.endpoint=https://10.4.7.10:7443
        - --accesslog
        - --accesslog.filepath=/var/log/traefik_access.log
        - --traefiklog
        - --traefiklog.filepath=/var/log/traefik.log
        - --metrics.prometheus
```
```buildoutcfg
# create svc.yaml
[root@hdss7-200 traefik]# cat svc.yaml
kind: Service
apiVersion: v1
metadata:
  name: traefik-ingress-service
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress
  ports:
    - protocol: TCP
      port: 80
      name: controller
    - protocol: TCP
      port: 8080
      name: admin-web
```
```buildoutcfg
# create ingress.yaml
[root@hdss7-200 traefik]# cat ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: traefik.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: traefik-ingress-service
          servicePort: 8080
```
#### [hdss7-22]
```buildoutcfg
# apply rbac.yaml
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/traefik/rbac.yaml
serviceaccount/traefik-ingress-controller created
clusterrole.rbac.authorization.k8s.io/traefik-ingress-controller created
clusterrolebinding.rbac.authorization.k8s.io/traefik-ingress-controller created
```
```buildoutcfg
# apply ds.yaml
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/traefik/ds.yaml
daemonset.extensions/traefik-ingress created
# apply svc.yaml
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/traefik/svc.yaml
service/traefik-ingress-service created
# apply ingress.yaml
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/traefik/ingress.yaml
ingress.extensions/traefik-web-ui created
```
```buildoutcfg
[root@hdss7-22 ~]# kubectl get pods -o wide -n kube-system
NAME                       READY   STATUS    RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
coredns-6c87bf5d98-7mfvb   1/1     Running   0          5h31m   172.7.22.3   hdss7-22.host.com   <none>           <none>
traefik-ingress-fwtww      1/1     Running   0          12m     172.7.21.3   hdss7-21.host.com   <none>           <none>
traefik-ingress-k9fkz      1/1     Running   0          92m     172.7.22.2   hdss7-22.host.com   <none>           <none>
```
```buildoutcfg
[root@hdss7-21 ~]# netstat -lntup | grep 81
tcp        0      0 0.0.0.0:81              0.0.0.0:*               LISTEN      80711/docker-proxy
```
#### [hdss7-11]
```buildoutcfg
[root@hdss7-11 ~]# cat /etc/nginx/conf.d/od.com.conf
upstream default_backend_traefik {
    server 10.4.7.21:81    max_fails=3 fail_timeout=10s;
    server 10.4.7.22:81    max_fails=3 fail_timeout=10s;
}
server {
    server_name *.od.com;

    location / {
        proxy_pass http://default_backend_traefik;
        proxy_set_header Host       $http_host;
        proxy_set_header x-forwarded-for $proxy_add_x_forwarded_for;
    }
}
[root@hdss7-11 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

[root@hdss7-11 ~]# nginx -s reload
```
#### [hdss7-12]
```buildoutcfg
[root@hdss7-12 ~]# cat /etc/nginx/conf.d/od.com.conf
upstream default_backend_traefik {
    server 10.4.7.21:81    max_fails=3 fail_timeout=10s;
    server 10.4.7.22:81    max_fails=3 fail_timeout=10s;
}
server {
    server_name *.od.com;

    location / {
        proxy_pass http://default_backend_traefik;
        proxy_set_header Host       $http_host;
        proxy_set_header x-forwarded-for $proxy_add_x_forwarded_for;
    }
}
[root@hdss7-12 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

[root@hdss7-12 ~]# nginx -s reload
```
#### Update A record
```buildoutcfg
[root@hdss7-11 ~]# cat /var/named/od.com.zone
$ORIGIN od.com.
$TTL 600        ; 10 minutes
@       IN SOA dns.od.com. dnsadmin.od.com. (
                        2021021004      ; serial
                        10800           ; refresh (3 hours)
                        900             ; retry (15 minutes)
                        604800          ; expire (1 week)
                        86400           ; minimum (1 day)
                        )
                        NS      dns.od.com.
$TTL 60 ; 1 minute
dns             A       10.4.7.11
harbor          A       10.4.7.200
k8s-yaml        A       10.4.7.200
traefik         A       10.4.7.10

[root@hdss7-11 ~]# systemctl restart named
```
```buildoutcfg
Chrome: http://traefik.od.com
[root@hdss7-200 ~]# curl http://traefik.od.com
<a href="/dashboard/">Found</a>.
```
***