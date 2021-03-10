# Lab 02 Deploy microservice IOT app

##


## Deploy backend microservices

### Task 1: (OPTIONAL) Declare persistent Volume for MariaDB

**Please do this task only if you are not using lab in DMZ (behind anyconnect) !!**

A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. It is a resource in the cluster just like a node is a cluster resource. PVs are volume plugins like Volumes, but have a lifecycle independent of any individual Pod that uses the PV. This API object captures the details of the implementation of the storage, be that NFS, iSCSI, or a cloud-provider-specific storage system.

In this lab we will use NFS storage. In modern systems like Hyperflex, Kubernetes leverages CSI - Container Storage Interface which is a module with storage driver.

    vi pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-studentXX ## replace XX with your student ID
spec:
  capacity:
    storage: 2Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /mnt/sharedfolder/studentXX ## replace XX with your student ID
    server: 10.62.86.11
```

    kubectl apply -f pv.yaml

Verify your persistent volume

    kubectl get pv
    kubectl describe pv pv-studentXX

### Task 2: Declare PersistentVolumeClaim

A **Kubernetes Persistent Volume Claim (PVC)** is a request for storage by a user. It is similar to a pod. Pods consume node resources and PVCs consume Persistent Volume (PV) resources. Pods can request specific levels of resources (CPU and Memory). Claims can request specific size and access modes (e.g., can be mounted once read/write or many times read-only).

1. To keep the sensor data safe during Pod restarts, you would create a new Persistent Volume Claim. 

    vi pvc.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mariadb-pv-claim
  labels:
    app: iot-backend
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

    kubectl apply -f pvc.yaml

or deploy directly using this command:

    kubectl create -f https://raw.githubusercontent.com/fwardzic/bthfy20/master/manifests/pvc.yaml

2. Verify if your persistent volume now has been claimed:

    kubectl get pvc mariadb-pv-claim

### Task 3: Declare secret with credentials for MariaDB

A **Kubernetes Secret** is an object that contains a small amount of sensitive data such as a password, a token, or a key. Such information might otherwise be put in a Pod specification or in an image; putting it in a Secret object allows for more control over how it is used, and reduces the risk of accidental exposure.

The MariaDB container image uses an environment variable named as 'MYSQL\_ROOT\_PASSWORD', it hold the root password required to access the database. So you would create a new secret with 'password' key (value as 'cisco123') which would later be used in mariaDB deployment yaml file.

    kubectl create secret generic mariadb-root-pass --from-literal=password=cisco123

### Task 4: Create MariaDB Deployment

1. Deploy MariaDB, using either command below, or copy paste the manifest to an empty file on the server and apply.

    vi mariadb.yaml

```yaml
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: iot-backend-mariadb
  labels:
    app: iot-backend
spec:
  selector:
    matchLabels:
      app: iot-backend
      tier: mariadb
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: iot-backend
        tier: mariadb
    spec:
      containers:
      - image: mariadb:10.3
        name: mariadb
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mariadb-root-pass
              key: password
        ports:
        - containerPort: 3306
          name: mariadb
        volumeMounts:
        - name: mariadb-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mariadb-persistent-storage
        persistentVolumeClaim:
          claimName: mariadb-pv-claim
```

    kubectl apply -f mariadb.yaml

or deploy directly using this command:

    kubectl create -f https://raw.githubusercontent.com/fwardzic/bthfy20/master/manifests/mariadb_deployment.yaml

2. check deployment:

    kubectl get deployment
    kubectl get pods

3. Expose MariaDB service as ClusterIP:

    kubectl expose deployment iot-backend-mariadb --type=ClusterIP --target-port=3306 --port=3306 --name mariadb-service

>NOTE: every user will execute the same command creating objects with the same names, however each student is using different namespace, therefore names can overlap.

4. check mariaDB service:

    kubectl get service mariadb-service

### Task 4: Deploy MQTT-to-DB-Agent microservice

1. Deploy mqtt_db_agent microservice, using either command below, or copy paste the manifest to an empty file on the server and apply.

    vi mqtt.yaml

```yaml
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: iot-backend-mqtt-db-agent
  labels:
    app: iot-backend
    tier: mqtt-db-agent
