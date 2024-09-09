[Notes overview](https://github.com/TheAbys/devops-bootcamp-certification-project/blob/master/README.md)

# 16 - Monitoring with Prometheus

## 1 - Introduction to Monitoring with Prometheus

In modern infrastructure with hundreds of servers and services it is very important to have an overview.
Prometheus can provide that overview.

Finding problems is hard work without a tool such as Prometheus

Prometheus can watch servers and application and notify if necessary before a problem occurs

Prometheus Server consists of 3 parts
- Retrieval, which actively pulls metrics data
- Storage, to store the data
- Tool to executes PromQL queries

It pulls data, as otherwise if all applications etc. would send data the Prometheus Server could crash.


Prometheus can monitor Linux, Windows, Applications, Webserver, Database, ...
retrieved metrics can be Memory usage, CPU usage, etc.

Metric types
- Counter, increasing counter, like how often is the application accessed
- Gauge, what cpu usage does the server have
- Histogram

To monitor a specific server which has no /metrics endpoint sometimes there is a metrics exporter especially for that
if not it can be developed

Configuration is done in prometheus.yaml file

Alertmanager is responsible for sending messages like email or slack or whatever else.

Prometheus is meant to work when other applications don't.
Prometheus is a standlone application. To scale it Prometheus Federations can be used.
Example for federation, one Prometheus in each cluster used, and one main Prometheus outside the clusters that federates the other instances.

## 2 - Install Prometheus Stack in Kubernetes

Multiple ways to install Prometheus

1. setting up everything with configuration YAML files, inefficient and a lot of work
2. using an operator, the operator manages everything
3. using Helm to deploy an operator

Create a cluster

    eksctl create cluster
    kubectl get node

Deploy the microservice applications in the default namespace

    kubectl apply -f config-microservices.yaml

Installing prometheus

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update
    kubectl create namespace monitoring
    helm install monitoring prometheus-commmunity/kube-prometheus-stack -n monitoring
    kubectl get all -n monitoring

3 main deployments operator, grafana and kube state metrics
grafana is used to display metrics
kube state metrics handles the cluster metrics and provides them to prometheus

prometheus node exporter is deployed as a deamonset as this way it is deployed on each node
it provides the metrics of the nodes for prometheus

to see all the configurations on what is tracked

    kubectl get configmap -n monitoring

Extract some examples to see what was created

    kubectl describe statefulset prometheus-monitoring-kube-prometheus-prometheus -n monitoring > prom.yaml
    kubectl describe statefulset alertmanager-monitoring-kube-prometheus-alertmanager -n monitoring > alert.yaml
    kubectl describe deployment monitoring-kube-prometheus-operator -n monitoring > oper.yaml

prom.yaml
=> prometheus is configured to automatically reload the configuration whenever something was changed
=> configurations and rules are set via configmaps or secrets
=> changing anything here doesn't work, it will get overwritten through the reload

## 3 - Data Visualization with Prometheus UI

Make application available on localhost:9090

    kubectl port-forward service/monitoring-kube-prometheus-prometheus -n monitoring 9090:9090 &

Look up the targets and metrics that are available.
Even look into the configuration that is active.

Each target automatically gets labels added. Like jobname, namespace or the instance ip address.

For propper visualisation use Grafana.

## 4 - Introduction to Grafana

Make application available on localhost:8080

    kubectl port-forward service/monitoring-grafana -n monitoring 8080:80 &

default Grafana within Prometheus Operator is setup with user admin, password prom-operator

In case of this setup all the Dashboards for monitoring the kubernetes cluster are already given.

Create and run a pod "curl-test" of a specific image in the cluster

    kubectl run curl-test --image=radial/busyboxplus:curl -i --tty --rm

This opens the commandline of the pod and let's us execute a curl command.
This way the frontend-service can be curl'd and load can be generated.

Script to curl 1000 times and send response to test.txt

    for i in $(seq 1 10000)
    do
        curl <frontend-service-url> > test.txt
    done

CPU spike can then be watched in Grafana.

Grafana supports multiple datasource, Prometheus is only one of them.
Other Examples: Loki, ElasticSearch, Jaeger, ...

## 5 - Alert Rules in Prometheus

Define rules about what and when we want to be notified.
Under <prometheus-url>/alerts all currently defined alerts are listed.
When Prometheus Operator was installed, all alerts for the Prometheus Stack are preconfigured.

PromQL functions
https://prometheus.io/docs/prometheus/latest/querying/functions/

Example rule:

``` 
  - alert: HighRequestLatency
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
    for: 10m
    labels:
        severity: page
    annotations:
        summary: "High request latency"
        description: "The request latency for job 'myjob' has been above 0.5 seconds for more than 10 minutes."
        runbook: "https://runbooks.prometheus-operator.dev/docs/HighRequestLatency/"
```

Best practise for rules is to define and link a runbook.
Initial state of an alert is "Inactive".
After its first trigger it changes to "Pending".
If the condition of "expr" is met "for" 10 minutes, then the alert is "Firing".

## 6 - Create own Alert Rules - Part 1

Copy an example rule to alert-rules.yaml and structuring the first rule.

## 7 - Create own Alert Rules - Part 2

Changing rule file to kubernetes configuration yaml.
https://docs.openshift.com/container-platform/4.15/rest_api/monitoring_apis/prometheus-monitoring-coreos-com-v1.html

PrometheusRule is a custom resource

Apply the rules and check them

```
kubectl apply -f alert-rules.yaml
kubectl get PrometheusRules -n monitoring
```

## 8 - Create own Alert Rules - Part 3

Stress the application to trigger the rules.

Running the image containerstack/cpustress as a pod within the cluster to show one pod getting high cpu usage.

```
kubectl run cpustress --rm containerstack/cpustress -- --cpu 4 --timeout 30s --metrics-brief
```

## 9 - Introduction to Alertmanager

Until now the alerts were firing, but no reaction on this event was defined.

Make alertmanager application accessable from localhost:9093

    kubectl port-forward svc/monitoring-kube-prometheus-alertmanager -n monitoring 9093:9093 &

Default 

## 10 - Configure Alertmanager with Email Receiver

See example configuration of alertmanager alert.yaml

The volume "config-volume" was configured as a secret "alertmanager-monitoring-kube-prometheus-alertmanager-generated".

    kubectl get secret alertmanager-monitoring-kube-prometheus-alertmanager-generated -n monitoring -o yaml | less

This shows the content of the alertmanager.yaml as base64 encoded string.

    echo <base64-encoded-string> | base64 -D | less

Just to see the configuration of alertmanager.
But here nothing can be changed.

Another custom resource called AlertmanagerConfig (see alert-manager-configuration.yaml)

    kubectl apply -f alert-manager-configuration.yaml
    kubectl get alertmanagerconfig -n monitoring

To check the logs of the config-reloader, if the configuration was actually reloaded

    kubectl logs alertmanager-monitoring-kube-prometheus-alertmanager-0 -n monitoring -c config-reloader

## 11 - Trigger Alerts for Email Receiver

<alertmanagerurl>/v2/alerts to see if alerts are sent

## 12 - Monitor Third-Party Applications

Redis does not have an /metrics endpoint for Prometheus to scrape data from.
This is why a redis exporter was written https://github.com/oliver006/redis_exporter
This exporter knows how to read metrics from redis and expose them over /metrics endpoint.

## 13 - Deploy Redis Exporter

Helmchart to install the redis exporter

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm install redis-exporter prometheus-community/prometheus-redis-exporter -f redis-values.yaml
```

The pod and servicemonitor are created and Prometheus starts scraping the metrics.

## 14 - Alert Rules & Grafana Dashboard for Redis

https://samber.github.io/awesome-prometheus-alerts/
https://samber.github.io/awesome-prometheus-alerts/rules#redis

See redis-rules.yaml

Alert can be triggered by scaling down the redis pod to zero.

https://grafana.com/grafana/dashboards/763-redis-dashboard-for-prometheus-redis-exporter-1-x/

With the number 763 and the dashboard import in Grafana the dashboard is imported and created.

## 15 - Collect & Expose Metrics with Prometheus Client Library (Monitor own App - Part 1)

Example documentation for using prometheus_client in python to generate metrics.
https://prometheus.github.io/client_python/getting-started/three-step-demo/

Another example for nodejs
https://www.npmjs.com/package/prom-client/v/11.5.3

see "nodejs-app" folder for the example application

## 16 - Scrape Own Application Metrics & Configure Own Grafana Dashboard (Monitor own App - Part 1)

After creating the service and servicemonitor for the "nodejs-app" Prometheus scrapes metrics.
The metrics can then be used within Prometheus or Grafana to create dashboards.

description of the rate function
https://prometheus.io/docs/prometheus/latest/querying/functions/#rate

rate(http_request_operations_total[2m])
rate(http_request_duration_seconds_sum[2m])