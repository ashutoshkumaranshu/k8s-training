# Basic services

We need a few basic services for mimic a real cluster

* Monitoring services
  * To collect metrics 
  * To collect logs
  * To display monitoring data

## Monitoring services

However prometheus is very popular I don't like it for several reason (eg.: not reliable database backend, awful configuration language, pull based behavior, etc, etc.). Also it is not good to store logs and traces. To cover logs and traces we need additional products like Elasticsearch or OpenSearch, Jaeger, or many more. And for these we need the shippers also (metricbeat, filebeat, logstash, fluentd, etc., etc.)
Instead of the above mess let's create a standard solution that uses only one shipper and one storage:
* OpenTelemetry collector(s) to ship the data
* Clickhouse to store the data
* Grafana to visualize the data

### Install OpenTelemetry Operator

You can install it via ArgoCd app
```
$ kubectl apply -f argocd/apps/opentelemetry-operator.yaml
```



