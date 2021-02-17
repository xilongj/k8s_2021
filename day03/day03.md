## Kubernetes Dashboard
### [Dashboard](https://github.com/kubernetes/dashboard)
Kubernetes Dashboard is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage applications running in the cluster and troubleshoot them, as well as manage the cluster itself.

#### [hdss7-200]
```buildoutcfg
[root@hdss7-200 ~]# docker pull k8scn/kubernetes-dashboard-amd64:v1.8.3
[root@hdss7-200 ~]# docker pull k8scn/kubernetes-dashboard-amd64

[root@hdss7-200 ~]# docker images | grep dashboard
k8scn/kubernetes-dashboard-amd64   latest         1b604d831cb5   2 years ago     102MB
k8scn/kubernetes-dashboard-amd64   v1.8.3         fcac9aa03fd6   2 years ago     102MB
```
```buildoutcfg
[root@hdss7-200 ~]# docker tag fcac9aa03fd6 harbor.od.com/public/dashboard.od.com:v1.8.3
[root@hdss7-200 ~]# docker images | grep dashboard
k8scn/kubernetes-dashboard-amd64        latest         1b604d831cb5   2 years ago     102MB
k8scn/kubernetes-dashboard-amd64        v1.8.3         fcac9aa03fd6   2 years ago     102MB
harbor.od.com/public/dashboard.od.com   v1.8.3         fcac9aa03fd6   2 years ago     102MB

[root@hdss7-200 ~]# docker login harbor.od.com
Login Succeeded
[root@hdss7-200 ~]# docker push harbor.od.com/public/dashboard.od.com:v1.8.3
The push refers to repository [harbor.od.com/public/dashboard.od.com]
23ddb8cbb75a: Pushed
v1.8.3: digest: sha256:ebc993303f8a42c301592639770bd1944d80c88be8036e2d4d0aa116148264ff size: 529
```
