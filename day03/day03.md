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
```buildoutcfg
# delete incorrect tagged image on harbor.od.com
[root@hdss7-200 ~]# docker images | grep dashboard
k8scn/kubernetes-dashboard-amd64        latest         1b604d831cb5   2 years ago     102MB
k8scn/kubernetes-dashboard-amd64        v1.8.3         fcac9aa03fd6   2 years ago     102MB
harbor.od.com/public/dashboard.od.com   v1.8.3         fcac9aa03fd6   2 years ago     102MB
[root@hdss7-200 ~]# docker rmi harbor.od.com/public/dashboard.od.com:v1.8.3
Untagged: harbor.od.com/public/dashboard.od.com:v1.8.3
Untagged: harbor.od.com/public/dashboard.od.com@sha256:ebc993303f8a42c301592639770bd1944d80c88be8036e2d4d0aa116148264ff
[root@hdss7-200 ~]# docker images | grep dashboard
k8scn/kubernetes-dashboard-amd64   latest         1b604d831cb5   2 years ago     102MB
k8scn/kubernetes-dashboard-amd64   v1.8.3         fcac9aa03fd6   2 years ago     102MB
[root@hdss7-200 ~]# docker tag fcac9aa03fd6 harbor.od.com/public/dashboard:v1.8.3
[root@hdss7-200 ~]# docker images | grep dashboard
k8scn/kubernetes-dashboard-amd64   latest         1b604d831cb5   2 years ago     102MB
k8scn/kubernetes-dashboard-amd64   v1.8.3         fcac9aa03fd6   2 years ago     102MB
harbor.od.com/public/dashboard     v1.8.3         fcac9aa03fd6   2 years ago     102MB
[root@hdss7-200 ~]# docker push harbor.od.com/public/dashboard:v1.8.3
The push refers to repository [harbor.od.com/public/dashboard]
23ddb8cbb75a: Mounted from public/dashboard.od.com
v1.8.3: digest: sha256:ebc993303f8a42c301592639770bd1944d80c88be8036e2d4d0aa116148264ff size: 529
```
### [Dashboard Resource](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dashboard)
#### [hdss7-200]
```buildoutcfg
[root@hdss7-200 ~]# mkdir -p /data/k8s-yaml/dashboard && cd /data/k8s-yaml/dashboard
```
```buildoutcfg
[root@hdss7-200 ~]# ls -l /data/k8s-yaml/dashboard/
total 16
-rw-r--r-- 1 root root 1381 Feb 17 10:20 deployment.yaml
-rw-r--r-- 1 root root  318 Feb 17 10:22 ingress.yaml
-rw-r--r-- 1 root root  610 Feb 17 10:15 rbac.yaml
-rw-r--r-- 1 root root  322 Feb 17 10:21 svc.yaml
```
```buildoutcfg
# rbac.yaml
[root@hdss7-200 dashboard]# cat rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
  name: kubernetes-dashboard-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard-admin
  namespace: kube-system
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard-admin
  namespace: kube-system
```
```buildoutcfg
# deployment.yaml
[root@hdss7-200 dashboard]# cat deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
  labels:
    k8s-app: kubernetes-dashboard
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ''
    spec:
      priorityClassName: system-cluster-critical
      containers:
      - name: kubernetes-dashboard
        image: harbor.od.com/public/dashboard:v1.8.3
        resources:
          limits:
            cpu: 100m
            memory: 300Mi
          requests:
            cpu: 50m
            memory: 100Mi
        ports:
        - containerPort: 8443
          protocol: TCP
        args:
          # PLATFORM-SPECIFIC ARGS HERE
          - --auto-generate-certificates
        volumeMounts:
        - name: tmp-volume
          mountPath: /tmp
        livenessProbe:
          httpGet:
            scheme: HTTPS
            path: /
            port: 8443
          initialDelaySeconds: 30
          timeoutSeconds: 30
      volumes:
      - name: tmp-volume
        emptyDir: {}
      serviceAccountName: kubernetes-dashboard-admin
      tolerations:
      - key: "CriticalAddonsOnly"
        operator: "Exists"
```
```buildoutcfg
# svc.yaml
[root@hdss7-200 dashboard]# cat svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
  labels:
    k8s-app: kubernetes-dashboard
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  selector:
    k8s-app: kubernetes-dashboard
  ports:
  - port: 443
    targetPort: 8443
```
```buildoutcfg
# ingress.yaml
[root@hdss7-200 dashboard]# cat ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: dashboard.od.com
    http:
      paths:
      - backend:
          serviceName: kubernetes-dashboard
          servicePort: 443
```
#### apply *.yaml
```buildoutcfg
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/dashboard/rbac.yaml
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/dashboard/deployment.yaml
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/dashboard/svc.yaml
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/dashboard/ingress.yaml
```
```buildoutcfg
[root@hdss7-22 ~]# kubectl get pods -n kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-6c87bf5d98-7mfvb                1/1     Running   0          19h
kubernetes-dashboard-76dcdb4677-hrpt2   1/1     Running   0          56s
traefik-ingress-2rss8                   1/1     Running   0          12h
traefik-ingress-pc8fw                   1/1     Running   0          12h

[root@hdss7-22 ~]# kubectl get pods -n kube-system | grep dashboard
kubernetes-dashboard-76dcdb4677-hrpt2   1/1     Running   0          67s
```
```buildoutcfg
[root@hdss7-22 ~]# kubectl get pods -n kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-6c87bf5d98-7mfvb                1/1     Running   0          19h
kubernetes-dashboard-76dcdb4677-hrpt2   1/1     Running   0          4m7s
traefik-ingress-2rss8                   1/1     Running   0          12h
traefik-ingress-pc8fw                   1/1     Running   0          12h

[root@hdss7-22 ~]# kubectl get svc -n kube-system
NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
coredns                   ClusterIP   192.168.0.2      <none>        53/UDP,53/TCP,9153/TCP   20h
kubernetes-dashboard      ClusterIP   192.168.123.30   <none>        443/TCP                  4m12s
traefik-ingress-service   ClusterIP   192.168.93.197   <none>        80/TCP,8080/TCP          15h

[root@hdss7-22 ~]# kubectl get ingress -n kube-system
NAME                   HOSTS              ADDRESS   PORTS   AGE
kubernetes-dashboard   dashboard.od.com             80      4m28s
traefik-web-ui         traefik.od.com               80      15h
```
#### [hdss7-11]
```buildoutcfg
# update DNS
[root@hdss7-11 ~]# cat /var/named/od.com.zone
$ORIGIN od.com.
$TTL 600        ; 10 minutes
@       IN SOA dns.od.com. dnsadmin.od.com. (
                        2021021005      ; serial
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
dashboard       A       10.4.7.10
```
```buildoutcfg
# [hdss7-10]
[root@hdss7-200 dashboard]# dig -t A dashboard.od.com @10.4.7.11 +short
10.4.7.10

# [hdss7-21]
[root@hdss7-21 ~]# kubectl get svc -n kube-system
NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
coredns                   ClusterIP   192.168.0.2      <none>        53/UDP,53/TCP,9153/TCP   20h
kubernetes-dashboard      ClusterIP   192.168.123.30   <none>        443/TCP                  11m
traefik-ingress-service   ClusterIP   192.168.93.197   <none>        80/TCP,8080/TCP          15h

[root@hdss7-21 ~]# dig -t A dashboard.od.com @192.168.0.2 +short
10.4.7.10
```
```buildoutcfg
# disable Windows network
Chrome: dashboard.od.com 
```