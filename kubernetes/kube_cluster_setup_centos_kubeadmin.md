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

    $wget https://download.docker.com/linux/centos/docker-ce.repo
    $mv docker-ce.repo /etc/yum.repos.d/
    $yum repolist
    $yum install -y docker # I used ibm-yum.sh install docker

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
    $yum install -y kubelet kubeadm kubectl # if not working, use ./ibm-yum.sh install -y kubelet kubeadm kubectl
    $systemctl enable kubelet && systemctl start kubelet

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

## Initializing your master

$kubeadm init --pod-network-cidr=192.168.0.0/16
    init: unrecognized option '--pod-network-cidr=192.168.0.0/16'
    [root@lexbz2226 kube]# kubeadm init --pod-network-cidr=192.168.0.0/16
    [init] Using Kubernetes version: v1.9.6
    [init] Using Authorization modes: [Node RBAC]
    [preflight] Running pre-flight checks.
        [WARNING FileExisting-crictl]: crictl not found in system path
    [preflight] Starting the kubelet service
    [certificates] Generated ca certificate and key.
    [certificates] Generated apiserver certificate and key.
    [certificates] apiserver serving cert is signed for DNS names [lexbz2226.lexington.ibm.com kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 9.51.102.226]
    [certificates] Generated apiserver-kubelet-client certificate and key.
    [certificates] Generated sa key and public key.
    [certificates] Generated front-proxy-ca certificate and key.
    [certificates] Generated front-proxy-client certificate and key.
    [certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
    [kubeconfig] Wrote KubeConfig file to disk: "admin.conf"
    [kubeconfig] Wrote KubeConfig file to disk: "kubelet.conf"
    [kubeconfig] Wrote KubeConfig file to disk: "controller-manager.conf"
    [kubeconfig] Wrote KubeConfig file to disk: "scheduler.conf"
    [controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
    [controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
    [controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
    [etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
    [init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests".
    [init] This might take a minute or longer if the control plane images have to be pulled.
    [apiclient] All control plane components are healthy after 28.002252 seconds
    [uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
    [markmaster] Will mark node lexbz2226.lexington.ibm.com as master by adding a label and a taint
    [markmaster] Master lexbz2226.lexington.ibm.com tainted and labelled with key/value: node-role.kubernetes.io/master=""
    [bootstraptoken] Using token: 4d35bb.7afcbc5b331a0322
    [bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
    [bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
    [bootstraptoken] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
    [bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
    [addons] Applied essential addon: kube-dns
    [addons] Applied essential addon: kube-proxy

    Your Kubernetes master has initialized successfully!

    To start using your cluster, you need to run the following as a regular user:

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

    You should now deploy a pod network to the cluster.
    Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
    https://kubernetes.io/docs/concepts/cluster-administration/addons/

    You can now join any number of machines by running the following on each node
    as root:

    kubeadm join --token 4d35bb.7afcbc5b331a0322 9.51.102.226:6443 --discovery-token-ca-cert-hash sha256:73ac21d22e7fea51960efd3ddbefe73d053eb600294ae352a0d5b85594043452

## Deploy a pod network to the cluster (calico)

    @Master
    $mkdir -p $HOME/.kube
    $sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    $sudo chown $(id -u):$(id -g) $HOME/.kube/config

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
        <!DOCTYPE html>
        <html>
        <head>
        <title>Welcome to nginx!</title>
        <style>
            body {
                width: 35em;
                margin: 0 auto;
                font-family: Tahoma, Verdana, Arial, sans-serif;
            }
        </style>
        </head>
        <body>
        <h1>Welcome to nginx!</h1>
        <p>If you see this page, the nginx web server is successfully installed and
        working. Further configuration is required.</p>

        <p>For online documentation and support please refer to
        <a href="http://nginx.org/">nginx.org</a>.<br/>
        Commercial support is available at
        <a href="http://nginx.com/">nginx.com</a>.</p>

        <p><em>Thank you for using nginx.</em></p>
        </body>
        </html>

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

## Tear down

kubectl drain lexbz2184.lexington.ibm.com --delete-local-data --force --ignore-daemonsets
kubectl drain lexbz2202.lexington.ibm.com --delete-local-data --force --ignore-daemonsets
kubectl drain lexbz2226.lexington.ibm.com --delete-local-data --force --ignore-daemonsets

kubectl delete node lexbz2184.lexington.ibm.com
kubectl delete node lexbz2202.lexington.ibm.com
kubectl delete node lexbz2226.lexington.ibm.com

kubeadm reset

* logs
    [root@lexbz2226 kube]# kubectl drain lexbz2184.lexington.ibm.com --delete-local-data --force --ignore-daemonsets
    node "lexbz2184.lexington.ibm.com" cordoned
    WARNING: Ignoring DaemonSet-managed pods: calico-node-7l5jn, kube-proxy-hrpm4
    pod "kube-dns-6f4fd4bdf-87skp" evicted
    pod "nginx-deployment-6c54bd5869-njxrv" evicted
    node "lexbz2184.lexington.ibm.com" drained
    [root@lexbz2226 kube]# kubectl drain lexbz2202.lexington.ibm.com --delete-local-data --force --ignore-daemonsets
    node "lexbz2202.lexington.ibm.com" cordoned

    WARNING: Ignoring DaemonSet-managed pods: calico-node-65v5p, kube-proxy-rkn86
    pod "nginx-deployment-6c54bd5869-4tkws" evicted
    pod "kube-dns-6f4fd4bdf-88fzn" evicted
    pod "calico-kube-controllers-6b7986ddff-vlggb" evicted
    node "lexbz2202.lexington.ibm.com" drained
    [root@lexbz2226 kube]#
    [root@lexbz2226 kube]# kubectl delete node lexbz2184.lexington.ibm.com
    node "lexbz2184.lexington.ibm.com" deleted
    [root@lexbz2226 kube]# kubectl delete node lexbz2202.lexington.ibm.com
    node "lexbz2202.lexington.ibm.com" deleted
    [root@lexbz2226 kube]# kubectl get nodes
    NAME                          STATUS    ROLES     AGE       VERSION
    lexbz2226.lexington.ibm.com   Ready     master    14h       v1.9.6
    [root@lexbz2226 kube]# kubectl drain lexbz2226.lexington.ibm.com --delete-local-data --force --ignore-daemonsets
    node "lexbz2226.lexington.ibm.com" cordoned
    WARNING: Ignoring DaemonSet-managed pods: calico-node-459bv, kube-proxy-gbbfk; Deleting pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet: etcd-lexbz2226.lexington.ibm.com, kube-apiserver-lexbz2226.lexington.ibm.com, kube-controller-manager-lexbz2226.lexington.ibm.com, kube-scheduler-lexbz2226.lexington.ibm.com
    pod "nginx-deployment-6c54bd5869-rt9dm" evicted
    pod "nginx-deployment-6c54bd5869-g8pkf" evicted
    pod "calico-kube-controllers-6b7986ddff-npzjq" evicted
    pod "kube-dns-6f4fd4bdf-jbsvf" evicted
    node "lexbz2226.lexington.ibm.com" drained
    [root@lexbz2226 kube]# kubectl delete node lexbz2226.lexington.ibm.com
    node "lexbz2226.lexington.ibm.com" deleted
    [root@lexbz2226 kube]# kubectl get nodes
    No resources found.
    [root@lexbz2226 kube]# kubeadm reset
    [preflight] Running pre-flight checks.
    [reset] Stopping the kubelet service.
    [reset] Unmounting mounted directories in "/var/lib/kubelet"
    [reset] Removing kubernetes-managed containers.
    [reset] Deleting contents of stateful directories: [/var/lib/kubelet /etc/cni/net.d /var/lib/dockershim /var/run/kubernetes /var/lib/etcd]
    [reset] Deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
    [reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]


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

