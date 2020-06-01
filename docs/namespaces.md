# Namespaces

K8s supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces. To create a namespaces is quite simple:

```bash
kubectl create namespace my-namespace
namespace/my-namespace created

kubectl get namespaces
NAME                   STATUS   AGE
default                Active   7d16h
docker                 Active   7d16h
kube-node-lease        Active   7d16h
kube-public            Active   7d16h
kube-system            Active   7d16h
kubernetes-dashboard   Active   7d15h
lens-metrics           Active   7d13h
my-namespace           Active   27s
```

Now we can associate the resources with the namespaces they belong to:

```bash
kubectl run nginx --image=nginx --restart=OnFailure -n my-namespace
job.batch/nginx created

kubectl get all -n my-namespace
NAME              READY   STATUS    RESTARTS   AGE
pod/nginx-8ncqh   1/1     Running   0          40s

NAME              COMPLETIONS   DURATION   AGE
job.batch/nginx   0/1           40s        40s
```

This becomes useful to define the borders within deferent parts of the application/domains. It also allows to reduce the impact someone can have on parts that shouldn't be touched. Most k8s resources (e.g. pods, services, etc.) are in some namespaces. However namespace resources are not themselves in a namespace. And low-level resources, such as nodes and persistentVolumes, are not in any namespace.

To see which Kubernetes resources are and aren’t in a namespace:

```bash
# In a namespace
kubectl api-resources --namespaced=true

# Not in a namespace
kubectl api-resources --namespaced=false
```
