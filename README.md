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
or just download from offical document. 

to check version if you dont have download it

```
minikube version
```
you can specify your desired driver using the --driver flag and add Docker as your driver:

```
minikube start --driver=docker
```

- Install Argo CD on Your Kubernetes Cluster: Follow the official Argo CD documentation to install and set up Argo CD.
- Install Argo Rollouts: Install the Argo Rollouts controller in your Kubernetes cluster, following the official guide.







