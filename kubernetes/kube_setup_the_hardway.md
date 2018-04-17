# Kubernetes The Hard Way

## [Ref](https://github.com/kelseyhightower/kubernetes-the-hard-way)

## Steps

    Prerequisites
    Installing the Client Tools
    Provisioning Compute Resources
    Provisioning the CA and Generating TLS Certificates
    Generating Kubernetes Configuration Files for Authentication
    Generating the Data Encryption Config and Key
    Bootstrapping the etcd Cluster
    Bootstrapping the Kubernetes Control Plane
    Bootstrapping the Kubernetes Worker Nodes
    Configuring kubectl for Remote Access
    Provisioning Pod Network Routes
    Deploying the DNS Cluster Add-on
    Smoke Test
    Cleaning Up

## Prerequisites

    The original doc uses Google Cloud Platform, but I'd use my testing servers:
    Master - in.9.lexbz2226 
    Worker1 - in.10.lexbz2202
    Worker2 - in.11.lexbz2184

## Installing the Client Tools

Install cfssl and kubectl

* @Master, install cfssl and cfssljson

    mkdir kube && cd kube

    wget  --timestamping \
    https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 \
    https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64

    chmod +x cfssl_linux-amd64 cfssljson_linux-amd64

    mv cfssl_linux-amd64 /usr/local/bin/cfssl && mv cfssljson_linux-amd64 /usr/local/bin/cfssljson

    cfssl version
        Version: 1.2.0
        Revision: dev
        Runtime: go1.6

* @Master, install kubectl

    wget https://storage.googleapis.com/kubernetes-release/release/v1.9.0/bin/linux/amd64/kubectl

    chmod +x kubectl && mv kubectl /usr/local/bin/ && kubectl version --client
        Client Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.0", GitCommit:"925c127ec6b946659ad0fd596fa959be43f0cc05", GitTreeState:"clean", BuildDate:"2017-12-15T21:07:38Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}

## Provisioning Compute Resources

    I'm using Master, Worker1 and Worker2

## TODO:





