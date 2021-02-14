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
***
```buildoutcfg
[root@hdss7-200 ~]# cfssl-certinfo -domain www.xilinx.com
{
  "subject": {
    "common_name": "www.xilinx.com",
    "names": [
      "www.xilinx.com"
    ]
  },
  "issuer": {
    "common_name": "Sectigo RSA Domain Validation Secure Server CA",
    "country": "GB",
    "organization": "Sectigo Limited",
    "locality": "Salford",
    "province": "Greater Manchester",
    "names": [
      "GB",
      "Greater Manchester",
      "Salford",
      "Sectigo Limited",
      "Sectigo RSA Domain Validation Secure Server CA"
    ]
  },
  "serial_number": "63421351399747298645501159832218948933",
  "sans": [
    "www.xilinx.com",
    "china.xilinx.com",
    "developer.xilinx.com",
    "japan.xilinx.com",
    "license.xilinx.com",
    "petalinux.xilinx.com"
  ],
  "not_before": "2020-03-03T00:00:00Z",
  "not_after": "2021-03-03T23:59:59Z",
  "sigalg": "SHA256WithRSA",
  "authority_key_id": "8D:8C:5E:C4:54:AD:8A:E1:77:E9:9B:F9:9B:5:E1:B8:1:8D:61:E1",
  "subject_key_id": "CD:94:AF:E4:A2:EB:E5:A0:97:96:AB:13:7D:E0:CA:B2:79:A5:12:C3",
  "pem": "-----BEGIN CERTIFICATE-----\nMIIGDzCCBPegAwIBAgIQL7aCCs8HyiXlNf4SV07VRTANBgkqhkiG9w0BAQsFADCB\njzELMAkGA1UEBhMCR0IxGzAZBgNVBAgTEkdyZWF0ZXIgTWFuY2hlc3RlcjEQMA4G\nA1UEBxMHU2FsZm9yZDEYMBYGA1UEChMPU2VjdGlnbyBMaW1pdGVkMTcwNQYDVQQD\nEy5TZWN0aWdvIFJTQSBEb21haW4gVmFsaWRhdGlvbiBTZWN1cmUgU2VydmVyIENB\nMB4XDTIwMDMwMzAwMDAwMFoXDTIxMDMwMzIzNTk1OVowGTEXMBUGA1UEAxMOd3d3\nLnhpbGlueC5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC1Q8F+\nuSgBw9XQR+mbn2c1CywmdwTvpK8Gsllcu/hc8WOPWHyqY5b9PcjUTTlflFy35hZH\nTkinvjiaJTEFtxGQmbhOOI3ozepfuFAK53mIB6HVpAJi/fJ5dFRDxEtWRck8eFFq\nXie3BnMd2g9sUAW1I2BvU1tNR/CzyfwbWEU7hMMMMJ8t3o9UMq4+ujEoBVOOUX16\nGm/qkbHzNmBtOWXEE22KZRG3ajyH7PBm6YEAi8wlHyX0bioSZR5xJqk96/cDmABm\n/f8UPG7pfS89anGw1wIqf6/PA1vsOgKXkVpcoV0FmkCfcgqTOUutbb5lkJTeGYVN\n6tGWMUlcmnCXjuGPAgMBAAGjggLaMIIC1jAfBgNVHSMEGDAWgBSNjF7EVK2K4Xfp\nm/mbBeG4AY1h4TAdBgNVHQ4EFgQUzZSv5KLr5aCXlqsTfeDKsnmlEsMwDgYDVR0P\nAQH/BAQDAgWgMAwGA1UdEwEB/wQCMAAwHQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsG\nAQUFBwMCMEkGA1UdIARCMEAwNAYLKwYBBAGyMQECAgcwJTAjBggrBgEFBQcCARYX\naHR0cHM6Ly9zZWN0aWdvLmNvbS9DUFMwCAYGZ4EMAQIBMIGEBggrBgEFBQcBAQR4\nMHYwTwYIKwYBBQUHMAKGQ2h0dHA6Ly9jcnQuc2VjdGlnby5jb20vU2VjdGlnb1JT\nQURvbWFpblZhbGlkYXRpb25TZWN1cmVTZXJ2ZXJDQS5jcnQwIwYIKwYBBQUHMAGG\nF2h0dHA6Ly9vY3NwLnNlY3RpZ28uY29tMIIBBAYKKwYBBAHWeQIEAgSB9QSB8gDw\nAHYAfT7y+I//iFVoJMLAyp5SiXkrxQ54CX8uapdomX4i8NcAAAFwomZ8pAAABAMA\nRzBFAiBZg2Mjy24pzRfPbjdDFvp12HFAOOctAtUHP8F0949rMAIhAMmu68Wk19Ub\nb1CCEvSHQ99xmabT8CKksfsrv262ou06AHYAlCC8Ho7VjWyIcx+CiyIsDdHaTV5s\nT5Q9YdtOL1hNosIAAAFwomZ8bwAABAMARzBFAiBXq5vI1o99EpvtsGqCk3ryi6Ju\nCvkdosNLMM1bM/V6sAIhANwfj3sxsNw+oHy+Pnq6wNH2LxLGZ4cjEHAEhH3sSm8T\nMH0GA1UdEQR2MHSCDnd3dy54aWxpbnguY29tghBjaGluYS54aWxpbnguY29tghRk\nZXZlbG9wZXIueGlsaW54LmNvbYIQamFwYW4ueGlsaW54LmNvbYISbGljZW5zZS54\naWxpbnguY29tghRwZXRhbGludXgueGlsaW54LmNvbTANBgkqhkiG9w0BAQsFAAOC\nAQEA0ZIbs04XUHe42av/glIsS14saWm0A377BGhImg1eZYPcMP/ptiGouFBKI5ag\nNyHvnGrmYoOBCwDJndeyrN3zObwj6TX7aRSem74p8LCBWCe4nkaTzdUCa7w6EPP2\noeRtJKjhevyftnJhd90HdAc/9N4UABMqX7zALkbW5ixoC+HskfjfAf/V4QPUFg+K\njcs/2H+0E6rUpU3FAcOaxVPPsLTTj2PgwTxHhmX16KjfB3ftsfv3JQcLl8x+fBiQ\nm06haeNQ8u2Ig8yK0te6Cs8Yv5JpHl/CCibdutXC5T/PoryYaJSVxCbeZgi0U6SY\n62lyReGsiTKJ+A8UdjoyHXQy1Q==\n-----END CERTIFICATE-----\n"
}
```