---
title: "Customization"
linkTitle: "Customization"
description: >
  Helm chart parameters to configure a KRE deployment via a `values.yaml` file or the flag `--set`.
weight: 50
---

It is posible to configure a lot of aspects of a KRE deployment, depending on your environment and your usecase the default values are not enough. 
Below are described all the parameters suceptible to be changed to adapt KRE Helm Chart to your environment. You can create your own `values.yaml` with these 
parameters customized to be applied when run Helm. In order to get a more clear idea where can find 
the parameter that you need to modify we have split the parameters in some sections by configuration goals.

- [Config](#config)
- [Runtime](#runtime)
- [Admin API](#admin-api)
- [Admin UI](#admin-ui)
- [MongoDB](#mongodb)
- [Cert Manager](#cert-manager)
  - [DNS01](#dns01)
  - [HTTP01](#http01)
- [Prometheus](#prometheus)
  
## Config 

Under this section are all the parameters that are used by multiple components in order to not repeat, also general 
configuration related with the behaviour of KRE and not just specific components requirements like Docker images, services ports,
storage size, etc.

| Parameter                                       | Description                                | Default                 |
| ----------------------------------------------- | ------------------------------------------ | ----------------------- |
| `config.baseDomainName`                         | Domain name used to access KRE             | `local`                 |
| `config.admin.apiAddress`                       | Base internal URL for Api Server           | `:3000`                 |
| `config.admin.apiBaseURL`                       | Base public URL for Api Server             | `api.kre.local`         |
| `config.admin.frontendBaseURL`                  | Base URL to connect from frontent          | `http://localhost:3000` |
| `config.admin.corsEnabled`                      | Activate CORS in Admin API                 | `true`                  |
| `config.admin.userEmail`                        | Emain for default Admin user               | `dev@local.local`       |
| `config.smtp.enabled`                           | Activate SMTP                              | `false`                 |
| `config.smtp.sender`                            | SMTP Sender Email                          | <not_defined>           |
| `config.smtp.senderName`                        | SMTP Sender Name                           | <not_defined>           |
| `config.smtp.user`                              | SMTP User to connect                       | <not_defined>           |
| `config.smtp.pass`                              | SMTP Password to connect                   | <not_defined>           |
| `config.smtp.host`                              | SMTP Host to connect                       | <not_defined>           |
| `config.smtp.port`                              | SMTP Port to connect                       | <not_defined>           |
| `config.auth.verificationCodeDurationInMinutes` | User Verification Code Duration In Minutes | `1`                     |
| `config.auth.jwtSignSecret`                     | JWT Sign Secret Key                        | `jwt_secret`            |
| `config.auth.secureCookie`                      | Activate secure cookie                     | `false`                 |

## Runtime

Within `config.runtime` you can setup some parameters that will affect when a new Runtime is created, for example the storageClass
to create the volumes of the base service of a runtime.

| Parameter                                       | Description                                | Default                 |
| ----------------------------------------------- | ------------------------------------------ | ----------------------- |
| `config.runtime.sharedStorageClass`             | StorageClass to use RWX volume for Minio and Runners on Runtimes| `standard`                 |
| `config.runtime.sharedStorageSize`              | Volume size for Minio and Runners on Runtimes| `2Gi`                 |
| `config.runtime.nats_streaming.storage.className`             | StorageClass to create volumes for NATS pods | `standard`                 |
| `config.runtime.nats_streaming.storage.size`             | Size of volume attached to NATS Pod | `1Gi`                 |
| `config.runtime.mongodb.persistentVolume.enabled`             | This parameter enable the persistence of MongoDB | `true`                 |
| `config.runtime.mongodb.persistentVolume.storageClass`             | StorageClass to create volumes for MongoDB | `standard`                 |
| `config.runtime.mongodb.persistentVolume.size`             | Size of volume attached to MongoDB Pod  | `5Gi`                 |
| `config.runtime.chronograf.persistentVolume.enabled`             | This parameter enable the persistence of Chronograf | `true`                 |
| `config.runtime.chronograf.persistentVolume.storageClass`             | StorageClass to create volumes for Chronograf | `standard`                 |
| `config.runtime.chronograf.persistentVolume.size`             | Size of volume attached to Chronograf Pod | `1Gi`                 |
| `config.runtime.influxdb.persistentVolume.enabled`             | This parameter enable the persistence of InfluxDB | `true`                 |
| `config.runtime.influxdb.persistentVolume.storageClass`             | StorageClass to create volumes for InfluxDB | `standard`                 |
| `config.runtime.influxdb.persistentVolume.size`             | Size of volume attached to InfluxDB Pod | `5Gi`                 |

## Admin API

Specific configuration for Admin API

| Parameter                    | Description                                                                                               | Default                       |
| ---------------------------- | --------------------------------------------------------------------------------------------------------- | ----------------------------- |
| `admin-api.image.repository` | Docker registry to download the admin-api image                                                           | `konstellation/kre-admin-api` |
| `admin-api.image.tag`        | Version of the admin-api Docker image to deploy                                                           | `latest`                      |
| `admin-api.image.pullPolicy` | Define when Kubernetes has to pull a Docker image                                                         | `IfNotPresent`                |
| `admin-api.service.port`     | TCP port where is going to listen the internal service                                                    | `4000`                        |
| `admin-api.tls.enabled`      | If we want to enable HTTPS access to the API. For this Cert Manager is required in the Kuberentes cluster | `false`                       |
| `admin-api.host`             | Public hostname to generate SSL certificate with Cert Manager                                             | `false`                       |


## Admin UI
Specific configuration for Admin UI

| Parameter                   | Description                                                                                              | Default                      |
| --------------------------- | -------------------------------------------------------------------------------------------------------- | ---------------------------- |
| `admin-ui.image.repository` | Docker registry to download the admin-ui image                                                           | `konstellation/kre-admin-ui` |
| `admin-ui.image.tag`        | Version of the admin-ui Docker image to deploy                                                           | `latest`                     |
| `admin-ui.image.pullPolicy` | Define when Kubernetes has to pull a Docker image                                                        | `IfNotPresent`               |
| `admin-ui.service.port`     | TCP port where is going to listen the internal service                                                   | `5000`                       |
| `admin-ui.tls.enabled`      | If we want to enable HTTPS access to the UI. For this Cert Manager is required in the Kuberentes cluster | `false`                      |
| `admin-ui.host`             | Public hostname to generate SSL certificate with Cert Manager                                            | `false`                      |


## MongoDB

Specific configuration for the MongoDB on the Engine.

| Parameter                 | Description                                                 | Default    |
| ------------------------- | ----------------------------------------------------------- | ---------- |
| `mongodb.mongodbDatabase` | Database to create                                          | `localKRE` |
| `mongodb.mongodbUsername` | MongoDB custom user (mandatory if `mongodbDatabase` is set) | `admin`    |
| `mongodb.mongodbPassword` | MongoDB custom user password                                | `123456`   |
| `mongodb.image.tag`       | MongoDB version 4.2                                         | `3.6`      |



## Cert Manager

As described on previous sections of this guide KRE use Cert Manager to manage the lifecycle of the certificates to 
add a security layer on the comunication from outside.

| Parameter                 | Description                                                                            | Default                                  |
| ------------------------- | -------------------------------------------------------------------------------------- | ---------------------------------------- |
| `certManager.enabled`     | Enable Cert Manager to validate certificates                                           | `false`                                  |
| `certManager.acme.server` | Default certificate authority server to validate certificates, more instructions below | `acme-v02.api.letsencrypt.org/directory` |
| `certManager.acme.email`  | Default email for the certificate owner                                                | `user@email.com`                         |

You can fill in the field `certManager.acme.server` with one of the following values depend of your environment:

**Production environment**
```
  certManager:
    acme:
        server: https://acme-v02.api.letsencrypt.org/directory
```
Rate limit of 50 per day on certificates request with a week block if the limit is passed.[+ info](https://letsencrypt.org/docs/rate-limits/)

No web-browser action required.

**Staging environment** 
```
  certManager:
    acme:
        server: https://acme-staging-v02.api.letsencrypt.org/directory
```
Rate limit of 1500 each three hours on certificates request.[+ Info](https://letsencrypt.org/docs/staging-environment/)



This option needs the following action from user to set-up the staging certification authority.

**How add the fake certificate on chrome**

- Download the certificate [Fake Certificate](https://letsencrypt.org/certs/fakeleintermediatex1.pem)
- Go to settings -> Search Certificates -> Manage Certificates -> Issuers Entities
- Import the previous certificate.
- Enable the first option.
- Reload the https://admin.<your-domain> page
- You have a certificate for any kre domain.

Regarding the challenge method used for validation of the CSR we are going to describe two kinds supported by Cert Manager, but 
may be there are another methods that fits better for your environment, please refer to the [Cert Manager documentation](https://cert-manager.io/docs/configuration/) in order to 
create a more accurate configuration.

### DNS01

If you hosted the subdomain `kre.yourdomain.com` in Route53 is posible to configure the validation of the CSR with DNS via a 
Route53 plugin. Below there is a snippet of config to add to your `values.yaml` to configure this challenge. This is very
usefull in deployments that are behind a firewall with restricted access from Internet or just in a private network.

| Parameter                 | Description                                                 | Default    |
| ------------------------- | ----------------------------------------------------------- | ---------- |
| `certManager.dns01.route53.region` | AWS Region where the hosted zome is created        | `<not_defined>` |
| `certManager.dns01.route53.hostedZoneID` | AWS Hosted Zone ID                           | `<not_defined>` |
| `certManager.dns01.route53.accessKeyID` | AWS Access Key ID                             | `<not_defined>` |
| `certManager.dns01.route53.secretAccessKey` | AWS Access Secret Key                     | `<not_defined>` |

### HTTP01

When KRE is open to Internet and you can not configure your subdomain to be hosted in Route53 you can use this challenge.
The validation is done just with a HTTP request from the Let's Encrypt servers. Just need to set as enable.

| Parameter                 | Description                                                 | Default    |
| ------------------------- | ----------------------------------------------------------- | ---------- |
| `certManager.http01.enabled` | Enable the HTTP01 challenge        | `<not_defined>` |

## Prometheus

| Parameter                 | Description                                                 | Default    |
| ------------------------- | ----------------------------------------------------------- | ---------- |
| `prometheusOperator.enabled` | This parameter indicate if you want that when KRE is deployed also deploy Prometheus Operator, may be you have already deployed this on your cluster and is not required to be installed. | `true` |
| `prometheusOperator.grafana.enabled` | Disable the installation of Grafana with the Prometheus Operator | `true` |
