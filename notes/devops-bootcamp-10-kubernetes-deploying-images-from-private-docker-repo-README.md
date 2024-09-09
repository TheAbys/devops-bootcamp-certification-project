[Notes overview](https://github.com/TheAbys/devops-bootcamp-certification-project/blob/master/README.md)

# 17 - Deploying Images in Kubernetes from private Docker repository

At first login into aws ecr (again company proxy is required)
I use zsh:

    vim ~/.zshrc

Add following lines

export NO_PROXY=localhost,127.0.0.1,10.96.0.0/12,192.168.59.0/24,192.168.49.0/24,192.168.39.0/24
export HTTP_PROXY=http://http-proxy.krones-deu.krones-group.com:3128
export HTTPS_PROXY=http://http-proxy.krones-deu.krones-group.com:3128

Reload shell

    source ~/.zshrc

Check if variables are set

    env | grep PROXY

## login

For this to work the aws cli must be preconfigured

    aws configure

    aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin 561656302811.dkr.ecr.eu-central-1.amazonaws.com

This is not sufficent, as minikube won't have access to this stored credentials.

## ssh into minikube and configure there

Get the login password

    aws ecr get-login-password

SSH into minikube cluster

    minikube ssh

Login to ECR within the cluster

    docker login --username AWS -p eyJwYXlsb2FkIjoiMlg0RTlKMS92Z045Vy8yaE1TNjdXVHNjMGMwMXhtaFVIRHpwTWJTbDJBbXpmbjZGVkdBY041T3RZeThvcnByVS93YjdZY2pnaklhZ1BkeW83QU5pQXhuZ3NmOVg2ai93dDRlalROdno3SCt2K3hRTjRHTko0bXJ6MXNOeHNiUFdIcGJnWERlVGVtbGVFWGhFbUFGV1JMRktaQXR3MTQ0NUtDWnlPZ01OUzRvQkhLdGxRM1p6ckEzNEFCVjhXZXVlb0tVaEZobFFRY2J3Z0p2YUdReVh3OE5oRnNrOStpZzNqdE5YVmg4NlVibWs0TTdPdnh2Z0lRNXVaTEpwaE0wTXBtd05ERlNaMzNteW9KQ202MnZQblN6a09KRzVvaVI2RXkzT0h4S3RkTzJUYysxUndNbXVNdmdONHRJcU54Y29sUHp2Wm1rQWN0WlY0NFV3UEN5cFZ3TWZYNk1kRlpRc1VObENLMmR3cTl6bjdLdnBGTUc5L2M4cUxrcldLSWY3TjNYeTFyem5PUEFMd3ZWQ3ZSRjN3OXZmWGtZWTZNaGF2ZFNzYlBuOVlDU2tUaHlCaUZ0MVlWa0pFQ3FsektmWjlGQnhyYnZWWXhTQ3NaSGVkY1RpSm4rZVpHMUtiWmI0MDdHN1ViWFdsSFh6U3V5dVM4cWk5aFJoYUpvRDFQMG5uaS9iWkI1eGNkRS8wMHlOWFowNDRzbWVsZ3hYdkhvWDJtRFU3ZkF4bVBsVXo2M0s5RlBQOFBET1U0NnBZS3ljYlZna3RmdXdDMThVSTFRTmRscXRtY2Y5ZlJVOGtaUEZXL2ZNbHh1WkhHeUFnQWNkZjJPVHhaOEZnZU9UdzB3MHlIdkZoeVhhaW9ER3htNVBSWEd3WFU4MGNQQk83N3R0dUxNSGJ2UXJtM1J0OS85TzkxNVVMcUlhYll2UmJIbkNsbmhQT1FwTHNBbCs5cjR5UVlYWGdxaXozdURBalg5a1QzQ3pBZ2RNbHZRSjJ6UzhWTkdWNUlDQjE1aDByM2V0cmRCYlphMVBEUmtFb3l5Ky81MTlXYk9nNExsOXFKYXkzem1SR1FheG1kZ1hGZmFUUVNSSUl5TDAxRGFOLzYzUThtYTVBZXcyNnBZRllTN0xML3dOY2dXNS9jTCsxYUVzOFFDZTVUNExIWDVoVWU1SklwRUVKREp0aDQ4MlIrRjJBWTk0Q1AwSkt2cE5UWmNGZ3lpVHMzWUdlbWM1ZWdBditINDVDVlFOY3JFc3VURzdQWUt6QUJNUDVtMk9HdEhPWEE4M05NNzh3WWw3OENSYlhGYzVkamVSL3U3OWlTUzNIeElyNnl6bkVDVnZ0cVluV0dxdUxaMmorMHROSHFiU1hFMFBJTzM2N2w5c1RtQXpTaU9OUnF5NnhlRURqTDNqQzRoanFFSDFiZ0hnIiwiZGF0YWtleSI6IkFRRUJBSGgzellPZHRwQkJTVncvWTJhbjhCWElENGt3TFFmbm9UajV6d1p0R0pab1pRQUFBSDR3ZkFZSktvWklodmNOQVFjR29HOHdiUUlCQURCb0Jna3Foa2lHOXcwQkJ3RXdIZ1lKWUlaSUFXVURCQUV1TUJFRURKUGFYcHQ4eXJDcUhJVTZLZ0lCRUlBN3BvQnJxWHpyTmlUOVlDZ2x1cnRIMmltSUlPOWpBUzV1OGxHUTcxMllkUTB2WkFyMzNabWdlUHhXMU9Sc3ZmanAzYW9hMVIrZHNiMjQ3UTA9IiwidmVyc2lvbiI6IjIiLCJ0eXBlIjoiREFUQV9LRVkiLCJleHBpcmF0aW9uIjoxNzA3ODczMzQ4fQ== 561656302811.dkr.ecr.eu-central-1.amazonaws.com

Copy from minikube cluster to local

    minikube cp minikube:/home/docker/.docker/config.json /home/mueller/.docker/config.json

Convert to base64

    ~/.docker/config.json | base64

or use kubectl

    kubectl create secret generic my-registry-key \
    --from-file=.dockerconfigjson=/home/mueller/.docker/config.json \
    --type=kubernetes.io/dockerconfigjson

## doing all in one step

    kubectl create secret docker-registry my-registry-key-two \
    --docker-server=https://561656302811.dkr.ecr.eu-central-1.amazonaws.com \
    --docker-username=AWS \
    --docker-password=eyJwYXlsb2FkIjoiMlg0RTlKMS92Z045Vy8yaE1TNjdXVHNjMGMwMXhtaFVIRHpwTWJTbDJBbXpmbjZGVkdBY041T3RZeThvcnByVS93YjdZY2pnaklhZ1BkeW83QU5pQXhuZ3NmOVg2ai93dDRlalROdno3SCt2K3hRTjRHTko0bXJ6MXNOeHNiUFdIcGJnWERlVGVtbGVFWGhFbUFGV1JMRktaQXR3MTQ0NUtDWnlPZ01OUzRvQkhLdGxRM1p6ckEzNEFCVjhXZXVlb0tVaEZobFFRY2J3Z0p2YUdReVh3OE5oRnNrOStpZzNqdE5YVmg4NlVibWs0TTdPdnh2Z0lRNXVaTEpwaE0wTXBtd05ERlNaMzNteW9KQ202MnZQblN6a09KRzVvaVI2RXkzT0h4S3RkTzJUYysxUndNbXVNdmdONHRJcU54Y29sUHp2Wm1rQWN0WlY0NFV3UEN5cFZ3TWZYNk1kRlpRc1VObENLMmR3cTl6bjdLdnBGTUc5L2M4cUxrcldLSWY3TjNYeTFyem5PUEFMd3ZWQ3ZSRjN3OXZmWGtZWTZNaGF2ZFNzYlBuOVlDU2tUaHlCaUZ0MVlWa0pFQ3FsektmWjlGQnhyYnZWWXhTQ3NaSGVkY1RpSm4rZVpHMUtiWmI0MDdHN1ViWFdsSFh6U3V5dVM4cWk5aFJoYUpvRDFQMG5uaS9iWkI1eGNkRS8wMHlOWFowNDRzbWVsZ3hYdkhvWDJtRFU3ZkF4bVBsVXo2M0s5RlBQOFBET1U0NnBZS3ljYlZna3RmdXdDMThVSTFRTmRscXRtY2Y5ZlJVOGtaUEZXL2ZNbHh1WkhHeUFnQWNkZjJPVHhaOEZnZU9UdzB3MHlIdkZoeVhhaW9ER3htNVBSWEd3WFU4MGNQQk83N3R0dUxNSGJ2UXJtM1J0OS85TzkxNVVMcUlhYll2UmJIbkNsbmhQT1FwTHNBbCs5cjR5UVlYWGdxaXozdURBalg5a1QzQ3pBZ2RNbHZRSjJ6UzhWTkdWNUlDQjE1aDByM2V0cmRCYlphMVBEUmtFb3l5Ky81MTlXYk9nNExsOXFKYXkzem1SR1FheG1kZ1hGZmFUUVNSSUl5TDAxRGFOLzYzUThtYTVBZXcyNnBZRllTN0xML3dOY2dXNS9jTCsxYUVzOFFDZTVUNExIWDVoVWU1SklwRUVKREp0aDQ4MlIrRjJBWTk0Q1AwSkt2cE5UWmNGZ3lpVHMzWUdlbWM1ZWdBditINDVDVlFOY3JFc3VURzdQWUt6QUJNUDVtMk9HdEhPWEE4M05NNzh3WWw3OENSYlhGYzVkamVSL3U3OWlTUzNIeElyNnl6bkVDVnZ0cVluV0dxdUxaMmorMHROSHFiU1hFMFBJTzM2N2w5c1RtQXpTaU9OUnF5NnhlRURqTDNqQzRoanFFSDFiZ0hnIiwiZGF0YWtleSI6IkFRRUJBSGgzellPZHRwQkJTVncvWTJhbjhCWElENGt3TFFmbm9UajV6d1p0R0pab1pRQUFBSDR3ZkFZSktvWklodmNOQVFjR29HOHdiUUlCQURCb0Jna3Foa2lHOXcwQkJ3RXdIZ1lKWUlaSUFXVURCQUV1TUJFRURKUGFYcHQ4eXJDcUhJVTZLZ0lCRUlBN3BvQnJxWHpyTmlUOVlDZ2x1cnRIMmltSUlPOWpBUzV1OGxHUTcxMllkUTB2WkFyMzNabWdlUHhXMU9Sc3ZmanAzYW9hMVIrZHNiMjQ3UTA9IiwidmVyc2lvbiI6IjIiLCJ0eXBlIjoiREFUQV9LRVkiLCJleHBpcmF0aW9uIjoxNzA3ODczMzQ4fQ==

Last version is only one step, but also only for one specific repository.
If there are multiple repositories accessable through the same authentication the first solution would be more practical.

    kubectl delete -f my-app-deployment.yaml

When it doesn't work it shows an error ImagePullBackOff or ErrImagePull

    kubectl get pods

Secrets must be in the same namespace.

# 18 - Kubernetes Operators for Managing Complex Applications

Kubernetes can completely manage lifecycle of stateless apps.
But stateful apps are not that easy to automate. Therefore you can use the operator.
Mostly required for stateful applications that cannot really be hosted outside the cluster.

The operator can work with CRD (custom resource defintions) which are specifically defined for the stateful app like a specific database.

# 19 - Secure your cluster - Authorization with RBAC

RBAC Role Based Access Control

## Ressource Role

It is required to give each user the least privilege.
This means a developer user should maybe not have the rights to create a new storage and also should not be allowed to edit specific services or deployments.

For this we can use a namespace specific rolebinding on a user or group.

rules:
    - apiGroups
      resources -> resource types pods, services, deployments,...
      verbs -> actions like create or delete deployment

## Ressource ClusterRole

Administrators need permissions to administer the cluster, therefore here we can't use the namespace specific rolebindings.
We use ClusterRole for that.

API Server handles the authentication of all requests and checks if a user is allowed to connect.

An Administrator manually creates certificates OR ldap for users which allows them to access the cluster.

## Ressource ServiceAccount

But we also need authorization for applications. Other services or tools do access the cluster.

## API Access

Check if the current user is able to execute a command

    kubectl auth can-i create deployments


User at first logs in through username/password, certificate, whatever