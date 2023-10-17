# LAB :: Building Kubernetes Monitoring

Monitoring systems must contain the following elements

```
log/metric/trace shipper --> monitoring data storage --> Visualization --> Analysis and Alerting
```

We try to cover all the areas. 

Jump to sections:
* [Install Opentelemetry Operator](#install-opentelemetry-operator) 
* [Install ClickHouse Operator](#install-clickhouse-operator)
* [Install Clickhouse server instance (from CRD)](#install-clickhouse-server-instance)
* [Install Opentelemetry Collectors (from CRD)](#install-opentelemetry-collectors)
* [Install Opentelemetry Demo App](#install-opentelemetry-demo-app)
* [Install Grafana](#install-grafana)
* [Install Grafana Dashboards (Custom helm chart)](#install-grafana-dashboards)

## Monitoring services

Components:
* To collect metrics/logs/traces: OpenTelemetry
* To store monitoring Data: ClickHouse
* To display and analyze monitoring data: Grafana
* To send alerts: NOT IMPLEMENTED (Grafana could be used)

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

NOTE: Currently ClickHouse exporter supports only MergeTree tables (and not ReplicatedMergeTree) so we create a one node instance instead of servers with replications. ([wip](https://github.com/open-telemetry/opentelemetry-collector-contrib/pull/26599))


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

### Install Grafana Dashboards and Datasource

You can install it as an ArgoCD app
```
$ kubectl apply -f argocd/apps/grafana-dashboards-app.yaml
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
