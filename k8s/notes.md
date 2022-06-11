---
layout: page
title:  Kubernetes in Action
permalink: /k8s/book-in-action
---

This pages summarizes my notes for the book [Kubernetes in
Action](https://www.manning.com/books/kubernetes-in-action-second-edition) by
[Marko Lukša](https://twitter.com/markoluksa).

**Part II: Core Concepts**

* ✅ Pods
* ✅ Replication
* ✅ Services
* ✅ Volumes
* ⌛ ConfigMaps and Secrets 
* ⌛ Communiation with application
* ⌛ Deployments
* ⌛ StatefulSets

**Part III: Beyond the basics**

* Internals
* Securing API
* Securing pods and network
* Managing ressources
* Scaling
* Scheduling
* Best Practice
* Extending K8S

Legend:
* ✅ Notes available below
* ⌛ Chapter read and notes in progress

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

The `kubectl exec` command allows you to remotely run arbitrary commands inside an existing container of a pod.

A **sidecar** container is a container that augments the operation of the main container of the pod. 


### Replication

If a node fails, the pods on the node are lost and will not be replaced with new ones, unless those pods are managed by a ReplicaSets or similar. Always create ReplicaSets instead of ReplicationControllers.

Kubernetes can check if a container is still alive through **liveness probes**.

Kubernetes can probe a container with:

* An *HTTP GET* probe performs an HTTP GET request on the container’s IP address, a port and path you specify. Make sure the /health HTTP endpoint doesn’t require authentication.
* A *TCP* Socket probe tries to open a TCP connection to the specified port of the container.
* An *Exec* probe executes an arbitrary command inside the container and checks the command’s exit status code.

Kubernetes allows you to also define a **readiness probe** for your pod. Three types of readiness probes as for liveness probes. Readiness probes are also useful when rolling out new deployments, as a bad pod will be prevented to take over all the running ones if its readiness probe fails.

Unlike liveness probes, if a container fails the readiness check, it won’t be killed or restarted. The readiness probe is checked periodically—every 10 seconds by default.

You should always define a readiness probe, even if it’s as simple as sending an HTTP request to the base URL.

When a pod is being shut down, the app running in it usually stops accepting connections as soon as it receives the termination signal. Because of this, you might think you need to make your readiness probe start failing as soon as the shutdown procedure is initiated.

Implementing your own retry loop into the probe is wasted effort.

A **ReplicaSet** is a Kubernetes resource that ensures its pods are always kept running. It constantly monitors the list of running pods (pods that match a certain label selector) and makes sure the actual number of pods of a “type” always matches the desired number. Its job is to make sure that an exact number of pods always matches its label selector.

Changing a ReplicaSet's pod template has no effect on existing pods.

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


### Services

Pods need a way of finding other pods. Specifying the exact IP address or hostname wouldn’t work, because

* Pods are ephemeral
* Kubernetes assigns an IP address to a pod after the pod has been scheduled to a node and before it’s started

A Kubernetes **Service** is a resource you create to make a single, constant point of entry to a group of pods providing the same service. Each service has an IP address and port that never change while the service exists. 

Services can also support multiple ports. 

You can also give a name to each pod’s port and refer to it by name in the service spec. The biggest benefit of doing so is that it enables you to change port numbers later without having to change the service spec.

Connections to the service are load-balanced across all the backing pods. You want all requests made by a certain client to be redirected to the same pod every time, you can set the service’s sessionAffinity property to ClientIP. Kubernetes supports only two types of service session affinity: None and ClientIP.

```
apiVersion: v1
kind: Service
metadata:
  name: [SERVICE_NAME]
spec:
  ports:
  port: 80
    targetPort: 8080
  selector:
app: [APP_NAME]
```

The Kubernetes service proxy intercept the connection, select a random pod among the ones matching the selector, and forward the request to it.

When a pod is started, Kubernetes initializes a set of environment variables pointing to each service that exists at that moment.

Kubernetes runs a DNS server, which all other pods running in the cluster are automatically configured to use. Each service gets a DNS entry in the internal DNS server, and client pods that know the name of the service can access it through its fully qualified domain name (FQDN) instead of resorting to environment variables. You can omit the `svc.cluster.local` suffix and even the namespace, when the pods are in the same namespace. 

`curl`-ing the service works, but pinging it doesn’t. That’s because the service’s cluster IP is a virtual IP, and only has meaning when combined with the service port.

To expose external services through the Kubernetes services feature, you can use endpoints. Instead of exposing an external service by manually configuring the service’s Endpoints, a simpler method allows you to refer to an external service by its fully qualified domain name (FQDN). To create a service that serves as an alias for an external service, you create a Service resource with the type field set to *ExternalName*. This hides the actual service name and its location from pods consuming the service, allowing you to modify the service definition and point it to a different service any time later, by only changing the externalName attribute. ExternalName services are implemented solely at the DNS level—a simple CNAME DNS record is created for the service. 

Services don’t link to pods directly. Instead, a resource sits in between—the Endpoints resource. An **Endpoints** resource is a list of IP addresses and ports exposing a service. Endpoints are a separate resource and not an attribute of a service. Because you created the service without a selector, the corresponding Endpoints resource hasn’t been created automatically. After both the Service and the Endpoints resource are posted to the server, the service is ready to be used like any regular service with a pod selector.

To make a service accessible externally:

* Setting the service type to NodePort—For a NodePort service.
* Setting the service type to LoadBalancer, an extension of the NodePort type
* Creating an Ingress resource, a radically different mechanism for exposing multiple services through a single IP address

By creating a **NodePort** service, you make Kubernetes reserve a port on all its nodes (the same port number is used across all of them) and forward incoming connections to the pods that are part of the service. It can be accessed not only through the service’s internal cluster IP, but also through any node’s IP and the reserved node port.

A connection received on one NodePort of a node might be forwarded to a pod running on another node. If you only point your clients to the first node, when that node fails, your clients can’t access the service anymore. 

Kubernetes clusters running on cloud providers usually support the automatic provision of a load balancer from the cloud infrastructure. If Kubernetes is running in an environment that doesn’t support **LoadBalancer** services, the load balancer will not be provisioned, but the service will still behave like a NodePort service.

When an external client connects to a service through the node port (this also includes cases when it goes through the load balancer first), the randomly chosen pod may or may not be running on the same node that received the connection. An additional network hop is required to reach the pod, but this may not always be desirable.

You can prevent this additional hop by configuring the service to redirect external traffic only to pods running on the node that received the connection with `externalTrafficPolicy` field.  If no local pods exist, the connection will hang.

When the connection is received through a node port, the packets’ source IP is changed, because Source Network Address Translation (SNAT) is performed on the packets.
The backing pod can’t see the actual client’s IP. The Local external traffic policy affects the preservation of the client’s IP. There is no additional hop between the node receiving the connection and the node hosting the target pod (SNAT isn’t performed).

**Ingresses** operate at the application layer of the network stack (HTTP) and vide features such as cookie-based session affinity. You need an Ingress controller running in your cluster, to create an Ingress resource.

Example: The client first performed a DNS lookup of kubia.example.com, and the DNS server (or the local operating system) returned the IP of the Ingress controller. The client then sent an HTTP request to the Ingress controller and specified kubia.example.com in the Host header. From that header, the controller determined which service the client is trying to access, looked up the pod IPs through the Endpoints object associated with the service, and forwarded the client’s request to one of the pods.

When a client opens a TLS connection to an Ingress controller, the controller terminates the TLS connection. The application running in the pod doesn’t need to support TLS. you need to attach a certificate and a private key to the Ingress. The two need to be stored in a Kubernetes resource called a Secret.

if you tell Kubernetes you don’t need a cluster IP for your service (you do this by setting the clusterIP field to None in the service specification), the DNS server will return the pod IPs instead of the single service IP.

Setting the clusterIP field in a service `spec` to `None` makes the service **headless**, as Kubernetes won’t assign it a cluster IP through which clients could connect to the pods backing it.

Services are a crucial Kubernetes concept and the source of frustration for many developers. I’ve seen many developers lose heaps of time figuring out why they can’t connect to their pods through the service IP or FQDN.

Troubleshooting tips:

* Make sure you’re connecting to the service’s cluster IP from within the cluster
* Don’t bother pinging
* Check readiness probe, make sure it’s succeeding
* Examine the corresponding Endpoints object with `kubectl get endpoints`.
* See if you can access it using its cluster IP instead of the FQDN.
* Check whether you’re connecting to the port exposed by the service and not the target port.
* Try connecting to the pod IP directly 
* Make sure your app isn’t only binding to localhost.


### Volumes

Every new container starts off with the exact set of files that was added to the image at build time. New container will not see anything that was written to the filesystem by the previous container, even though the newly started container runs in the same pod.

Kubernetes provides this by defining storage volumes. They aren’t top-level resources like pods, but are instead defined as a part of a pod and share the same lifecycle as the pod. This means a volume is created when the pod is started and is destroyed when the pod is deleted. Because of this, a volume’s contents will persist across container
restarts. 

Kubernetes volumes are a component of a pod and are thus defined in the pod’s specification. 
The several of the available volume types:

* `emptyDir` - A simple empty directory used for storing transient data.
* `hostPath` - Used for mounting directories from the worker node’s filesystem into the pod.
* `gitRepo` - A volume initialized by checking out the contents of a Git repository.
* `nfs` - An NFS share mounted into the pod.
* Cloud provider-specific storage.
* Other types of network storage.
* `configMap`, `secret`, `downwardAPI` - Special types of volumes used by K8S
* `persistentVolumeClaim` - A way to use a pre- or dynamically provisioned per-sistent storage.

The simplest volume type is the `emptyDir` volume. An `emptyDir` volume is especially useful for sharing files between containers running in the same pod. The `emptyDir` is created on the actual disk of the worker node hosting your pod, so its performance depends on the type of the node’s disks.

A `gitRepo` volume is basically an emptyDir volume that gets populated by cloning a Git repository and checking out a specific revision when the pod is starting up (but before its containers are created). If you want to clone a private Git repo into your container, you should use a *git-sync* sidecar or a similar method instead of a `gitRepo` volume.

Most pods should be oblivious of their host node, so they shouldn’t access any files on
the node’s filesystem. A `hostPath` volume points to a specific file or directory on the node’s filesystem. Pods running on the same node and using the same path in their `hostPath` volume see the same files. 

Don't use `hostPath` volume as the place to store a database’s data directory. Because the volume’s contents are stored on a specific node’s filesystem, when the database pod gets rescheduled to another node, it will no longer see the data. You should use other types of volumes, depending on the underlying infrastructure.

Including infrastructure-related information into a pod definition means the pod definition is pretty much tied to a specific Kubernetes cluster. You can’t use the same pod definition in another one. Thereby, volumes isn’t the best way to attach persistent storage to your pods. 

Ideally, a developer deploying their apps on Kubernetes should never have to know what kind of storage technology is used underneath, like they don't know the type of physical servers used to run their pods.

**PersistentVolumes** and **PersistentVolumeClaims** enable apps to request storage in a Kubernetes cluster without having to deal with infrastructure specifics.

Key steps:

1. Cluster admin sets up some type of network storage (NFS export or similar)
2. Admin then creates a PersistentVolume (PV) by posting a PV descriptor to the Kubernetes API
3. User creates a PersistentVolumeClaim (PVC)
4. Kubernetes ﬁnds a PV of adequate size and access mode and binds the PVC to the PV
5. User creates a pod with a volume referencing the PVC

A PersistentVolumeClaim manifest specify the minimum size and the access mode they require. 

When creating a PersistentVolume, the administrator needs to tell Kubernetes what its capacity is and whether it can be read from and/or written to by a single node or by multiple nodes at the same time. They also need to tell Kubernetes what to do with the PersistentVolume when it’s released.

Claiming a PersistentVolume is a completely separate process from creating a pod, because you want the same PersistentVolumeClaim to stay available even if the pod is rescheduled.

The only way to manually recycle the PersistentVolume to make it available again is to delete and recreate the PersistentVolume resource. 

Two other possible reclaim policies exist: `Recycle` and `Delete`. The first one deletes
the volume’s contents and makes the volume available to be claimed again.

PersistentVolume still requires a cluster administrator to provision the actual storage up front. Kubernetes can also perform this job. Kubernetes includes provisioners for the most popular cloud providers. If Kubernetes is deployed on-premises, a custom provisioner needs to be deployed.

Before a user can create a PersistentVolumeClaim, which will result in a new PersistentVolume being provisioned, an admin needs to create one or more **StorageClass** resources. StorageClasses are refered by name. The PVC definitions are therefore portable across different clusters, as long as the StorageClass names are the same across all of them. 
PersistentVolumeClaim can also specifies the class of storage you want to use. 