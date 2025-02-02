# Harbor Scanner Trivy

Trivy as a plug-in vulnerability scanner in the Harbor registry.

## TL;DR;

```
$ helm repo add aqua https://helm.aquasec.com
```

### Without TLS

```
$ helm install harbor-scanner-trivy aqua/harbor-scanner-trivy \
    --namespace harbor
```

### With TLS

1. Generate certificate and private key files:
   ```
   $ openssl genrsa -out tls.key 2048
   $ openssl req -new -x509 \
       -key tls.key \
       -out tls.crt \
       -days 365 \
       -subj /CN=harbor-scanner-trivy.harbor
   ```
2. Install the `harbor-scanner-trivy` chart:
   ```
   $ helm install harbor-scanner-trivy aqua/harbor-scanner-trivy \
       --namespace harbor \
       --set service.port=8443 \
       --set scanner.api.tlsEnabled=true \
       --set scanner.api.tlsCertificate="`cat tls.crt`" \
       --set scanner.api.tlsKey="`cat tls.key`"
   ```

## Introduction

This chart bootstraps a scanner adapter deployment on a [Kubernetes](http://kubernetes.io) cluster using the
[Helm](https://helm.sh) package manager.

## Prerequisites

- Kubernetes 1.12+
- Helm 2.11+ or Helm 3+

## Installing the Chart

To install the chart with the release name `my-release`:

```
$ helm install my-release aqua/harbor-scanner-trivy
```

The command deploys scanner adapter on the Kubernetes cluster in the default configuration. The [Parameters](#parameters)
section lists the parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`.

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```
$ helm delete my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Parameters

The following table lists the configurable parameters of the scanner adapter chart and their default values.

| Parameter                             | Description                                                                                                                                                                                                                                                                        | Default                            |
|---------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------|
| `image.registry`                      | Image registry                                                                                                                                                                                                                                                                     | `docker.io`                        |
| `image.repository`                    | Image name                                                                                                                                                                                                                                                                         | `aquasec/harbor-scanner-trivy`     |
| `image.tag`                           | Image tag                                                                                                                                                                                                                                                                          | `{TAG_NAME}`                       |
| `image.pullPolicy`                    | Image pull policy                                                                                                                                                                                                                                                                  | `IfNotPresent`                     |
| `replicaCount`                        | Number of scanner adapter Pods to run                                                                                                                                                                                                                                              | `1`                                |
| `scanner.logLevel`                    | The log level of `trace`, `debug`, `info`, `warn`, `warning`, `error`, `fatal` or `panic`. The standard logger logs entries with that level or anything above it                                                                                                                   | `info`                             |
| `scanner.api.tlsEnabled`              | The flag to enable or disable TLS for HTTP                                                                                                                                                                                                                                         | `true`                             |
| `scanner.api.tlsCertificate`          | The absolute path to the x509 certificate file                                                                                                                                                                                                                                     |                                    |
| `scanner.api.tlsKey`                  | The absolute path to the x509 private key file                                                                                                                                                                                                                                     |                                    |
| `scanner.api.readTimeout`             | The maximum duration for reading the entire request, including the body                                                                                                                                                                                                            | `15s`                              |
| `scanner.api.writeTimeout`            | The maximum duration before timing out writes of the response                                                                                                                                                                                                                      | `15s`                              |
| `scanner.api.idleTimeout`             | The maximum amount of time to wait for the next request when keep-alives are enabled                                                                                                                                                                                               | `60s`                              |
| `scanner.trivy.cacheDir`              | Trivy cache directory                                                                                                                                                                                                                                                              | `/home/scanner/.cache/trivy`       |
| `scanner.trivy.reportsDir`            | Trivy reports directory                                                                                                                                                                                                                                                            | `/home/scanner/.cache/reports`     |
| `scanner.trivy.debugMode`             | The flag to enable or disable Trivy debug mode                                                                                                                                                                                                                                     | `false`                            |
| `scanner.trivy.vulnType`              | Comma-separated list of vulnerability types. Possible values are `os` and `library`.                                                                                                                                                                                               | `os,library`                       |
| `scanner.trivy.ignorepolicy`          | The OPA rego script used by Trivy to evaluate each vulnerability                                                                                                                                                                                                                   | `     `                            |
| `scanner.trivy.severity`              | Comma-separated list of vulnerabilities severities to be displayed                                                                                                                                                                                                                 | `UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL` |
| `scanner.trivy.ignoreUnfixed`         | The flag to display only fixed vulnerabilities                                                                                                                                                                                                                                     | `false`                            |
| `scanner.trivy.timeout`               | The duration to wait for scan completion                                                                                                                                                                                                                                           | `5m0s`                             |
| `scanner.trivy.skipUpdate`            | The flag to enable or disable Trivy DB downloads from GitHub                                                                                                                                                                                                                       | `false`                            |
| `scanner.trivy.offlineScan`           | The flag to disable external API requests to identify dependencies                                                                                                                                                                                                                 | `false`                            |
| `scanner.trivy.gitHubToken`           | The GitHub access token to download Trivy DB                                                                                                                                                                                                                                       |                                    |
| `scanner.trivy.insecure`              | The flag to skip verifying registry certificate                                                                                                                                                                                                                                    | `false`                            |
| `scanner.store.redisNamespace`        | The namespace for keys in the Redis store                                                                                                                                                                                                                                          | `harbor.scanner.trivy:store`       |
| `scanner.store.redisScanJobTTL`       | The time to live for persisting scan jobs and associated scan reports                                                                                                                                                                                                              | `1h`                               |
| `scanner.jobQueue.redisNamespace`     | The namespace for keys in the scan jobs queue backed by Redis                                                                                                                                                                                                                      | `harbor.scanner.trivy:job-queue`   |
| `scanner.jobQueue.workerConcurrency`  | The number of workers to spin-up for a jobs queue                                                                                                                                                                                                                                  | `1`                                |
| `scanner.redis.poolURL`               | The Redis server URI. The URI supports schemas to connect to a standalone Redis server, i.e. `redis://:password@standalone_host:port/db-number` and Redis Sentinel deployment, i.e. `redis+sentinel://:password@sentinel_host1:port1,sentinel_host2:port2/monitor-name/db-number`. |
| `scanner.redis.poolMaxActive`         | The max number of connections allocated by the Redis connection pool                                                                                                                                                                                                               | `5`                                |
| `scanner.redis.poolMaxIdle`           | The max number of idle connections in the Redis connection pool                                                                                                                                                                                                                    | `5`                                |
| `scanner.redis.poolIdleTimeout`       | The duration after which idle connections to the Redis server are closed. If the value is zero, then idle connections are not closed.                                                                                                                                              | `5m`                               |
| `scanner.redis.poolConnectionTimeout` | The timeout for connecting to the Redis server                                                                                                                                                                                                                                     | `1s`                               |
| `scanner.redis.poolReadTimeout`       | The timeout for reading a single Redis command reply                                                                                                                                                                                                                               | `1s`                               |
| `scanner.redis.poolWriteTimeout`      | The timeout for writing a single Redis command                                                                                                                                                                                                                                     | `1s`                               |
| `service.type`                        | Kubernetes service type                                                                                                                                                                                                                                                            | `ClusterIP`                        |
| `service.port`                        | Kubernetes service port                                                                                                                                                                                                                                                            | `8080`                             |
| `httpProxy`                           | The URL of the HTTP proxy server                                                                                                                                                                                                                                                   |                                    |
| `httpsProxy`                          | The URL of the HTTPS proxy server                                                                                                                                                                                                                                                  |                                    |
| `noProxy`                             | The URLs that the proxy settings do not apply to                                                                                                                                                                                                                                   |                                    |

The above parameters map to the env variables defined in [harbor-scanner-trivy](https://github.com/aquasecurity/harbor-scanner-trivy#configuration).

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`.

```
$ helm install my-release aqua/harbor-scanner-trivy \
    --namespace my-namespace \
    --set "service.port=9090" \
    --set "scanner.trivy.vulnType=os\,library"
```
