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
## [Apache Dubbo](https://github.com/apache/dubbo)
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
  
## Infrastructure Map
Hostname | Role | IP
:-----: | :----: | :-----:
hdss7-11.host.com  | zookeeper1 | 10.4.7.11
hdss7-12.host.com  | zookeeper2 | 10.4.7.12
hdss7-21.host.com  | zookeeper3 | 10.4.7.21
hdss7-22.host.com  |  Jenkins   | 10.4.7.22
hdss7-200.host.com |  Harbor    | 10.4.7.200

### Install JDK
#### [hdss7-11]
```buildoutcfg
[root@hdss7-11 ~]# mkdir /opt/src
[root@hdss7-11 ~]# mv jdk-8u281-linux-x64.tar.gz /opt/src
[root@hdss7-11 ~]# mkdir -p /usr/java
[root@hdss7-11 ~]# cd /opt/src/
[root@hdss7-11 src]# tar xvf jdk-8u281-linux-x64.tar.gz -C /usr/java/
[root@hdss7-11 ~]# ln -s /usr/java/jdk1.8.0_281/ /usr/java/jdk
[root@hdss7-11 ~]# tail -3 /etc/profile
export JAVA_HOME=/usr/java/jdk
export PATH=$JAVA_HOME/bin:$JAVA_HOME/bin:$PATH
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar

[root@hdss7-11 ~]# source /etc/profile
[root@hdss7-11 ~]# java -version
java version "1.8.0_281"
Java(TM) SE Runtime Environment (build 1.8.0_281-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.281-b09, mixed mode)
```
#### [hdss7-12]
```buildoutcfg
[root@hdss7-12 ~]# scp hdss7-11:/opt/src/jdk-8u281-linux-x64.tar.gz .
[root@hdss7-12 ~]# mv jdk-8u281-linux-x64.tar.gz /opt/src/
[root@hdss7-12 ~]# mkdir -p /usr/java
[root@hdss7-12 ~]# cd /opt/src/
[root@hdss7-12 src]# tar xvf jdk-8u281-linux-x64.tar.gz -C /usr/java/
[root@hdss7-12 ~]# ln -s /usr/java/jdk1.8.0_281/ /usr/java/jdk
[root@hdss7-12 ~]# tail -3 /etc/profile
export JAVA_HOME=/usr/java/jdk
export PATH=$JAVA_HOME/bin:$JAVA_HOME/bin:$PATH
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar

[root@hdss7-12 ~]# source /etc/profile
[root@hdss7-12 ~]# java -version
java version "1.8.0_281"
Java(TM) SE Runtime Environment (build 1.8.0_281-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.281-b09, mixed mode)
```
#### [hdss7-21]
```buildoutcfg
[root@hdss7-21 ~]# scp hdss7-11:/opt/src/jdk-8u281-linux-x64.tar.gz .
[root@hdss7-21 ~]# mv jdk-8u281-linux-x64.tar.gz /opt/src/
[root@hdss7-21 ~]# mkdir -p /usr/java
[root@hdss7-21 ~]# cd /opt/src/
[root@hdss7-21 src]# tar xf jdk-8u281-linux-x64.tar.gz -C /usr/java/
[root@hdss7-21 src]# ln -s /usr/java/jdk1.8.0_281/ /usr/java/jdk
[root@hdss7-21 ~]# tail -3 /etc/profile
export JAVA_HOME=/usr/java/jdk
export PATH=$JAVA_HOME/bin:$JAVA_HOME/bin:$PATH
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar

[root@hdss7-21 ~]# source /etc/profile
[root@hdss7-21 ~]# java -version
java version "1.8.0_281"
Java(TM) SE Runtime Environment (build 1.8.0_281-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.281-b09, mixed mode)
```
### ZooKeeper(https://zookeeper.apache.org/index.html)
Welcome to Apache ZooKeeper™
Apache ZooKeeper is an effort to develop and maintain an open-source server which enables highly reliable distributed coordination.

