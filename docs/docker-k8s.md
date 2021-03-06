# 第一部分：Docker

## 虚拟化技术的发展

![](https://github.com/majinghe/DevOps/blob/main/images/container-history.png)

## docker 概述
### 什么是 docker
* 是一家公司
* 是一种虚拟化技术
* 是一个容器
* 将应用程序和运行时环境进行打包封装(host, runtime, code, operating system, tools, libraries)，实现了 **build once, running anywhere**。

### docker 的核心概念
* 镜像
* 镜像仓库
* Dockerfile
* 容器

### docker 的架构
![](https://github.com/majinghe/DevOps/blob/main/images/docker-arch.png)

## docker 的使用

### 常规应用部署

* Step 1 写代码
```
package main

import (
    "fmt"
    "log"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello DevOpsDay, this is xiaomage")
}

func main() {
    http.HandleFunc("/", handler)
    log.Fatal(http.ListenAndServe(":9999", nil))
}
```

* Step 2 安装测试环境

需要确保环境安装了合适版本的运行环境`GO`。

* Step 3 启动运行

```
go main.go
```

* Step 4 测试应用

打开浏览器的 `localhost:9999`，查看输出内容

![](https://github.com/majinghe/DevOps/blob/main/images/check.png)

如果开发测试完毕，就继续其他环境的安装、部署和测试工作。其实是上面 2、3、4 步的重复工作。这其中会带来以下几个问题：

* 环境漂移（无法保证所有测试环境的一致性）
* 应用程序运行环境的维护繁琐（安装、升级甚至回退）
* 迁移成本高（需要 clone 环境）

### 使用 docker 来部署

* Step 1 编写代码（同上）
* Step 2 编写 `Dockerfile`

```
FROM golang:1.12.9-alpine3.9

MAINTAINER <devops008@sina.com>

WORKDIR /usr/src/app/

COPY main.go /usr/src/app

CMD ["go","run", "main.go"]
```
* Step 3 构建镜像并推送至镜像仓库

[![Build Docker Image](https://asciinema.org/a/6JPchmyFPnoetUUc7HJnZc7qh.svg)](https://asciinema.org/a/6JPchmyFPnoetUUc7HJnZc7qh)

* 部署应用

使用如下部署应用
```
$ docker run --rm -p 9999:9999 dllhb/docker-share:v0.3
```
然后可以在浏览器中输入 `localhost:9999` 查看应用。或者用如下的命令行查看结果

```
$ curl localhost:9999
Hello DevOpsDay, this is xiaomage
```

如果需要部署其他环境时，可以直接拿上述的镜像来直接使用 `docker run` 命令来部署。**这样就省去了在其他服务器上安装运行时环境和依赖库的麻烦了**。这就是 `docker` 为我们带来的便利之一。

# 第二部分：Kubernetes

## 演进史

![](https://github.com/lhb008/DevOps/blob/main/images/k8s_history.png)

**Kubernetes 是一个开源容器编排引擎，用于对容器化应用程序的部署、扩展和管理进行自动化管理**。KuberKubernetes简称K8S(除去开头的K和结尾的S，中间正好有八个字符，故号称K8S)是谷歌开源的一个全新的基于容器技术的分布式架构方案，同时也是一个全新的容器编排工具。Kubernetes的前身是谷歌的Borg系统，始于2003年，它是在谷歌内部使用的一个容器管理系统，它管理着来自数千个不同应用程序的数十万个作业，跨越许多集群，而每个集群拥有多达数万台计算机。2013年，另外一个Omega系统在谷歌内部应用，Omega可以看作是Borg的延伸，在大规模集群管理以及性能方面优于Borg。但是Borg和Omega都是谷歌内部使用的系统，也就是所谓的闭源系统。直到2014年，谷歌以开源的形式推出了Kubernetes系统，同年6月7号在github上完成了第一次提交，随后的7月10号，微软，IBM，Redhat，Docker加入了Kubernetes社区。在2015年7月Kubernetes正式发布了v1.0版本，直到今年6月19号发布了v1.15版本，而且1.16版本正在紧张开发之中。可以看出短短四年已经发布了十六个版本，而且速度越来越快。其实在容器编排领域除了有Kubernetes，还有Docker Swarm和Mesos，在初期，这三者呈三足鼎立之势，但是现在Kubernetes却一枝独秀，Kubernetes已经发展成为一个Kubernetes生态圈，而且以Kubernetes为核心发展起来的的CloudNative的发展势头也很迅猛。

## Kubernetes 架构

`Kubernetes`是一种`Master/Node`架构。

#![](https://github.com/majinghe/DevOps/blob/main/images/k8s-arch.png)
![](https://github.com/majinghe/DevOps/blob/main/images/full-kubernetes-model-architecture.png)

### Master 节点

Master 负责管理和维护Kubernetes集群的状态。

![](https://github.com/majinghe/DevOps/blob/main/images/kubernetes-master-elements.png)

### Node 节点

接受来自于Master的指令，运行和维护运行在Node节点上的相关负载(容器)。

![](https://github.com/majinghe/DevOps/blob/main/images/components-worker-node-kubernetes.png)

## 核心组件

### 控制面核心组件（Master）

* kube-apiserver

API server 是`Kubernetes`控制面的一个组件，用来暴漏`Kubernetes`的 API。也是所有外部请求的唯一入口。

* etcd

是一个高可用的键值存储数据库，用来存储`kubernetes`集群的所有数据。

* kube-scheduler

控制面的组件之一，用来对 `pod`（`Kubernetes` 最基本的调度单元）进行调度，为其分配合适的`node`。

* kube-controller-manager

控制面的组件之一，用来运行 `controller`逻辑。有多种`controller`，比如常用的`Replication controller`、`Node controller` 等等。

> controller 是一个异步运行的脚本，来确保`Kubernetes`系统的当前状态和我们期望的状态是一致的。

### 数据面核心组件（Node）

* kubelet

运行在每个`Node`上的一个代理，用来确保每个`Pod`中的`Container`运行正常。

* kube-proxy 

运行于集群每个节点上的一个网络代理，主要用来实现`Kubernetes`的`Service`（下面会讲到）。

* Container runtime

用来负责对运行容器进行管理的组件。也是`Kubernetes`编排和管理的重点。目前`Kubernetes`支持多种容器运行时`Docker`、`Containerd`、`CRI-O`等。

### Kubernetes 集群

**一个特定领域的软件部署在多台服务器上并做为一个整体对外提供服务，这个整体就称之为集群**。 Kubernetes 集群有 Master 节点和 Node 节点组成。

#### Master 托管

```kubectl get  nodes
NAME                                               STATUS   ROLES    AGE   VERSION
ip-172-31-12-242.cn-northwest-1.compute.internal   Ready    <none>   34d   v1.17.9-eks-4c6976
ip-172-31-30-20.cn-northwest-1.compute.internal    Ready    <none>   34d   v1.17.9-eks-4c6976
ip-172-31-40-170.cn-northwest-1.compute.internal   Ready    <none>   34d   v1.17.9-eks-4c6976
```

#### Master 非托管

```$ kubectl get  nodes
NAME        STATUS   ROLES    AGE    VERSION
k8stest01   Ready    master   204d   v1.11.0
k8stest02   Ready    <none>   204d   v1.11.0
k8stest03   Ready    <none>   204d   v1.11.0
k8stest04   Ready    <none>   204d   v1.11.0
```

## 一些核心概念

### Pod

Pod 是 Kubernetes 中最基本的调度单元，是一个或者一组容器的一个集合，Pod 内的容器共享网络、存储，且按照规范控制着容器运行。

#### Pod 在 Kubernetes 中的使用方式

* 一个`Pod`中一个`Container`

```
$ kubectl -n test get pods
NAME   READY   STATUS    RESTARTS   AGE
test   1/1     Running   0          7s
```

* 一个`Pod`中多个`Container`（可参考这个[使用场景](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/))

```
$ kubectl -n test get pods
NAME   READY   STATUS    RESTARTS   AGE
test   2/2     Running   0          6s
```

#### Pod 的几种状态

* **Pending**：集群已经接受创建请求并创建了`Pod`，但是`Pod`内的容器还没有运行成功。

* **Running**：`Pod`已经在`Node`上创建成功，且运行成功。**这是大家期望得到的状态**。

* **Succeeded**：`Pod`中的所有容器都成功终止，且不会重启。

* **Failed**：`Pod`中的所有容器都被终止，但至少其中一个容器终止失败。

* **Unknown**：无法确定`Pod`的状态。


#### Pod 的创建

`yaml` 文件
```
apiVersion: v1
kind: Pod
metadata:
  name: test
  namespace: test
spec:
  containers:
  - name: test
    image: ubuntu:18.04
    command: ["tail"]
    args: ["-f","/dev/null"]
```
将上述内容写到 `xxx.yaml` 文件中，然后执行如下内容即可创建一个 `Pod`:
```
$ kubectl -n your_namespace apply -f xxx.yaml
```
整个流程可以查看这个[链接](https://asciinema.org/a/tr60QeEzIyovgByBlMCj0hFzL)

#### Pod 的创建流程

![](https://github.com/majinghe/DevOps/blob/main/images/k8s-pod-creation.png)

* 客户端发送`Pod`创建请求（发送至`API server`之前，客户端会有一个验证，确保创建资源的合法性）

* `API server`将`Pod`的信息写入到`etcd`数据库中。写入成功会给`API server`和`etcd`发送确认信息。

* `API server`将`etcd`的状态进行反映。此时所有的`Kubernetes`组件都在监听`API server`的变化。

* `kube-scheduler`通过它自身的监听器探测到`API server`中关于`Pod`创建的信息，然后`kube-scheduler`将分配一个`Node`用来承载这个`Pod`。同时将此信息更新到`API server`。

* `etcd` 将`Node`与`Pod`的信息进行更新。

* `kubelet`感知到了`API server`的关于`Pod`创建的请求，且将`Pod`分配至自己所在的`Node`。

* `kubelet`创建并启动`Pod`，随后向`API server`更新信息。

* `API server`更新`etcd`。

* `etcd`返回确认信息，`API server`告知`kubelet`事件生效。

#### Pod 的设计模式

##### Init（初始化）容器

**Init 容器是在容器中启动主容器之前应该运行并完成的容器。**

主要使用场景：

* 应用程序或主容器需要一些先决条件（比如启动之前安装一些软件、完成数据库设置、文件系统的权限修改等）。

* 希望延迟主容器启动。

##### Sidecar（边车）容器

**Sidecar 模式可以在不更改的情况下扩展并增强当前容器的功能。**

主要使用场景：

* 将日志事件发送到外部服务器。

* 希望扩展或增强现有单个容器`Pod`的功能但不想更改现有容器`Pod`功能。

>Init 容器在主容器启动之前会退出；Sidecar 容器却和主容器同生共死。


### Deployment

`Deployment` 对 `Pod` 和 `ReplicaSets` 进行声明式的更新。(说人话就是 `Deployment` 能够对 `Pod` 进行管理)。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test
  labels:
    app: test
spec:
  replicas: 3
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - name: test
        image: ubuntu:18.04
        command: ["tail"]
        args: ["-f","/dev/null"]
```
> 可以理解为你 `ReplicaSet` + `Pod` = `Deployment`。

### Service

将运行在一组pod上的应用程序公开为网络服务的抽象方法。因为 `Pod` 的 `IP` 是经常变化的，为了确保其他服务能够准确发现并连接到`Pod` 所提供服务，需要将链接信息进行固定化。

```
$ kubectl -n demo get svc
NAME             TYPE           CLUSTER-IP       EXTERNAL-IP                                                                       PORT(S)          AGE
apple-service    LoadBalancer   10.100.62.213    a3e5ac0f2186e45dd95d0bc1818d6a6e-940893763.cn-northwest-1.elb.amazonaws.com.cn    9999:30173/TCP   25d
banana-service   LoadBalancer   10.100.160.152   ae410e74749ba42b091e8d53ce070886-2128295197.cn-northwest-1.elb.amazonaws.com.cn   9999:31580/TCP   25d
```
> Serivce 的类型一般有 NodePort，LoadBlancer 和 Ingress 三种。

### Ingress

Kubernetes 中的一种服务暴露方式。

![](https://github.com/lhb008/DevOps/blob/main/images/k8s-svc.png)

## Kubernetes 环境

* Minikube

* Docker Desktop Kubernetes

* kubernetes kind


## 参考资料

[Docker 官网](https://docs.docker.com/)

[Kubernetes 官网](https://kubernetes.io/docs/)

[Pod 创建流程](https://github.com/jamiehannaford/what-happens-when-k8s)

[Kubernetes 架构](https://phoenixnap.com/kb/understanding-kubernetes-architecture-diagrams)
