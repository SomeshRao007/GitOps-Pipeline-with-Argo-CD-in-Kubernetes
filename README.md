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

![Screenshot 2024-04-02 152145](https://github.com/SomeshRao007/GitOps-Pipeline-with-Argo-CD-in-Kubernetes/assets/111784343/59723569-107d-4da7-8a28-70fa3b77e69f)

If you are running a VMware Virtual machine then please wait it will take time as per allocated resources.
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

Editing __argocd-server__ service from cluster ip to nodeport so can we can access it. Simply check for _type_ at the bottom of the document, which will be opened after running this command: 

~~~
kubectl edit svc argocd-server -n argocd

service/argocd-server edited
~~~

now, after editing that we need to create a http and https tunnel for us to use.

~~~
minikube service argocd-server -n argocd
~~~

you will get a IP address something like 192.27..... its a private ip you can just copy paste that in your browser.

Or as an alternative you can do port forwarding 

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

To Fetch username and password for Admin (as we are😅) follow the steps below:
all confidentails data is stored in secret servers

~~~
kubectl get secret -n argocd
~~~
~~~
NAME                          TYPE     DATA   AGE
argocd-initial-admin-secret   Opaque   1      30m
argocd-notifications-secret   Opaque   0      102m
argocd-secret                 Opaque   5      102m
~~~

we will open the file as if we are going to edit it but we will copy the password, you can use describe command too but it wont work for our purpose since, it will hide all the passwords. 

The password you copied is an encrypted password we will decrypt it with:

~~~
echo <password> | base64 --decode #to decode the password
~~~

Sign up into argocd by entering ipaddress given by minikube tunnel & you willsee a login page 
#### vollaaa!!

Lastly, installation of Argo Rollouts
Again, we install the Argo Rollouts controller in your Kubernetes cluster, by following the official guide. but i will share the commands i used. 

~~~
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
~~~
this will work.

And for kubectl plugins:
basically we are downloading that scrip file __curl__, then making it executable file with __chmod +x__ and we are running that file with __./__ and moving the output to /usr/local/bin/kubectl-argo-rollouts location.

~~~
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x ./kubectl-argo-rollouts-linux-amd64
sudo mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
~~~

installation is complete. 

## Task 2: Creating the GitOps Pipeline
## Dockerize the Application
we will build a Docker image for our web application and push it to a _**public container registry**_ of your choice.


### Creating Docker Image 

For docker image, i am using my porfolio website so starting with cloning my git repo u can use you can use your own website: 

~~~
git clone https://github.com/SomeshRao007/SomeshRao007.github.io.git

cd SomeshRao007.github.io.git
~~~

how to create a image for my website? basically we need to install all dependencies if node.js or anyother stack based then refer docker hub for that image and take a glance at commands. For me it was httpd, so i search at docker hub for httpd and found how to write docker commands follow: https://hub.docker.com/_/httpd#:~:text=Create%20a%20Dockerfile%20in%20your%20project

Lets create a Dockerfile

~~~
vi Dockerfile 
~~~

write this in that file

~~~
FROM httpd:2.4
COPY ./ /usr/local/apache2/htdocs/
~~~

FROM command takes the base image of http2.4 and then with COPY command, we are copying everthing in our local directory to server location. 
to save esc wq

__Next step building a image__ 

```
docker build -t <your image name> <your docker file location use . if you are in the same dir>
```

To check image

~~~
docker images
~~~

> Note:
> > Docker internally runs on a client/server architecture. In particular, when you run docker build, it creates a tar file of the directory you specify, sends that tar file across a socket to the Docker daemon, and unpacks it there. (True even on a totally local system.)

~~~
docker run -itd -port 3000:80 --name website myweb
~~~

We are running docker in background __-itd tag__ on port 3000 (__container port : host port__) and name of the container website from the image _myweb_.

to check go to 127.0.0.1/3000 (or type localhost:3000) 

![Screenshot 2024-04-02 191017](https://github.com/SomeshRao007/GitOps-Pipeline-with-Argo-CD-in-Kubernetes/assets/111784343/4992705b-a474-4ace-b96b-85882d52e2a8)

__vollaa!!__


Now, lets push this into our registry, I am using AWS ECR as i am comfortable with AWS services SignIn into the AWS console and naviagte to ECR (Elastic contriner registry) in search box. I am creating a a public registry and if you want enhanced security enable encryption but, i am leaving it disabled.

if you go to the registry and click on push commands it will give you commands follow that: 

To retrieve an authentication token and authenticate your Docker client to your registry.

```
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/n8y8c3i1
```

After the build completes, tag your image so you can push the image to this repository:

~~~~
docker tag argocdtest:latest public.ecr.aws/n8y8c3i1/argocdtest:latest
~~~~

Run the following command to push this image to your newly created AWS repository

~~~
docker push public.ecr.aws/n8y8c3i1/argocdtest:latest
~~~

After a sucessfull push u will see this: 

```
The push refers to repository [public.ecr.aws/n8y8c3i1/argocdtest]
ac324cd774ab: Pushed 
d91409980a4e: Pushed 
2ef15bc08e25: Pushed 
1f9ac7ca16f1: Pushed 
5f70bf18a086: Pushed 
3ee8143dd880: Pushed 
a483da8ab3e9: Pushed 
latest: digest: sha256:98ba59f775811691803162b8d95418530e81623f3fb88144c891b9ea794a7adc size: 1784
```

![Screenshot 2024-04-02 191700](https://github.com/SomeshRao007/GitOps-Pipeline-with-Argo-CD-in-Kubernetes/assets/111784343/d647caae-6897-428c-90d2-30819ccdadcf)


### create a kubernetes deployment resource

> It is best practice to create a namespace to work and test... why ?
>> Imagine a Kubernetes cluster with hundreds or even thousands of applications running inside Pods. Each of these Pods has its own set of Kubernetes objects, such as Deployments, Services, ConfigMaps, Secrets, and more. When all these resources are deployed in the same Namespace, it can be difficult to know which resource belongs to which application. there is a high risk of accidentally updating the wrong resource and breaking a running application.Namespaces provide a solution to this problem. By grouping resources into logically isolated groups, Namespaces help reduce the chance of making mistakes when managing resources.

To create a namespace to test and work env:

~~~
kubectl create namespace testenv
~~~

Let's create a pod first and understand the basics 

~~~
vim pod.yaml
~~~

write this to create a pod that takes image from our ECR repositry 

~~~
apiVersion: v1
kind: Pod
metadata:
   name: website
spec:
   containers:
   - name: website
     image: public.ecr.aws/n8y8c3i1/argocdtest:latest
     ports:
     - containerPort: 80
~~~

inorder to create this pod and keep it up running we do:

~~~
kubectl apply -f pod.yaml
~~~

check the pod by:

~~~
kubectl get pods -n <namespace>
~~~

To get complete Info about the pod:
pod/<name>
~~~
kubectl describe pod/website
~~~

Output:
- - - -
~~~
Name:             website
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Tue, 02 Apr 2024 21:37:33 +0700
Labels:           <none>
Annotations:      cni.projectcalico.org/containerID: 80219f618b64694e7da455ce1553c9bffe53a5293c6694f1dd6d25aaf75675fa
                  cni.projectcalico.org/podIP: 10.244.120.126/32
                  cni.projectcalico.org/podIPs: 10.244.120.126/32
Status:           Running
IP:               10.244.120.126
IPs:
  IP:  10.244.120.126
Containers:
  website:
    Container ID:   docker://0b4aa8ea49c0e3ca994c0cff107ba86c1a55907dfa5e66382a91fb5ff9dc7226
    Image:          public.ecr.aws/n8y8c3i1/argocdtest:latest
    Image ID:       docker-pullable://public.ecr.aws/n8y8c3i1/argocdtest@sha256:98ba59f775811691803162b8d95418530e81623f3fb88144c891b9ea794a7adc
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 02 Apr 2024 21:38:43 +0700
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qhcw7 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-qhcw7:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m56s  default-scheduler  Successfully assigned default/website to minikube
  Normal  Pulling    2m54s  kubelet            Pulling image "public.ecr.aws/n8y8c3i1/argocdtest:latest"
  Normal  Pulled     107s   kubelet            Successfully pulled image "public.ecr.aws/n8y8c3i1/argocdtest:latest" in 1m7.082979603s 
  Normal  Created    107s   kubelet            Created container website
  Normal  Started    107s   kubelet            Started container website
~~~

Now you cannot access this my website directly, as our pods are handled by minikube. Minikube create a vm at the beginneing so, to access our website for now go inside minikube for that simply type:

~~~
minikubve ssh 
~~~

After SSH into to the minikube, reach our website which is on IP mentioned in above describe part.

~~~
curl 10.244.120.126
~~~

Since we are doing curl we will get underlying HTML page i dont want to show that but i can gurantee that if you follow the steps it will work!!

__vollaaa!!__

> Note:
> > if u delete the pod youcannot access to my webite (TRY ON UROWN)


## Deploy the Application Using Argo CD

we will create a Kubernetes manifests(deployment and service files) in our repository to use the Docker image we pushed eariler.
After that, we will set up Argo CD to monitor our repository and automatically deploy changes to your Kubernetes cluster.

Enough chit chat!! 
now, lets create deployment.yml file 

~~~
vim deployment.yaml
~~~

~~~
apiVersion: apps/v1
kind: Deployment
metadata:
  name: website-deployment
  labels:
    app: website
spec:
  replicas: 3
  selector:
    matchLabels:
      app: website
  template:
    metadata:
      labels:
        app: website
    spec:
      containers:
      - name: website
        image: public.ecr.aws/n8y8c3i1/argocdtest:latest
        ports:
        - containerPort: 80
~~~

Quick breakdown
- - - -
**apiVersion:** Under the hood, a Kubernetes cluster exposes its functionality via a REST API. We seldom interact with this API directly because we have kubectl that takes care of it for us. 

**kind:** It represents the type of object that the configuration file defines. 

**metadata.name:** Quite simply, the name of the object. It’s how we and Kubernetes refer to it.

**metadata.labels:** These help us further categorize cluster objects. These have no real effect in the system

**spec:** This contains the actual functional specification for the behavior of the deployment.

**spec.replicas:** The number of replica pods that the deployment should create.

**spec.template:** Specifies the configuration of the pods that will be part of the deployment

**spec.template.metadata.labels:** Very similar to metadata.labels. The only difference is that those labels are added to the deployment while these ones are added to the pods.
- - - -

To deploy the menifest file we created:

~~~
kubectl apply -f deployment.yml

deployment.apps/website-deployment created
~~~

To check deploymnt, replica sets and pods:

~~~
kubectl get deploy
~~~

~~~
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
website-deployment   3/3     3            3           75s
~~~

~~~
kubectl get rs
~~~

~~~
NAME                            DESIRED   CURRENT   READY   AGE
website-deployment-84c497887b   3         3         3       86s
~~~

~~~
kubectl get pods 
~~~

~~~
NAME                                  READY   STATUS    RESTARTS   AGE
website-deployment-84c497887b-b8xzp   1/1     Running   0          97s
website-deployment-84c497887b-dlsbd   1/1     Running   0          97s
website-deployment-84c497887b-rnv4s   1/1     Running   0          97s
~~~

> If you notice, our replical set is set 3, Hence the number of pods. The number of replica set is actually decided by number of concrete users accessing the following application. So it is an Important decision that a DevOps engineer takes is to take adequate number of replica count. That means if you delete any pod or due to some disaster, a pod got delted then it will regenerate to desire count (similar to ASG in AWS).


To get more info or detailed info of our pods, clusters etc etc access the dashboard :

~~~
 minikube dashboard
~~~


Now lets create services 

Its all about how to interact with the pods, We use services to expose an application running in a set of pods to the world outside the cluster.

~~~
vim service.yml 
~~~

~~~
apiVersion: v1
kind: Service
metadata:
  name: website-service
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: website
  ports:
    - name: "http"
      protocol: TCP
      port: 80
      targetPort: 3000
      nodePort: 30080
~~~

Quick Breakdown
- - - -
**spec.ports.port:** The port that will be exposed by the service.

**spec.ports.targetPort:** The port that the service will target in the container.

**spec.ports.nodePort:** The port that the service will expose in all the nodes of the cluster
- - - -

> Note
>> The port field, the website service is available at node port 30080 now, you can access the web page at http://<node IP address>:30080.

** To set up ArgoCD **

Go to the brower page where you open [GO](####vollaaa!! "Argocd UI")