spec:
  selector:
    matchLabels:
      app: iot-backend-mqtt-db-agent
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: iot-backend-mqtt-db-agent
    spec:
      containers:
      - image: fwardzic/mqtt_db_plugin:latest
        name: mqtt-db-agent
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mariadb-root-pass
              key: password
```

    kubectl apply -f mqtt.yaml

or deploy directly using this command:

    kubectl create -f https://raw.githubusercontent.com/fwardzic/bthfy20/master/manifests/mqtt_db_agent_deployment.yaml

2. MQTT is not exposed, rather it is connecting to AWS IoT platform (over proxy which is hardcode in container) and writing data to MariaDB, leveraging mariadb-service to reach the database.

check deployment and pod status. 

    kubectl get deployments
    kubectl get pods

3. Check logs to see whether MQTT agent successfully connected to AWS IoT platform, create database and tables in MariaDB and finally start receiving data.

    kubectl logs <mqtt_pod_name>

### Task 5: Deploy RestAPI agent

1. This microservice reads data from database, and expose it over HTTP, so it will be later consumed by Frontend server using RestAPI channel rather direct connection to the database.

    vi restapi.yaml

```yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: iot-backend-rest-api-agent
  labels:
    app: iot-backend-rest-api-agent
spec:
  replicas: 2
  selector:
    matchLabels:
      app: iot-backend-rest-api-agent
      tier: rest-api-agent
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: iot-backend-rest-api-agent
        tier: rest-api-agent
    spec:
      containers:
      - image: pradeesi/rest_api_agent:v1
        name: rest-api-agent
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mariadb-root-pass
              key: password
```

    kubectl apply -f restapi.yaml

or deploy directly using this command:

    kubectl create -f https://raw.githubusercontent.com/fwardzic/bthfy20/master/manifests/restapi.yaml

2. Expose RestAPI agent, so Frontend server can read data using RestAPI. This time we will expose service externally so you can see the raw data in your browser:

    kubectl expose deployment iot-backend-rest-api-agent --type=NodePort --target-port=5050 --port=5050 --name=rest-api-service

You need to find the NodePort and Kubernetes Node external IP to access the 'rest-api-agent.

3. Use the following command to display the port exposed by 'rest-api-service' -
    
    kubectl get service rest-api-service

4. Use the following command to display the 'External-IP' of you kubernetes nodes -

	  kubectl get nodes -o wide

5. Note down the Node External IP Address and NodePort Service Port Number. These values would be used in next section for deploying the frontend app as the environment variables values ('**BACKEND\_HOST**' and '**BACKEND\_PORT**'). 
You can open web browser and enter IP:port values like this:

	http://<kubernetes node's external ip>:<nodePort>/cities
	
	http://<kubernetes node's external ip>:<nodePort>/temperature
	
	http://<kubernetes node's external ip>:<nodePort>/humidity
	
	http://<kubernetes node's external ip>:<nodePort>/sensor_data/city

## Task 6: Deploy frontend microservice

1. In the previous step you noted the the external IP and port. Now we have to pass this information inside frontend container. we will leverage ConfigMap and it will be used by POD to import those values as environment variables inside container.

    kubectl create configmap frontend-to-backend --from-literal=BACKEND_HOST=<any_node_IP> --from-literal=BACKEND_PORT=<port>

2. Deploy Frontend server then:

    vi frontend.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iot-frontend
  labels:
    app: iot-frontend
spec:
  selector:
    matchLabels:
      app: iot-frontend
      tier: web
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: iot-frontend
        tier: web
    spec:
      containers:
      - image: fwardzic/frontend_server
        name: frontend-server
        envFrom:
        - configMapRef:
            name: frontend-to-backend
      - image: fwardzic/nginx_srvr
        name: nginx-server
```

or use direct command:

    kubectl apply -f https://raw.githubusercontent.com/fwardzic/bthfy20/master/manifests/frontend.yaml

3. Expose our frontend webserver externally.

    kubectl expose deployment iot-frontend --name frontend-service --type=NodePort --port=80 --target-port=80

Check external port again:

    kubectl get service iot-frontend

in the browser you can now type:

    http://<kubernetes node's external ip>:<nodePort>

Click on Open Dashboard


Voila !