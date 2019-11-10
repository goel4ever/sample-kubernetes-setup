# Logging and Metrics - (Yet to be finished)

> This instruction is dependent on setting up `guestbook-with-redis` for now until the it is finished being worked on. Please do not proceed with these steps yet unless you know what you are looking for already.

This tutorial builds upon the PHP Guestbook with Redis tutorial. Lightweight log, metric, and network data open source shippers, or Beats, from Elastic are deployed in the same Kubernetes cluster as the guestbook.
The Beats collect, parse, and index the data into Elasticsearch so that you can view and analyze the resulting operational information in Kibana. This example consists of the following components:

- A running instance of the PHP Guestbook with Redis tutorial
- Elasticsearch and Kibana
- Filebeat
- Metricbeat
- Packetbeat

## Pre-requisites

### Add a Cluster role binding

`kube-system` is the namespace for objects created by the Kubernetes system. Typically, this would contain service accounts which are used to run the kubernetes controllers, pods like kube-dns, kube-proxy, kubernetes-dashboard and stuff like fluentd, heapster, ingresses and so on. You should avoid putting anything `personal` in that namespace since kube considers it to be "owned" by kube and the permissions for the service-accounts inside are quite high.

Here is how you see all the details within kube-system:

```sh
kubectl get pods --namespace=kube-system
kubectl get deploy --namespace=kube-system
kubectl get rs --namespace=kube-system
kubectl get ds --namespace=kube-system
```

Create a cluster level role binding so that you can deploy `kube-state-metrics` and the `Beats` at the cluster level (in kube-system).

```sh
kubectl create clusterrolebinding cluster-admin-binding \
 --clusterrole=cluster-admin --user=<your email associated with the k8s provider account>

OR

kubectl create clusterrolebinding cluster-admin-binding \
 --clusterrole=cluster-admin --user=$(gcloud info --format='value(config.account)')
# Note that your GCP identity is case sensitive but gcloud info as of Google Cloud SDK 221.0.0 is not.
# This means that if your IAM member contains capital letters, the above one-liner may not work for you.
```

### Install kube-state-metrics

Kubernetes `kube-state-metrics` is a simple service that listens to the Kubernetes API server and generates metrics about the state of the objects. It is not focused on the health of the individual Kubernetes components, but rather on the health of the various objects inside, such as deployments, nodes and pods.

kube-state-metrics is about generating metrics from Kubernetes API objects without modification. This ensures that features provided by kube-state-metrics have the same grade of stability as the Kubernetes API objects themselves. In turn, this means that kube-state-metrics in certain situations may not show the exact same values as kubectl, as kubectl applies certain heuristics to display comprehensible messages.

The metrics are exported on the HTTP endpoint `/metrics` on the `listening port` (default 80). They are served as `plaintext`. They are designed to be consumed either by Prometheus itself or by a scraper that is compatible with scraping a Prometheus client endpoint. You can also open /metrics in a browser to see the raw metrics.

`Metricbeat` reports these metrics. Add kube-state-metrics to the Kubernetes cluster that the guestbook is running in.

```sh
# Check to see if kube-state-metrics is running
kubectl get pods --namespace=kube-system | grep kube-state

# Install kube-state-metrics if needed
git clone https://github.com/kubernetes/kube-state-metrics.git kube-state-metrics
kubectl create -f kube-state-metrics/kubernetes
kubectl create -f kube-state-metrics/examples/standard
kubectl get pods --namespace=kube-system | grep kube-state

# Verify that kube-state-metrics is running and ready
kubectl get pods -n kube-system -l k8s-app=kube-state-metrics
```

### A running Elasticsearch and Kibana deployment

```sh

```

## References

```sh
https://kubernetes.io/docs/tutorials/stateless-application/guestbook-logs-metrics-with-elk/
```
