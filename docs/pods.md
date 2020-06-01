# Pods

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

## Creating a pod

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

## Port forwarding

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

Now, next thing that is really useful to check (when troubleshooting or just to see what was created) is to `describe` the resource. This means that you can get all the information associated with the resource using the following command:

```bash
kubectl describe pod my-nginx
#the output is as follows
Name:         my-nginx
Namespace:    default
Priority:     0
Node:         docker-desktop/192.168.65.3
Start Time:   Mon, 25 May 2020 09:57:46 +0200
Labels:       name=my-nginx
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"name":"my-nginx"},"name":"my-nginx","namespace":"default"},"spec":...
Status:       Running
IP:           10.1.0.37
IPs:
  IP:  10.1.0.37
Containers:
  my-nginx:
    Container ID:   docker://8ca36a7c77cf4b65dd03a487cd2b572c320ccbc62aebc2ba7da4a96023f6b665
    Image:          nginx:alpine
    Image ID:       docker-pullable://nginx@sha256:763e7f0188e378fef0c761854552c70bbd817555dc4de029681a2e972e25e30e
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 25 May 2020 09:57:47 +0200
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  128Mi
    Requests:
      cpu:        500m
      memory:     128Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-kxzww (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-kxzww:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-kxzww
    Optional:    false
QoS Class:       Guaranteed
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From                     Message
  ----    ------     ----       ----                     -------
  Normal  Scheduled  <unknown>  default-scheduler        Successfully assigned default/my-nginx to docker-desktop
  Normal  Pulled     20m        kubelet, docker-desktop  Container image "nginx:alpine" already present on machine
  Normal  Created    20m        kubelet, docker-desktop  Created container my-nginx
  Normal  Started    20m        kubelet, docker-desktop  Started container my-nginx
  ```

There is a lot of information here. Also we can see all the information for our pod that we provided (for example, labels, resources, etc.). This is really detailed and good to know when you want to check a particular resource in case of some problems. The `Events` section in this output can usually indicate some high level problems and be a good starting point in troubleshooting the issue.

## Pod health

Now let us touch on the subject that can help k8s to help you when something goes wrong. That is using the `probs` to determine the state/health of the `pod container`. There are 2 types of the `probes` k8s are using to determine what is happening (or what it should to with a particular `container`):

1. **Liveness prob**
2. **Readiness prob**

There is a subtle difference between the two. Liveness prob says is `pod` alive. Readiness prob checks is the requests can be forwarded to the `pod`. `Probs` are something that will k8s run behind a scene for you, sort of diagnostic, periodically. So it is not something that we need to worry about, except when it starts failing :)

So in case that `container` fails on the check, there is a policy associated with the `probs` on what should happen next. By default this policy is set to **restart** always (`restartPolicy: Always`). This can be changed, depending on the use-case and expectations on what should happen in case of the failure.

There are different types of checks we can perform in combination when defining the `probs`:

1. ExecAction: Run some action inside of the `container` (file check, etc.)
2. TCPSocketAction: TCP check on the port inside of the `container`
3. HTTPGetAction: Run the HTTP GET request on the endpoint defined within the `container`

The "result" of the check can be:

1. **Success**
2. **Failure**
3. **Unknown**

Now, the following example shows how to define the `liveness and readiness prob` in declarative way (again, this only means using `YAML`):

```yml
livenessProbe:
  httpGet:
    path: /index.html
    port: 80
  initialDelaySeconds: 10
  timeoutSeconds: 2
  periodSeconds: 10
  failureThreshold: 1
readinessProbe:
  httpGet:
    path: /index.html
    port: 80
  initialDelaySeconds: 10
  timeoutSeconds: 2
  periodSeconds: 10
  failureThreshold: 0
```

Honestly, depending on the situation it us up to us to define this checks properly. I know that most common answer on 99,999% of questions in developer world ends in that answer "it depends". So a simple rule of thumb for me, as the description of `probs` defines them, is:

1. Liveness probe:  a simple check if the application is running (process check, an endpoint that returns Success/Failure) and nothing more.
2. Readiness probe: a check of dependencies (if you know that you can't serve the request because your db is down) and can you guarantee a success when processing the request.

The entire example can be found in the `src/nginx.health.pod.yaml`. To test this you can `exec` into the `container` and try renaming the file. When you describe the `pod` you should see something like this:

```bash
  Normal   Scheduled  <unknown>          default-scheduler        Successfully assigned default/my-nginx-health to docker-desktop
  Warning  Unhealthy  20s                kubelet, docker-desktop  Liveness probe failed: HTTP probe failed with statuscode: 404
  Normal   Killing    20s                kubelet, docker-desktop  Container my-nginx-health failed liveness probe, will be restarted
```

Now when you go and fetch `pods` and `exec` into it, you should see that `index.html` for example is now back (even after you removed it or renamed it). This is because the `pod` was removed and the new instance was started. That new instance has no relation with that file change we did, so it only has a fresh new state.

## Deletion of pod

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

In case that pod was created using the *declarative* way (that is, using `YAML`) like in example above, we can actually just delete the `pod` and it will be gone. This is because the only thing we created was `pod`, the command will not create any associated resource (`deployment` that is) that will try to put the desired state back (in case of deletion).

Also really nice way to cleanup/delete resources created using file is their deletion using the file:

```bash
kubectl delete -f src/nginx.pod.yaml
```

## Getting inside of the pod

There is an useful command that allows you to connect to the `pod/container` and do some troubleshooting on the file system for example (check if all files were being deployed and so on). That is using the command:

```bash
kubectl exec [pod-name] -it sh

# example
kubectl exec my-nginx -it sh
```

This will give you access to the file system inside of the `container` so you can check what is happening inside.

```bash
# example of command above
/ # ls
bin    dev    etc    home   lib    media  mnt    opt    proc   root   run    sbin   srv    sys    tmp    usr    var
/ #
```
