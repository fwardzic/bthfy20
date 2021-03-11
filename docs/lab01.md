## Lab 01 - Kubernetes lab access

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