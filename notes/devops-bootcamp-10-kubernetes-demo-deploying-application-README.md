[Notes overview](https://github.com/TheAbys/devops-bootcamp-certification-project/blob/master/README.md)

# 4 - Minikube and kubectl - Local Kubernetes Cluster

Installing minikube https://minikube.sigs.k8s.io/docs/start/

    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube

Starting minikube and checking for the status

    minikube start -driver docker

    minikube status

List kubectl components

    kubectl get pods
    kubectl get services
    kubectl get nodes
    ...

Creating an example deployment and checking its state

    kubectl create deployment nginx-depl --image=nginx
    kubectl get deployments #lists 0/1 READY, the deployment is not ready yet
    # did not work, because kubectl was not able to pull the image due to company proxy

Use proxy for starting minikube (https://minikube.sigs.k8s.io/docs/handbook/vpn_and_proxy/)

    export HTTP_PROXY=http://http-proxy.krones-deu.krones-group.com:3128
    export HTTPS_PROXY=http://http-proxy.krones-deu.krones-group.com:3128
    export NO_PROXY=localhost,127.0.0.1,10.96.0.0/12,192.168.59.0/24,192.168.49.0/24,192.168.39.0/24
    minikube start


    kubectl describe pod <podname>
    kubectl logs <podname>

    kubectl create deployment mongo-deployment --image=mongo
    # did not work at first, deleted the deployment and recreated it, then it worked
    kubectl delete deployment mongo-deployment

Connect into a pod

    kubectl exec -it mongo-deployment-64d449895-l9x8j -- bin/bash
    # QUESTION: how does that work with a deployment that is replicated multiple times? in which container am i connecting?

Apply a new file

    kubectl apply -f nginx-deployment.yaml

# 6 - YAML Configuration File

Creating or updating a deployment

    kubectl apply -f nginx-deployment.yaml
    kubectl apply -f nginx-service.yaml
    kubectl delete -f nginx-deployment.yaml

    kubectl get services
    kubectl get svc

Output with more information like IP address

    kubectl get pods -o wide

# 7 - Complete Demo Project - Deploying Application in Kubernetes Cluster

Create base64 password in bash for usage in secrets.
This is still not secure and not encrypted!

    echo -n 'password' | base64

Prepared all the deployments, services, secret and configmap
When the application should be exposed and opened in the browser I had massive problems getting it to run.
On Linux (at least with the minikube docker driver) there was no automatic created tunnel from localhost to the cluster. This resulted in having to find the solution online.

Solution: Forward the port for the pod (mongo express)

    kubectl port-forward <podname> <localhostport>:<containerport>

I had an error in one of the service definitions, but it was really just coincidence that I found this typo.

# 8 - Namespace

    kubectl get namespace

kube-system is not meant for use, it is for system processes like from the control plane or kubectl  

kube-public public accessiable data, accessable through

    kubectl cluster-info

kube-node-lease, holds information about heartbeat of nodes, each node gets its own object

default, is the default if you did not create your own namespace
best practise: do not use the default namespace for larger applications/infrastructure, as it get's pretty overwhelming
Separate for example everything database related into a database namespace or monitoring related stuff goes into a monitoring namespace etc.


You can create your own namespace through:

    kubectl create namespace <namespacename>

Can also be created through the YAML configuration files


Volume and node are not within a namespace, they are global.

Selecting a configmap of a specific namespace (here default namespace)

    kubectl get configmap -n default

While applying a configuration we could append the namespace argument and set the namespace, but a better way is to use configuration files under metadata>namespace

    kubectl apply -f mongo-configmap.yaml --namespace=my-namespace

Set the current "default namespace" to a specific namespace

    kubectl config set-context --current --namespace=my-namespace

Installing kubectx

    sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
    sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
    sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens

    mkdir -p ~/.oh-my-zsh/custom/completions
    chmod -R 755 ~/.oh-my-zsh/custom/completions
    ln -s /opt/kubectx/completion/_kubectx.zsh ~/.oh-my-zsh/custom/completions/_kubectx.zsh
    ln -s /opt/kubectx/completion/_kubens.zsh ~/.oh-my-zsh/custom/completions/_kubens.zsh
    echo "fpath=($ZSH/custom/completions $fpath)" >> ~/.zshrc

    source ~/.zshrc


# 9 - Services - Connecting to Applications inside cluster

There are four types of services ClusterIP, Headless, NodePort and LoadBalancer
Pods normally do not have fixed IP addresses, while a Service has.

A service works over all pods on different nodes. There is always one single service basically on all nodes which handles all matched pods on any node.

## Most Common/default: ClusterIP Service (internal service)

Service acts like a LoadBalancer and moves requests to the pod, relevant pods are selected through "selector".

        [SERVICE]
+-----------+----------+
|           |          |
[POD]     [POD]     [POD]

each pod has its own dynamic ip address while the service ip address is static
Which port is selected is random or probably based on current load of each pod

List all endpoints

    kubectl get endpoints

Describe a specific endpoint. If replicas of a deployment is set and multiple pods are deployed the endpoint addresses also show all addresses for this pods.

    kubectl describe endpoints mongo-express-service

    # example output
    Name:         mongo-express-service
    Namespace:    default
    Labels:       <none>
    Annotations:  endpoints.kubernetes.io/last-change-trigger-time: 2024-02-12T12:42:56Z
    Subsets:
    Addresses:          10.244.0.51,10.244.0.59
    NotReadyAddresses:  <none>
    Ports:
        Name     Port  Protocol
        ----     ----  --------
        <unset>  8081  TCP

    Events:  <none>

## Headless Service

Is mostly used for stateful applications like databases because in that case the pod replicas are not identical.
It is necessary for the pod to be able to connect to another pod to clone the database data.
Only on of multiple database pods must be allowed to write data, all other only can read.

The ip information for connection could be collected through the K8s API Server, but this would be unperformant (and makes App very reliant on K8s API)

When the clusterIP spec of the service is set to None the DNS lookup for that service will return the *IP addresses* of all the pods instead of the service. This results in multiple IP addresses and the client then must decide what to do with that. Connect to one, many or all of them.

## NodePort Service

Creates a service that is accessable on a specific port on the Node itself, therefore only one service per Node can have a nodePort (30000-32767) set.
Makes it possible to directly communicate with a service. This is good or practical for testing purposes but otherwise insecure. Normally the request goes to for example Ingress which then forwards to a service.

Extension of the ClusterIP type.

## LoadBalancer Service

AWS, GC, DigitalOcean, Linode, ... generate a LoadBalancer by default.
ClusterIP and NodePort are automatically set.
The service then is reachable directly but only from the LoadBalancer itself.

Extension of the NodePort type.

# 10 - Ingress - Connecting to Applications outside cluster

Activate ingress, it's an addon which is disabled by default. Alternatives do exist.

    minikube addons enable ingress

We do make a specific service available through ingress with our configuration dashboard-ingress.yaml

    kubectl apply -f dashboard-ingress.yaml

Get all pods, services, ... for specific namespace

    kubectl get all -n kubernetes-dashboard

Get information about ingress config

    kubectl get ingress -n kubernetes-dashboard

In /etc/host add following line:

    127.0.0.1 dashboard.com

Start the minikube tunnel for accessing the application through 127.0.0.1

    minikube tunnel

Here it's just working in the tutorial, but on my local Ubuntu machine it doesn't.
Even the port-forward solution I did previously for the other demo doesn't work here.

Checked the configuration of the existing dashboard service, but firstly there shouldn't be an issue and secondly there's nothing which I would change with my current knowledge.

    kubectl get service kubernetes-dashboard -n kubernetes-dashboard -o yaml

ALTERNATIVE to always adding namespace (installed previously)

    kubens kubernetes-dashboard

Can't get it to work with following line, as it has no nodeport defined

    minikube service kubernetes-dashboard -n kubernetes-dashboard

SKIPPING solving this issue for now...

Checking ingress

    kubectl describe ingress dashboard-ingress -n kubernetes-dashboard


# 11 - Volumes - Persisting Application Data

Persistant volumes are not namespaced and available to the whole cluster.
You can basically "link" Cloud storage, local storage, whatever as a "plugin" into Kubernetes.
Someone has to manage, meaning backup etc., the storage.

Local volume types violate the 2nd and 3rd requirement for data persistance, as they are tied to one specific node and do not survive cluster crashes.

## Persistant volume

Persistant volume is basically on the administrator side. An administrator must provide the storage whereever he thinks it's reasonable. For example in the cloud, self-hosted, ...
There are multiple types to choose as each storage works different and needs different configuration.

## Persistant volume claim

The persistance volume claim just claims a specific size on that peristant volume.
A developer knows his application or database requires x amount of file space. Therefore he can claim the space without ever knowing where it is hosted.

## Storage class

Can also by claimed by a persistant volume claim.
The storage class then provisions a persistant volume which then is actually claimed.