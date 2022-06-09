---
title: Probes in K8S
tags: k8s devops
---

Types of probes

Probes for ASP.NET Core Application

Probes for Postgres

```yaml
livenessProbe:
  exec:
    command:
    - /bin/sh
    - -c
    - exec pg_isready -U "postgres" -h 127.0.0.1 -p 5432
  failureThreshold: 6
  initialDelaySeconds: 30
  periodSeconds: 10
  successThreshold: 1
  timeoutSeconds: 5
```


```yaml
readinessProbe:
  exec:
    command:
    - /bin/sh
    - -c
    - -e
    - |
      exec pg_isready -U "postgres" -h 127.0.0.1 -p 5432
      [ -f /opt/bitnami/postgresql/tmp/.initialized ] || [ -f /bitnami/postgresql/.initialized ]
  failureThreshold: 6
  initialDelaySeconds: 5
  periodSeconds: 10
  successThreshold: 1
  timeoutSeconds: 5
```

Probes for RabbitMQ

liveness

```yaml
livenessProbe:
  exec:
    command: ["rabbitmq-diagnostics", "status"]
  initialDelaySeconds: 60
  periodSeconds: 60
  timeoutSeconds: 15
```

readiness

```yaml
readinessProbe:
  exec:
    command: ["rabbitmq-diagnostics", "ping"]
  initialDelaySeconds: 20
  periodSeconds: 60
  timeoutSeconds: 10
```

More information on

* https://www.rabbitmq.com/monitoring.html#health-checks
* https://www.rabbitmq.com/monitoring.html