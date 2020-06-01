# Secrets

Now, let us touch on the subject of the sensitive data and how to get that inside of the k8s cluster and our `pods`. For this, the k8s provide a concept of `secretes` that is pretty similar to the ConfigMaps. Sensitive data can be of any kind, API keys, passwords, etc. Within the k8s there is a store that this secretes will be stored that prevent us to need to store it within file system of the container, manifest and so on. The secretes, same as ConfigMaps, can be mounted using the same principles. That is, using files or environment variables (guess which one I prefer). The k8s will expose the `secret` to the `nodes` that are requesting it. The requested secrete will be stored on `tmpfs` of the `node` to prevent someone to accessing it directly.

Creating a `secret` is (or it should) be more involved then creating a ConfigMap. It usually is not a good idea to have them part of the repository, right? It defeats the purpose of having them separated from the ConfigMaps. They will be only `base64` encoded and if someone gets a hold of them, well you can guess. This is a big topic that I'm not gonna touch upon here. Here we are gonna show one way of creating a `secret` and that is using command line:

```bash
# example: create a secret using a literal
# generic is used for simple key-value pairs
kubectl create secret generic my-secret --from-literal=password=1234!

# example: loading a secret from the file
kubectl create secrete generic my-secret --from-file=privatekey=~/.ssh/id_rsa --from-file=publickey=~/.ssh/id_rsa.pub

# example: when working with TLS
kubectl create secret tls my-secret --cert=/path/to/tls.cert --key=/path/to/tls.key
```

Now, let us do an example and see what we are gonna have within our cluster:

```bash
kubectl create secret generic my-secret --from-literal=password=1234!
secret/my-secret created

# ok, so far so good, now let us get that secret
kubectl get secret my-secret -o yaml
apiVersion: v1
data:
  password: MTIzNCE=
kind: Secret
metadata:
  creationTimestamp: "2020-05-29T06:33:43Z"
  name: my-secret
  namespace: default
  resourceVersion: "251694"
  selfLink: /api/v1/namespaces/default/secrets/my-secret
  uid: a96a2762-dbce-40a8-8433-7623e8d76cb9
type: Opaque

# now let use decode the base64 string
base64 -d
MTIzNCE=
1234!%

# you can also do it like this, if the string in in the clipboard :)
# pbpaste | base64 -d
# 1234!%

# and here it is, our password
```

Now let us see how we can use this secret from within the `pod`:

```bash
# first we create a secret
kubectl create secret generic my-nginx --from-literal=password='1234!'
secret/my-nginx created

# now let us create a pod that uses the secret
kubectl apply -f src/nginx.secret.yaml
deployment.apps/my-nginx created

# get the pods
kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
my-nginx-78f7944647-j5fxt   1/1     Running   0          5s

# now we exec into the pod
kubectl exec my-nginx-78f7944647-j5fxt -it sh

# run the command to echo out the env variable
echo $MYPASSWORD
1234!

# and that is it
```
