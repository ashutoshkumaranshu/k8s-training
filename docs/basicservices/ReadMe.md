# Basic services

We need a few basic services for mimic a real cluster

* Monitoring services
  * To collect metrics 
  * To collect logs
  * To display monitoring data

Jump to sections:
* [Install Opentelemetry Operator](#install-opentelemetry-operator) 
* [Install ClickHouse Operator](#install-clickhouse-operator)
* [Install Clickhouse server instance (from CRD)](#install-clickhouse-server-instance)
* [Install Opentelemetry Collectors (from CRD)](#install-opentelemetry-collectors)
* [Install Opentelemetry Demo App](#install-opentelemetry-demo-app)
* [Install Grafana](#install-grafana)
* [Install Grafana Dashboards (Custom helm chart)](#install-grafana-dashboards)

## Monitoring services

However prometheus is very popular I don't like it for several reason (eg.: not reliable database backend, awful configuration language, pull based behavior, etc, etc.). Also it is not good to store logs and traces. To cover logs and traces we need additional products like Elasticsearch or OpenSearch, Jaeger, or many more. And for these we need the shippers also (metricbeat, filebeat, logstash, fluentd, etc., etc.)
Instead of the above mess let's create a standard solution that uses only one shipper and one storage:
* OpenTelemetry collector(s) to ship the data (contrib image)
  * It could act as Jaeger, Zipkin, Prometheus server, etc.
* ClickHouse to store the data
* Grafana to visualize the data


### Install OpenTelemetry Operator

You can install it as an ArgoCd app
```
$ kubectl apply -f argocd/apps/opentelemetry-operator-app.yaml
```

Now, don't forget to sync the application on the UI or with argocd cli
```
$ argocd app sync argocd/opentelemetry-operator
```

### Install ClickHouse Operator

You can install it as an ArgoCD app
```
$ kubectl apply -f argocd/apps/clickhouse-operator-app.yaml
```

Now, don't forget to sync the application on the UI or with argocd cli
```
$ argocd app sync argocd/clickhouse-operator
```

Check if all is green on ArgoCD UI.


### Install ClickHouse server instance

You can install it as an ArgoCD app
```
$ kubectl apply -f argocd/apps/clickhouse-cluster-app.yaml
```

Now, don't forget to sync the application on the UI or with argocd cli
```
$ argocd app sync argocd/clickhouse-cluster
```

Check if all is green on ArgoCD UI, also you can reach ClickHouse play web page on https://clickhouse.k8s.local/play (set usrname to admin and password to adminpwd) with running some query like "show tables in system;"


### Install Opentelemetry Collectors

This will install the necessary cluster role and cluster role binding that allow opentelemetry-operator service user to get data from kubernetes and create two collectors to collect logs and metrics.
It is possible to run one collector with merged config but I wanted to handle them differently to show examples only for metrics and only for logs. 

You can install it as an ArgoCD app
```
$ kubectl apply -f argocd/apps/opentelemetry-collectors-app.yaml
```

Now, don't forget to sync the application on the UI or with argocd cli
```
$ argocd app sync argocd/opentelemetry-collectors
```

Check if all is green on ArgoCD UI. If you want you can allow "debug" exporter in the pipeline if you want to check the raw messages in the pod logs.

After a few seconds you can check if the data is flowing into Clickhouse on the play UI (or any DB admin tool where you can install clickhouse jdbc driver)
```
show tables in otel;
select count(*) from otel.otel_metrics_gauge;
select count(*) from otel.otel_logs;
```

### Install Opentelemetry Demo App

You can install it as an ArgoCD app
```
$ kubectl apply -f argocd/apps/opentelemetry-demo-app.yaml
```

Now, don't forget to sync the application on the UI or with argocd cli
```
$ argocd app sync argocd/opentelemetry-demo
```

You can check if the tables were created. We put the demo app data into different tables prefixed as "otel_demo_". It will install its own collector. You can check all the bunch of resources it creates [here](../../custom-apps/opentelemetry-operator/demo/opentelemetry-demo.yaml)

### Install Grafana

You can install it as an ArgoCD app
```
$ kubectl apply -f argocd/apps/grafana-app.yaml
```

Now, don't forget to sync the application on the UI or with argocd cli
```
$ argocd app sync argocd/grafana
```

### Install Grafana Dashboards

You can install it as an ArgoCD app
```
$ kubectl apply -f argocd/apps/grafana-app.yaml
```

Now, don't forget to sync the application on the UI or with argocd cli
```
$ argocd app sync argocd/grafana-dashboards
```

Check if all is green on ArgoCD UI.
Now you can check the dashboards on Grafana UI https://grafana.k8s.local. For that you need the admin user password that is stored in a secret
```
$ kubectl get secret -n grafana grafana -o jsonpath='{.data.admin-password}' | base64 -d ; echo
```