What is ZooKeeper?
ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. All of these kinds of services are used in some form or another by distributed applications. Each time they are implemented there is a lot of work that goes into fixing the bugs and race conditions that are inevitable. Because of the difficulty of implementing these kinds of services, applications initially usually skimp on them, which make them brittle in the presence of change and difficult to manage. Even when done correctly, different implementations of these services lead to management complexity when the applications are deployed.

#### Download Zookeeper
```buildoutcfg
[root@hdss7-11 ~]# wget http://archive.apache.org/dist/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz -P /opt/src/

[root@hdss7-11 ~]# md5sum /opt/src/zookeeper-3.4.14.tar.gz
b0ecf30e2cd83eac8de3454e06bcae28  /opt/src/zookeeper-3.4.14.tar.gz
```
```buildoutcfg
[root@hdss7-12 ~]# wget http://archive.apache.org/dist/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz -P /opt/src/

[root@hdss7-12 ~]# md5sum /opt/src/zookeeper-3.4.14.tar.gz
b0ecf30e2cd83eac8de3454e06bcae28  /opt/src/zookeeper-3.4.14.tar.gz
```
```buildoutcfg
[root@hdss7-21 ~]# wget http://archive.apache.org/dist/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz -P /opt/src/

[root@hdss7-21 ~]# md5sum /opt/src/zookeeper-3.4.14.tar.gz
b0ecf30e2cd83eac8de3454e06bcae28  /opt/src/zookeeper-3.4.14.tar.gz
```
#### Install ZooKeeper
```buildoutcfg
[root@hdss7-11 src]# tar xvf zookeeper-3.4.14.tar.gz -C /opt/
[root@hdss7-11 ~]# ln -s /opt/zookeeper-3.4.14/ /opt/zookeeper
[root@hdss7-11 ~]# mkdir -pv /data/zookeeper/data /data/zookeeper/logs
[root@hdss7-11 ~]# cat /opt/zookeeper/conf/zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper/data
dataLogDir=/data/zookeeper/logs
clientPort=2181
server.1=zk1.od.com:2888:3888
server.2=zk2.od.com:2888:3888
server.3=zk3.od.com:2888:3888
```
```buildoutcfg
[root@hdss7-12 ~]# cd /opt/src/ && tar xvf zookeeper-3.4.14.tar.gz -C /opt/
[root@hdss7-12 ~]# ln -s /opt/zookeeper-3.4.14/ /opt/zookeeper
[root@hdss7-12 ~]# mkdir -pv /data/zookeeper/data /data/zookeeper/logs
[root@hdss7-12 ~]# cat /opt/zookeeper/conf/zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper/data
dataLogDir=/data/zookeeper/logs
clientPort=2181
server.1=zk1.od.com:2888:3888
server.2=zk2.od.com:2888:3888
server.3=zk3.od.com:2888:3888
```
```buildoutcfg
[root@hdss7-21 ~]# cd /opt/src/ && tar xvf zookeeper-3.4.14.tar.gz -C /opt/
[root@hdss7-21 ~]# ln -s /opt/zookeeper-3.4.14/ /opt/zookeeper
[root@hdss7-21 ~]# mkdir -pv /data/zookeeper/data /data/zookeeper/logs
[root@hdss7-21 ~]# cat /opt/zookeeper/conf/zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper/data
dataLogDir=/data/zookeeper/logs
clientPort=2181
server.1=zk1.od.com:2888:3888
server.2=zk2.od.com:2888:3888
server.3=zk3.od.com:2888:3888
```
#### [hdss7-11]
```buildoutcfg
[root@hdss7-11 ~]# cat /var/named/od.com.zone
$ORIGIN od.com.
$TTL 600        ; 10 minutes
@       IN SOA dns.od.com. dnsadmin.od.com. (
                        2021021006      ; serial
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
zk1             A       10.4.7.11
zk2             A       10.4.7.12
zk3             A       10.4.7.21

[root@hdss7-11 ~]# systemctl restart named
```
```buildoutcfg
[root@hdss7-12 ~]# dig -t A zk1.od.com @10.4.7.11 +short
10.4.7.11
[root@hdss7-12 ~]# dig -t A zk2.od.com @10.4.7.11 +short
10.4.7.12
[root@hdss7-12 ~]# dig -t A zk3.od.com @10.4.7.11 +short
10.4.7.21
```
```buildoutcfg
[root@hdss7-11 ~]# echo 1 > /data/zookeeper/data/myid
[root@hdss7-11 ~]# cat /data/zookeeper/data/myid
1
[root@hdss7-12 ~]# echo 2 > /data/zookeeper/data/myid
[root@hdss7-12 ~]# cat /data/zookeeper/data/myid
2
[root@hdss7-21 ~]# echo 3 > /data/zookeeper/data/myid
[root@hdss7-21 ~]# cat /data/zookeeper/data/myid
3
```
```buildoutcfg
[root@hdss7-11 ~]# /opt/zookeeper/bin/zkServer.sh start
[root@hdss7-11 ~]# /opt/zookeeper/bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Mode: follower

[root@hdss7-12 ~]# /opt/zookeeper/bin/zkServer.sh start
[root@hdss7-12 ~]# /opt/zookeeper/bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Mode: leader

[root@hdss7-21 ~]# /opt/zookeeper/bin/zkServer.sh start
[root@hdss7-21 ~]# /opt/zookeeper/bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Mode: follower
```
***

