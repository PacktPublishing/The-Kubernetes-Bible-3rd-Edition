# Using Multi-Container Pods and Design Patterns

- [Using Multi-Container Pods and Design Patterns](#using-multi-container-pods-and-design-patterns)
  - [Creating multi-container Pod](#creating-multi-container-pod)
  - [Volumes](#volumes)
  - [Hostpath Volume](#hostpath-volume)
  - [Ambassador multi-container Pod](#ambassador-multi-container-pod)


## Creating multi-container Pod

```shell
$ kubectl apply -f multi-container-pod.yaml
pod/multi-container-pod created


$ kubectl get pod
NAME                  READY   STATUS    RESTARTS   AGE
multi-container-pod   2/2     Running   0          2m7s
```

```shell
$ kubectl logs multi-container-pod -c debian-container
Sat Aug 29 02:05:49 UTC 2026
debian-container
Sat Aug 29 02:05:54 UTC 2026
debian-container
...<removed for brevity>...
```

```shell
$ kubectl logs multi-container-pod -c nginx-container
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
...<removed for brevity>...
2026/08/29 02:05:39 [notice] 1#1: start worker process 29
2026/08/29 02:05:39 [notice] 1#1: start worker process 30
```

Testing multi-container Pod with invalid image

```shell
$ kubectl apply -f failed-multi-container-pod.yaml
pod/failed-multi-container-pod created

$ kubectl get pod
NAME                         READY   STATUS         RESTARTS   AGE
failed-multi-container-pod   1/2     ErrImagePull   0          33s
```

```shell
$ kubectl describe pod failed-multi-container-pod
Name:             failed-multi-container-pod
Namespace:        default
...<removed for brevity>...
Events:
  Type     Reason     Age                 From               Message
...<removed for brevity>...
  {nginx-container}: Pulling image "nginx:i-do-not-exist"
  Warning  Failed     16s (x4 over 104s)  kubelet            spec.containers{nginx-container}: Failed to pull image "nginx:i-do-not-exist": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/nginx:i-do-not-exist": failed to resolve reference "docker.io/library/nginx:i-do-not-exist": docker.io/library/nginx:i-do-not-exist: not found
  Warning  Failed     16s (x4 over 104s)  kubelet            spec.containers{nginx-container}: Error: ErrImagePull
  Normal   BackOff    2s (x6 over 102s)   kubelet            spec.containers{nginx-container}: Back-off pulling image "nginx:i-do-not-exist"
  Warning  Failed     2s (x6 over 102s)   kubelet            spec.containers{nginx-container}: Error: ImagePullBackOff
```

Deleting multi-container pods

```shell
$ kubectl delete -f multi-container-pod.yaml

## Otherwise, if you already know the Pod's name, you can do this as follows:
$ kubectl delete pods/multi-pod

## or equivalent
$ kubectl delete pods multi-pod

## Force deletion
$ kubectl delete pod failed-multi-container-pod --grace-period=0 --force
Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "failed-multi-container-pod" force deleted from default namespace
```

Accessing Container

```shell
$ kubectl describe pod multi-container-pod
```

```shell
$ kubectl get pod/multi-container-pod -o jsonpath="{.spec.containers[*].name}"
nginx-container debian-container

$ kubectl exec -it multi-container-pod --container nginx-container -- /bin/bash
root@multi-container-pod:/# hostname
multi-container-pod
root@multi-container-pod:/#
```

```shell
$ kubectl exec pods/multi-container-pod -c nginx-container -- ls
bin
boot
dev
docker-entrypoint.d
docker-entrypoint.sh
etc
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```

```shell
$ kubectl apply -f nginx-debian-with-custom-command-and-args.yaml
pod/nginx-debian-with-custom-command-and-args created

$ kubectl get po -w
NAME                                        READY   STATUS              RESTARTS   AGE
nginx-debian-with-custom-command-and-args   0/2     ContainerCreating   0          2s
nginx-debian-with-custom-command-and-args   2/2     Running             0          6s
nginx-debian-with-custom-command-and-args   1/2     NotReady            0          66s
```


```shell
$ kubectl apply -f nginx-with-init-container.yaml
pod/nginx-with-init-container created

$ kubectl get po -w
NAME                        READY   STATUS     RESTARTS   AGE
nginx-with-init-container   0/1     Init:0/1   0          3s
nginx-with-init-container   0/1     Init:0/1   0          4s
nginx-with-init-container   0/1     PodInitializing   0          19s
nginx-with-init-container   1/1     Running           0          22s
```

```shell
$ kubectl logs multi-container-pod
Defaulted container "nginx-container" out of: nginx-container, debian-container
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/08/30 13:33:48 [notice] 1#1: using the "epoll" event method
2026/08/30 13:33:48 [notice] 1#1: nginx/1.31.4
2026/08/30 13:33:48 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2026/08/30 13:33:48 [notice] 1#1: OS: Linux 6.6.95
2026/08/30 13:33:48 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/08/30 13:33:48 [notice] 1#1: start worker processes
2026/08/30 13:33:48 [notice] 1#1: start worker process 29
2026/08/30 13:33:48 [notice] 1#1: start worker process 30
```

## Volumes

```shell
$ kubectl apply -f multi-container-with-emptydir-pod.yaml
pod/multi-container-with-emptydir-pod created

$ kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
multi-container-with-emptydir-pod   2/2     Running   0          25s
```

```shell
$  kubectl exec multi-container-with-emptydir-pod -c debian-container -- ls /var
backups
cache
i-am-empty-dir-volume
lib
local
lock
log
mail
opt
run
spool
tmp
```

```shell
$  kubectl exec multi-container-with-emptydir-pod -c debian-container -- bin/sh -c "echo 'hello world' >> /var/i-am-empty-dir-volume/hello-world.txt"

$ kubectl exec multi-container-with-emptydir-pod -c nginx-container -- cat /var/i-am-empty-dir-volume/hello-world.txt
hello world

$ kubectl exec multi-container-with-emptydir-pod -c debian-container -- cat /var/i-am-empty-dir-volume/hello-world.txt
hello world
```

## Hostpath Volume

```shell
$ echo "Hello World" >> /tmp/hello-world.txt
```

If minikube

```shell
$ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ echo "Hello World" > /tmp/hello-world.txt
$ exit
logout
```

If Podman

```shell
$ sudo podman exec -it minikube /bin/bash
root@minikube:/# cat /tmp/hello-world.txt
```

Check file

```shell
$ kubectl exec multi-container-with-hostpath -c nginx-container -- cat /foo/hello-world.txt
```

## Ambassador multi-container Pod
