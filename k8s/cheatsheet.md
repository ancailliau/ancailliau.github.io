---
layout: page
title:  kubectl cheat sheet
permalink: /k8s/cheatsheet
---

### Pods

| Description | Command |
| ----------- | ------- |
| Create a pod | `kubectl create -f [YAML_CONFIG]` |
| Get configuration of the pod | `kubectl get po [POD] -o (yaml|json)` |
| List the pods | `kubectl get pods` |
| Delete pod | `kubectl delete po [POD]` |
| Delete all pods of a namespace | `kubectl delete po --all` |
| Delete pods with a label | `kubectl delete po -l [LABEL(=VALUE)]` |

### Labels

| Description | Command |
| ----------- | ------- |
| List the pods with labels | `kubectl get po --show-labels` |
| List the pods with some labels in columns | `kubectl get po -L [LABELS]` |
| List the pods filtered by label (comma = AND combination) | `kubectl get po -l [LABEL]` |
| List the pods filtered by label value | `kubectl get po -l [LABEL=VALUE]` |
| List the pods filtered by label values | `kubectl get po -l [LABEL] in ([VALUES])` |
| List the pods excluding a label | `kubectl get po -l '![LABEL]'` |
| List the pods excluding a label value | `kubectl get po -l '[LABEL!=VALUE]'` |
| List the pods excluding a label value | `kubectl get po -l '[LABEL] notin ([VALUES])'`|
| Add a label to a pod | `kubectl label po [POD] [LABEL=VALUE]` |
| Change a label to a pod | `kubectl label po [POD] [LABEL=VALUE] --overwrite` |
| Add a label | `kubectl label node [NODE] [LABEL=VALUE]` |

### Annotations

| Description | Command |
| ----------- | ------- |
| Annotate a pod | `kubectl annotate pod [POD] [KEY=VALUE]` |
| Display annotations | `kubectl describe pod [POD]` |

#### Namespace

| Description | Command |
| ----------- | ------- |
| List | `kubectl get ns` |
| Create | `kubectl create namespace [NS]` |
| List pods of a namespace | `kubectl get po --namespace [NS]` |

### Logging

| Description | Command |
| ----------- | ------- |
| Get the logs of a pod | `kubectl logs [POD]` |
| Get the logs of a job | `kubectl logs [JOB]` |
| Get the logs of a container | `kubectl logs [POD] -c [CONTAINER_NAME]` |
| Get the logs of a crashed container | `kubectl logs mypod --previous` |

### Network

| Description | Command |
| ----------- | ------- |
| Setup port forwarding | `kubectl port-forward [POD] [LOCAL_POST]:[CONTAINER_PORT]` |

### ReplicationController

| Description | Command |
| ----------- | ------- |
| List Replication Controller | `kubectl get rc` |
| Scale | `kubectl scale rc [RC] --replicas=[INT]` |
| Delete but keep pods | `kubectl delete rc [RC] --cascade=false` |

### ReplicaSet

| Description | Command |
| ----------- | ------- |
| List Replication Controller | `kubectl get rs` |
| Scale | `kubectl scale rs [RC] --replicas=[INT]` |
| Delete but keep pods | `kubectl delete rs [RC] --cascade=false` |

### DeamonSet

| Description | Command |
| ----------- | ------- |
| List deamon sets | `kubectl get ds`

### Jobs

| Description | Command |
| ----------- | ------- |
| List jobs | `kubectl get jobs`