## [Jenkins](https://www.jenkins.io/)
The leading open source automation server, Jenkins provides hundreds of plugins to support building, deploying and automating any project.

### Install Jenkins
#### [hdss7-200]
```buildoutcfg
[root@hdss7-200 ~]# docker pull jenkins/jenkins:2.190.3
docker.io/jenkins/jenkins:2.190.3
[root@hdss7-200 ~]# docker images | grep jenkins
jenkins/jenkins                    2.190.3        22b8b9a84dbe   15 months ago   568MB
[root@hdss7-200 ~]# docker tag 22b8b9a84dbe harbor.od.com/public/jenkins:v2.190.3
[root@hdss7-200 ~]# docker images | grep jenkins
jenkins/jenkins                    2.190.3        22b8b9a84dbe   15 months ago   568MB
harbor.od.com/public/jenkins       v2.190.3       22b8b9a84dbe   15 months ago   568MB

[root@hdss7-200 ~]# docker push harbor.od.com/public/jenkins:v2.190.3
```
#### Generate Private Key
```buildoutcfg
[root@hdss7-200 ~]# ssh-keygen -t rsa -b 2048 -C "xilongus@gmail.com" -N "" -f /root/.ssh/id_rsa
Generating public/private rsa key pair.
Created directory '/root/.ssh'.
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:12y/zJ0ZBfZYfdCwAHSca0X1BCsDZ0cGaFH3CSaHRYY xilongus@gmail.com
The key's randomart image is:
+---[RSA 2048]----+
|         .=B%%X=o|
|          oEB=o*=|
|         .  oo=.*|
|           oo+ =.|
|        S ..+ . o|
|         . . .  .|
|              .. |
|             o o+|
|              +o.|
+----[SHA256]-----+

[root@hdss7-200 ~]# ls -l .ssh
total 8
-rw------- 1 root root 1679 Feb 18 14:11 id_rsa
-rw-r--r-- 1 root root  400 Feb 18 14:11 id_rsa.pub
```
```buildoutcfg
[root@hdss7-200 ~]# mkdir -p /data/dockerfile
[root@hdss7-200 ~]# cd /data/dockerfile
[root@hdss7-200 dockerfile]# mkdir jenkins
[root@hdss7-200 dockerfile]# cd jenkins/
```
```buildoutcfg
# Dockerfile
[root@hdss7-200 jenkins]# cat Dockerfile
FROM harbor.od.com/public/jenkins:v2.190.3
USER root
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo 'Asia/Shanghai' > /etc/timezone
ADD id_rsa /root/.ssh/id_rsa
ADD config.json /root/.docker/config.json
ADD get-docker.sh /get-docker.sh
RUN echo "    StrictHostKeyChecking no" >> /etc/ssh/ssh_config && \
    /get-docker.sh
```
```buildoutcfg
[root@hdss7-200 jenkins]# cp /root/.ssh/id_rsa .
[root@hdss7-200 jenkins]# cp /root/.docker/config.json .
[root@hdss7-200 jenkins]# curl -fsSL get.docker.com -o get-docker.sh
[root@hdss7-200 jenkins]# chmod +x get-docker.sh
```
```buildoutcfg
[root@hdss7-200 jenkins]# cat config.json
{
        "auths": {
                "harbor.od.com": {
                        "auth": "YWRtaW46eGxueEAyMDIx"
                }
        }
}
```
```buildoutcfg
[root@hdss7-200 jenkins]# cat config.json
{
        "auths": {
                "harbor.od.com": {
                        "auth": "YWRtaW46eGxueEAyMDIx"
                }
        }
}

[root@hdss7-200 jenkins]# echo YWRtaW46eGxueEAyMDIx | base64 -d
admin:xlnx@2021
```
```buildoutcfg
[root@hdss7-200 jenkins]# ls -l /data/dockerfile/jenkins/
total 28
-rw------- 1 root root    77 Feb 18 14:25 config.json
-rw-r--r-- 1 root root   352 Feb 18 14:24 Dockerfile
-rwxr-xr-x 1 root root 13857 Feb 18 14:26 get-docker.sh
-rw------- 1 root root  1679 Feb 18 14:24 id_rsa
```
#### Create Private Project
```buildoutcfg
# login harbor.od.com
# create new project as infra & Private
```
#### build log
```buildoutcfg
[root@hdss7-200 jenkins]# docker build . -t harbor.od.com/infra/jenkins:v2.190.3
Sending build context to Docker daemon  20.48kB
Step 1/7 : FROM harbor.od.com/public/jenkins:v2.190.3
 ---> 22b8b9a84dbe
Step 2/7 : USER root
 ---> Using cache
 ---> d197ff9fc121
Step 3/7 : RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&     echo 'Asia/Shanghai' > /etc/timezone
 ---> Using cache
 ---> 2b7b703f273e
Step 4/7 : ADD id_rsa /root/.ssh/id_rsa
 ---> Using cache
 ---> 26ea744560d7
Step 5/7 : ADD config.json /root/.docker/config.json
 ---> Using cache
 ---> 7b99a3f5afe4
Step 6/7 : ADD get-docker.sh /get-docker.sh
 ---> Using cache
 ---> 9bb9cf91d820
Step 7/7 : RUN echo "    StrictHostKeyChecking no" >> /etc/ssh/ssh_config &&     /get-docker.sh
 ---> Running in cf216a4f2401
# Executing docker install script, commit: 3d8fe77c2c46c5b7571f94b42793905e5b3e42e4
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq apt-transport-https ca-certificates curl >/dev/null
debconf: delaying package configuration, since apt-utils is not installed
+ sh -c curl -fsSL "https://download.docker.com/linux/debian/gpg" | apt-key add -qq - >/dev/null
Warning: apt-key output should not be parsed (stdout is not a terminal)
+ sh -c echo "deb [arch=amd64] https://download.docker.com/linux/debian stretch stable" > /etc/apt/sources.list.d/docker.list
+ sh -c apt-get update -qq >/dev/null
+ [ -n  ]
+ sh -c apt-get install -y -qq --no-install-recommends docker-ce >/dev/null
debconf: delaying package configuration, since apt-utils is not installed
If you would like to use Docker as a non-root user, you should now consider
adding your user to the "docker" group with something like:

  sudo usermod -aG docker your-user

Remember that you will have to log out and back in for this to take effect!

WARNING: Adding a user to the "docker" group will grant the ability to run
         containers which can be used to obtain root privileges on the
         docker host.
         Refer to https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface
         for more information.
Removing intermediate container cf216a4f2401
 ---> 6ed14fcfb081
Successfully built 6ed14fcfb081
Successfully tagged harbor.od.com/infra/jenkins:v2.190.3
```
```buildoutcfg
[root@hdss7-200 jenkins]# docker push harbor.od.com/infra/jenkins:v2.190.3
The push refers to repository [harbor.od.com/infra/jenkins]
281af236df81: Pushed
f127106cfd86: Pushed
819040809720: Pushed
a718a9c3025d: Pushed
2ee0e38a5c12: Pushed
e0485b038afa: Mounted from public/jenkins
2950fdd45d03: Mounted from public/jenkins
cfc53f61da25: Mounted from public/jenkins
29c489ae7aae: Mounted from public/jenkins
473b7de94ea9: Mounted from public/jenkins
6ce697717948: Mounted from public/jenkins
0fb3a3c5199f: Mounted from public/jenkins
23257f20fce5: Mounted from public/jenkins
b48320151ebb: Mounted from public/jenkins
911119b5424d: Mounted from public/jenkins
5051dc7ca502: Mounted from public/jenkins
a8902d6047fe: Mounted from public/jenkins
99557920a7c5: Mounted from public/jenkins
7e3c900343d0: Mounted from public/jenkins
b8f8aeff56a8: Mounted from public/jenkins
687890749166: Mounted from public/jenkins
2f77733e9824: Mounted from public/jenkins
97041f29baff: Mounted from public/jenkins
v2.190.3: digest: sha256:53d6f3a507b87b2fdae88533d8389a977273cfabc3269fd63a02477e0d484af8 size: 5130
```
```buildoutcfg
[root@hdss7-200 jenkins]# docker images | grep jenkins
harbor.od.com/infra/jenkins        v2.190.3       6ed14fcfb081   2 minutes ago   1.02GB
jenkins/jenkins                    2.190.3        22b8b9a84dbe   15 months ago   568MB
harbor.od.com/public/jenkins       v2.190.3       22b8b9a84dbe   15 months ago   568MB
```
### Add Key on Gitee
```buildoutcfg
[root@hdss7-200 jenkins]# cat /root/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD*****************************************************Ty2DAZaP4hQGj/FbZb2PnhUviT4QS7IZSkekfYdx4idR0ENki1FqCbWWA8NkPot3YfkBXh4wW75V1DS1eMzIf8RX2M4W/M0Q9Zym2nLyXyffaU8fqNXzAO8pgK8+3ZfTksxz5q8FnjVTWraOUnQ/U9b/8OL9DNsmNBi3UTNAjtCSZXChNb1uTcrntQRsSqEP9csAfdoJ1yqpSH/O1/FfqknC7b1AQLVgMfrwWSHXdyqc7U6A4TqweJgDB3HUV4q/0XSwAWP4jEJUyy34CyqSAYWBNb xilongus@gmail.com

# https://gitee.com
# Settings --> Key manages --> Add key
```
```buildoutcfg
[root@hdss7-200 jenkins]# docker run --rm harbor.od.com/infra/jenkins:v2.190.3 ssh -i /root/.ssh/id_rsa -T git@gitee.com
Warning: Permanently added 'gitee.com,212.64.62.183' (ECDSA) to the list of known hosts.
Hi Xilong Jin (DeployKey)! You've successfully authenticated, but GITEE.COM does not provide shell access.
Note: Perhaps the current use is DeployKey.
Note: DeployKey only supports pull/fetch operations
```
### Create Namespace infra
#### [hdss7-21]
```buildoutcfg
[root@hdss7-21 ~]# kubectl create ns infra
[root@hdss7-21 ~]# kubectl create secret docker-registry harbor --docker-server=harbor.od.com --docker-username=admin --docker-password=xlnx@2021 -n infra
secret/harbor created
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get secret -n infra
NAME                  TYPE                                  DATA   AGE
default-token-cqfh7   kubernetes.io/service-account-token   3      101m
harbor                kubernetes.io/dockerconfigjson        1      4m29s
```
```buildoutcfg
# https://dashboard.od.com
# Namespace: infra  
# Config and Storage: Secrets
 .dockerconfigjson:
{"auths":{"harbor.od.com":{"username":"admin","password":"xlnx@2021","auth":"YWRtaW46eGxueEAyMDIx"}}}
```
```buildoutcfg
# base64
[root@hdss7-21 ~]# echo "YWRtaW46eGxueEAyMDIx" | base64 -d
admin:xlnx@2021
```
### Install NFS
#### [hdss7-200]
```buildoutcfg
[root@hdss7-200 ~]# yum -y install nfs-utils
[root@hdss7-200 ~]# rpm -qa nfs-utils
nfs-utils-1.3.0-0.68.el7.x86_64
```
```buildoutcfg
[root@hdss7-200 ~]# cat /etc/exports
/data/xsj-storage       10.4.7.0/24(rw,no_root_squash)
[root@hdss7-200 ~]# mkdir -p /data/xsj-storage

[root@hdss7-200 ~]# systemctl start nfs
[root@hdss7-200 ~]# systemctl enable nfs
```
```buildoutcfg
# man exports
   User ID Mapping
       nfsd bases its access control to files on the server machine on the uid and gid provided in each NFS RPC request. The normal behavior  a  user  would
       expect  is  that she can access her files on the server just as she would on a normal file system. This requires that the same uids and gids are used
       on the client and the server machine. This is not always true, nor is it always desirable.

       Very often, it is not desirable that the root user on a client machine is also treated as root when accessing files on the NFS server. To  this  end,
       uid  0  is normally mapped to a different id: the so-called anonymous or nobody uid. This mode of operation (called `root squashing') is the default,
       and can be turned off with no_root_squash.

       By default, exportfs chooses a uid and gid of 65534 for squashed access. These values can also be overridden by  the  anonuid  and  anongid  options.
       Finally, you can map all user requests to the anonymous uid by specifying the all_squash option.

       Here's the complete list of mapping options:

       root_squash
              Map requests from uid/gid 0 to the anonymous uid/gid. Note that this does not apply to any other uids or gids that might be equally sensitive,
              such as user bin or group staff.

       no_root_squash
              Turn off root squashing. This option is mainly useful for diskless clients.

       all_squash
              Map all uids and gids to the anonymous user. Useful for NFS-exported public FTP directories, news spool directories, etc. The opposite  option
              is no_all_squash, which is the default setting.

       anonuid and anongid
              These  options  explicitly  set the uid and gid of the anonymous account.  This option is primarily useful for PC/NFS clients, where you might
              want all requests appear to be from one user. As an example, consider the export entry for /home/joe in the example section below, which  maps
              all requests to uid 150 (which is supposedly that of user joe).
