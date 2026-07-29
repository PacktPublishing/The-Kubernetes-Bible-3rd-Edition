# Installing Your First Kubernetes Cluster

- [Installing Your First Kubernetes Cluster](#installing-your-first-kubernetes-cluster)
  - [Installing minikube](#installing-minikube)
  - [minikube configurations](#minikube-configurations)
  - [Deploying Kubernetes using minikube](#deploying-kubernetes-using-minikube)
  - [Stop/Pause/Delete minikube clusters](#stoppausedelete-minikube-clusters)
  - [Multi-node Kubernetes using minikube](#multi-node-kubernetes-using-minikube)
  - [Multiple Kubernetes clusters using minikube](#multiple-kubernetes-clusters-using-minikube)
    - [minikube check profiles](#minikube-check-profiles)
  - [Installing Kind](#installing-kind)
  - [Creating cluster using kind](#creating-cluster-using-kind)

## Installing minikube

Linux

```shell
$ curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64

# Verify minikube command and path
$ which minikube
/usr/local/bin/minikube
```

macOS

```shell
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
# Verify minikube command and path
$ which minikube
```

Windows

```shell
# Using Windows Package Manager (if installed)
winget install Kubernetes.minikube

# Using Chocolatey
choco install minikube

# Or download the .exe file from https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe
```

## minikube configurations

```shell
$ minikube config set cpus 4
❗  These changes will take effect upon a minikube delete and then a minikube start

$ minikube config set memory 16000
❗  These changes will take effect upon a minikube delete and then a minikube start

$ minikube config set driver podman
❗  These changes will take effect upon a minikube delete and then a minikube start

$ minikube config view driver
- driver: podman
- rootless: false
```

## Deploying Kubernetes using minikube

On your workstation where you have installed minikube and VirtualBox, execute the following command.

```shell
$ minikube start --driver=virtualbox --memory=8000m --cpus=2
```

If you are using an old version of minikube but you want to install different version of Kubernetes version, then you can mention the specific version as follows.

```shell
$ minikube start --driver=virtualbox --memory=8000m --cpus=2 --kubernetes-version=1.36.3
```

Selecting correct driver:

```shell
😄  minikube v1.38.1 on Fedora 43
❗  Specified Kubernetes version 1.36.3 is newer than the newest supported version: v1.35.1. Use `minikube config defaults kubernetes-version` for details.
❗  Specified Kubernetes version 1.36.3 not found in Kubernetes version list
🤔  Searching the internet for Kubernetes version...
✅  Kubernetes version 1.36.3 found in GitHub version list
✨  Using the virtualbox driver based on user configuration
👍  Starting "minikube" primary control-plane node in "minikube" cluster
🔥  Creating virtualbox VM (CPUs=2, Memory=8000MB, Disk=20000MB) ...
📦  Preparing Kubernetes v1.36.3 on containerd 2.2.1 ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                   │
│    You have selected "virtualbox" driver, but there are better options !                          │
│    For better performance and support consider using a different driver:                          │
│            - kvm2                                                                                 │
│            - qemu2                                                                                │
│                                                                                                   │
│    To turn off this warning run:                                                                  │
│                                                                                                   │
│            $ minikube config set WantVirtualBoxDriverWarning false                                │
│                                                                                                   │
│                                                                                                   │
│    To learn more about on minikube drivers checkout https://minikube.sigs.k8s.io/docs/drivers/    │
│    To see benchmarks checkout https://minikube.sigs.k8s.io/docs/benchmarks/cpuusage/              │
│                                                                                                   │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

```shell
$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

Using container

```shell
$ minikube start --driver=podman --kubernetes-version=1.36.3
😄  minikube v1.38.1 on Fedora 43
❗  Specified Kubernetes version 1.36.3 is newer than the newest supported version: v1.35.1. Use `minikube config defaults kubernetes-version` for details.
❗  Specified Kubernetes version 1.36.3 not found in Kubernetes version list
🤔  Searching the internet for Kubernetes version...
✅  Kubernetes version 1.36.3 found in GitHub version list
✨  Using the podman driver based on user configuration
📌  Using Podman driver with root privileges
👍  Starting "minikube" primary control-plane node in "minikube" cluster
🚜  Pulling base image v0.0.50 ...
E0728 21:55:30.136370 1044574 cache.go:239] Error downloading kic artifacts:  not yet implemented, see issue #8426
🔥  Creating podman container (CPUs=2, Memory=8000MB) ...
📦  Preparing Kubernetes v1.36.3 on containerd 2.2.1 ...
🔗  Configuring CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

If driver not available,

```shell
$ minikube start --driver=docker
😄  minikube v1.38.1 on Fedora 43
✨  Using the docker driver based on user configuration

💣  Exiting due to PROVIDER_DOCKER_VERSION_EXIT_1: "docker version --format <no value>-<no value>:<no value>" exit status 1: failed to connect to the docker API at unix:///var/run/docker.sock; check if the path is correct and if the daemon is running: dial unix /var/run/docker.sock: connect: no such file or directory
📘  Documentation: https://minikube.sigs.k8s.io/docs/drivers/docker/
```
Kubeconfig:

```shell
$ cat ~/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/gineesh/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Tue, 28 Jul 2026 22:25:46 +08
        provider: minikube.sigs.k8s.io
        version: v1.38.1
      name: cluster_info
    server: https://192.168.49.2:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Tue, 28 Jul 2026 22:25:46 +08
        provider: minikube.sigs.k8s.io
        version: v1.38.1
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
users:
- name: minikube
  user:
    client-certificate: /home/gineesh/.minikube/profiles/minikube/client.crt
    client-key: /home/gineesh/.minikube/profiles/minikube/client.key
```

Check nodes

```shell
$ kubectl get nodes
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   2m49s   v1.36.3
```

```shell
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

```shell
$ kubectl get componentstatuses  # deprecated
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE   ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   ok
```

## Stop/Pause/Delete minikube clusters

```shell
# Pause
$ minikube pause
⏸️  Pausing node minikube ...
⏯️  Paused 8 containers in: kube-system, kubernetes-dashboard, istio-operator

# Check status
$  minikube status
minikube
type: Control Plane
host: Running
kubelet: Stopped
apiserver: Paused
kubeconfig: Configured

# Resume
$ minikube unpause
⏸️  Unpausing node minikube ...
⏸️  Unpaused 8 containers in: kube-system, kubernetes-dashboard, istio-operator
```

```shell
$ minikube stop
✋  Stopping node "minikube"  ...
🛑  Powering off "minikube" via SSH ...
🛑  1 node stopped.

$ minikube status
minikube
type: Control Plane
host: Stopped
kubelet: Stopped
apiserver: Stopped
kubeconfig: Stopped

$  minikube start
😄  minikube v1.38.1 on Fedora 43
❗  Specified Kubernetes version 1.36.3 is newer than the newest supported version: v1.35.1. Use `minikube config defaults kubernetes-version` for details.
❗  Specified Kubernetes version 1.36.3 not found in Kubernetes version list
🤔  Searching the internet for Kubernetes version...
✅  Kubernetes version 1.36.3 found in GitHub version list
✨  Using the podman driver based on existing profile
👍  Starting "minikube" primary control-plane node in "minikube" cluster
🚜  Pulling base image v0.0.50 ...
E0728 22:49:28.695173 1073729 cache.go:239] Error downloading kic artifacts:  not yet implemented, see issue #8426
🔄  Restarting existing podman container for "minikube" ...
📦  Preparing Kubernetes v1.36.3 on containerd 2.2.1 ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

```shell
$ minikube delete
🔥  Deleting "minikube" in podman ...
🔥  Deleting container "minikube" ...
🔥  Removing /home/gmadappa/.minikube/machines/minikube ...
💀  Removed all traces of the "minikube" cluster.
```

## Multi-node Kubernetes using minikube

```shell
$ minikube start --driver=podman --nodes=3 --kubernetes-version=1.36.3

$ kubectl get nodes
NAME           STATUS   ROLES           AGE     VERSION
minikube       Ready    control-plane   8m16s   v1.36.3
minikube-m02   Ready    <none>          8m1s    v1.36.3
minikube-m03   Ready    <none>          7m47s   v1.36.3
```

```shell
$ minikube start \
  --driver=podman \
  --nodes 5 \
  --ha true \
  --cpus=2 \
  --memory=2g \
  --kubernetes-version=1.36.3
```

```shell
$ kubectl get nodes
NAME           STATUS   ROLES           AGE     VERSION
minikube       Ready    control-plane   2m57s   v1.36.3
minikube-m02   Ready    control-plane   2m19s   v1.36.3
minikube-m03   Ready    control-plane   100s    v1.36.3
minikube-m04   Ready    <none>          83s     v1.36.3
minikube-m05   Ready    <none>          65s     v1.36.3
```


## Multiple Kubernetes clusters using minikube

```shell
# Start a minikube cluster using Podman as driver.
$ minikube start --profile cluster-podman --driver=podman

$ minikube start --profile cluster-vbox --driver=virtualbox
```

### minikube check profiles

```shell
$ $  minikube profile list
┌────────────────┬────────────┬─────────┬────────────────┬─────────┬────────┬───────┬────────────────┬────────────────────┐
│    PROFILE     │   DRIVER   │ RUNTIME │       IP       │ VERSION │ STATUS │ NODES │ ACTIVE PROFILE │ ACTIVE KUBECONTEXT │
├────────────────┼────────────┼─────────┼────────────────┼─────────┼────────┼───────┼────────────────┼────────────────────┤
│ cluster-podman │ podman     │ docker  │ 192.168.58.2   │ v1.35.1 │ OK     │ 1     │                │                    │
│ cluster-vbox   │ virtualbox │ docker  │ 192.168.59.191 │ v1.35.1 │ OK     │ 1     │                │ *                  │
└────────────────┴────────────┴─────────┴────────────────┴─────────┴────────┴───────┴────────────────┴────────────────────┘

# Stop cluster
$ minikube stop --profile cluster-podman

# Remove the cluster
$ minikube delete --profile cluster-podman
```


## Installing Kind

```shell
## Linux:
$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.27.1/kind-$(uname)-amd64
$ chmod +x ./kind
$ mv ./kind /usr/local/bin/kind

# macOS:
$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.27.1/kind-$(uname)-amd64
$ chmod +x ./kind
$ mv ./kind /usr/local/bin/kind

# Homebrew:
$ brew install kind

# Windows:
$ curl.exe -Llo kind-windows-amd64.exe https

# Chocolatey:
$ choco install kind
```

## Creating cluster using kind

```shell
$ kind create cluster --name test-kind
```

```shell
$ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:42547
CoreDNS is running at https://127.0.0.1:42547/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl get --raw='/readyz?verbose'
[+]ping ok
[+]log ok
[+]etcd ok
[+]etcd-readiness ok
[+]informer-sync ok
[+]poststarthook/start-apiserver-admission-initializer ok
[+]poststarthook/generic-apiserver-start-informers ok
[+]poststarthook/priority-and-fairness-config-consumer ok
[+]poststarthook/priority-and-fairness-filter ok
[+]poststarthook/storage-object-count-tracker-hook ok
[+]poststarthook/start-apiextensions-informers ok
[+]poststarthook/start-apiextensions-controllers ok
[+]poststarthook/crd-informer-synced ok
[+]poststarthook/start-service-ip-repair-controllers ok
[+]poststarthook/rbac/bootstrap-roles ok
[+]poststarthook/scheduling/bootstrap-system-priority-classes ok
[+]poststarthook/priority-and-fairness-config-producer ok
[+]poststarthook/start-system-namespaces-controller ok
[+]poststarthook/bootstrap-controller ok
[+]poststarthook/start-cluster-authentication-info-controller ok
[+]poststarthook/start-kube-apiserver-identity-lease-controller ok
[+]poststarthook/start-kube-apiserver-identity-lease-garbage-collector ok
[+]poststarthook/start-legacy-token-tracking-controller ok
[+]poststarthook/aggregator-reload-proxy-client-cert ok
[+]poststarthook/start-kube-aggregator-informers ok
[+]poststarthook/apiservice-registration-controller ok
[+]poststarthook/apiservice-status-available-controller ok
[+]poststarthook/apiservice-discovery-controller ok
[+]poststarthook/kube-apiserver-autoregistration ok
[+]autoregister-completion ok
[+]poststarthook/apiservice-openapi-controller ok
[+]poststarthook/apiservice-openapiv3-controller ok
[+]shutdown ok
readyz check passed
```

Config file creating multi-node cluster - eg: `~/.kube/kind_cluster`

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker
```

Create cluster

```shell
$ kind create cluster --config ~/.kube/kind_cluster
```

Start with Podman instead of Docker

```shell
$ KIND_EXPERIMENTAL_PROVIDER=podman kind create cluster --config ~/.kube/kind_cluster
```

Mention the Kubernetes version

```shell
# 1.29.0
$ kind create cluster \
  --name my-kind-cluster \
  --config ~/.kube/kind_cluster \
  --image kindest/node:v1.29.0@sha256:eaa1450915475849a73a9227b8f201df25e55e268e5d619312131292e324d570

# 1.29.0
$ kind create cluster \
  --name my-kind-cluster \
  --config ~/.kube/kind_cluster \
  --image kindest/node:v1.29.2@sha256:51a1434a5397193442f0be2a297b488b6c919ce8a3931be0ce822606ea5ca245
```


Refer to [github.com/kubernetes-sigs/kind/releases](https://github.com/kubernetes-sigs/kind/releases) to learn more.
