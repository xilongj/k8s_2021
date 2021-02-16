### curl
```buildoutcfg
[root@hdss7-200 ~]# curl -sIL -w "%{http_code}\n" -o /dev/null http://k8s-yaml.od.com/
200
```
### dig
```buildoutcfg
[root@hdss7-21 ~]# dig -t A nginx-dp.kube-public.svc.cluster.local. @192.168.0.2 +short
192.168.150.147
```