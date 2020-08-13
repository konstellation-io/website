---
title: "Validate"
linkTitle: "Validate"
description: >
  Some check to validate the KRE deployment.
weight: 35
---

## Check pods on KRE namespace

Command:

```bash
kubectl -n kre get pods
```
Expected output:

```
NAME                                                     READY   STATUS    RESTARTS   AGE
alertmanager-kre-local-prometheus-opera-alertmanager-0   2/2     Running   0          28h
kre-local-admin-api-75d8bdc6b9-n4qcv                     1/1     Running   5          27h
kre-local-admin-ui-5d96987f95-9g59q                      1/1     Running   0          27h
kre-local-grafana-67f89f8977-pqdmw                       2/2     Running   0          28h
kre-local-k8s-manager-bddc586c4-kjhgf                    1/1     Running   0          27h
kre-local-kube-state-metrics-78fbbcbfb8-64ds8            1/1     Running   0          28h
kre-local-prometheus-node-exporter-sx9sb                 1/1     Running   0          28h
kre-local-prometheus-opera-operator-5b6f794b67-8lrdj     2/2     Running   0          28h
kre-mongo-0                                              1/1     Running   0          28h
prometheus-kre-local-prometheus-opera-prometheus-0       3/3     Running   1          28h
```

## Open Admin web console

If everything works fine we should open the URL https://admin.kre.yourdomain.com in a browser and the admin interface should open like the figure below.

{{< imgproc admin_web Resize "600x" >}}

{{< /imgproc >}}

## Check API is up

Command:

```bash
curl -v https://api.kre.yourdomain.com
```
Expected output:

```
*   Trying 1.2.3.4:443...
* TCP_NODELAY set
* Connected to api.kre.yourdomain.com (1.2.3.4) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server accepted to use h2
* Server certificate:
*  subject: CN=api.kre.yourdomain.com
*  start date: Aug  5 14:18:15 2020 GMT
*  expire date: Nov  3 14:18:15 2020 GMT
*  subjectAltName: host "api.kre.yourdomain.com" matched cert's "api.kre.yourdomain.com"
*  issuer: C=US; O=Let's Encrypt; CN=Let's Encrypt Authority X3
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x555789f5ddb0)
> GET / HTTP/2
> Host: api.kre.yourdomain.com
> user-agent: curl/7.68.0
> accept: */*
> 
* Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
< HTTP/2 404 
< server: nginx/1.17.10
< date: Thu, 13 Aug 2020 08:53:16 GMT
< content-type: application/json; charset=UTF-8
< content-length: 24
< access-control-allow-credentials: true
< access-control-allow-origin: 
< vary: Origin
< x-request-id: 74e620a75d284453082aabe95ca13958
< strict-transport-security: max-age=15724800; includeSubDomains
< 
{"message":"Not Found"}
* Connection #0 to host api.kre.yourdomain.com left intact
```

With this command we are validating that the API is up and running properly, because the expected answer is `{"message":"Not Found"}`. Also we can see that the certificate is issued correctly by Cert Manager.

## Check the API logs

Command:

```bash
kubectl -n kre logs kre-local-admin-api-75d8bdc6b9-n4qcv
```

Expected output:

```
2020-08-11T12:35:25.356756843Z INFO MongoDB connecting...
2020-08-11T12:35:25.356831185Z INFO MongoDB ping...
2020-08-11T12:35:25.371577437Z INFO MongoDB connected
2020-08-11T12:35:25.372054104Z INFO [RBAC] Reloading user roles
2020-08-11T12:35:25.372382218Z INFO [RBAC] Removing roles for user kre_admin_user (dev@local.local)
2020-08-11T12:35:25.372396432Z INFO [RBAC] Adding role ADMIN to user kre_admin_user (dev@local.local)
2020-08-11T12:35:25.372769609Z INFO HTTP server started on :80
2020-08-11T12:35:29.48550533Z INFO Request from user kre_admin_user
2020-08-11T12:35:29.501522116Z INFO Request from user kre_admin_user
2020-08-11T12:35:30.479367476Z INFO Request from user kre_admin_user
```
We should see at the very beginning of the API logs that the connection to MongoDB is fine and the service have started.
