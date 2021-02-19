### ImagePullBackOff

```buildoutcfg
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/coredns/rbac.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/coredns/configmap.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/coredns/deployment.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/coredns/svc.yaml

[root@hdss7-21 ~]# kubectl get all -n kube-system
NAME                           READY   STATUS             RESTARTS   AGE
pod/coredns-7f8897b78c-gvfh7   0/1     ImagePullBackOff   0          59s


NAME              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
service/coredns   ClusterIP   192.168.0.2   <none>        53/UDP,53/TCP,9153/TCP   44s


NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   0/1     1            0           59s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-7f8897b78c   1         1         0       59s


```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get pods -n kube-system
NAME                       READY   STATUS             RESTARTS   AGE
coredns-7f8897b78c-gvfh7   0/1     ImagePullBackOff   0          2m28s
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl describe pod coredns-7f8897b78c-gvfh7 -n kube-system
Name:                 coredns-7f8897b78c-gvfh7
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Node:                 hdss7-21.host.com/10.4.7.21
Start Time:           Tue, 16 Feb 2021 13:49:40 +0800
Labels:               k8s-app=coredns
                      pod-template-hash=7f8897b78c
Annotations:          <none>
Status:               Pending
IP:                   172.7.21.3
Controlled By:        ReplicaSet/coredns-7f8897b78c
Containers:
  coredns:
    Container ID:
    Image:         harbor.od.com/k8s/coredns:1.6.1
    Image ID:
    Ports:         53/UDP, 53/TCP, 9153/TCP
    Host Ports:    0/UDP, 0/TCP, 0/TCP
    Args:
      -conf
      /etc/coredns/Corefile
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Liveness:       http-get http://:8080/health delay=60s timeout=5s period=10s #success=1 #failure=5
    Environment:    <none>
    Mounts:
      /etc/coredns from config-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from coredns-token-hlplx (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      coredns
    Optional:  false
  coredns-token-hlplx:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  coredns-token-hlplx
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                   From                        Message
  ----     ------     ----                  ----                        -------
  Normal   Scheduled  3m35s                 default-scheduler           Successfully assigned kube-system/coredns-7f8897b78c-gvfh7 to hdss7-21.host.com
  Normal   Pulling    114s (x4 over 3m34s)  kubelet, hdss7-21.host.com  Pulling image "harbor.od.com/k8s/coredns:1.6.1"
  Warning  Failed     104s (x4 over 3m29s)  kubelet, hdss7-21.host.com  Failed to pull image "harbor.od.com/k8s/coredns:1.6.1": rpc error: code = Unknown desc = Error response from daemon: unauthorized: project not found, name: k8s: project not found, name: k8s
  Warning  Failed     104s (x4 over 3m29s)  kubelet, hdss7-21.host.com  Error: ErrImagePull
  Warning  Failed     91s (x6 over 3m28s)   kubelet, hdss7-21.host.com  Error: ImagePullBackOff
  Normal   BackOff    77s (x7 over 3m28s)   kubelet, hdss7-21.host.com  Back-off pulling image "harbor.od.com/k8s/coredns:1.6.1"
```

```buildoutcfg
# edit deployment.yaml
image: harbor.od.com/public/coredns:1.6.1
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get deployment -n kube-system
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   0/1     1            0           72m

[root@hdss7-21 ~]# kubectl delete deployment coredns -n kube-system
deployment.extensions "coredns" deleted
```
```buildoutcfg
# apply updated deployment.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/coredns/deployment.yaml
deployment.apps/coredns created
[root@hdss7-21 ~]# kubectl get all -n kube-system
NAME                           READY   STATUS    RESTARTS   AGE
pod/coredns-6c87bf5d98-7mfvb   1/1     Running   0          5s


NAME              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
service/coredns   ClusterIP   192.168.0.2   <none>        53/UDP,53/TCP,9153/TCP   75m


NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   1/1     1            1           5s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-6c87bf5d98   1         1         1       5s
```
***

### Error response from daemon
```buildoutcfg
Warning  FailedCreatePodSandBox  3m34s  kubelet, hdss7-21.host.com  Failed create pod sandbox: rpc error: code = Unknown desc = failed to start sandbox container for pod "traefik-ingress-fwtww": Error response from daemon: driver failed programming external connectivity on endpoint k8s_POD_traefik-ingress-fwtww_kube-system_9ca6378a-f855-42da-8e12-9ac40935b01f_1 (acd6e27836ac2a7a9c1f8b60121cea61ab997ca70c4dbe6da682da9b2e98d495):  (iptables failed: iptables --wait -t filter -A DOCKER ! -i docker0 -o docker0 -p tcp -d 172.7.21.3 --dport 80 -j ACCEPT: iptables: No chain/target/match by that name.
(exit status 1))
```
```buildoutcfg
[root@hdss7-22 ~]# kubectl get pods -o wide -n kube-system
NAME                       READY   STATUS              RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
coredns-6c87bf5d98-7mfvb   1/1     Running             0          5h19m   172.7.22.3   hdss7-22.host.com   <none>           <none>
traefik-ingress-fwtww      0/1     ContainerCreating   0          5s      <none>       hdss7-21.host.com   <none>           <none>
traefik-ingress-k9fkz      1/1     Running             0          80m     172.7.22.2   hdss7-22.host.com   <none>           <none>
```
```buildoutcfg
# solutions
[root@hdss7-21 ~]# systemctl restart docker

#Output
[root@hdss7-22 ~]# kubectl get pods -o wide -n kube-system
NAME                       READY   STATUS    RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
coredns-6c87bf5d98-7mfvb   1/1     Running   0          5h31m   172.7.22.3   hdss7-22.host.com   <none>           <none>
traefik-ingress-fwtww      1/1     Running   0          12m     172.7.21.3   hdss7-21.host.com   <none>           <none>
traefik-ingress-k9fkz      1/1     Running   0          92m     172.7.22.2   hdss7-22.host.com   <none>           <none>
```
***

### wrong fs type, bad option, bad superblock
```buildoutcfg
Events:
  Type     Reason       Age   From                        Message
  ----     ------       ----  ----                        -------
  Normal   Scheduled    118s  default-scheduler           Successfully assigned infra/jenkins-576b469db4-cfbb7 to hdss7-22.host.com
  Warning  FailedMount  114s  kubelet, hdss7-22.host.com  MountVolume.SetUp failed for volume "data" : mount failed: exit status 32
Mounting command: systemd-run
Mounting arguments: --description=Kubernetes transient mount for /data/kubelet/pods/5114ccb4-5c3b-47eb-8382-cd38a1615232/volumes/kubernetes.io~nfs/data --scope -- mount -t nfs hdss7-200:/data/xsj-storage/jenkins_home /data/kubelet/pods/5114ccb4-5c3b-47eb-8382-cd38a1615232/volumes/kubernetes.io~nfs/data
Output: Running scope as unit run-128825.scope.
mount: wrong fs type, bad option, bad superblock on hdss7-200:/data/xsj-storage/jenkins_home,
       missing codepage or helper program, or other error
       (for several filesystems (e.g. nfs, cifs) you might
       need a /sbin/mount.<type> helper program)

       In some cases useful info is found in syslog - try
       dmesg | tail or so.
```
```buildoutcfg
[root@hdss7-21 ~]# yum -y install nfs-utils
[root@hdss7-21 ~]# rpm -qa nfs-utils
nfs-utils-1.3.0-0.68.el7.x86_64
[root@hdss7-21 ~]# systemctl start nfs
[root@hdss7-21 ~]# systemctl enable nfs
```
***