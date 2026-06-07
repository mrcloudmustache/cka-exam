# Cluster Architecture, Installation & Configuration

## Upgrade Control node

View the cluster components current versions

```
kubeadm upgrade plan

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE      CURRENT   TARGET
kubelet     control   v1.35.3   v1.35.5
kubelet     worker1   v1.35.3   v1.35.5
kubelet     worker2   v1.35.3   v1.35.5

Upgrade to the latest version in the v1.35 series:

COMPONENT                 NODE      CURRENT   TARGET
kube-apiserver            control   v1.35.3   v1.35.5
kube-controller-manager   control   v1.35.3   v1.35.5
kube-scheduler            control   v1.35.3   v1.35.5
kube-proxy                          1.35.3    v1.35.5
CoreDNS                             v1.13.1   v1.13.1
etcd                      control   3.6.6-0   3.6.6-0
```

### Switch to another Kubernetes package repository

```
sudo vim /etc/apt/sources.list.d/kubernetes.list
```

Update to the next available minor version

```
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35<----UPDATE/deb/
```

### Find the lastest verion to update to

```
sudo apt update
sudo apt-cache madison kubeadm

Fetched 14.3 MB in 2s (6,165 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
50 packages can be upgraded. Run 'apt list --upgradable' to see them.
   kubeadm | 1.35.5-1.1 | https://pkgs.k8s.io/core:/stable:/v1.35/deb  Packages
   kubeadm | 1.35.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.35/deb  Packages
   kubeadm | 1.35.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.35/deb  Packages
   kubeadm | 1.35.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.35/deb  Packages
   kubeadm | 1.35.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.35/deb  Packages
   kubeadm | 1.35.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.35/deb  Packages
```

### Upgrade kubeadm

Install updated packages

```
# replace x in 1.36.x-* with the latest patch version
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.35.5' && \
sudo apt-mark hold kubeadm
```

### View the updated version of kubeadm

```
kubeadm verison
``` 

### Upgrade Cluster components

```
sudo kubeadm upgrade apply v1.35.5
```

## Upgrade Worker nodes


### Upgrade kubeadmn

Install updated packages

```
# replace x in 1.36.x-* with the latest patch version
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.35.5' && \
sudo apt-mark hold kubeadm
```

Run the upgrade

```
sudo kubeadm upgrade node
```

### Drain the node

```
kubectl drain worker1 --ignore-daemonsets
```

### Upgrade kubelet and kubectl

```
# replace x in 1.36.x-* with the latest patch version
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.35.5' kubectl='1.35.5' && \
sudo apt-mark hold kubelet kubectl
```

### Restart kubelet

```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### Uncordon the node

```
kubectl uncordon worker1
```

## Scheduling

### Taints and Tolerations

Taint worker1

value is optional.

```
kubectl taint node worker1 dedicated=special-user:NoSchedule
```

Remove taint with no value from worker1

```
kubectl taint node worker1 dedicated=:NoSchedule-
```

Add Toleration to depolyment so that pods tolerate the taint on worker1 and possibly scheduled onto the node.

Operator
* Equal - must enter key, value and effect
* Exists - must enter only key and effect

Effects
* NoSchedule - This means that no pod will be able to schedule onto a node unless it has a matching toleration.
* PreferNoSchedule - The control plane will try to avoid placing a Pod that does not tolerate the taint on the node, but it is not guaranteed.
* NoExecute - Pods that do not tolerate the taint are evicted immediately

```
spec:
   containers:
      - .....
   # Equal example
   tolerations:
   - key: "dedicated"
     operator: "Equal"
     value: "special-user"
     effect: "NoSchedule"
   # Exists example
   tolerations:
   - key: "dedicated"
     operator: "Exists"
     effect: "NoSchedule"
```

## DaemonSets

Create Deployment imperatively and update to DaemonSet

```
kubectl create deployment fluentd --image=quay.io/fluentd_elasticsearch/fluentd:v5.0.1 -n kube-system --dry-run=client -o yaml > fluentd-ds.yaml
```

Change kind from Deployment to DaemonSet 

Remove replicas, strategy and status  

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: fluentd
  name: fluentd
  namespace: kube-system
spec:
  # replicas: 1
  selector:
    matchLabels:
      app: fluentd
  # strategy: {}
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - image: quay.io/fluentd_elasticsearch/fluentd:v5.0.1
        name: fluentd
        resources: {}
# status: {}
```

## Static Pods

Pods created and managed by kubelet. The control plane is viewing read-only versions of pod mirrors.

Deploys control plane components as static pods.

Add pod(not deployment, replicasets etc) definition files to the manifest directory.

kubelet config location

```
/var/lib/kubelet/config.yaml
```

Path to the manifest directory

```
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
```

View static pods without the API server

```
sudo crictl ps

CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD                                       NAMESPACE
2c0612bcb7a49       53ed370019059       5 days ago          Running             kube-proxy                0                   d3260da58a81c       kube-proxy-cghjx                          kube-system
b4e75f398b752       87c9b0e4f80d3       5 days ago          Running             kube-scheduler            0                   02df7a82650f2       kube-scheduler-control                    kube-system
7ce385752d77e       5e785d005ccc1       5 days ago          Running             calico-kube-controllers   29                  36b9056e394a7       calico-kube-controllers-b45f49df6-njks7   kube-system
f262f07f15879       0f2b96c93465f       5 days ago          Running             kube-apiserver            0                   a622195ca1b92       kube-apiserver-control                    kube-system
ff67c464b5276       0eb506280f9bc       5 days ago          Running             kube-controller-manager   1                   9e869696fd7fd       kube-controller-manager-control           kube-system
188d985460341       0a108f7189562       8 weeks ago         Running             etcd                      1                   a57755899f1fc       etcd-control                              kube-system
0e6c3268f9af4       08616d26b8e74       2 months ago        Running             calico-node               0                   6db91db2df2eb       calico-node-rfpsp   
```




