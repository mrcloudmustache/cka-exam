# cka-exam

## Helpful commands

### List all available API resources

Displays resource names, short names and API endpoints.

```
$ kubectl api-resources

NAME                                SHORTNAMES                        APIVERSION                        NAMESPACED   KIND
bindings                                                              v1                                true         Binding
componentstatuses                   cs                                v1                                false        ComponentStatus
configmaps                          cm                                v1                                true         ConfigMap
endpoints                           ep                                v1                                true         Endpoints

```

## kubectl

A CLI tool used to access Kubernetes objects.

kubeconfig is a configuration file used by kubectl to authentication and connect to Kubernetes clusters.

* clusters
* contexts
* users

kubeconfig file location ```~/.kube/config```

## Cluster components

* ```DaemonSets``` ensure a single pod is running on each node in the cluster. Kube proxy and CNI(calico) run as a DaemonSet.
* ```kubelet``` is a background Linux process. It's not managed by Kuburnetes.

View the kubelet service

```
$ sudo systemctl status kubelet
```

View kubelet logs

```
$ sudo journalctl -u kubelet
```

View containerd logs

```
$ sudo journalctl -u containerd
```

View pod logs in directory

```
$ sudo ls /var/log/pods/
```


### Control Plane nodes

Runs the API server, DNS and controller manager, scheduler, kube-proxy and etcd datastore.

View system pods on control node 

```
$ kubectl get pods -n kube-system
```

* coredns
* ETCD datastore - Stores all the Kubernetes configuration data. Losing the datastore is catastrophic.
* API server - Exposes the Kubernetes API and acts as the entry point in to the entire cluster.
* kube-proxy
* Sheduler - Shedules pods to run a select nodes based on taints, tolerations and available node resources.
* Controller manager - Manager of all other controllers such as Deployments and Replicasets. A controller matches the current state to the desired state.
* metrics-server (must be installed seperately)
* calico (CNI puligin installed seperately)

### Worker nodes

Runs application containers by receiving instructions from the control plane node via the kubelet systemctl service.

* Kube proxy - Responsible for allocating traffic to pods within a service.
* Kubelet - Makes sure containers are running in a pod. Can take corrective action to restart a failed container per the YAML specifications. Reports the status of pods and containers to the API server.
* Container runtime (containerd) - The engine that runs containers.


## Services and networking

To expose your application outside of the Kubernetes cluster you create an object called a service. 

Components

* ClusterIP
* NodePort
* LoadBalancer

### Ingress


### API Gateway

## Storage

In Kubernetes storage is decoupled from the ephemeral applications creating the persistence of data.

Components

* Persisant volumes
* Persistant volume claims
* Storage Classes
