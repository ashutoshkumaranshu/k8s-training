# Install Loki log monitoring

This is a step-by-step guide about how to install Loki log monitoring on Kubernetes using the fllowing components:
* Minikube: the standalone k8s cluster
* Metallb: on-prem loadbalancer for K8S 
* Traefik: K8s ingest/ingestroute controller
* Filebeat: the leighweight log shipper
* Logstash: the filebeat log collector that ships all thelohs to Loki (Filebeat could be replaced by logstash instances, but Filebeat has better description language in yaml)
* Loki: The log collector engine (installed in a distributed microservice way)
* Grafana: Loki UI

Optional:

* Minio could be used as an S3 storage for Loki

# Problems

Loki is very unstable this time installing it on minikube with distributed manner. There are many missing log chunks frequently happening when the storage is local storage
* https://github.com/grafana/loki/issues/5412
* https://community.grafana.com/t/logs-disappearing/59956/5

Unfortunately I couldn't setup minio on my minikube stack. Even the documentation is not clear about this  https://grafana.com/docs/loki/next/installation/helm/install-scalable/

So we will go with the basic version.

# Installing and starting Minikube

Install minikube with using the documentation (I used it on Linux so the backend is KVM).

* Set up minikube resources (we need at least 8GB RAM and 4-6 vCPU if we use Minio)
```
$ minikube config set memory 8192
$ minikube config set cpus 6
$ minikube config set disk-size 20G
```

# Install K8S tools

* Install kubectl with using the doc: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
* Install Helm with using the doc: https://helm.sh/docs/intro/install/

# Start minikube

```
$ minikube start
```

# Install metallb

This will install metallb. Please change metallb-values correspond to your minikube VM network range
To get the range:
```
# stop minikube
$ minikube stop
# get the vnets
$ virsh net-list
# Get the definition of minikube vnet
$ virsh net-dumpxml mk-minikube | grep range
# Change the dhcp range to have a fix ip range for metallb eg.:
$ virsh net-update mk-minikube delete ip-dhcp-range "<range start='192.168.39.2' end='192.168.39.254'/>" --live --config
$ virsh net-update mk-minikube add ip-dhcp-range "<range start='192.168.39.150' end='192.168.39.254'/>" --live --config
# start minikube
$ minikube start
# let's update metallb-values.yml file with the new range eg.: 192.168.39.2-192.168.39.149
```

```
$ helm repo add metallb https://metallb.github.io/metallb
$ helm install --create-namespace -n metallb metallb metallb/metallb
$ kubectl apply -f metallb/metallb-iprange.yml
```

# Install traefik

```
$ helm repo add traefik https://helm.traefik.io/traefik
$ helm install --create-namespace -n traefik traefik traefik/traefik -f traefik/traefik-values.yml
# Now you can access the ui on http://treafik.k8s.local/dashboard/
```

# Update your Linux dnsmasq setting to point to traefik loadbalanced IP

``` 
# get the service LB ip
$ kubectl get svc -n traefik -o jsonpath='{.items[0].status.loadBalancer.ingress[0].ip}'
# Add line to /etc/Networkmanager/dnsmasq.d/k8s.conf with the proper IP
# > address=/k8s.local/192.168.39.2
# restart NetworkManager service
$ sudo systemctl restart NetworkManager
# add NetworkManager as local resolver to systemd-resolved
# add the below lines to [Resolve] section
# DNS=127.0.1.1 
# Domains=~k8s.lab
$ sudo systemctl restart systemd-resolved

# Test
$ dig something.k8s.local
...
; ANSWER SECTION:
something.k8s.local.    0       IN      A       192.168.39.2
....
```

# Install minio

```
$ helm repo add minio https://operator.min.io/
$ helm install --create-namespace -n minio minio minio/minio-operator -f minio/minio-values.yml
# above kubernetes 1.22 you need to run this
$ kubectl apply -f minio/console-token.yml
# then i don't know somehow you can expose the s3 api endpoint but i'm confused here :D
```

# Install Loki

```
$ helm repo add grafana https://grafana.github.io/helm-charts
$ helm install --create-namespace -n monitoring loki grafana/loki -f loki/loki-values.yaml
```

# Install Garafana

```
# repo has been already added for loki
$ helm install grafana --create-namespace --namespace=monitoring grafana/grafana -f grafana/grafana-values.yml
# Open grafana UI on https://grafana.k8s.local
# Get the admin user secret 
# Add loki datasource as https://loki.k8s.local and disable tls settings
# Since you haven1t sent data yet you will get an error about the missing labels, don!t mind about now
```

# Install logstash

For logstash I created a custom image that have loki-output. Alos it is important that we had to remove the original logstash.conf pipeline that is wired in the original image and must be overwritten

```
$ helm repo add elastic https://helm.elastic.co
$ helm install logstash --create-namespace --namespace=monitoring elastic/logstash -f logstash/logstash-values.yml
# add logstash ingress that is optional since you can use the service itself in filebeat config and actually it is not working now since we need a tcp router in an IngressRoute
$ kubectl apply -f logstash/logstash-ingress.yml
```

# Install filebeat

```
# helm rapo has been added in the previous step
$ helm install filebeat --create-namespace --namespace=monitoring elastic/filebeat -f filebeat/filebeat-values.yml
```


# Required improvements

* Install Minio or use an existing one
* Install Loki in distributed way and set it up to use minio bucket