```
```buildoutcfg
# deployment.yaml
[root@hdss7-200 jenkins]# cat deployment.yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: jenkins
  namespace: infra
  labels:
    name: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      name: jenkins
  template:
    metadata:
      labels:
        app: jenkins
        name: jenkins
    spec:
      volumes:
      - name: data
        nfs:
          server: hdss7-200
          path: /data/xsj-storage/jenkins_home        # /data/xsj-storage
      - name: docker
        hostPath:
          path: /run/docker.sock
          type: ''
      containers:
      - name: jenkins
        image: harbor.od.com/infra/jenkins:v2.190.3
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
          protocol: TCP
        env:
        - name: JAVA_OPTS
          value: -Xmx512m -Xms512m
        resources:
          limits:
            cpu: 500m
            memory: 1Gi
          requests:
            cpu: 500m
            memory: 1Gi
        volumeMounts:
        - name: data
          mountPath: /var/jenkins_home
        - name: docker
          mountPath: /run/docker.sock
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      imagePullSecrets:
      - name: harbor
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      securityContext:
        runAsUser: 0
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  revisionHistoryLimit: 7
  progressDeadlineSeconds: 600
```
```buildoutcfg
[root@hdss7-21 ~]# file /run/docker.sock
/run/docker.sock: socket
[root@hdss7-22 ~]# file /run/docker.sock
/run/docker.sock: socket
```
```buildoutcfg
# svc.yaml
[root@hdss7-200 jenkins]# cat svc.yaml
kind: Service
apiVersion: v1
metadata:
  name: jenkins
  namespace: infra
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  selector:
    app: jenkins
