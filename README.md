# Redis

[Redis](http://redis.io/) is an advanced key-value cache and store. It is often referred to as a data structure server since keys can contain strings, hashes, lists, sets, sorted sets, bitmaps and hyperloglogs.


By default this chart install 3 pods total:
 * one pod containing a redis master, sentinel containers and a prometheus exporter.
 * two pods each containing redis slave, sentinel containers and a prometheus exporter.

## Introduction

This chart bootstraps a [Redis](https://redis.io) highly available master/slave statefulset in a [Kubernetes](http://kubernetes.io) cluster using the Helm package manager.

## Prerequisites

- Kubernetes 1.8+ with Beta APIs enabled
- PV provisioner support in the underlying infrastructure


## Installing the Chart

To install the chart

```bash
$ helm install . -f=values.yaml
```

The command deploys Redis on the Kubernetes cluster in the default configuration. By default this chart install one master pod containing redis master container and sentinel container along with 2 redis slave pods each containing their own sentinel sidecars. The [configuration](#configuration) section lists the parameters that can be configured during installation.


## Uninstalling the Chart

To uninstall/delete the deployment:

```bash
$ helm del <chart-name>
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following table lists the configurable parameters of the Redis chart and their default values.

| Parameter                        | Description                                                                                                                  | Default                                                   |
| -------------------------------- | -----------------------------------------------------                                                                        | --------------------------------------------------------- |
| `image`                          | Redis image                                                                                                                  | `redis`                                                   |
| `tag`                            | Redis tag                                                                                                                    | `5.0.3-alpine`                                          |
| `replicas`                       | Number of redis master/slave pods                                                                                            | `3`                                                       |
| `redis.port`                     | Port to access the redis service                                                                                             | `6379`                                                    |
| `redis.masterGroupName`          | Redis convention for naming the cluster group                                                                                | `mymaster`                                                |
| `redis.config`                   | Any valid redis config options in this section will be applied to each server (see below)                                    | see values.yaml                                           |
| `redis.customConfig`             | Allows for custom redis.conf files to be applied. If this is used then `redis.config` is ignored                             | ``                                                        |
| `redis.resources`                | CPU/Memory for master/slave nodes resource requests/limits                                                                   | `{}`                                                      |
| `sentinel.port`                  | Port to access the sentinel service                                                                                          | `26379`                                                   |
| `sentinel.quorum`                | Minimum number of servers necessary to maintain quorum                                                                       | `2`                                                       |
| `sentinel.config`                | Valid sentinel config options in this section will be applied as config options to each sentinel (see below)                 | see values.yaml                                           |
| `sentinel.customConfig`          | Allows for custom sentinel.conf files to be applied. If this is used then `sentinel.config` is ignored                       | ``                                                        |
| `sentinel.resources`             | CPU/Memory for sentinel node resource requests/limits                                                                        | `{}`                                                      |
| `auth`                           | Enables or disables redis AUTH (Requires `redisPassword` to be set)                                                          | `false`                                                   |
| `redisPassword`                  | A password that configures a `requirepass` and `masterauth` in the conf parameters (Requires `auth: enabled`)                | ``                                                        |
| `existingSecret`                  | An existing secret containing an `auth` key that configures `requirepass` and `masterauth` in the conf parameters (Requires `auth: enabled`, cannot be used in conjunction with `.Values.redisPassword`)                | ``                                                        |
| `nodeSelector`                   | Node labels for pod assignment                                                                                               | `{}`                                                      |
| `tolerations`                    | Toleration labels for pod assignment                                                                                         | `[]`                                                      |
| `podAntiAffinity.server`         | Antiaffinity for pod assignment of servers, `hard` or `soft`                                                                 | `Hard node and soft zone anti-affinity`                   |
| `exporter.enabled`               | Enable prometheus exporter                                                                                                   | `true`                                                    |
| `exporter.image.repository`      | Exporter image                                                                                                               | `oliver006/redis_exporter`                                |
| `exporter.image.tag`             | Exporter tag                                                                                                                 | `v0.25.0`                                                 |
| `exporter.pullPolicy`            | Image PullPolicy                                                                                                             | `IfNotPresent`                                            |
| `exporter.annotations`           | Annotations for prometheus service discovery                                                                                 |  `{}`                                                     |
| `exporter.annotations.extraArgs` | Extra args for redis exporter                                                                                                | `{}`                                                      |

## Custom Redis and Sentinel config options

This chart allows for most redis or sentinel config options to be passed as a key value pair through the `values.yaml` under `redis.config` and `sentinel.config`. See links below for all available options.

[Example redis.conf](http://download.redis.io/redis-stable/redis.conf)
[Example sentinel.conf](http://download.redis.io/redis-stable/sentinel.conf)

For example `repl-timeout 60` would be added to the `redis.config` section of the `values.yaml` as:

```yml
    repl-timeout: "60"
```

Sentinel options supported must be in the the `sentinel <option> <master-group-name> <value>` format. For example, `sentinel down-after-milliseconds 30000` would be added to the `sentinel.config` section of the `values.yaml` as:

```yml
    down-after-milliseconds: 30000
```

If more control is needed from either the redis or sentinel config then an entire config can be defined under `redis.customConfig` or `sentinel.customConfig`. Please note that these values will override any configuration options under their respective section. For example, if you define `sentinel.customConfig` then the `sentinel.config` is ignored.
