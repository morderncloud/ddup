# Using kubeadm to Create a Cluster

## [Ref Using kubeadm to Create a Cluster](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)

## My Server Resources

    4u16m250d, RHEL7.4

    Master - in.9.lexbz2226 
    Worker1 - in.10.lexbz2202
    Worker2 - in.11.lexbz2184

## Objectives

    Install a secure Kubernetes cluster on your machines
    Install a Pod network on the cluster so that your Pods can talk to each other

## [Install kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/)

* Disable swap
    Make sure no swap enabled on the servers in order for the kubelet to work properly:

    $cat /proc/swaps
    $swapoff -a
    $cat /etc/fstab # remove swap item if any

* Verify the MAC address and product_uuid are unique for every node
    $ip link
    $cat /sys/class/dmi/id/product_uuid

* Check required ports
    Master node(s)
    Protocol	Direction	Port Range	Purpose
    TCP	Inbound	6443*	Kubernetes API server
    TCP	Inbound	2379-2380	etcd server client API
    TCP	Inbound	10250	Kubelet API
    TCP	Inbound	10251	kube-scheduler
    TCP	Inbound	10252	kube-controller-manager
    TCP	Inbound	10255	Read-only Kubelet API

    Worker node(s)
    Protocol	Direction	Port Range	Purpose
    TCP	Inbound	10250	Kubelet API
    TCP	Inbound	10255	Read-only Kubelet API
    TCP	Inbound	30000-32767	NodePort Services**


## Install Docker

    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    yum install docker-ce
    systemctl enable docker.service
    systemctl start docker

## Install Go 1.10 on all nodes (Optional)

This is to avoid: [WARNING FileExisting-crictl]: crictl not found in system path
But I don't see any problem without crictl

wget https://dl.google.com/go/go1.10.1.linux-amd64.tar.gz && tar -C /usr/local -xzvf go1.10.1.linux-amd64.tar.gz


Default GOPATH is $HOME/go, so add path info to .bash_profile as:
PATH=$PATH:$HOME/bin:/usr/local/go/bin:$HOME/go/bin

yum install git -y
go get github.com/kubernetes-incubator/cri-tools/cmd/crictl


## Installing kubeadm, kubelet and kubectl

    $cat <<EOF > /etc/yum.repos.d/kubernetes.repo
        [kubernetes]
        name=Kubernetes
        baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64 # attention here, it was $basearch
        enabled=1
        gpgcheck=1
        repo_gpgcheck=1
        gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
        EOF

    $setenforce 0
    $yum install -y kubelet kubeadm kubectl // if not working, use ./ibm-yum.sh install -y kubelet kubeadm kubectl
    $systemctl enable kubelet

    $cat <<EOF >  /etc/sysctl.d/k8s.conf
        net.bridge.bridge-nf-call-ip6tables = 1
        net.bridge.bridge-nf-call-iptables = 1
        EOF
    $sysctl --system

## Configure cgroup driver used by kubelet on Master Node

    $docker info | grep -i cgroup
        Cgroup Driver: systemd
    $cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
        Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"

    If the Docker cgroup driver and the kubelet config don’t match, change the kubelet config to match the Docker cgroup driver. The flag you need to change is --cgroup-driver. If it’s already set, you can update like so:

    cp /etc/systemd/system/kubelet.service.d/10-kubeadm.conf /etc/systemd/system/kubelet.service.d/10-kubeadm.conf.bk && sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

## Initializing your master

// kube-router still not working for me, investigating
//$kubeadm init --pod-network-cidr=10.1.0.0/16

// Calico
$kubeadm init --pod-network-cidr=192.168.0.0/16

    Your Kubernetes master has initialized successfully!

    To start using your cluster, you need to run the following as a regular user:

rm -rf $HOME/.kube && mkdir -p $HOME/.kube && cp -i /etc/kubernetes/admin.conf $HOME/.kube/config && chown $(id -u):$(id -g) $HOME/.kube/config

