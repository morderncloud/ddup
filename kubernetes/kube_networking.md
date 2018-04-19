# Kubernetes Networking

## DNS entry for kube objects

* Pod

pod-ip-address.my-namespace.pod.cluster.local

For example, a pod with IP 1.2.3.4 in the namespace default with a DNS name of cluster.local would have an entry: 1-2-3-4.default.pod.cluster.local

* Service

my-svc.my-namespace.svc.cluster.local

