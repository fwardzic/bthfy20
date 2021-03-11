## Lab 01 - Kubernetes lab access

### Google cloud access

Download google cloud SDK appropriate to your OS:

[https://cloud.google.com/sdk/docs/quickstart](https://cloud.google.com/sdk/docs/quickstart)

Download will take a bit of time. Once downloaded, please follow instruction to initialize:

[https://cloud.google.com/sdk/docs/quickstart#initializing_the](https://cloud.google.com/sdk/docs/quickstart#initializing_the)

Use this command:

```gcloud init```

Use following options:
- Project: fwardz001-poc-ci1s
- Zone: europe-west-4a

You will be asked to provide credentials, use your cisco.com address.

Install kubectl - the command line tool for managing Kubernetes clusters:

```gcloud components install kubectl```

Once installed, obtain access to the following cluster:

```gcloud container clusters get-credentials autopilot-bth-2021 --region europe-west4 --project fwardz001-poc-ci1s```

Verify you have access to cluster and you are able to list cluster nodes.

```kubectl get nodes```

---

### Play with Kubernetes

Go to page: [https://labs.play-with-k8s.com/](https://labs.play-with-k8s.com/)

Login using your github or docker account.

<img src="https://raw.githubusercontent.com/fwardzic/bthfy20/master/docs/images/play-with-k8s-login.png">

Add new instance:

<img src="https://raw.githubusercontent.com/fwardzic/bthfy20/master/docs/images/k8s-add-1node.png">

Initialize kubernetes cluster:

<img src="https://raw.githubusercontent.com/fwardzic/bthfy20/master/docs/images/k8s-kubeadm-init.png">

Record kubeadm join command from the output.
Once done, check status of cluster - you should see it is in the "NotReady" state. 

<img src="https://raw.githubusercontent.com/fwardzic/bthfy20/master/docs/images/k8s-kube-1node-notready.png">

Initialize pod networking CNI

```kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml```

Wait until status of the cluster node will be Ready.

Add one more instance. 

<img src="https://raw.githubusercontent.com/fwardzic/bthfy20/master/docs/images/k8s-kube-1node-notready.png">

Copy kubeadm join command from previous output from the first node and execute it on the second node.

<img src="https://raw.githubusercontent.com/fwardzic/bthfy20/master/docs/images/k8s-join-command.png">

Wait couple minutes and check status of the nodes 

```kubectl get nodes -o wide```

This command will print IP addresses of your nodes, in addition. Status of the nodes should be "Ready" now.