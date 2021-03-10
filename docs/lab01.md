## Lab 01 - Kubernetes lab access

### Play with Kubernetes

Go to page: [https://labs.play-with-k8s.com/](https://labs.play-with-k8s.com/)

Login using your github or docker account.

<img src="https://raw.githubusercontent.com/fwardzic/bthfy20/master/docs/images/play-with-k8s-login.png">

Add new instance:

<img src="https://raw.githubusercontent.com/fwardzic/bthfy20/master/docs/images/k8s-add-1node.png">

Initilaize kubernetes cluster:

<img src="https://raw.githubusercontent.com/fwardzic/bthfy20/master/docs/images/k8s-kubeadm-init.png">

Once done, check status of cluster - you should see it is in the "NotReady" state. 

<img src="https://raw.githubusercontent.com/fwardzic/bthfy20/master/docs/images/k8s-kube-1node-notready.png">

Initialize pod networking CNI

```kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml```

Add one more instance. 

<img src="https://raw.githubusercontent.com/fwardzic/bthfy20/master/docs/images/k8s-kube-1node-notready.png">