```
```buildoutcfg
# ingress.yaml
[root@hdss7-200 jenkins]# cat ingress.yaml
kind: Ingress
apiVersion: extensions/v1beta1
metadata:
  name: jenkins
  namespace: infra
spec:
  rules:
  - host: jenkins.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: jenkins
          servicePort: 80
```
```buildoutcfg
[root@hdss7-200 ~]# mkdir -p /data/xsj-storage/jenkins_home
```
#### [hdss7-21]
```buildoutcfg
[root@hdss7-21 ~]# yum -y install nfs-utils
[root@hdss7-21 ~]# rpm -qa nfs-utils
nfs-utils-1.3.0-0.68.el7.x86_64
[root@hdss7-21 ~]# systemctl start nfs
[root@hdss7-21 ~]# systemctl enable nfs
[root@hdss7-21 ~]# #kubectl apply -f http://k8s-yaml.od.com/jenkins/deployment.yaml
[root@hdss7-21 ~]# #kubectl apply -f http://k8s-yaml.od.com/jenkins/svc.yaml
[root@hdss7-21 ~]# #kubectl apply -f http://k8s-yaml.od.com/jenkins/ingress.yaml
```
```buildoutcfg
[root@hdss7-21 ~]# kubectl get pods -o wide -n infra
NAME                       READY   STATUS    RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
jenkins-576b469db4-9tmth   1/1     Running   0          7m17s   172.7.21.4   hdss7-21.host.com   <none>           <none>
# jenkins logs
[root@hdss7-21 ~]# kubectl describe pod jenkins-576b469db4-9tmth -n infra
Name:           jenkins-576b469db4-9tmth
Namespace:      infra
Priority:       0
Node:           hdss7-21.host.com/10.4.7.21
Start Time:     Fri, 19 Feb 2021 02:14:55 +0800
Labels:         app=jenkins
                name=jenkins
                pod-template-hash=576b469db4
