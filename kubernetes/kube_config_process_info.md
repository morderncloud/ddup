# Kubernetes process info samples

## Environment

RHEL 7.4
Kubernetes 1.9
Docker 1.13.1 (API version 1.26)

## Configuration file/path

/etc/systemd/system/kubelet.service.d/10-kubeadm.conf

--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf
--kubeconfig=/etc/kubernetes/kubelet.conf

--pod-manifest-path=/etc/kubernetes/manifests
--cni-conf-dir=/etc/cni/net.d
--cni-bin-dir=/opt/cni/bin
--client-ca-file=/etc/kubernetes/pki/ca.crt
--cert-dir=/var/lib/kubelet/pki

kube-scheduler --kubeconfig=/etc/kubernetes/scheduler.conf

--authorization-mode=Node,RBAC

/usr/local/bin/kube-proxy --config=/var/lib/kube-proxy/config.conf
/kube-dns --domain=cluster.local. --dns-port=10053 --config-dir=/kube-dns-config --v=2
/sidecar --v=2 --logtostderr --probe=kubedns,127.0.0.1:10053,kubernetes.default.svc.cluster.local,5,SRV --probe=dnsmasq,127.0.0.1:53,kubernetes.default.svc.cluster.local,5,SRV

/usr/bin/kube-controllers

* kube-controller-manager

--kubeconfig=/etc/kubernetes/controller-manager.conf 
--leader-elect=true --controllers=*,bootstrapsigner,tokencleaner 
--kubeconfig=/etc/kubernetes/controller-manager.conf 
--allocate-node-cidrs=true 
--cluster-cidr=192.168.0.0/16 
--node-cidr-mask-size=24


## Master node

[root@lexbz2226 ~]# ps -ef|grep kube
root      5112     1  3 Apr10 ?        07:10:26 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true --network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin --cluster-dns=10.96.0.10 --cluster-domain=cluster.local --authorization-mode=Webhook --client-ca-file=/etc/kubernetes/pki/ca.crt --cadvisor-port=0 --cgroup-driver=systemd --rotate-certificates=true --cert-dir=/var/lib/kubelet/pki
root      5559  5519  0 Apr10 ?        01:17:18 kube-scheduler --address=127.0.0.1 --leader-elect=true --kubeconfig=/etc/kubernetes/scheduler.conf
root      5612  5545  4 Apr10 ?        08:33:36 kube-apiserver --insecure-port=0 --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --requestheader-username-headers=X-Remote-User --requestheader-group-headers=X-Remote-Group --service-cluster-ip-range=10.96.0.0/12 --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --admission-control=Initializers,NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota --enable-bootstrap-token-auth=true --requestheader-extra-headers-prefix=X-Remote-Extra- --client-ca-file=/etc/kubernetes/pki/ca.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --allow-privileged=true --requestheader-allowed-names=front-proxy-client --secure-port=6443 --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --advertise-address=9.51.102.226 --service-account-key-file=/etc/kubernetes/pki/sa.pub --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --authorization-mode=Node,RBAC --etcd-servers=http://127.0.0.1:2379
root      5633  5582  2 Apr10 ?        04:17:58 kube-controller-manager --cluster-signing-key-file=/etc/kubernetes/pki/ca.key --leader-elect=true --use-service-account-credentials=true --controllers=*,bootstrapsigner,tokencleaner --root-ca-file=/etc/kubernetes/pki/ca.crt --service-account-private-key-file=/etc/kubernetes/pki/sa.key --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt --address=127.0.0.1 --kubeconfig=/etc/kubernetes/controller-manager.conf --allocate-node-cidrs=true --cluster-cidr=192.168.0.0/16 --node-cidr-mask-size=24
root      6681  6663  0 Apr10 ?        00:22:13 /usr/local/bin/kube-proxy --config=/var/lib/kube-proxy/config.conf
root      6984  6967  0 Apr10 ?        00:07:04 /kube-dns --domain=cluster.local. --dns-port=10053 --config-dir=/kube-dns-config --v=2
nfsnobo+  7095  7073  0 Apr10 ?        00:16:28 /sidecar --v=2 --logtostderr --probe=kubedns,127.0.0.1:10053,kubernetes.default.svc.cluster.local,5,SRV --probe=dnsmasq,127.0.0.1:53,kubernetes.default.svc.cluster.local,5,SRV
root      7458  7441  0 Apr10 ?        00:05:21 /usr/bin/kube-controllers

## Worker

root      2383     1  1 Mar28 ?        09:02:38 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true --network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin --cluster-dns=10.96.0.10 --cluster-domain=cluster.local --authorization-mode=Webhook --client-ca-file=/etc/kubernetes/pki/ca.crt --cadvisor-port=0 --cgroup-driver=systemd --rotate-certificates=true --cert-dir=/var/lib/kubelet/pki
root      2569  2535  0 Mar28 ?        00:56:05 /usr/local/bin/kube-proxy --config=/var/lib/kube-proxy/config.conf
root      4132  4109  0 Apr11 ?        00:56:43 /kube-state-metrics --port=8080

## Docker

root      5076     1  1 Apr10 ?        03:44:29 /usr/bin/dockerd-current --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current --default-runtime=docker-runc --authorization-plugin=rhel-push-plugin --exec-opt native.cgroupdriver=systemd --userland-proxy-path=/usr/libexec/docker/docker-proxy-current --seccomp-profile=/etc/docker/seccomp.json --selinux-enabled --log-driver=journald --signature-verification=false --add-registry docker.io
root      5091  5076  0 Apr10 ?        00:12:23 /usr/bin/docker-containerd-current -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --shim docker-containerd-shim --runtime docker-runc --runtime-args --systemd-cgroup=true
root      5224     1  0 Apr10 ?        01:42:52 /usr/libexec/docker/rhel-push-plugin
root      5342  5091  0 Apr10 ?        00:00:00 /usr/bin/docker-containerd-shim-current 0aee71c9061c9606fbfeb99013ccf5081c013eb261a7d75977ba27de287a691b /var/run/docker/libcontainerd/0aee71c9061c9606fbfeb99013ccf5081c013eb261a7d75977ba27de287a691b /usr/libexec/docker/docker-runc-current

## pod path on host

* Volume

tmpfs           7.8G   12K  7.8G   1% /var/lib/kubelet/pods/9211b8ea-329e-11e8-a320-00505600feb9/volumes/kubernetes.io~secret/calico-cni-plugin-token-5bbs7
tmpfs           7.8G   12K  7.8G   1% /var/lib/kubelet/pods/419e27f8-329e-11e8-a320-00505600feb9/volumes/kubernetes.io~secret/kube-proxy-token-ksptw
