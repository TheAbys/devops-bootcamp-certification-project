# Bonus-Project

## Disclaimer

I can't share any of the code, but I can describe what I'm doing and what I'm trying to accomplish.

## Stack

- Router API
- Python
- Prometheus
- Open telemetry
- Loki
- Grafana
- Kubernetes
- Helm/Helmfile
- Ansible
- Bitbucket
- Jfrog
- Jenkins

## What's the goal?

Collect router data and metrics, bring them into a Grafana to generate alerts based on reachability of a router.

## What's it doing?

Retrieve router data from the API, convert the information into metrics and send them to open telemetry in Python.
Any logs are being sent to Loki.
In the first thread the API is queried in an interval and retrieved data is stored in a queue.
The second thread handles processing the queue one by one.

Open telemetry is used in between the python application and Prometheus.

Prometheus is used as a datasource for Grafana.

Grafana is used to display the generated metrics and handling alerts.
The first dashboard shows an overview of all routers with basic data and the reachability status.
A second dashboard is for a detail view with additional information like uptime, cpu and memory usage.

If the reachability status changes to **UNREACHABLE** the responsible team will be alerted after 24 hours to check the issue.

## Learnings in regard of this bootcamp

In this specific case Open telemetry means overhead.
It is only used as a middleware for metrics. Traces and logs are not sent over it.
Therefore it could probably be removed and it can be solved only with Prometheus.
Obviously the python application would require an update too using the prom client library to expose a /metrics endpoint.

## What are the other tools for?

Jenkins is used to build a docker image and a helm chart and store it in our Jfrog artefactory.

Ansible is used for the installation of everything on the production system.

Bitbucket is used as git repository.

Kubernetes is used to deploy all said applications on the servers.


