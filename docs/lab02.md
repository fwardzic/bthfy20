## Lab02 - Pods, ReplicaSets, Deployments

1. Deploy single Pod, in the imperative way, check what happens when you will leave the container

```kubectl run -it test-pod --image=alpine --restart=Never -- /bin/sh```

- check status of your container
- check on which node container is running
- check locally pulled docker images on that node

2. Deploy pod using declarative approach:

create a file with following manifest:


```apiVersion: v1
kind: Pod
metadata:
  name: hello
  namespace: default
spec:
  containers:
  - name: hello
    image: busybox
    command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
  restartPolicy: OnFailure```

