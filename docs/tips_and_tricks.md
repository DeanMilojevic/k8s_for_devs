# Tips & Tricks

To leverage k8s to create a `YAML` for the resource you can run the command bellow:

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > pod.yaml

```

And now in the *pod.yaml* we have the following:

```yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

This can be used also if you are creating something that needs to run a specific command:

```bash
kubectl run busybox --image=busybox --restart=Never --dry-run -o yaml --command -- env > pod.yaml
```

and we can see that command is now included as part of the resulting `YAML`:

```yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - command:
    - env
    image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

To see the resulting output in creating a namespace (without creating it in the cluster) we can add `--dry-run` to the command:

```bash
kubectl create namespace my-namespace -o yaml --dry-run
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: my-namespace
spec: {}
status: {}
```

In case you want to listen the pods (if you have permissions) on all namespaces add `--all-namespaces`:

```bash
kubectl get pods --all-namespaces
NAMESPACE      NAME                                     READY   STATUS      RESTARTS   AGE
default        busybox                                  0/1     Completed   0          7m35s
docker         compose-78f95d4f8c-dffhf                 1/1     Running     0          7d16h
docker         compose-api-6ffb89dc58-q5fgp             1/1     Running     0          7d16h
kube-system    coredns-5644d7b6d9-b2fsj                 1/1     Running     0          7d16h
kube-system    coredns-5644d7b6d9-kk6tf                 1/1     Running     0          7d16h
kube-system    etcd-docker-desktop                      1/1     Running     0          7d16h
kube-system    kube-apiserver-docker-desktop            1/1     Running     0          7d16h
kube-system    kube-controller-manager-docker-desktop   1/1     Running     0          7d16h
kube-system    kube-proxy-qv4mf                         1/1     Running     0          7d16h
kube-system    kube-scheduler-docker-desktop            1/1     Running     0          7d16h
kube-system    storage-provisioner                      1/1     Running     1          7d16h
kube-system    vpnkit-controller                        1/1     Running     0          7d16h
lens-metrics   kube-state-metrics-94b8476c4-rfcxm       1/1     Running     1          7d13h
lens-metrics   node-exporter-46xdz                      1/1     Running     0          7d13h
lens-metrics   prometheus-0                             1/1     Running     0          7d13h
```
