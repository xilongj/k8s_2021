### Kubernetes Certs
#### [hdss7-200]
```buildoutcfg
[root@hdss7-200 ~]# ls -l /opt/certs/
total 100
-rw-r--r-- 1 root root 1249 Feb 12 08:30 apiserver.csr
-rw-r--r-- 1 root root  565 Feb 12 08:27 apiserver-csr.json
-rw------- 1 root root 1679 Feb 12 08:30 apiserver-key.pem
-rw-r--r-- 1 root root 1598 Feb 12 08:30 apiserver.pem
-rw-r--r-- 1 root root  836 Feb 11 21:10 ca-config.json
-rw-r--r-- 1 root root  997 Feb 11 11:24 ca.csr
-rw-r--r-- 1 root root  331 Feb 11 11:23 ca-csr.json
-rw------- 1 root root 1675 Feb 11 11:24 ca-key.pem
-rw-r--r-- 1 root root 1346 Feb 11 11:24 ca.pem
-rw-r--r-- 1 root root  997 Feb 12 08:23 client.csr
-rw-r--r-- 1 root root  283 Feb 12 08:17 client-csr.json
-rw------- 1 root root 1679 Feb 12 08:23 client-key.pem
-rw-r--r-- 1 root root 1371 Feb 12 08:23 client.pem
-rw-r--r-- 1 root root 1066 Feb 11 21:18 etcd-peer.csr
-rw-r--r-- 1 root root  366 Feb 11 21:15 etcd-peer-csr.json
-rw------- 1 root root 1675 Feb 11 21:18 etcd-peer-key.pem
-rw-r--r-- 1 root root 1432 Feb 11 21:18 etcd-peer.pem
-rw-r--r-- 1 root root 1123 Feb 12 19:41 kubelet.csr
-rw-r--r-- 1 root root  456 Feb 12 19:27 kubelet-csr.json
-rw------- 1 root root 1675 Feb 12 19:41 kubelet-key.pem
-rw-r--r-- 1 root root 1472 Feb 12 19:41 kubelet.pem
-rw-r--r-- 1 root root 1009 Feb 13 08:52 kube-proxy-client.csr
-rw------- 1 root root 1679 Feb 13 08:52 kube-proxy-client-key.pem
-rw-r--r-- 1 root root 1383 Feb 13 08:52 kube-proxy-client.pem
-rw-r--r-- 1 root root  270 Feb 13 08:47 kube-proxy-csr.json
```

```buildoutcfg
[root@hdss7-200 certs]# cfssl-certinfo -cert apiserver.pem
{
  "subject": {
    "common_name": "apiserver",
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
      "apiserver"
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
  "serial_number": "77592573585988226506448246262331906296623050583",
  "sans": [
    "kubernetes.default",
    "kubernetes.default.svc",
    "kubernetes.default.svc.cluster",
    "kubernetes.default.svc.cluster.local",
    "127.0.0.1",
    "192.168.0.1",
    "10.4.7.10",
    "10.4.7.21",
    "10.4.7.22",
    "10.4.7.23"
  ],
  "not_before": "2021-02-12T00:26:00Z",
  "not_after": "2041-02-07T00:26:00Z",
  "sigalg": "SHA256WithRSA",
  "authority_key_id": "FA:3C:F3:89:A8:AF:94:74:C9:AD:68:66:AE:F2:54:FE:B8:43:13:4A",
  "subject_key_id": "B6:92:57:F0:9:FF:FE:4A:DB:C6:95:EE:DD:11:92:1B:53:80:E4:8F",
  "pem": "-----BEGIN CERTIFICATE-----\nMIIEbzCCA1egAwIBAgIUDZdfD8pMeydLyju2MMcWtrsGr1cwDQYJKoZIhvcNAQEL\nBQAwYTELMAkGA1UEBhMCQ04xEDAOBgNVBAgTB0JlaWppbmcxEDAOBgNVBAcTB0Jl\naWppbmcxDTALBgNVBAoTBFhMTlgxDTALBgNVBAsTBFhFVVMxEDAOBgNVBAMTB0hv\nbWVFZHUwHhcNMjEwMjEyMDAyNjAwWhcNNDEwMjA3MDAyNjAwWjBjMQswCQYDVQQG\nEwJDTjEQMA4GA1UECBMHQmVpamluZzEQMA4GA1UEBxMHQmVpamluZzENMAsGA1UE\nChMEWExOWDENMAsGA1UECxMEWEVVUzESMBAGA1UEAxMJYXBpc2VydmVyMIIBIjAN\nBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAy4vE13LlnbB8PMxybYhjh7Zrm415\nHA4tYbVSToPy5EmlNnSizg6dS/rk1Bh5VRiOtHrQc50UCjG9JkWb58hbUkbZbarm\nm1M0ojrKi/DqT8/R1DjXIfw2ZFFnhC1n8cDM/n0upEwiWAsTwi9eT6G4/wQVprYq\nL+JsJGlmTbj7S6c8PP24I4yXRcZSOaHEOv2Q2THJMkut2rhJc5aDaHNhTkoVdkNa\nGQtH9zmsHP8V7SsNY4OuADjRXuHCHH1+50SwLw2u5LxxTcTmz36pZ6SbrypeN/J2\nSfAqko+lfM8RNbb844ySZQ8mE+KVAQibTa+Mlgo2X44UVmm2yROX3dYhpQIDAQAB\no4IBGzCCARcwDgYDVR0PAQH/BAQDAgWgMBMGA1UdJQQMMAoGCCsGAQUFBwMBMAwG\nA1UdEwEB/wQCMAAwHQYDVR0OBBYEFLaSV/AJ//5K28aV7t0RkhtTgOSPMB8GA1Ud\nIwQYMBaAFPo884mor5R0ya1oZq7yVP64QxNKMIGhBgNVHREEgZkwgZaCEmt1YmVy\nbmV0ZXMuZGVmYXVsdIIWa3ViZXJuZXRlcy5kZWZhdWx0LnN2Y4Iea3ViZXJuZXRl\ncy5kZWZhdWx0LnN2Yy5jbHVzdGVygiRrdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNs\ndXN0ZXIubG9jYWyHBH8AAAGHBMCoAAGHBAoEBwqHBAoEBxWHBAoEBxaHBAoEBxcw\nDQYJKoZIhvcNAQELBQADggEBAAMy0MQC0P9XvvHRLl0mdSnflNej7FzvbqbNH/BZ\nxM/HXuMucDgdiV8dNe32TI1s8liwsLJjEPxO6EwLJm1jqLN/o5hFEAaILqpQJn2/\ngIzTENKFmuzXlrD1yqFqs9wqfIT8+sZQxNxT/4AzLyh9BllSfWZh/jzZ38FjXgQ6\nQYgHGMG4scbUphLMj8MHlAjufL+TcS2twg8hev7zIpKixysiUqxRq7g9LyxN1Jrf\n4hFxrz1mQCvmxriydA4J1J0K+htHOdof4Mg0N1GpsepxM0DRW6fym/g5Ca+jtKUC\nMxqcZHioCIfnld8a/H5QxJn0bf2zCkEq+tKlQKkJP6bK77E=\n-----END CERTIFICATE-----\n"
}
```