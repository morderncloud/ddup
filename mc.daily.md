# Log learning progress

## 2018-07-18 14:49:15

* [@Javascript iterator, module, iterator, ES6]

function* fibs() {
  let a = 1;
  let b = 2;
  console.log('b4 while');
  while (true) {
  	console.log('b4 yield');
    yield a;
    console.log('af yield');
    [a, b] = [b, a + b];
    console.log('af assign');
  }
  console.log('out while');
}

let [first, second] = fibs();
console.log(first + ',' + second + ',' );

## 2018-07-17 12:48:55

* [@Javascript versions](https://www.w3schools.com/js/js_versions.asp)

* [@Javascript let const](http://es6.ruanyifeng.com/#docs/let)

## 2018-04-18 11:44:16

* [Kube API client libraries, supporting Go, Java, Javascript, Python, .Net](https://kubernetes.io/docs/reference/client-libraries/)

## 2018-04-17 14:58:11

* [Nginx Ingress Controller](https://github.com/kubernetes/ingress-nginx/blob/master/README.md)
* [Kube external storage provisioner](https://github.com/kubernetes-incubator/external-storage)
* [Glusterfs](https://www.gluster.org/, http://blog.51cto.com/wzlinux/1949441)
* [NFS](https://github.com/kubernetes-incubator/external-storage/tree/master/nfs)

## 2018-04-11 09:08:43

* [Helm](https://github.com/kubernetes/helm)
* [charts](https://github.com/kubernetes/helm/blob/master/docs/charts.md)

## 2018-03-29 08:10:27

* [Installing GlusterFS - a Quick Start Guide](https://docs.gluster.org/en/latest/Quick-Start-Guide/Quickstart/)

## 2018-03-28 18:10:21

* Deployed ICP on 117, 3 nodes

## 2018-03-27 18:21:35

* Deployed Kubernetes on 226, 3 nodes

## 2018-03-26

* Practised the useful commands of kubectl on minikube

## 2018-03-20

* Tina - added instruction document of install minikube on Linux VM

## 2018-03-15 17:46:26

* [An open platform to connect, manage, and secure microservices](https://istio.io/)
* [Habitat for build and deploy](https://www.habitat.sh/)
  A simple, flexible way to build, deploy, and manage cloud native applications.

* Kubernetes CN
  * [IBM开源技术微讲堂](http://www.k8smeetup.com/ibmcourse, https://www.ibm.com/developerworks/community/wikis/home?lang=zh#!/wiki/W30b0c771924e_49d2_b3b7_88a2a2bc2e43/page/IBM%E5%BC%80%E6%BA%90%E6%8A%80%E6%9C%AF%E5%BE%AE%E8%AE%B2%E5%A0%82)
  * https://feisky.gitbooks.io/kubernetes/zh/architecture/architecture.html
  * https://jimmysong.io/kubernetes-handbook/
  * https://www.gitbook.com/book/feisky/kubernetes/details
  * [Cloud Native expert](https://feisky.xyz/)
  * https://www.kubernetes.org.cn/practice
  * [Chris Richardson 的微服务架构网站](http://microservices.io/patterns/cn/index.html)

* Kubenetes Playground
  * [Play with Kubernetes](https://labs.play-with-k8s.com/)
  * [katacoda Kube Playground](https://www.katacoda.com/courses/kubernetes/playground)

## 2018-03-14 09:17:10

* [Shared by Tina: 将 Java 微服务部署在支持多语言的 Kubernetes 上](https://developer.ibm.com/cn/journey/deploy-java-microservices-on-kubernetes-with-polyglot-support/?cm_mmc=dwchina-_-homepage-_-dev-_-news)

## 2018-03-09 12:12:52

* [scalable-microservices-with-kubernetes learning](https://www.udacity.com/course/scalable-microservices-with-kubernetes--ud615)
* Added TODO list & Tools using
* Added learning note: Notes_Kubernetes_Basics.pdf
* Initialize the org/repo