Annotations:    <none>
Status:         Running
IP:             172.7.21.4
Controlled By:  ReplicaSet/jenkins-576b469db4
Containers:
  jenkins:
    Container ID:   docker://4dfd7c3c7cccf622d3dff27f3e6d76f6b9b5466ee2e4f7c2b665c0847d526d4c
    Image:          harbor.od.com/infra/jenkins:v2.190.3
    Image ID:       docker-pullable://harbor.od.com/infra/jenkins@sha256:53d6f3a507b87b2fdae88533d8389a977273cfabc3269fd63a02477e0d484af8
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 19 Feb 2021 02:14:57 +0800
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  1Gi
    Requests:
      cpu:     500m
      memory:  1Gi
    Environment:
      JAVA_OPTS:  -Xmx512m -Xms512m
    Mounts:
      /run/docker.sock from docker (rw)
      /var/jenkins_home from data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-cqfh7 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  data:
    Type:      NFS (an NFS mount that lasts the lifetime of a pod)
    Server:    hdss7-200
    Path:      /data/xsj-storage/jenkins_home
    ReadOnly:  false
  docker:
    Type:          HostPath (bare host directory volume)
    Path:          /run/docker.sock
    HostPathType:
  default-token-cqfh7:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-cqfh7
    Optional:    false
QoS Class:       Guaranteed
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From                        Message
  ----    ------     ----   ----                        -------
  Normal  Scheduled  7m23s  default-scheduler           Successfully assigned infra/jenkins-576b469db4-9tmth to hdss7-21.host.com
  Normal  Pulled     7m22s  kubelet, hdss7-21.host.com  Container image "harbor.od.com/infra/jenkins:v2.190.3" already present on machine
  Normal  Created    7m22s  kubelet, hdss7-21.host.com  Created container jenkins
  Normal  Started    7m21s  kubelet, hdss7-21.host.com  Started container jenkins
```
#### [hdss7-200]
```buildoutcfg
# jenkins storage
[root@hdss7-200 ~]# ls -l /data/xsj-storage/
total 4
drwxr-xr-x 13 root root 4096 Feb 19 02:15 jenkins_home
[root@hdss7-200 ~]#
[root@hdss7-200 ~]# ls -l /data/xsj-storage/jenkins_home/
total 36
-rw-r--r--  1 root root 1643 Feb 19 02:15 config.xml
-rw-r--r--  1 root root   50 Feb 19 02:14 copy_reference_file.log
-rw-r--r--  1 root root  156 Feb 19 02:15 hudson.model.UpdateCenter.xml
-rw-------  1 root root 1712 Feb 19 02:15 identity.key.enc
-rw-r--r--  1 root root    7 Feb 19 02:15 jenkins.install.UpgradeWizard.state
-rw-r--r--  1 root root  171 Feb 19 02:15 jenkins.telemetry.Correlator.xml
drwxr-xr-x  2 root root    6 Feb 19 02:14 jobs
drwxr-xr-x  3 root root   19 Feb 19 02:15 logs
-rw-r--r--  1 root root  907 Feb 19 02:15 nodeMonitors.xml
drwxr-xr-x  2 root root    6 Feb 19 02:15 nodes
drwxr-xr-x  2 root root    6 Feb 19 02:14 plugins
-rw-r--r--  1 root root   64 Feb 19 02:14 secret.key
-rw-r--r--  1 root root    0 Feb 19 02:14 secret.key.not-so-secret
drwx------  4 root root  265 Feb 19 02:15 secrets
drwxr-xr-x  2 root root   67 Feb 19 02:16 updates
drwxr-xr-x  2 root root   24 Feb 19 02:15 userContent
drwxr-xr-x  3 root root   56 Feb 19 02:15 users
drwxr-xr-x 11 root root 4096 Feb 19 02:14 war
```
#### [hdss7-11]
```buildoutcfg
[root@hdss7-11 ~]# cat /var/named/od.com.zone
$ORIGIN od.com.
$TTL 600        ; 10 minutes
@       IN SOA dns.od.com. dnsadmin.od.com. (
                        2021021007      ; serial
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
zk1             A       10.4.7.11
zk2             A       10.4.7.12
zk3             A       10.4.7.21
jenkins         A       10.4.7.10

[root@hdss7-11 ~]# systemctl restart named
[root@hdss7-11 ~]# dig -t A jenkins.od.com @10.4.7.11 +short
10.4.7.10
```
#### Windows hosts
```buildoutcfg
# Windows hosts (C:\Windows\System32\drivers\etc)
10.4.7.10		dashboard.od.com  harbor.od.com	 traefik.od.com  jenkins.od.com
10.4.7.200		k8s-yaml.od.com
```

#### Jenkins initial password
```buildoutcfg
[root@hdss7-200 ~]# cat /data/xsj-storage/jenkins_home/secrets/initialAdminPassword
90cafb5fddc743a690bd552641df70a0
```
```buildoutcfg
URL: http://jenkins.od.com/
Username: admin
Password: admin123
```
#### Manage Jenkins
* Configure Global Security
  * Authorization
    * [Y] Allow anonymous read access
  * CSRF Protection
    * [N] Prevent Cross Site Request Forgery exploits
* Manage Plugins
  * Available
    * [Y] Blue Ocean (Download now and install after restart)

#### BlueOcean Aggregator  
Warning: 
This plugin requires dependent plugins that require Jenkins 2.204.1 or newer. Jenkins will refuse to load the dependent plugins requiring a newer version of Jenkins, and in turn loading this plugin will fail.
