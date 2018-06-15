# IBM Cloud Private (ICP) Installation

## Servers

sbybz221092 worker
sbybz221116 proxy
sbybz221117 master

## Reference

[Version] 	IBM Cloud Private 2.1.0.3 for Linux (64-bit) Docker (CNT27EN )

Size 9,775MB Date posted 25 May 2018


[ICP architecture](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.1/getting_started/architecture.html)
[Installation](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.1/installing/install_containers.html)
[Uninstall](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.1/installing/uninstall.html)

## Uninstall (skip it if this is a new install)

* For high availability (HA) environments, on all master nodes, unmount the /var/lib/registry directory.
    umount /var/lib/registry

* Log in to the boot node as a user with root permissions. The boot node is usually your master node. 
    cd /opt/ibm-cloud-private-2.1.0.1/cluster

* Uninstall IBM Cloud Private
    docker run -e LICENSE=accept --net=host -t -v "$(pwd)":/installer/cluster ibmcom/icp-inception:2.1.0.1-ee uninstall

* Restart docker on all 3 nodes
    (I installed pssh to run the same cmd on all nodes specified in ~/.pssh_hosts_files
    alias psh="/usr/bin/pssh -i -h ~/.pssh_hosts_files " in .bash_profile)

    psh systemctl restart docker

* Restart all nodes in your cluster
    psh shutdown -r now 
    (pssh does not work very well on rebooting system, so need to double check if the target system is rebooted)

## Prerequisites

    On all nodes,

    yum install -y yum-utils   device-mapper-persistent-data   lvm2

## Install docker

    On all nodes,

    yum-config-manager     --add-repo     https://download.docker.com/linux/centos/docker-ce.repo
    cat /etc/yum.repos.d/docker-ce.repo
    yum install docker-ce
    systemctl start docker

## Install on master

* Extract the images and load them into Docker. Extracting the images might take a few minutes
    cd /DST/cluster/images/
    tar xf ibm-cloud-private-x86_64-2.1.0.1.tar.gz -O | sudo docker load

* mkdir /opt/ibm-cloud-private-2.1.0.1 && cd /opt/ibm-cloud-private-2.1.0.1

* Stop iptables & firewalld, disable SELinux
    psh setenforce  0
    psh systemctl stop iptables
    psh systemctl stop firewalld

* Extract the sample configuration file from the installer image
    docker run -v $(pwd):/data -e LICENSE=accept ibmcom/icp-inception:2.1.0.1-ee cp -r cluster /data

* vim /opt/ibm-cloud-private-2.1.0.1/cluster/hosts
    [master]
    9.45.221.117

    [worker]
    9.45.221.92

    [proxy]
    9.45.221.116

    #[management]
    #4.4.4.4

* SSH keys
    (I'll bypass the key generation because it's already there. For more, see to https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.1/installing/ssh_keys.html)
    cd /opt/ibm-cloud-private-2.1.0.1 && cp ~/.ssh/id_rsa ./cluster/ssh_key

* mv installation file
    mkdir -p cluster/images && cp /DST/ibm-cloud-private-x86_64-2.1.0.1.tar.gz  ./cluster/images/

* Setup Calico network
    vim /opt/ibm-cloud-private-2.1.0.1/cluster/config.yaml
    calico_ip_autodetection_method: can-reach=9.45.221.117

* I skipped HA cluster (HA master, proxy nodes) because we only have 3 servers for testing purpose

* Enable iptables & disable firewalld

    psh systemctl enable iptables && psh systemctl start iptables
    psh systemctl enable firewalld && psh systemctl stop firewalld

* Deploy the environment
    cd /opt/ibm-cloud-private-2.1.0.1/cluster/ && docker run --net=host -t -e LICENSE=accept -v $(pwd):/installer/cluster ibmcom/icp-inception:2.1.0.1-ee install

        PLAY RECAP *****************************************************************************************************************************************************************************
        9.45.221.116               : ok=113  changed=32   unreachable=0    failed=0
        9.45.221.117               : ok=172  changed=51   unreachable=0    failed=0
        9.45.221.92                : ok=113  changed=32   unreachable=0    failed=0
        localhost                  : ok=231  changed=115  unreachable=0    failed=0


        POST DEPLOY MESSAGE ********************************************************************************************************************************************************************

        The Dashboard URL: https://9.45.221.117:8443, default username/password is admin/admin

        Playbook run took 0 days, 0 hours, 20 minutes, 27 seconds

## How to

* Restart cluster
    @Master, run:
    psh systemctl restart kubelet docker

    If you don't have psh installed, run the following cmd on each of your nodes:
    systemctl restart kubelet docker

## Issue & fix

TASK [check : Validating Firewall rules] ***********************************************************************************************************************************************
fatal: [9.45.221.92]: FAILED! => {"changed": false, "failed": true, "msg": "Please disable/stop your firewalld since this installation is in firewall disabled mode"}
fatal: [9.45.221.117]: FAILED! => {"changed": false, "failed": true, "msg": "Please disable/stop your firewalld since this installation is in firewall disabled mode"}
fatal: [9.45.221.116]: FAILED! => {"changed": false, "failed": true, "msg": "Please disable/stop your firewalld since this installation is in firewall disabled mode"}

NO MORE HOSTS LEFT *********************************************************************************************************************************************************************

NO MORE HOSTS LEFT *********************************************************************************************************************************************************************

PLAY RECAP *****************************************************************************************************************************************************************************
9.45.221.116               : ok=26   changed=12   unreachable=0    failed=1
9.45.221.117               : ok=26   changed=12   unreachable=0    failed=1
9.45.221.92                : ok=26   changed=12   unreachable=0    failed=1

Playbook run took 0 days, 0 hours, 6 minutes, 7 seconds

This error happened after I disabled iptables, firewalld by using: 
psh systemctl stop firewalld && psh systemctl disable firewalld
psh systemctl stop iptables && psh systemctl disable iptables

What I did to resolve the issue was:
enable and start both firewalld and iptables, then stop and disable firewalld only. Seems iptables should not be disabled?



