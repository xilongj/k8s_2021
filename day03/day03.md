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
### [RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
Role-based access control (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users within your organization.
RBAC authorization uses the rbac.authorization.k8s.io API group to drive authorization decisions, allowing you to dynamically configure policies through the Kubernetes API.

#### Check cluster-admin
```buildoutcfg
[root@hdss7-21 ~]# kubectl get clusterrole cluster-admin -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: "2021-02-12T05:09:31Z"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: cluster-admin
  resourceVersion: "40"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/cluster-admin
  uid: c7939c05-cfd7-4fe8-a6d2-b4d75421cfb3
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
  - '*'
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get clusterrole system:node -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: "2021-02-12T05:09:31Z"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:node
  resourceVersion: "51"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/system%3Anode
  uid: 9b20e7ea-fc2f-4590-ad1e-7a50b90a0095
rules:
- apiGroups:
  - authentication.k8s.io
  resources:
  - tokenreviews
  verbs:
  - create
- apiGroups:
  - authorization.k8s.io
  resources:
  - localsubjectaccessreviews
  - subjectaccessreviews
  verbs:
  - create
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - create
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
  - update
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - patch
  - update
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - create
  - patch
  - update
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - create
  - delete
- apiGroups:
  - ""
  resources:
  - pods/status
  verbs:
  - patch
  - update
- apiGroups:
  - ""
  resources:
  - pods/eviction
  verbs:
  - create
- apiGroups:
  - ""
  resources:
  - configmaps
  - secrets
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - persistentvolumeclaims
  - persistentvolumes
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - endpoints
  verbs:
  - get
- apiGroups:
  - certificates.k8s.io
  resources:
  - certificatesigningrequests
  verbs:
  - create
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - persistentvolumeclaims/status
  verbs:
  - get
  - patch
  - update
- apiGroups:
  - ""
  resources:
  - serviceaccounts/token
  verbs:
  - create
- apiGroups:
  - storage.k8s.io
  resources:
  - volumeattachments
  verbs:
  - get
- apiGroups:
  - storage.k8s.io
  resources:
  - csidrivers
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - storage.k8s.io
  resources:
  - csinodes
  verbs:
  - create
  - delete
  - get
  - patch
  - update
- apiGroups:
  - coordination.k8s.io
  resources:
  - leases
  verbs:
  - create
  - delete
  - get
  - patch
  - update
- apiGroups:
  - node.k8s.io
  resources:
  - runtimeclasses
  verbs:
  - get
  - list
  - watch
```
### Login Dashboard with Token
```buildoutcfg
# Authentication failed. Please try again.
# Dashboard Version: v1.8.3
```
#### [hdss7-200]
```buildoutcfg
[root@hdss7-200 certs]# (umask 077; openssl genrsa -out dashboard.od.com.key 2048)
Generating RSA private key, 2048 bit long modulus
......................................+++
.+++
e is 65537 (0x10001)
```
```buildoutcfg
[root@hdss7-200 certs]# openssl req -new -key dashboard.od.com.key -out dashboard.od.com.csr -subj "/CN=dashboard.od.com/C=CN/ST=Beijing/L=Beijing/O=XLNX/OU=XEUS"
```
```buildoutcfg
[root@hdss7-200 certs]# ls -l /opt/certs/dashboard*
-rw-r--r-- 1 root root 1009 Feb 17 18:34 /opt/certs/dashboard.od.com.csr
-rw------- 1 root root 1675 Feb 17 18:30 /opt/certs/dashboard.od.com.key
```
```buildoutcfg
[root@hdss7-200 certs]# openssl x509 -req -in dashboard.od.com.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out dashboard.od.com.crt -days 3650
Signature ok
subject=/CN=dashboard.od.com/C=CN/ST=Beijing/L=Beijing/O=XLNX/OU=XEUS
Getting CA Private Key
[root@hdss7-200 certs]# ls -l dashboard*
-rw-r--r-- 1 root root 1196 Feb 17 18:39 dashboard.od.com.crt
-rw-r--r-- 1 root root 1009 Feb 17 18:34 dashboard.od.com.csr
-rw------- 1 root root 1675 Feb 17 18:30 dashboard.od.com.key
```
```buildoutcfg
[root@hdss7-200 certs]# cfssl-certinfo -cert dashboard.od.com.crt
{
  "subject": {
    "common_name": "dashboard.od.com",
    "country": "CN",
    "organization": "XLNX",
    "organizational_unit": "XEUS",
    "locality": "Beijing",
    "province": "Beijing",
    "names": [
      "dashboard.od.com",
      "CN",
      "Beijing",
      "Beijing",
      "XLNX",
      "XEUS"
    ]
  },
  "issuer": {
    "common_name": "HomeEdu",
    "country": "CN",
    "organization": "XLNX",
    "organizational_unit": "XEUS",
    "locality": "Beijing",
    "province": "Beijing",
    "names": [
      "CN",
      "Beijing",
      "Beijing",
      "XLNX",
      "XEUS",
      "HomeEdu"
    ]
  },
  "serial_number": "10788697195439947958",
  "not_before": "2021-02-17T10:39:59Z",
  "not_after": "2031-02-15T10:39:59Z",
  "sigalg": "SHA256WithRSA",
  "authority_key_id": "",
  "subject_key_id": "",
  "pem": "-----BEGIN CERTIFICATE-----\nMIIDRzCCAi8CCQCVuSbwM3OAtjANBgkqhkiG9w0BAQsFADBhMQswCQYDVQQGEwJD\nTjEQMA4GA1UECBMHQmVpamluZzEQMA4GA1UEBxMHQmVpamluZzENMAsGA1UEChME\nWExOWDENMAsGA1UECxMEWEVVUzEQMA4GA1UEAxMHSG9tZUVkdTAeFw0yMTAyMTcx\nMDM5NTlaFw0zMTAyMTUxMDM5NTlaMGoxGTAXBgNVBAMMEGRhc2hib2FyZC5vZC5j\nb20xCzAJBgNVBAYTAkNOMRAwDgYDVQQIDAdCZWlqaW5nMRAwDgYDVQQHDAdCZWlq\naW5nMQ0wCwYDVQQKDARYTE5YMQ0wCwYDVQQLDARYRVVTMIIBIjANBgkqhkiG9w0B\nAQEFAAOCAQ8AMIIBCgKCAQEAwVOox2lhQjrCOdRfLEnSkM0ioj1PMFywk+N2Vwjl\n8ajQMQw/q3Lr3uqliSIqP5OJrSLbdqj2x8Pqhpgk03Zt+z4USrPQlvxuN49//1bk\n99Lsu1Vb01r5bAfpi24pYsF8YSP0mbp7sh9aAzooZAdPPQNArxw+ZQ9sdgy5OQWX\nAFnjovG0R9/ewWXjpROqM15R61zIdpDyZknBZLL1vyQqIr2IkMSnvWI3KVKxkUCq\nc7B5G74i8oLJHpcXyhbB9I6gicVPvaX6U1DToa6IYRTeQDqaQVpYxo45tvuX3QF8\nu4LzpI7ez22Ol9/F0GDOx/V9XapBJ3Pw8VW5o6PDmVjsXwIDAQABMA0GCSqGSIb3\nDQEBCwUAA4IBAQCja3gtHUQVSi5ubLd9BnTjVts3Fj+kOOOgo4a9S5uv64baDisB\n/6QXbgxtwBz7idSySFB5uDjmp9lrJyB7U4o0+YCBkdKA2yN8FHctpFBDtAMLyet9\nFZVUaaWUofvUEvYLD5XE9arxblYveW9ZN30+B+z97MoAA/wKXjj/SpdEUt0vaIA3\ngElUhR3nZBgMDeMlpPmpTqJFzdpmldANohzdc3lFyZ+SzgBUMdoU3Oph1HQt+uF5\n/xvCiFasZkY2sd8pAPz7monuBcrh12vqVpNRNVW0l6ygwyKBsSEome29GMcZI0n/\njliZyC3AGG8nDvJYxupTdY4p1V2OXFSpNJHO\n-----END CERTIFICATE-----\n"
}
```
#### [hdss7-11]
```buildoutcfg
[root@hdss7-11 ~]# mkdir -p /etc/nginx/certs
[root@hdss7-11 ~]# cd /etc/nginx/certs/ && scp hdss7-200:/opt/certs/dashboard.od.com.crt .
                                                                         
[root@hdss7-11 certs]# scp hdss7-200:/opt/certs/dashboard.od.com.key .                                                                
[root@hdss7-11 certs]# ls -l
total 8
-rw-r--r-- 1 root root 1196 Feb 17 18:46 dashboard.od.com.crt
-rw------- 1 root root 1675 Feb 17 18:47 dashboard.od.com.key
```
```buildoutcfg
[root@hdss7-11 ~]# cat /etc/nginx/conf.d/dashboard.od.com.conf
server {
    listen       80;
    server_name  dashboard.od.com;

    rewrite ^(.*)$ https://${server_name}$1 permanent;
}
server {
    listen       443 ssl;
    server_name  dashboard.od.com;

    ssl_certificate "certs/dashboard.od.com.crt";
    ssl_certificate_key "certs/dashboard.od.com.key";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://default_backend_traefik;
        proxy_set_header Host       $http_host;
        proxy_set_header x-forwarded-for $proxy_add_x_forwarded_for;
    }
}
```
```buildoutcfg
[root@hdss7-11 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@hdss7-11 ~]# nginx -s reload
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get secret -n kube-system
NAME                                     TYPE                                  DATA   AGE
coredns-token-hlplx                      kubernetes.io/service-account-token   3      29h
default-token-m8jr5                      kubernetes.io/service-account-token   3      4d22h
kubernetes-dashboard-admin-token-qj2wb   kubernetes.io/service-account-token   3      8h
kubernetes-dashboard-key-holder          Opaque                                2      8h
traefik-ingress-controller-token-mwzh5   kubernetes.io/service-account-token   3      23h

[root@hdss7-21 ~]# kubectl describe secret kubernetes-dashboard-admin-token-qj2wb -n kube-system
Name:         kubernetes-dashboard-admin-token-qj2wb
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: kubernetes-dashboard-admin
              kubernetes.io/service-account.uid: d7ed0cfd-6afd-4c7e-b805-813f1bcfb781

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1346 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbi10b2tlbi1xajJ3YiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImQ3ZWQwY2ZkLTZhZmQtNGM3ZS1iODA1LTgxM2YxYmNmYjc4MSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiJ9.dwWbSd6OPm3BTiUU8kX3qQntSNnj0or4nL5uSAPmb2XQhNsbVDfynd3iEmX-ivDo6linAkmqNM_LVhZyYBRFT-dKx41V4zqSxeldb3163icBUGbaFRm1F8W82-_Mr4GOKBEQxttfz6kwITG_u5G6K8fghMvKGDMeHeCogapvDobzTCmO_2hN5zk5G7puPL8TNyUdOyfuLTD-JlAh9VJ3NNI4_O-UKIUHwRtyHfLkP-Wrwkd7xSstwXja_CGREr-4CKJ4VPJfNuOR5fAFH1iCBcJfOSVbrH4VMvfcyTwJNoeAvQHmR3PmT3RUU0-QIJRY1EdhINJplA1brtjaE6lA_Q
[root@hdss7-21 ~]#
```
```buildoutcfg
# login with Token: eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbi10b2tlbi1xajJ3YiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImQ3ZWQwY2ZkLTZhZmQtNGM3ZS1iODA1LTgxM2YxYmNmYjc4MSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiJ9.dwWbSd6OPm3BTiUU8kX3qQntSNnj0or4nL5uSAPmb2XQhNsbVDfynd3iEmX-ivDo6linAkmqNM_LVhZyYBRFT-dKx41V4zqSxeldb3163icBUGbaFRm1F8W82-_Mr4GOKBEQxttfz6kwITG_u5G6K8fghMvKGDMeHeCogapvDobzTCmO_2hN5zk5G7puPL8TNyUdOyfuLTD-JlAh9VJ3NNI4_O-UKIUHwRtyHfLkP-Wrwkd7xSstwXja_CGREr-4CKJ4VPJfNuOR5fAFH1iCBcJfOSVbrH4VMvfcyTwJNoeAvQHmR3PmT3RUU0-QIJRY1EdhINJplA1brtjaE6lA_Q
```
### Dashboard Upgrade [1.10.1]
```buildoutcfg
[root@hdss7-200 ~]# docker pull hexun/kubernetes-dashboard-amd64:v1.10.1

[root@hdss7-200 ~]# docker tag f9aed6605b81 harbor.od.com/public/dashboard:v1.10.1

[root@hdss7-200 ~]# docker images | grep dashboard
harbor.od.com/public/dashboard     v1.10.1        f9aed6605b81   2 years ago     122MB
hexun/kubernetes-dashboard-amd64   v1.10.1        f9aed6605b81   2 years ago     122MB
k8scn/kubernetes-dashboard-amd64   v1.8.3         fcac9aa03fd6   2 years ago     102MB
harbor.od.com/public/dashboard     v1.8.3         fcac9aa03fd6   2 years ago     102MB

[root@hdss7-200 ~]# docker push harbor.od.com/public/dashboard:v1.10.1
```
```buildoutcfg
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/dashboard/upgrade.yaml
[root@hdss7-200 dashboard]# diff upgrade.yaml deployment.yaml
24c24
<         image: harbor.od.com/public/dashboard:v1.10.1
---
>         image: harbor.od.com/public/dashboard:v1.8.3
```
#### Homework - create devops account
```buildoutcfg
We are building an Adaptable Intelligent World!
```
### [heapster](https://github.com/kubernetes-retired/heapster)
#### [hdss7-200]
```buildoutcfg
[root@hdss7-200 ~]# mkdir -p /data/k8s-yaml/dashboard/heapster/
[root@hdss7-200 ~]# cd /data/k8s-yaml/dashboard/heapster/
```
```buildoutcfg
[root@hdss7-200 heapster]# docker pull bitnami/heapster:1.5.4
# or
[root@hdss7-200 heapster]# docker pull quay.io/bitnami/heapster:1.5.4

[root@hdss7-200 heapster]# docker images | grep heapster
quay.io/bitnami/heapster           1.5.4          c359b95ad38b   2 years ago     136MB
bitnami/heapster                   1.5.4          c359b95ad38b   2 years ago     136MB
```
```buildoutcfg
[root@hdss7-200 heapster]# docker images | grep heapster
quay.io/bitnami/heapster           1.5.4          c359b95ad38b   2 years ago     136MB
[root@hdss7-200 heapster]# docker tag c359b95ad38b harbor.od.com/public/heapster:v1.5.4
[root@hdss7-200 heapster]# docker images | grep heapster
harbor.od.com/public/heapster      v1.5.4         c359b95ad38b   2 years ago     136MB
quay.io/bitnami/heapster           1.5.4          c359b95ad38b   2 years ago     136MB

[root@hdss7-200 heapster]# docker push harbor.od.com/public/heapster:v1.5.4
The push refers to repository [harbor.od.com/public/heapster]
20d37d828804: Pushed
b9b192015e25: Pushed
b76dba5a0109: Pushed
v1.5.4: digest: sha256:bfb71b113c26faeeea27799b7575f19253ba49ccf064bac7b6137ae8a36f48a5 size: 952
```
```buildoutcfg
# rbac.yaml
[root@hdss7-200 heapster]# cat rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: heapster
  namespace: kube-system
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: heapster
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:heapster
subjects:
- kind: ServiceAccount
  name: heapster
  namespace: kube-system
```
```buildoutcfg
# deployment.yaml
[root@hdss7-200 heapster]# cat deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: heapster
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: heapster
    spec:
      serviceAccountName: heapster
      containers:
      - name: heapster
        image: harbor.od.com/public/heapster:v1.5.4
        imagePullPolicy: IfNotPresent
        command:
        - /opt/bitnami/heapster/bin/heapster
        - --source=kubernetes:https://kubernetes.default
```
```buildoutcfg
# svc.yaml
[root@hdss7-200 heapster]# cat svc.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    task: monitoring
    # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
    # If you are NOT using this as an addon, you should comment out this line.
    kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: Heapster
  name: heapster
  namespace: kube-system
spec:
  ports:
  - port: 80
    targetPort: 8082
  selector:
    k8s-app: heapster
```
#### [hdss7-22]
```buildoutcfg
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/dashboard/heapster/rbac.yaml
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/dashboard/heapster/deployment.yaml
[root@hdss7-22 ~]# kubectl apply -f http://k8s-yaml.od.com/dashboard/heapster/svc.yaml

[root@hdss7-22 ~]# kubectl get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-6c87bf5d98-7mfvb               1/1     Running   0          32h
heapster-b5b9f794-qr5hl                1/1     Running   0          20s
kubernetes-dashboard-67989c548-zxrkg   1/1     Running   0          101m
traefik-ingress-2rss8                  1/1     Running   0          24h
traefik-ingress-pc8fw                  1/1     Running   0          24h
```
```buildoutcfg
[root@hdss7-22 ~]# kubectl describe secret kubernetes-dashboard-admin-token-qj2wb -n kube-system
Name:         kubernetes-dashboard-admin-token-qj2wb
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: kubernetes-dashboard-admin
              kubernetes.io/service-account.uid: d7ed0cfd-6afd-4c7e-b805-813f1bcfb781

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1346 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbi10b2tlbi1xajJ3YiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImQ3ZWQwY2ZkLTZhZmQtNGM3ZS1iODA1LTgxM2YxYmNmYjc4MSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiJ9.dwWbSd6OPm3BTiUU8kX3qQntSNnj0or4nL5uSAPmb2XQhNsbVDfynd3iEmX-ivDo6linAkmqNM_LVhZyYBRFT-dKx41V4zqSxeldb3163icBUGbaFRm1F8W82-_Mr4GOKBEQxttfz6kwITG_u5G6K8fghMvKGDMeHeCogapvDobzTCmO_2hN5zk5G7puPL8TNyUdOyfuLTD-JlAh9VJ3NNI4_O-UKIUHwRtyHfLkP-Wrwkd7xSstwXja_CGREr-4CKJ4VPJfNuOR5fAFH1iCBcJfOSVbrH4VMvfcyTwJNoeAvQHmR3PmT3RUU0-QIJRY1EdhINJplA1brtjaE6lA_Q
```
```buildoutcfg
Chrome: http://dashboard.od.com
```
***
### Kubernetes Upgrade
#### [hdss7-21], [hdss7-22]
```buildoutcfg
[root@hdss7-21 ~]# kubectl get node -o wide
NAME                STATUS   ROLES         AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME
hdss7-21.host.com   Ready    master,node   5d3h    v1.15.2   10.4.7.21     <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   docker://20.10.3
hdss7-22.host.com   Ready    master,node   4d23h   v1.15.2   10.4.7.22     <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   docker://20.10.3
```

***
## [Dubbo Deployment](https://github.com/apache/dubbo)
Apache Dubbo is a high-performance, java based open source RPC framework.
Dubbo |ˈdʌbəʊ| offers six key functionalities, which include transparent interface based RPC, intelligent load balancing, automatic service registration and discovery, high extensibility, runtime traffic routing, and visualized service governance.
* Transparent interface based RPC
  * Dubbo provides high performance interface based RPC, which is transparent to users.
* Intelligent load balancing
  * Dubbo supports multiple load balancing strategies out of the box, which perceives downstream service status to reduce overall latency and improve system throughput.
* Automatic service registration and discovery
  * Dubbo supports multiple service registries, which can detect service online/offline instantly.
* High extensibility
  * Dubbo’s micro-kernel and plugin design ensures that it can easily be extended by third party implementation across core features like Protocol, Transport, and Serialization.
* Runtime traffic routing
  * Dubbo can be configured at runtime so that traffic can be routed according to different rules, which makes it easy to support features like blue-green deployment, data center aware routing, etc.
* Visualized service governance
  * Dubbo provides rich tools for service governance and maintenance such as querying service metadata, health status and statistics.
  
### Infrastructure
Hostname | Role | IP
:-----: | :----: | :-----:
hdss7-11.host.com  | zk1 | 10.4.7.11
hdss7-12.host.com  | zk2 | 10.4.7.12
hdss7-21.host.com  | zk2 | 10.4.7.21
hdss7-22.host.com  | jenkins | 10.4.7.22
hdss7-200.host.com | harbor | 10.4.7.200