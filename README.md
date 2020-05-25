
<p align=center>
  <img alt="k8s" src="./resources/k8s.svg" />
</p>

# K8s for Developers

[Kubernetes](https://kubernetes.io/) is an open-source system for automating deployment, scaling, and management of containerized applications.

Some of the key features of k8s are:

1. Service discovery & load balancing
2. Automated rollouts/rollbacks
3. Storage orchestration
4. Self-healing in case of problems
5. Horizontal scaling
6. Secretes management
7. Configuration management

This is just some of the benefits system like k8s offers. From developer point of view, this are some of the most crucial ones, imo. From development point of view, there is some additional benefits that come with using `containers` and `k8s`. Yes, we could use the `docker` and `docker-compose` for most of the things that we are gonna mention. On the other hand, combining power of `docker` and benefits mentioned above from k8s, we are getting a really powerful tool in the arsenal.

Lets reflect on the benefits that we have using tools like `docker` and `containers` during development:

<p align=center>
  <img alt="benefits of containers" src="./resources/benefits_of_containers.svg" />
</p>

With this out of the way, lets see some really useful benefits of the k8s for developers:

<p align=center>
  <img alt="benefits of k8s" src="./resources/benefits_of_k8s.svg" />
</p>

Some of the reasons you would like to run k8s during development lifecycle:

1. Emulating environment setup (probably just partially) locally
2. E2E testing
3. Load testing
4. Troubleshooting

One of the personal examples on when it is useful to run the k8s locally was implementing a `CronJob` for a feature I was working on. When run with `docker-compose` the container was behaving without any issues, but when finally deployed to the test environment (that runs on k8s) it would almost it would end up in a `CrashLoopBackOff` on the start. After some time trying to figure out (logging was also not showing any useful information about failure), I started the k8s locally and deployed my `CronJob`. After just minutes into looking into the problem, it was obvious what was the problem. The `secrete` injector was having a problem with one of the secretes being used for the `CronJob`.

Another example is when you start playing around with optimizing your cluster and defining resources (CPU/RAM) for your `pods`. It can be useful to run the setup locally and put under a load test to see what are some baseline numbers you can configure for proper scaling (and how many replicas is a good start for your scenario).

## Desired state and k8s

The k8s provide a declarative way to define a cluster state.

<p align=center>
  <img alt="desired state" src="./resources/desired_state.svg" />
</p>

This is offered by k8s by offering internal services that handle this kind of state transitions. K8s have a concept of the `master node` and `worker nodes`. The `master node` (one or more) is responsible of management/orchestration of the `worker nodes`. This is what in k8s is known as a `cluster`.

The `pod` is just a k8s way of hosting a `container`. The `container` is the "place" where the application will be running. This are just terms that are used to describe the ecosystem and different "abstraction" layers of the deployment. The analogy to describe `pod` and `container` is the packaging of the product. The `pod` is the package and the `container` is the product you will be "using".

The `node` can be physical servers or virtual machines. Each node can have multiple `pods` running in it. That is where the horizontal scaling comes into the picture. The communication towards the application itself goes through the multiple layers. This is to ensure all the benefits of the k8s that were mentioned and that developer doesn't need to think about.

The `master node` keeps track of cluster through [etcd](https://etcd.io/). This is a distributed, reliable key-value store for the most critical data of a distributed system. The next part in this picture is the `controller manager`, that together with `scheduler` subsystem handles incoming requests. When the request comes to the cluster, `controller manager` schedules it (using `scheduler`) for processing. `Scheduler` will take care of the request based on the available resources, that is `pods`. This will be done based on the health, available resources, etc.

<p align=center>
  <img alt="cluster" src="./resources/cluster.svg" />
</p>

The interaction between the developer and the cluster is done over the `API` endpoint. This is a RESTful service. When communicating with the `API` it is possible to send payload, to be executed on the `nodes`, in `JSON` or `YAML` format. Then they will be scheduled (using the same tools explained above) to run on the nodes in the `cluster`. The popular tool that comes with k8s, to leverage the `API`, is `kubectl`. This is a CLI tool, that is an client implementation of the `API` endpoints to allow the communication with the `cluster`.

<p align=center>
  <img alt="kubectl" src="./resources/kubectl.svg" />
</p>

To allow the communication between the `worker node` and `master node` each node has an agent called `kubelet`. This agent is responsible to register `node` with `cluster`, that is `controller manager`. It also reports the status of the node, for the `master node` to know what to do in  case something goes wrong and so on. The `container runtime` is also a subsystem in that allows the running of the `containers` within the `pods`. The `kube-proxy` is the system that allows the networking and proper functioning of the node in the network. This means getting a unique IP address for each `pod` running within the `node`. This goes hand in hand with `services` that will be explained down the line.

<p align=center>
  <img alt="node" src="./resources/node.svg" />
</p>

## Running k8s locally

There are different ways of running the k8s locally and to my knowledge most popular are:

1. [Docker Desktop](https://www.docker.com/products/docker-desktop)
2. [Minikube](https://github.com/kubernetes/minikube)

There are some other, but haven't tried them yet so not gonna touch them at this point. Currently, I am using `docker desktop` for all my development needs, including k8s. It is quite easy to enable the k8s locally:

<p align=center>
  <img alt="docker desktop" src="./resources/k8s_locally.png" />
</p>

In case of the problems enabling this (process is quite similar for Windows), I suggest StackOverflow :)

To view something using a GUI, there is a Web UI that can be configured for k8s. For more information:
[https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

Personally, if I need this I prefer tools like Lens as they do offer the same functionality without a hustle of needing to install the Web UI on the cluster and running the proxy and playing with all that.

![Lens for k8s](./resources/lens.png)

## Pods

`Pods` are the smallest deployable units of computing that can be created and managed in Kubernetes. In short, they are the fundamental layer of the k8s that we can manage/deploy. They are used to run one or more containers, that is they act as environment for the container to run in. Even if it is possible to have multiple `containers` per `pod`, this is not something that is encouraged. The recommended mantra for this "one process per container and one container per pod", or at least trying to make a "mantra" :)

Defining `replicas` when deploying the `pod` will indicate on how many instances/copies of the pod/container (this can be used interchangeably if we follow the mantra) we will be running within the `node`. This is the `horizontal scaling` that will be taken care automatically for you by k8s. K8s will then load balance the incoming traffic across available pods and allow high availability of the application. In case that one of the pods is "unhealthy", k8s will automatically remove it and replace it with a new instance. This means that `pods` never recover from failure, they are replaced with a new instance.

<p align=center>
  <img alt="failing pod" src="./resources/failing_pod.svg" />
</p>

The networking for pods works as following:

1. Each pod has a unique IP address that gets assigned to it. This is called `Cluster IP` address.
2. The `containers` within the `pod` *share* the same IP address (of the pod), but they **have** the different ports. No 2 `containers` can have the same `port` and be in the same `pod`. Also, `containers` can span multiple `pods`. In case you need to have 2 or more `containers` in the `pod` (on internet found it referred to as *sidecar*, but never experienced a need for this), then you must ensure the `ports` are different for **each** container. The communication in the container is done over the same loopback network interface (localhost), so that makes it simple.
3. Ports can be reused in separate `pod containers`. That means that if `container in pod A` has a port of `80` assigned, the `container in pod B` can also have the port `80` assigned.

<p align=center>
  <img alt="pod ip" src="./resources/pod_ip.svg" />
</p>

### Creating a pod

The simplest way to create a pod is using the following command:

```bash
kubectl run [pod-name] --image=[image-name]

# example
kubectl run my-nginx --image=nginx:alpine
```

This is something that will be deprecated in the future (you can see by the output of the command):

```bash
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead. deployment.apps/my-nginx created
```

Now when we execute the command:

```bash
kubectl get all
```

the result should look like following (on clean cluster):

```bash
NAME                            READY   STATUS    RESTARTS   AGE
pod/my-nginx-576bb7cb54-p45zj   1/1     Running   0          96s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   16h

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx   1/1     1            1           96s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/my-nginx-576bb7cb54   1         1         1       96s
```

Except `ClusterIP`, everything was created with the command we just run. Tho it is much more then expected as we see couple more extra resources associated with the `pod`. This topics will be discussed in the [Deployments](#deployments) section of this document. Tho the pod is now running, we can't get to it, so lets see how we can manage this in the following chapter.

### Port forwarding

When the `pod` starts, it gets an `Cluster IP` address. This is something that is internal for the `cluster` itself. In other words, we can't access it directly as the `containers` and `pods` are only accessible within the cluster. To make this happen, we need to do a *port forwarding*. This is similar to the `docker` and `docker-compose`. The command for *port forwarding*:

```bash
kubectl port-forward [pod-name] [external-port]:[internal-port]

# example

kubectl port-forward my-nginx-576bb7cb54-p45zj 8080:80
```

After running the command above (with your `pod` name), you should see the following:

```bash
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

Now lets try curl our endpoint and see what will the output be:

```bash
http GET localhost:8080
# I am using a CLI tool httpie (https://httpie.org/) that is a bit nicer, imo, when working with the examples
```

with the output of:

```html
HTTP/1.1 200 OK
Accept-Ranges: bytes
Connection: keep-alive
Content-Length: 612
Content-Type: text/html
Date: Mon, 25 May 2020 07:07:32 GMT
ETag: "5e95ccbe-264"
Last-Modified: Tue, 14 Apr 2020 14:46:22 GMT
Server: nginx/1.17.10

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Back in the terminal where we started *port forward* we should also see some information:

```bash
kubectl port-forward my-nginx-576bb7cb54-p45zj 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
```

### Imperative vs. Declarative

Ok, now that we covered how we can create a pod in *imperative* way, lets move towards what is considered a standard. That is, using *declarative* way of creating resources. This is done using `YAML`. There are tone of the information on this topic online, so not gonna cover it here. For this to work with k8s, there are certain rules (or to be exact a `schema` to be followed). So let us try create the same thing using `YAML` now.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
  labels:
    name: my-nginx
spec:
  containers:
  - name: my-nginx
    image: nginx:alpine
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 8080

```

There ar some key-value pairs in here that we can ignore for now (this is just a default template I have configured locally). Now there are 2 commands we can use to create the `pod` using `YAML`:

```bash
kubectl create -f [file-name.yaml|file-name.yml]
# or
kubectl apply -f [file-name.yaml|file-name.yml]
```

The difference is what will happen in case of that `pod` already exists on the cluster. The `create` command ends up in error while `apply` will do create or update of the `pod`. For most cases, I prefer the `apply` command.

```bash
kubectl apply -f src/nginx.pod.yaml
# file src/nginx.pod.yml can be found in the root of this repository
```

This should give an output like the following:

```bash
pod/my-nginx created
```

As you can see, the output is a bit different then originally, but also now when we run the command `kubectl get all` the number of resources created will be different as well:

```bash
NAME           READY   STATUS    RESTARTS   AGE
pod/my-nginx   1/1     Running   0          5m7s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   17h
```

To check if everything works as before, lets run `kubectl port-forward my-nginx 8080:80` and the `http GET localhost:8080`. The output should be exactly the same as in the previous scenario when we used the imperative way. Awesome.

### Deletion of pod

Deletion of `pod` has an interesting side-effect that can be confusing first time. WHen you delete the `pod` using the command bellow:

```bash
kubectl delete pod [pod-name]

# example

kubectl delete pod my-nginx
```

Afterwards, you can try to get the all `pods` within the cluster:

```bash
kubectl get pods
```

you will again see your `pod`. If you look carefully and compare before/after situation, you will notice that there is a difference. `Pod` has a new **id**. This is expected behavior of the k8s, as it tries to keep the desired state. The `master node` got notified of deletion of the `pod`, checked the `deployment` (discussed in upcoming chapter) and saw that `pod` should have `number of replicas` running and the started a new instance. So the `pod` was deleted, it is just that new instance of it was started automatically.

To fully remove the you need to remove the `deployment` that is associated with this `pod` (or that manages the `pod`). For this you can check the [Deleting the deployment](###deleting-the-deployment).

## Deployments

### ReplicaSet

### Deleting the deployment

```bash
kubectl delete deployment [deployment-name]

# example
kubectl delete deployment my-nginx
```

For this command, you should see something like this:

```bash
 deployment.apps "my-nginx" deleted
 ```

 Now when we try to run the following command `kubectl get all`, there should not be any resources associated with the deployment:

```bash
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   16h
```

## Services

## Storage

## ConfigMaps

## Secretes

## CronJobs

## kubectl commands

| Command  | Description  |
|---|---|
|``` kubectl version ```| K8s version  |
|``` kubectl cluster-info ```| Cluster information  |
|``` kubectl create [resource] ```| Create a resource |
|``` kubectl apply [resource] ```| Create or modify a resource |
|``` kubectl get all ```| Gets (well, everything) `pods`, `deployments`, `services`, etc.  |
|``` kubectl port-forward [pod-name] [ports] ```| Allows the external access to the forwarded port |
|``` kubectl delete ([-f FILENAME] | [-k DIRECTORY] | TYPE [(NAME | -l label | --all)]) [options] ``` | Deleting a resource within the cluster |
|``` kubectl delete pod [pod-name] ```| Delete a pod with given name |


## Tools

[Lens](https://k8slens.dev/)