# Learn k8s in seconds

## Conception

[Documents](https://kubernetes.io/docs/concepts/)

- Pod: The basic unit, consist of one or more container. These containers share network and drive.
- Deployment: A pod status to keep. 

### Pod

The smallest and simplest unit in k8s. Pod could include one or more container, they share a `ClusterIP`.

Usually use controller(e.g., Deployments) manage Pod. But now we learn how to use Pod only.

#### pod.yml

```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
```

`apiVersion: v1` declaration we use which version api.

`kind: Pod` declaration the type of object.

`metadata: ` set `name` and `labels`.

`spec: ` define the specific of object. If type is Pod, `spec` should include a `containers` to define specific of container.

Two `name` of `containers` and `metadata` is different. Metadata's name is unique name of pod in kubernetes. Containers' name will be a part of docker container's name.

Now use `kubectl` communicate with kubernetes.

`kubectl get pods` list all pods.

```shell
$ kubectl get pods
NAME        READY     STATUS    RESTARTS   AGE
myapp-pod   1/1       Running   0          1m
# `kubectl get pods myapp-pod` could filte one pod by Pod name
```

`kubectl logs myapp-pod` will show logs of the pod.

```shell
$ kubectl logs myapp-pod
Hello Kubernetes!
```

`kubectl exec ...` exec some thing.

```shell
$ kubectl exec -it myapp-pod /bin/sh
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # exit
$ kubectl exec -it myapp-pod pwd
/
$ kubectl exec -t myapp-pod pwd
/
```

Network will notes in `Service` section.