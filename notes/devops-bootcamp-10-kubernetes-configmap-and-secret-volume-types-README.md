[Notes overview](https://github.com/TheAbys/devops-bootcamp-certification-project/blob/master/README.md)

# 12 - ConfigMap & Secret Volume Types

Create the deployment

    kubectl apply -f mosquitto-without-volumes.yaml

Exec into the container

    kubectl exec -it mosquitto-6564db7cb5-lrlql -- /bin/sh

Update the configuration for the mosquitto deployment, add volumes and mounts

Execute all configuration files

    kubectl apply -f config-file.yaml
    kubectl apply -f secret-file.yaml
    kubectl apply -f mosquitto.yaml


The config file and secret file get mounted into the pod.
This are local volume types.

# 13 - StatefulSet - Deploying Stateful Applications

Only the master pod can read and write.
The replica pods can only read.
Everytime a new replica pod is created it copies the data from the previous pod.
This also means theoretically, if pods never die, the data will not get lost. But this means obviously if all pods crash at the same time data is lost.
Therefore it is still required to keep data peristant.

StatefulSet pods get fixed ordered names while a Deployment does not.
Deletion will start from the max number and then delete the number zero.

The created pods also get a fixed individual DNS name like mysql-0.svc2 which is podname + servicename.

Containerized environments are not perfect for Stateful applications.

# 14 - Managed Kubernetes Services Explained

Basically the Managed Services are provided by all Cloud providers. The all provide identical solutions but your still vendor-locked in as it's not easy to switch afterwards.

The managed services handle all the control plane nodes and everything storage related.
It automates a lot of stuff and therefore safes time.

# 15 - Helm - Package Manager for Kubernetes

Search for packages via CLI or artifacthub.io
    
    helm search hub icinga

Helm is also a templating engine, not just a package manager

Throw values.yaml you can override the variables in those templates and therefore reduce the amount of files required
The values can also be set through the CLI command

helm install <chartname> starts with creating a version 1.0 for this chart and everytime you do helm upgrade <chartname> it increases the version number.
This also means that helm rollback <chartname> can be used to switch back to the old version.