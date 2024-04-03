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

Secondly, we have Install Argo CD on Your Kubernetes Cluster
We can follow the official Argo CD documentation to install and set up Argo CD. But I will list all the command for you:

To Install: 
~~~
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
~~~

To confirm the installation check for the running pods :

~~~
kubectl get pods -A
~~~
~~~
NAMESPACE     NAME                                                READY   STATUS    RESTARTS       AGE
argocd        argocd-application-controller-0                     1/1     Running   0              51s
argocd        argocd-applicationset-controller-75b78554fd-55r6b   1/1     Running   0              52s
argocd        argocd-dex-server-869fff9967-k5mlz                  1/1     Running   0              52s
argocd        argocd-notifications-controller-5b8dbb7c86-rldkm    1/1     Running   0              52s
argocd        argocd-redis-66d9777b78-smc4h                       1/1     Running   0              52s
argocd        argocd-repo-server-7b8d97c767-96rzh                 1/1     Running   0              52s
argocd        argocd-server-5c797497fb-xklvq                      1/1     Running   0              51s
~~~
If you see all these pods up and running then you are good to go. 

Lets check services:
for default namespace
~~~
kubectl get svc 
~~~

for a specific namespace use -n tag:

~~~
kubectl get svc -n argocd
~~~
Output:

~~~
NAME                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP   10.97.219.135   <none>        7000/TCP,8080/TCP            2m7s
argocd-dex-server                         ClusterIP   10.106.76.191   <none>        5556/TCP,5557/TCP,5558/TCP   2m7s
argocd-metrics                            ClusterIP   10.111.161.2    <none>        8082/TCP                     2m7s
argocd-notifications-controller-metrics   ClusterIP   10.103.253.61   <none>        9001/TCP                     2m7s
argocd-redis                              ClusterIP   10.111.1.230    <none>        6379/TCP                     2m7s
argocd-repo-server                        ClusterIP   10.111.241.53   <none>        8081/TCP,8084/TCP            2m7s
argocd-server                             ClusterIP   10.100.12.23    <none>        80/TCP,443/TCP               2m7s
argocd-server-metrics                     ClusterIP   10.97.68.20     <none>        8083/TCP                     2m7s
~~~




- Install Argo Rollouts: Install the Argo Rollouts controller in your Kubernetes cluster, following the official guide.