## Deploy a pod network to the cluster (kube-router)

// kube-router still not working for me, investigating

* kube-router providing pod networking and network policy 

[root@lexbz2226 ~]# KUBECONFIG=/etc/kubernetes/admin.conf kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml
    configmap "kube-router-cfg" created
    daemonset "kube-router" created
    serviceaccount "kube-router" created
    clusterrole "kube-router" created
    clusterrolebinding "kube-router" created

* kube-router providing service proxy, firewall and pod networking

    KUBECONFIG=/etc/kubernetes/admin.conf kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter-all-features.yaml

    KUBECONFIG=/etc/kubernetes/admin.conf kubectl -n kube-system delete ds kube-proxy
    
    // failed ? 
    docker run --privileged --net=host gcr.io/google_containers/kube-proxy-amd64:v1.7.3 kube-proxy --cleanup-iptables
    docker run --privileged --net=host k8s.gcr.io/kube-proxy-amd64:v1.10.1 kube-proxy --cleanup-iptables
        W0419 12:25:01.656439       1 server.go:190] WARNING: all flags other than --config, --write-config-to, and --cleanup-iptables are deprecated. Please begin using a config file ASAP.
    E0419 12:25:01.946962       1 proxier.go:604] Failed to execute iptables-restore for filter: exit status 1 (iptables-restore: line 3 failed
    )
    error: encountered an error while tearing down rules.

