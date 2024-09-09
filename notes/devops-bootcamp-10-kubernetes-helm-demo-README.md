[Notes overview](https://github.com/TheAbys/devops-bootcamp-certification-project/blob/master/README.md)

# 16 - Helm Demo - Managed K8s cluster

Create linode account

Create new cluster, shared CPU, 4GB, 2 Nodes (48$/month)

After the cluster was created download the kubeconfig.yaml which contains all necessary secret and auth information

On local machine make the kubeconfig.yaml accessable

    chmod 400 test-kubeconfig.yaml
    export KUBECONFIG=test-kubeconfig.yaml
    kubectl get node

Now we can see the linode nodes.

Installing helm and add bitnami repo
https://helm.sh/docs/intro/install

    helm repo add bitname https://charts.bitname.com/bitnami

For each repo you can lookup the actual values that can be overwritten.
This way the chart can be customized.
For example if the default chart uses AWS storage but you actually wanna use linode block storage.

    helm install mongodb --values helm-mongodb.yaml bitnami/mongodb

Installing mongo-express as internal service

    kubectl apply -f helm-mongo-express.yaml

Installing ingress controller through helm chart

    helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true

Cloud providers have there own Load/NodeBalancer which is automatically created with the last statement.
Its the main entrypoint into our cluster.

YOUR_HOST_DNS_NAME in the helm-ingress.yaml must be replaced by the linode hostname (or any other hostname whereever it's hosted)

    kubectl apply -f helm-ingress.yaml

    kubectl get ingress

Scale down the statefulsets

    kubectl scale --replicas=0 statefulset/mongodb

    kubectl get pods

    kubectl scale --replicas=3 statefulset/mongodb

Uninstall

    helm uninstall mongodb

Delete the cluster from linode