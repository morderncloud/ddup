# Kubernetes components & addons

## Master Components

kube-apiserver
etcd
kube-scheduler
kube-controller-manager
cloud-controller-manager

## Node Components

kubelet
kube-proxy
Container Runtime

## Addons

DNS
Web UI (Dashboard)
Container Resource Monitoring
Cluster-level Logging

[More addons](https://kubernetes.io/docs/concepts/cluster-administration/addons/)

## Tools for Monitoring Compute, Storage, and Network Resources

[Ref](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/)

* [Heapster]

Heapster is a cluster-wide aggregator of monitoring and event data. It currently supports Kubernetes natively and works on all Kubernetes setups. Heapster runs as a pod in the cluster, similar to how any Kubernetes application would run. The Heapster pod discovers all nodes in the cluster and queries usage information from the nodes’ Kubelets, the on-machine Kubernetes agent. The Kubelet itself fetches the data from cAdvisor. Heapster groups the information by pod along with the relevant labels. This data is then pushed to a configurable backend for storage and visualization. Currently supported backends include InfluxDB (with Grafana for visualization), Google Cloud Monitoring and many others described in more details here. 

* [Grafana URL](http://lexbz2226.lexington.ibm.com:30161/)

* [cAdvisor]

cAdvisor is an open source container resource usage and performance analysis agent. It is purpose-built for containers and supports Docker containers natively. In Kubernetes, cAdvisor is integrated into the Kubelet binary. cAdvisor auto-discovers all containers in the machine and collects CPU, memory, filesystem, and network usage statistics. cAdvisor also provides the overall machine usage by analyzing the ‘root’ container on the machine.

On most Kubernetes clusters, cAdvisor exposes a simple UI for on-machine containers on port 4194. Here is a snapshot of part of cAdvisor’s UI that shows the overall machine usage:
