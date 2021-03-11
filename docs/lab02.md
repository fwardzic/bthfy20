## Lab02 - Pods, ReplicaSets, Deployments

### 1. Create your namespace:

```kubectl create namespace studentXX```

**replace XX with your assigned number**

change default namespace in your kubectl, so you don't have specify namespace everytime

```kubectl config set-context $(kubectl config current-context) --namespace=studentXX```

**Replace XX with your assigned number!!!**

---

### 2. Deploy single Pod, in the imperative way, check what happens when you will exit from shell

```kubectl run -it test-pod-1 --image=alpine --restart=Never -- /bin/sh```

- check ip address ```ip add```

exit from pod's shell, check status of your pod ```kubectl get pods```

What happend?

Run another pod: 
```kubectl run -it test-pod-2 --image=alpine --generator=run-pod/v1```

- check what dns is confgured ```cat /etc/resolv.conf```
- check ip address ```ip add```
- check environment variables ```env```
- check on which node container is running ```kubectl get pods -o wide```

exit from pod and check status of you second pod, what is the difference, and why?

---

### 3. Deploy pod using declarative approach:

create a file with following manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod3
spec:
  containers:
  - name: test-pod3
    image: alpine
  restartPolicy: OnFailure
```

exit from pod and check status of you third pod, what is the difference, and why?

delete all pods:

```kubectl delete pod XYZ```

---

### 4. Implement replicas for Guestbook app

apply following manifest to deploy redis-master database node:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: redis-master
  labels:
    name: redis-master
spec:
  replicas: 1
  selector:
    matchLabels:
      name: redis-master
  template:
    metadata:
      labels:
        name: redis-master
    spec:
      containers:
      - name: master
        image: redis:3.0.7-alpine
        ports:
        - containerPort: 6379
```

instead of copy paste manifiest to your PC, you can use following command to apply that manifest file hosted in github

```kubectl apply -f https://raw.githubusercontent.com/fwardzic/bthfy20/master/manifests/guestbook/guestbook-redis-master.yaml```

- Check status of your pods - ```kubectl get pods```
- check status of replicaset - ```kubectl get rs```

apply following manifest to deploy redis-slave database nodes:


```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: redis-slave
  labels:
    name: redis-slave
spec:
  replicas: 2
  selector:
    matchLabels:
      name: redis-slave
  template:
    metadata:
      labels:
        name: redis-slave
    spec:
      containers:
      - name: worker
        image: gcr.io/google_samples/gb-redisslave:v1
        env:
        - name: GET_HOSTS_FROM
          value: dns
        ports:
        - containerPort: 6379
```
instead of copy paste manifiest to your PC, you can use following command to apply that manifest file hosted in github

```kubectl apply -f https://raw.githubusercontent.com/fwardzic/bthfy20/master/manifests/guestbook/guestbook-redis-slave.yaml```

- Check status of your pods - ```kubectl get pods```
- check status of replicaset - ```kubectl get rs```
- inrease number of slaves from 2 to 3 - ```kubeclt scale replicaset redis-slave --replicas=3```
- try to delete one of the pods from replicaset ```kubectl delete pod redis-slave-XXXXX``` - where XXX will be random ID.
- check what will happen

---

### 5. Implement frontend server for guestbook app:

apply following manifest to deploy frontend web server nodes:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      name: frontend
  template:
    metadata:
      labels:
        name: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
        env:
        - name: GET_HOSTS_FROM
          value: dns
        ports:
        - containerPort: 80
```

instead of copy paste manifiest to your PC, you can use following command to apply that manifest file hosted in github

```kubectl apply -f https://raw.githubusercontent.com/fwardzic/bthfy20/master/manifests/guestbook/guestbook-frontend.yaml```

- Check status of your pods - ```kubectl get pods```
- check status of replicaset - ```kubectl get deployment```

---

### 6. Rolling update - deployment 

Apply following deployment of simple web page


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-rolling
  labels:
    name: hello-rolling
spec:
  replicas: 3
  selector:
    matchLabels:
      name: hello-rolling
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        name: hello-rolling
    spec:
      containers:
      - name: hello-rolling
        image: openshift/hello-openshift
        ports:
        - containerPort: 8080
```
instead of copy paste manifiest to your PC, you can use following command to apply that manifest file hosted in github

```kubectl apply -f https://raw.githubusercontent.com/fwardzic/bthfy20/master/manifests/rolling-update-deployment.yaml```


expose web service internally withing the cluster

```kubectl expose deployment hello-rolling --port=80 --target-port=8080```

run another pod for testing purposes:

```kubectl run web-test --image=nginx --generator=run-pod/v1```

check if pod is running and try to login to shell of the pod:

```kubectl exec -it web-test -- bash```

from inside container, run curl command:

```curl http://hello-rolling```

exit container

run following command to change the application configuration. 

```kubectl set env deployment/hello-rolling RESPONSE="Hello from an updated app !!"```

See what will happen with your pods

- how many pods are running
- what is the uptime of the pods?
- go back to the web-test pod and try again curl, what do you see?

---