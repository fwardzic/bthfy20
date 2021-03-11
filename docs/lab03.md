## Lab03 Services

### 1. Exposing app internally using ClusterIP:

In the previous lab you have already seen how to expose your app internally. Check the detials:

- what is the ClusterIP address assigned to the hello-rolling service?
- what is the endpoint list for that service?
- what is the port of the service and port of the target pods?

Login to web-test pod, and check environment variables.

### 2. Exposing guestbook app:

Expose previously deployed redis-master Replica Set using internally using ClusterIP.

```kubectl expose replicaset redis-master```

Expose previously deployed redis-slave Replica Set using internally using ClusterIP.

```kubectl expose replicaset redis-slave```

- what is the ClusterIP address assigned to the redis-master and redis-slave services?
- what is the endpoint list for each service?
- what is the port of the service and port of the target pods?

Expose previously deployed frontend deployment externally using LoadBalancer service:

```kubectl expose deployment frontend --target-port=80 --port=80 --type=LoadBalancer```

check service status, note External-IP address, wait couple minutes until public IP address will be assigned.

Open your browser and enter that IP address, you should be able to see Guestbook app. 
To verify end-to-end if an app is working - type some text in the field and submit. If the frontend has connectivity to the redis database, it will save the word, and display is under the "Guestbook" button. 