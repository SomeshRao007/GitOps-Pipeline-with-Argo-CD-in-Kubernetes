# GitOps-Pipeline-with-Argo-CD-in-Kubernetes

## Overview
we will dockerize a simple web application, deploy it to a Kubernetes cluster using Argo CD, and manage its release process with Argo Rollouts. This is divided into four main tasks: 
1) setup and configuration.
2) creating the GitOps pipeline.
3) implementing a canary release.
4) Clean up & documentation.

## Task 1: Setup and Configuration
Firstly, Create a Github repository and host your source code in the same repository.
Now log into your Linux system and install minikube, check for the installation script in my repositry https://github.com/SomeshRao007/Autoscalling_minikube_helm.git for installation script one and two, clone it then run:

```
bash installation_script.sh
```
or just download it from offical document. 

to check version if you dont have download it

```
minikube version
```
you can specify your desired driver using the --driver flag and add Docker as your driver:

```
minikube start --driver=docker
```
Use the following command to check your Kubernetes cluster status

```
minikube status
```
If output is this, then good to go. Otherwise check your installation.
```
somesh@somesh-virtual-machine:~/Desktop$ minikube status 
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```
I wanted to check is there any pods running for that

```
kubectl get pods -A

tag -A means in all namespaces
```
Output:

~~~
NAMESPACE     NAME                                       READY   STATUS    RESTARTS      AGE
kube-system   calico-kube-controllers-558d465845-77tsq   1/1     Running   1 (56s ago)   67s
kube-system   calico-node-fxzz5                          1/1     Running   0             67s
kube-system   coredns-5dd5756b68-qgzhf                   1/1     Running   1 (53s ago)   67s
kube-system   etcd-minikube                              1/1     Running   0             79s
kube-system   kube-apiserver-minikube                    1/1     Running   0             79s
kube-system   kube-controller-manager-minikube           1/1     Running   0             79s
kube-system   kube-proxy-gm7kt                           1/1     Running   0             67s
kube-system   kube-scheduler-minikube                    1/1     Running   0             79s
kube-system   storage-provisioner                        1/1     Running   1 (36s ago)   76s
~~~

 



- Install Argo CD on Your Kubernetes Cluster: Follow the official Argo CD documentation to install and set up Argo CD.
- Install Argo Rollouts: Install the Argo Rollouts controller in your Kubernetes cluster, following the official guide.