* [Ref](https://github.com/cloudnativelabs/kube-router/blob/master/docs/kubeadm.md)

## Deploy a pod network to the cluster (or calico if kube-router is not used)

    $kubectl apply -f https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/kubeadm/1.7/calico.yaml
        configmap "calico-config" created
        daemonset "calico-etcd" created
        service "calico-etcd" created
        daemonset "calico-node" created
        deployment "calico-kube-controllers" created
        clusterrolebinding "calico-cni-plugin" created
        clusterrole "calico-cni-plugin" created
        serviceaccount "calico-cni-plugin" created
        clusterrolebinding "calico-kube-controllers" created
        clusterrole "calico-kube-controllers" created
        serviceaccount "calico-kube-controllers" created
    $kubectl get pods
    $kubectl get pods -n kube-system
    $kubectl get pods --all-namespaces 
        (or -n kube-system)
        NAME                                                  READY     STATUS    RESTARTS   AGE
        calico-etcd-dgqq9                                     1/1       Running   0          1m
        calico-kube-controllers-675684d4bb-rfr6n              1/1       Running   0          1m
        calico-node-h6nzc                                     2/2       Running   0          1m
        etcd-lexbz2226.lexington.ibm.com                      1/1       Running   0          2m
        kube-apiserver-lexbz2226.lexington.ibm.com            1/1       Running   0          2m
        kube-controller-manager-lexbz2226.lexington.ibm.com   1/1       Running   0          2m
        kube-dns-6f4fd4bdf-qwz9b                              3/3       Running   0          3m
        kube-proxy-rr9zj                                      1/1       Running   0          3m
        kube-scheduler-lexbz2226.lexington.ibm.com            1/1       Running   0          2m

## Master isolation

    Once kube-dns is up, proceed the following steps.

    $kubectl taint nodes --all node-role.kubernetes.io/master-
        node "lexbz2226.lexington.ibm.com" untainted
        taint "node-role.kubernetes.io/master:" not found
        taint "node-role.kubernetes.io/master:" not found

## On other two workers, run: (If this is not the first join, you may see error. pls see to issue & fix section.)

    $kubeadm join --token 4d35bb.7afcbc5b331a0322 9.51.102.226:6443 --discovery-token-ca-cert-hash sha256:73ac21d22e7fea51960efd3ddbefe73d053eb600294ae352a0d5b85594043452
        [preflight] Running pre-flight checks.
            [WARNING FileExisting-crictl]: crictl not found in system path
        [discovery] Trying to connect to API Server "9.51.102.226:6443"
        [discovery] Created cluster-info discovery client, requesting info from "https://9.51.102.226:6443"
        [discovery] Requesting info from "https://9.51.102.226:6443" again to validate TLS against the pinned public key
        [discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "9.51.102.226:6443"
        [discovery] Successfully established connection with API Server "9.51.102.226:6443"

        This node has joined the cluster:
            * Certificate signing request was sent to master and a response
            was received.
            * The Kubelet was informed of the new secure connection details.

        Run 'kubectl get nodes' on the master to see this node join the cluster.

* $kubectl get nodes
    NAME                          STATUS    ROLES     AGE       VERSION
    lexbz2184.lexington.ibm.com   Ready     <none>    42s       v1.9.6
    lexbz2202.lexington.ibm.com   Ready     <none>    2m        v1.9.6
    lexbz2226.lexington.ibm.com   Ready     master    19m       v1.9.6

## Validation

    [Grafana URL](http://lexbz2226.lexington.ibm.com:30161/)

    $wget https://raw.githubusercontent.com/kubernetes/website/master/docs/tasks/run-application/deployment.yaml
    $kubectl apply -f deployment.yaml
    $kubectl get pods
    $kubectl describe pods nginx-deployment-6c54bd5869-8kqvq

    $kubectl get pods
        NAME                                READY     STATUS    RESTARTS   AGE
        nginx-deployment-6c54bd5869-8kqvq   1/1       Running   0          1m
        nginx-deployment-6c54bd5869-bljmh   1/1       Running   0          1m
    $kubectl get pods -o wide
        NAME                                READY     STATUS    RESTARTS   AGE       IP               NODE
        nginx-deployment-6c54bd5869-8kqvq   1/1       Running   0          5m        192.168.21.129   lexbz2184.lexington.ibm.com
        nginx-deployment-6c54bd5869-bljmh   1/1       Running   0          5m        192.168.173.65   lexbz2202.lexington.ibm.com

    $curl 192.168.21.129
        <p><em>Thank you for using nginx.</em></p>

## Expose service and test on Browser

    $kubectl expose deployment/nginx-deployment --type="NodePort" --port 80
        service "nginx-deployment" exposed

    $kubectl get deployments
    NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   2         2         2            2           18m

    $kubectl get services
    NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
    kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP        54m
    nginx-deployment   NodePort    10.96.117.230   <none>        80:30678/TCP   20s

    On browser:
    http://lexbz2226.lexington.ibm.com:30678/
    or
    http://lexbz2202.lexington.ibm.com:30678/
    or
    http://lexbz2184.lexington.ibm.com:30678/

## Restart

systemctl stop kubelet && systemctl stop docker && systemctl daemon-reload && systemctl start docker && systemctl start kubelet

* Docker: systemctl restart docker
* Kubelet: systemctl daemon-reload && systemctl restart kubelet

## Tear down

* On Master

kubectl drain lexbz2184.lexington.ibm.com --delete-local-data --force --ignore-daemonsets
kubectl drain lexbz2202.lexington.ibm.com --delete-local-data --force --ignore-daemonsets
kubectl drain lexbz2226.lexington.ibm.com --delete-local-data --force --ignore-daemonsets

kubectl delete node lexbz2184.lexington.ibm.com
kubectl delete node lexbz2202.lexington.ibm.com
kubectl delete node lexbz2226.lexington.ibm.com

* On nodes

kubeadm reset

* Docker administration

systemctl restart docker
docker rm -f $(docker ps -aq)
docker rmi -f $(docker images -q)
docker system prune -a
docker images purge


## Issue & fix

* Rejoin the master got error
    [root@lexbz2184 ~]# kubeadm join --token 4d35bb.7afcbc5b331a0322 9.51.102.226:6443 --discovery-token-ca-cert-hash sha256:73ac21d22e7fea51960efd3ddbefe73d053eb600294ae352a0d5b85594043452
    [preflight] Running pre-flight checks.
        [WARNING FileExisting-crictl]: crictl not found in system path
    [preflight] Some fatal errors occurred:
        [ERROR Port-10250]: Port 10250 is in use
        [ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
        [ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
    [preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`

    fix:
    [root@lexbz2184 ~]# mv /etc/kubernetes/pki/ca.crt /etc/kubernetes/pki/ca.crt.bk
    [root@lexbz2184 ~]# mv /etc/kubernetes/kubelet.conf /etc/kubernetes/kubelet.conf.bk
    [root@lexbz2184 ~]# systemctl restart kubelet
    [root@lexbz2184 ~]# netstat -anp|grep 10250

* kubectl drain cmd hung

On each nodes,
systemctl restart docker
systemctl restart kubelet

* Error: No such image when “docker images” shows image, “docker rmi” says “no such image” or “reference does not exist”

sudo service docker stop
sudo rm -rf /var/lib/docker
sudo service docker start

* [WARNING FileExisting-crictl]: crictl not found in system path

Suggestion: go get github.com/kubernetes-incubator/cri-tools/cmd/crictl
Default GOPATH is $HOME/go, so add path info to .bash_profile as:
PATH=$PATH:$HOME/bin:/usr/local/go/bin:$HOME/go/bin

* Adding node error: [ERROR FileAvailable--etc-kubernetes-bootstrap-kubelet.conf]: /etc/kubernetes/bootstrap-kubelet.conf already exists
    mv /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf.bk
    and rerun kubeadm join cmd

* kube-dns pods was with status CrashLoopBackOff

Before kube-router install:
[root@lexbz2226 ~]# kubectl get pods -n kube-system
NAME                                                  READY     STATUS    RESTARTS   AGE
etcd-lexbz2226.lexington.ibm.com                      1/1       Running   0          29s
kube-apiserver-lexbz2226.lexington.ibm.com            1/1       Running   0          37s
kube-controller-manager-lexbz2226.lexington.ibm.com   1/1       Running   0          18s
kube-dns-86f4d74b45-2jlgm                             0/3       Pending   0          1m
kube-proxy-fcbl8                                      1/1       Running   0          1m
kube-scheduler-lexbz2226.lexington.ibm.com            1/1       Running   0          26s

After kube-router install: 

[root@lexbz2226 ~]# kubectl get pods -n kube-system
    NAME                                                  READY     STATUS             RESTARTS   AGE
    etcd-lexbz2226.lexington.ibm.com                      1/1       Running            0          57m
    kube-apiserver-lexbz2226.lexington.ibm.com            1/1       Running            0          56m
    kube-controller-manager-lexbz2226.lexington.ibm.com   1/1       Running            0          57m
    kube-dns-86f4d74b45-xtqpr                             2/3       CrashLoopBackOff   25         58m
    kube-router-4m7j8                                     1/1       Running            0          20m
    kube-router-mtqvl                                     1/1       Running            0          29m
    kube-router-pn7b5                                     1/1       Running            0          25m
    kube-scheduler-lexbz2226.lexington.ibm.com            1/1       Running            0          57m

I restarted kubelet on all nodes, looked better:

[root@lexbz2226 ~]# kubectl get pods -n kube-system
    NAME                                                  READY     STATUS    RESTARTS   AGE
    etcd-lexbz2226.lexington.ibm.com                      1/1       Running   1          1h
    kube-apiserver-lexbz2226.lexington.ibm.com            1/1       Running   1          1h
    kube-controller-manager-lexbz2226.lexington.ibm.com   1/1       Running   1          1h
    kube-dns-86f4d74b45-xtqpr                             2/3       Running   51         1h
    kube-router-4m7j8                                     1/1       Running   1          48m
    kube-router-mtqvl                                     1/1       Running   1          57m
    kube-router-pn7b5                                     1/1       Running   1          53m
    kube-scheduler-lexbz2226.lexington.ibm.com            1/1       Running   1          1h

but still with error, checking further..

check container logs in the pod: kubectl logs kube-dns-86f4d74b45-xtqpr kubedns -n kube-system

I0419 02:54:01.578925       1 dns.go:173] Waiting for services and endpoints to be initialized from apiserver...
I0419 02:54:02.077329       1 dns.go:173] Waiting for services and endpoints to be initialized from apiserver...
I0419 02:54:02.577372       1 dns.go:173] Waiting for services and endpoints to be initialized from apiserver...
I0419 02:54:03.077326       1 dns.go:173] Waiting for services and endpoints to be initialized from apiserver...
E0419 02:54:03.127364       1 reflector.go:201] k8s.io/dns/pkg/dns/dns.go:147: Failed to list *v1.Endpoints: Get https://10.96.0.1:443/api/v1/endpoints?resourceVersion=0: dial tcp 10.96.0.1:443: getsockopt: no route to host
I0419 02:54:03.577310       1 dns.go:173] Waiting for services and endpoints to be initialized from apiserver...
I0419 02:54:04.077367       1 dns.go:173] Waiting for services and endpoints to be initialized from apiserver...
I0419 02:54:04.577336       1 dns.go:173] Waiting for services and endpoints to be initialized from apiserver...
F0419 02:54:05.077310       1 dns.go:167] Timeout waiting for initialization

kubectl describe pod kube-apiserver-lexbz2226.lexington.ibm.com -n kube-system

--service-cluster-ip-range=10.96.0.0/12
--secure-port=6443

/etc/kubernetes/manifests/kube-apiserver.yaml on the master and changing the liveness probe:

livenessProbe:
  failureThreshold: 8
  httpGet:
    host: 127.0.0.1
    path: /healthz
    port: 443           # was 6443
    scheme: HTTPS

https://github.com/kubernetes/kubeadm/issues/193

[root@lexbz2226 ~]# kubectl get services -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP   1h
[root@lexbz2226 ~]# kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   1h


* Failed to get properties: Connection timed out

[root@lexbz2226 ~]# systemctl status docker

[root@lexbz2226 ~]# systemctl --force --force reboot

* Failed to clear iptables created by kube-proxy

[root@lexbz2226 ~]#     docker run --privileged --net=host k8s.gcr.io/kube-proxy-amd64:v1.10.1 kube-proxy --cleanup

W0419 12:35:09.047554       1 server.go:195] WARNING: all flags other than --config, --write-config-to, and --cleanup are deprecated. Please begin using a config file ASAP.
I0419 12:35:09.047660       1 feature_gate.go:226] feature gates: &{{} map[]}
time="2018-04-19T12:35:09Z" level=warning msg="Running modprobe ip_vs failed with message: `modprobe: ERROR: ../libkmod/libkmod.c:586 kmod_search_moddep() could not open moddep file '/lib/modules/3.10.0-327.4.5.el7.x86_64/modules.dep.bin'\nmodprobe: WARNING: Module ip_vs not found in directory /lib/modules/3.10.0-327.4.5.el7.x86_64`, error: exit status 1"
I0419 12:35:09.051474       1 server.go:444] Version: v1.10.1

A: just add a volumn mount
docker run --privileged -v /lib/modules/:/lib/modules/:ro --net=host k8s.gcr.io/kube-proxy-amd64:v1.10.1 kube-proxy --cleanup-iptables

Flag --cleanup-iptables has been deprecated, This flag is replaced by --cleanup.
W0419 13:19:06.347266       1 server.go:195] WARNING: all flags other than --config, --write-config-to, and --cleanup are deprecated. Please begin using a config file ASAP.
I0419 13:19:06.347350       1 feature_gate.go:226] feature gates: &{{} map[]}
I0419 13:19:06.351409       1 server.go:444] Version: v1.10.1

/etc/kubernetes/manifests/kube-apiserver.yaml

## Ref

