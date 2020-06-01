# ConfigMaps

ConfigMaps provide a way to store configuration and use it within the `containers`. It is a way to inject the configuration into our `containers` using key-value pairs or a file. Tho using a file is also looked at as key-value pair (filename - file contents).

ConfigMaps can be accessed using:

1. Environment variables (key-value)
2. ConfigMap volume (files)

Defining a ConfigMap using a manifest:

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-nginx
  labels:
    app: my-nginx
data:
  test: "value"
```

Creating the ConfigMap using a values from the file:

```bash
kubectl create configmap [config-map-name] --from-file=[file-path]

# output in the ConfigMap => key becomes a filename
# ...
# data:
#   myfile.config: |-
#     config1=value
#     config2=false
```

Using a `.env` file (and `--from-env-file` flag with the command) will create a similar output as with using the `manifest`. It is also possible to create ConfigMap directly from the command, using literals:

```bash
kubectl create configmap [config-map-name] --from-literal=config1=false --from-literal=config2=value
```

Viewing existing ConfigMaps withing cluster is possible using the following command:

```bash
# get all
kubectl get cm

# get by name
kubectl get cm [config-map-name]

# get it by name and show as yaml
kubectl get cm [config-map-name] -o yaml
```

An example of this:

```bash
# let us create one using literal
kubectl create configmap my-config --from-literal=config=false

# now let us get it using the commands above
kubectl get cm my-config -o yaml
apiVersion: v1
data:
  config: "false"
kind: ConfigMap
metadata:
  creationTimestamp: "2020-05-28T15:09:37Z"
  name: my-config
  namespace: default
  resourceVersion: "241016"
  selfLink: /api/v1/namespaces/default/configmaps/my-config
  uid: dfc29982-afbd-4366-a75a-cf439859b572
```

Now let us see how we can use the ConfigMap within our `pod`. For me the preferred way is using environment variables, as it ties into the dependency injection (or configuration injection) principle.

First let us apply the following files so we can see what is happening:

```bash
kubectl apply -f src/nginx.configmap.yaml
deployment.apps/my-nginx created

kubectl apply -f src/configmap.yaml
configmap/my-nginx created
```

Now let us exec into the `pod` (get the `pod` name using `kubectl get pods`) `kubectl exec [pod-name] -it sh`. Now that we are within the `pod` we can now echo out the environment variables:

```bash
echo $MYCONFIG
value
```
