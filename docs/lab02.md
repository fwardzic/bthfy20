## Lab02 - Pods, ReplicaSets, Deployments

1. Create your namespace:

kubectl create namespace studentXX

replace studentXX with your number

Use 

Deploy single Pod, in the imperative way, check what happens when you will leave the container

```kubectl run -it test-pod --image=alpine --restart=Never -- /bin/sh```

replace studentXX with your number

- check status of your container
- check on which node container is running
- check locally pulled docker images on that node

2. Deploy pod using declarative approach:

create a file with following manifest:

```
apiVersion: v1
kind: Pod
metadata:
  name: web-pod1
  namespace: student_XX
spec:
  containers:
  - name: web-pod1
    image: nginx
  restartPolicy: OnFailure
```

- note name of your pod and in which namespace it is running
- check status of your container
- check on which node container is running
- check IP Address of your POD

3. Implement replica set to scale up your pod

Copy following ReplicaSet manifest to 

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
