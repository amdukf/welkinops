# Import redis rdb backup when aof is active

To import an RDB backup file into a Sentinel-replicated Redis setup, first reduce the replica count to 1. Then disable AOF.

If Redis is deployed via a Helm chart, it is recommended to disable AOF in the Helm configuration. After that, copy the RDB file into the /data directory of the master pod and roll out the Redis StatefulSet (Delete the pod).

Verify the keys and check DBSIZE. If everything looks correct, increase the replica count again.


Here is an example of redis bitnami chart with topolvm csi:

Chart.yaml:
```
apiVersion: v2
name: redis-ha
description: "High available redis cluster"
version: 0.1.0
type: application
dependencies:
- name: redis
  version: 18.x.x
  repository: oci://registry-1.docker.io/bitnamicharts
  condition: redis.enabled
```

values.yaml
```
redis:
  enabled: true
  global:
    storageClass: local-path
    imageRegistry: "repo.example.com"
    imagePullSecrets:
      - regcred

  image:
    registry: docker.io/bitnamilegacy
    repository: redis
    tag: 8.2.1-debian-12-r0

  clusterDomain: cluster.local

  architecture: replication

  auth:
    enabled: true
    sentinel: false
    existingSecret: redis-secret
    existingSecretPasswordKey: redis-password

    acl:
      enabled: true
      sentinel: false
      userSecret: redis-secret
      users:
        - username: "bourse"
          enabled: "on"
          commands: "+@all -@admin -@dangerous +keys +info"
          keys: "~*"
          channels: "&*"

  commonConfiguration: |-
    appendonly yes
    appendfsync everysec
    save ""

  replica:
    replicaCount: 4
    disableCommands:
      - FLUSHDB
      - FLUSHALL
    persistence:
      enabled: true
      size: 20Gi
      storageClass: topolvm-provisioner
    resources:
      limits: {}
      requests: {}
    nodeSelector:
      redis: "true"
    podAntiAffinityPreset: hard
    tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "redis"
        effect: "NoSchedule"

  sentinel:
    image:
      registry: docker.io/bitnamilegacy
      repository: redis-sentinel
      tag: 8.2.1-debian-12-r0
    enabled: true
    masterSet: ada-master
    quorum: 2
    persistence:
      enabled: true
      size: 1Gi
      storageClass: topolvm-provisioner
    nodeSelector:
      redis: "true"
    podAntiAffinityPreset: hard
    tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "redis"
        effect: "NoSchedule"

  metrics:
    image:
      registry: docker.io/bitnamilegacy
      repository: redis-exporter
      tag: 1.76.0-debian-12-r0
    enabled: true
    resources:
      limits: {}
      requests: {}
    serviceMonitor:
      enabled: true
      namespace: monitoring
      additionalLabels:
        release: monitoring
    podMonitor:
      enabled: true
      namespace: monitoring
      additionalLabels:
        release: monitoring
    prometheusRule:
      enabled: true
      namespace: monitoring
      additionalLabels:
        release: monitoring

```