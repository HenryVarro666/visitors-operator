<!--换了intel的cpu，小爷我卷土重来-->
安装docker，k8s，helm

docker：官网下载安装

安装helm
```shell
brew install helm
```

安装kubectl
```shell
brew install kubectl
```
> kubernetes-cli 和 kubectl是一个东西

安装minikube
```shell
brew install minikube
```

本地运行minikube环境
```shell
minikube start
```

> 1.  [Kubernetes 文档](https://kubernetes.io/zh-cn/docs/)
> 2.  [任务](https://kubernetes.io/zh-cn/docs/tasks/)
> 3.  [安装工具](https://kubernetes.io/zh-cn/docs/tasks/tools/)

安装工具：
1. - [x] kubectl
	Kubernetes 命令行工具，[kubectl](https://kubernetes.io/docs/reference/kubectl/kubectl/)，使得你可以对 Kubernetes 集群运行命令。 你可以使用 kubectl 来部署应用、监测和管理集群资源以及查看日志。
	有关更多信息，包括 kubectl 操作的完整列表，请参见[`kubectl` 参考文件](https://kubernetes.io/zh/docs/reference/kubectl/)。
2. kind(暂不需要)
	[`kind`](https://kind.sigs.k8s.io/docs/) 让你能够在本地计算机上运行 Kubernetes。 `kind` 要求你安装并配置好 [Docker](https://docs.docker.com/get-docker/)。
	kind [快速入门](https://kind.sigs.k8s.io/docs/user/quick-start/)页面展示了 开始使用 `kind` 所需要完成的操作。
3. - [x] minikube
	与 `kind` 类似，[`minikube`](https://minikube.sigs.k8s.io/) 是一个工具， 能让你在本地运行 Kubernetes。 `minikube` 在你本地的个人计算机（包括 Windows、macOS 和 Linux PC）运行一个单节点的 Kubernetes 集群，以便你来尝试 Kubernetes 或者开展每天的开发工作。
	如果你关注如何安装此工具，可以按官方的 [Get Started!](https://minikube.sigs.k8s.io/docs/start/)指南操作。
4. kubeadm(暂不需要)
	你可以使用 [kubeadm](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/) 工具来 创建和管理 Kubernetes 集群。 该工具能够执行必要的动作并用一种用户友好的方式启动一个可用的、安全的集群。
	[安装 kubeadm](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) 展示了如何安装 kubeadm 的过程。 一旦安装了 kubeadm，你就可以使用它来 [创建一个集群](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)。
	
	
	---
	安装operator-sdk
	```shell
	brew install operator-sdk
	```
	
	安装gopls
	```shell
	brew install gopls
	```
	
	安装go
	```shell
	brew install go
	```
	> 直接安装go 和 go@1.17还是有点区别的。直接安装1.17就查不到version

	Problem:
	> bash: go: command not found
	> make: *** [controller-gen] Error 127
	
	Solution:
	1. 前提：
	```shell
	brew install go
	```
	> 默认直接安装是go 1.18.3

	```shell
	brew install go @1.17
	```
	> 指定go的版本：go 1.17.11

	```shell
	go env #查看详细信息
	```
	
	2. 改动
	```shell
	brew unlink go
	brew link --force go @1.17
	```
	然后再部署
	```shell
	make deploy
	```
	
---
添加repo repo
```shell
helm repo add presslabs https://presslabs.github.io/charts
helm repo ungrade
helm repo list
```
	
---
> go get: installing executables with 'go get' in module mode is deprecated.

To adjust and download dependencies of the current module, use 'go get -d'.
To install using requirements of the current module, use 'go install'.
To install ignoring the current module, use 'go install' with a version,
like 'go install example.com/cmd@latest'.
For more information, see https://golang.org/doc/go-get-install-deprecation
or run 'go help get' or 'go help install'.
	
	
---
Problem:
> error: resource mapping not found for name: "visitors-operator-controller-manager-metrics-monitor" namespace: "visitors-operator-system" from "STDIN": no matches for kind "ServiceMonitor" in version "monitoring.coreos.com/v1"
> ensure CRDs are installed first
> make: *** [deploy] Error 1

Solution(**not solved**):
[# no matches for kind "ServiceMonitor" in version "monitoring.coreos.com/v1" #1866](https://github.com/prometheus-operator/prometheus-operator/issues/1866)
[Can't deploy v21 kube-prometheus to fresh cluster - CRDs race condition #1571](https://github.com/prometheus-operator/prometheus-operator/issues/1571)
这个是应对unable to recognize这个错误的

但是：
Problem:
> ServiceMonitor not found in monitoring.coreos.com/v1

Solution:
[ServiceMonitor not found in monitoring.coreos.com/v1](https://stackoverflow.com/questions/51095556/servicemonitor-not-found-in-monitoring-coreos-com-v1)
	The ServiceMonitor resource not part of Kubernetes itself. It is a custom resource which is part of the Prometheus operator as described here.
	**Make sure that you have installed the Prometheus operator (including the custom resources) beforehand to enable creating a ServiceMonitor object.**
	
	把dependencies.txt里面的东西先装一下
	add repo然后helm install
	Prometheus就是用来monitor的

---
Until now,
the necessary **Custom Resource Definition(CRD)** called ***VisitorsApp*** should be automatically created. 
Then create a **Custom Resource (CR)** by applying the yaml file in the **config/samples/** folder

```shell
kubectl apply -f config/samples/example.com_v1beta1_visitorsapp.yaml
```

A CR called visitorsapp-sample should have been generated. But note that by now, neither frontend pods nor backend pods are created. This is because they are all waiting for the database pods to be up. 
However, our operator no longer create MySQL pods by ourselves like the old version does. In order to make it easier to achieve capability levels 3 to 5, an open source MySQL operator is needed to create a MySQL cluster. And in our demo, we choose **presslabs MySQL Operator**.

Helm, a package manager for Kubernetes can be helpful for an easy installation of the Operators published on artifacthub.io. Only one single command should be enough. Make sure that you have Helm installed on your computer, and have added presslabs to your repositories.


---
报错
> Error: INSTALLATION FAILED: failed to install CRD crds/mysql_v1alpha1_mysqlbackup.yaml: resource mapping not found for name: "mysqlbackups.mysql.presslabs.org" namespace: "" from "": no matches for kind "CustomResourceDefinition" in version "apiextensions.k8s.io/v1beta1"
> ensure CRDs are installed first


<!--
false:
helm install mysql-operator bitpoke/mysql-operator -n visitors-operator-system
-->

---
pod status出现CrashLoopBackOff

`CrashloopBackOff` 表示pod经历了 `starting` , `crashing` 然后再次 `starting` 并再次 `crashing` 。
[Kubernetes pod CrashLoopBackOff错误排查](https://cloud-atlas.readthedocs.io/zh_CN/latest/kubernetes/debug/k8s_crashloopbackoff.html)

---
kubectl get events里面的message出现liveness（存活检查）和readiness（就绪检查）

[k8s健康检查失败问题，如何解决](https://cloud.tencent.com/developer/article/1849122)


---
```shell
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get --show-kind --ignore-not-found -n visitors-operator-system
```
> 这边 --show-all 不行，会报错

---
**Cannot delete cluster #349**
[Cannot delete cluster #349](https://github.com/bitpoke/mysql-operator/issues/349)

```shell
# 检查cluster
kubectl get mysql -n visitors-operator-system
# 检查还剩下什么
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get --show-kind --ignore-not-found -n visitors-operator-system
```


[Finalizers](https://kubernetes.io/docs/concepts/overview/working-with-objects/finalizers/)
> **Note:** In cases where objects are stuck in a deleting state, avoid manually removing finalizers to allow deletion to continue. Finalizers are usually added to resources for a reason, so forcefully removing them can lead to issues in your cluster. This should only be done when the purpose of the finalizer is understood and is accomplished in another way (for example, manually cleaning up some dependent object).