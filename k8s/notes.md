---
layout: page
title:  Kubernetes in Action
permalink: /k8s/book-in-action
---

This pages summarizes my notes for the book [Kubernetes in
Action](https://www.manning.com/books/kubernetes-in-action-second-edition) by
[Marko Lukša](https://twitter.com/markoluksa).

Table of content:

* Pods ✓
* Replication ✓
* Services
* Volumes
* ConfigMaps and Secrets
* Communiation with application
* Deployments
* StatefulSets
* Internals
* Securing API
* Securing pods and network
* Managing ressources
* Scaling
* Scheduling
* Best Practice
* Extending K8S

### Pods

A **pod** is a co-located group of containers and never spans multiple worker node. This means that all containers of a pod will always run on the same worker node. Kubernetes scales whole pods.

Containers are designed to run only a single process per container. Pods are the abstraction  that allow to bind containers together and manage them as a single unit.

All containers of a pod share the same set of Linux namespaces instead of each container having its own set. They thereby all share the same hostname and network interfaces. The processes running in containers of the same pod need to not to bind to the same port numbers. 

All pods in a Kubernetes cluster reside in a single flat, shared, network-address space.

Containers should be grouped into pods when:

* They need to be run together 
* They represent a single whole 
* They must be scaled together

A pod **definition** contains: 

* *Metadata* that includes the name, namespace, labels, etc.
* *Spec* that contains the actual description of the pod’s contents (e.g. containers, volumes, etc.)
* *Status* that contains the current information about the running pod (e.g. condition of the pod, the description and status of each container, internal IP, etc.)

Specifying ports in the pod definition is purely informational but is considered a good practice.

If a pod includes multiple containers, the container name needs be explicitly specified.

Kubernetes allows to configure **port forwarding** to the pod. This is done through the `kubectl port-forward` command. 

Organizing pods and all other Kubernetes objects is done through **labels**. A label is an arbitrary key-value pair you attach to a resource, which is then utilized when selecting resources using label selectors.

A good practice would be to tag pods with 

* app, which specifies which app, component, or microservice the pod belongs to. (e.g. `app=ui`, `app=as` for *account service*, etc.)
* rel, which shows whether the application running in the pod is a stable, beta,
or a canary release. (e.g. `rel=stable`, `rel=beta`, etc.)

A **selector** can also include multiple comma-separated criteria. Resources need to match all of them to match the selector.

**Annotations** are also key-value pairs but they aren’t meant to hold identifying information. Good practice is to have descriptions for each pod.

Kubernetes **namespaces** provide a scope for objects names. They can also be used for separating resources in a multi-tenant environment. Resource names only need to be unique within a namespace.

Namespaces enable you to separate resources that don’t belong together, e.g. several users sharing the same Kubernetes cluster. They each manage their own distinct set of resources, they should each use their own namespace.

Namespace don’t provide any kind of isolation of running objects. Whether namespaces provide network isolation depends on which networking solution is deployed with Kubernetes. 

### Replication

If a node fails, the pods on the node are lost and will not be replaced with new ones, unless those pods are managed by a ReplicaSets or similar. Always create ReplicaSets instead of ReplicationControllers.

Kubernetes can check if a container is still alive through **liveness probes**.

Kubernetes also supports **readiness probes**. Always remember to set an initial delay to account for your app’s startup time.

Kubernetes can probe a container with:

* An *HTTP GET* probe performs an HTTP GET request on the container’s IP address, a port and path you specify. Make sure the /health HTTP endpoint doesn’t require authentication.
* A *TCP* Socket probe tries to open a TCP connection to the specified port of the container.
* An *Exec* probe executes an arbitrary command inside the container and checks the command’s exit status code.

Implementing your own retry loop into the probe is wasted effort.

A **ReplicaSet** is a Kubernetes resource that ensures its pods are always kept running. It constantly monitors the list of running pods (pods that match a certain label selector) and makes sure the actual number of pods of a “type” always matches the desired number. Its job is to make sure that an exact number of pods always matches its label selector.

A **ReplicationController** behaves exactly like a ReplicaSet, but it has less expressive pod selectors.

A ReplicaSet is composed by

* A label *selector*, which determines what pods are in the ReplicationController’s scope
* A replica *count*, which specifies the desired number of pods that should be running 
* A pod *template*, which is used when creating new pod replicas

The pod labels in the template must obviously match the label selector of the ReplicationController.

Notifications trigger the controller to check the actual number of pods and take appropriate action.

Pods created by a ReplicaSet aren’t tied to the ReplicaSet in any way. At any moment, a ReplicaSet manages pods that match its label selector. 

Removing a pod from the scope of the ReplicaSet makes it "detached". It can be useful for debugging when you want to keep your pod unmanaged temporarily.

When you delete a ReplicaSet through kubectl delete, the pods are also deleted.

To run a pod on each node, use *DeamonSets*. A DaemonSet makes sure it creates as many pods as there are nodes and deploys each one on its own node. A DaemonSet deploys pods to all nodes in the cluster, unless you specify labels.

ReplicaSets, and DaemonSets run continuous tasks that are never considered completed.
Job resource allows you to run a pod whose container isn’t restarted when the process running inside finishes successfully. 

To run a task in a pod, use *jobs*. Jobs may be configured to create more than one pod instance and run them in parallel or sequentially. A pod’s time can be limited by setting the `activeDeadlineSeconds` property in the pod spec. If the pod runs longer than that, the system will try to terminate it and mark the Job as failed.

To run jobs at regular intervals or specific time, use *CronJobs*. The schedule for running the job is specified with cron format. You may have a hard requirement for the job to not be started too far over the scheduled time, if so specify a deadline with the `startingDeadlineSeconds` field.

