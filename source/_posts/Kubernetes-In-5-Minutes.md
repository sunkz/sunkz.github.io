title: Kubernetes In 5 Minutes
author: Sunkz
tags:
  - Kubernetes
  - devops

photos:

  - https://s2.ax1x.com/2019/09/29/uGAg8f.png

categories: []
date: 2019-07-12 15:26:00

---
### 核心

#### Cluster

```
Cluster是计算、存储和网络资源的集合，Kubernetes利用这些资源运行各种基于容器的应用。
```

#### Master

#### Node

#### Pod

#### Controller

```
定义了Pod的部署特性，比如有几个副本、在什么样的Node上运行等
```

- Deployment

- ReplicaSet

  ```
  使用Deployment时会自动创建ReplicaSet，也就是说Deployment是通过ReplicaSet来管理Pod的多个副本的，我们通常不需要直接使用ReplicaSet。
  ```

- DemonSet

- StatefuleSet

  ```
  能够保证Pod的每个副本在整个生命周期中名称是不变的，而其他Controller不提供这个功能。当某个Pod发生故障需要删除并重新启动时，Pod的名称会发生变化，同时StatefuleSet会保证副本按照固定的顺序启动、更新或者删除
  ```

- Job

  ```
  用于运行结束就删除的应用
  ```

#### Service

```
Service定义了外界访问一组特定Pod的方式。Service有自己的IP和端口，Service为Pod提供了负载均衡.
Kubernetes运行容器（Pod）与访问容器（Pod）这两项任务分别由Controller和Service执行。
```

#### Namespace

```
当多个用户或项目组使用同一个Kubernetes Cluster，Namespace可以将他们创建的Controller、Pod等资源分开.
Namespace可以将一个物理的Cluster逻辑上划分成多个虚拟Cluster，每个Cluster就是一个Namespace。不同Namespace里的资源是完全隔离的。
Kubernetes默认创建了两个Namespace. default：创建资源时如果不指定，将被放到这个Namespace中。kube-system：Kubernetes自己创建的系统资源将放到这个Namespace中。
```

### 架构

#### Master节点

```
Master是Kubernetes Cluster的大脑，运行着的Daemon服务包括kube-apiserver、kube-scheduler、kube-controller-manager、etcd和Pod网络（例如flannel）
```

![image.png](https://i.loli.net/2019/09/29/wEuTqnhJP6KtmSs.png)

- Api Server(kube-apiserver)

  ```
  API Server提供HTTP/HTTPS RESTful API，即Kubernetes API。API Server是Kubernetes Cluster的前端接口，各种客户端工具（CLI或UI）以及Kubernetes其他组件可以通过它管理Cluster的各种资源。
  ```

- Scheduler(kube-scheduler)

  ```
  Scheduler负责决定将Pod放在哪个Node上运行。Scheduler在调度时会充分考虑Cluster的拓扑结构，当前各个节点的负载，以及应用对高可用、性能、数据亲和性的需求。
  ```

- Controller Manager(kube-controller-manager)

  ```
  Controller Manager负责管理Cluster各种资源，保证资源处于预期的状态。Controller Manager由多种controller组成，包括replicationcontroller、endpoints controller、namespace controller、service accounts controller等。
  不同的controller管理不同的资源。例如，replication controller管理Deployment、StatefulSet、DaemonSet的生命周期，namespace controller管理Namespace资源。
  ```

- Etcd

  ```
  负责保存Kubernetes Cluster的配置信息和各种资源的状态信息。当数据发生变化时，etcd会快速地通知Kubernetes相关组件。
  ```

- Pod Network

  ```
  Pod要能够相互通信，Kubernetes Cluster必须部署Pod网络，flannel是其中一个可选方案。
  ```

#### Node节点

```
Node是Pod运行的地方，Kubernetes支持Docker、rkt等容器Runtime。Node上运行的Kubernetes组件有kubelet、kube-proxy和Pod网络（例如flannel）
```

![image.png](https://i.loli.net/2019/09/29/jI7aLACM6NKr1wp.png)

- kubelet

  ```
  kubelet是Node的agent，当Scheduler确定在某个Node上运行Pod后，会将Pod的具体配置信息（image、volume等）发送给该节点的kubelet，kubelet根据这些信息创建和运行容器，并向Master报告运行状态。
  ```

- kube-proxy

  ```
  每个Node都会运行kube-proxy服务，它负责将访问service的TCP/UDP数据流转发到后端的容器。如果有多个副本，kube-proxy会实现负载均衡。
  ```

- Pod Network

  ```
  Pod要能够相互通信，Kubernetes Cluster必须部署Pod网络，flannel是其中一个可选方案。
  ```

#### 1Master+2Nodes

![image.png](https://i.loli.net/2019/09/29/jGv8Qe9VAi7W3h1.png)

1. Master上也可以运行应用,master也同时是一个Node,其上也有kubelet kube-proxy
2. 几乎所有的Kubernetes组件也运行在Pod中.kubelet是唯一没有以容器形式运行的Kubernetes组件，它在Ubuntu中通过Systemd服务运行
3. Kubernetes的系统组件都被放到kube-system namespace中
4. 还有一个kube-dns组件，它为Cluster提供DNS服务

#### 部署一个httpd-app

![image.png](https://i.loli.net/2019/09/29/miHg5UpBeEovdD4.png)

1. kubectl发送部署请求到API Server
2. API Server通知Controller Manager创建一个deployment资源。
3. Scheduler执行调度任务，将两个副本Pod分发到k8s-node1和k8s-node2。
4. k8s-node1和k8s-node2上的kubectl在各自的节点上创建并运行Pod
5. 应用的配置和当前状态信息保存在etcd中，执行kubectl get pod时API Server会从etcd中读取这些数据。
6. flannel会为每个Pod都分配IP。因为没有创建service，所以目前kube-proxy还没参与进来。

### 运行应用

```
两种资源创建方式 : 命令式 & 声明式
```

#### Deployment

![image.png](https://i.loli.net/2019/09/29/GQNy42nuYgkfhq7.png)

1. 用户通过kubectl创建Deployment
2. Deployment创建ReplicaSet
3. ReplicaSet创建Pod

##### nginx.yml

```yaml
apiVersion: extensions/v1beta1  #当前配置格式的版本
kind: Deployment #要创建的资源类型
metadata: #该资源的元数据,name是必须的元数据
	name: nginx-deployment
spec: #该deployment的规格说明
	replicas: 2 #副本数,默认为1
	template: #定义Pod的模板
		metadata: #Pod的元数据,至少要定义一个label
			labels: #label的key和value任意指定
				app: web_server
		spec: #定义Pod中每一个容器的属性
			containers:
			- name: nginx
              images: nginx:1.7.9
```

```shell
kubectl apply -f nginx.yml
```

##### Scalling & Failover

```
默认Pod不会调度到Master节点
```

```
node2 shutdown k8s将在node1上再启动一个Pod维持replicas=2, 进行故障转移
node2 再次恢复时, k8s不会将node1上的Pod调度到node2上
```

##### Label

```
label是key-value对，各种资源都可以设置label.默认配置下，Scheduler会将Pod调度到所有可用的Node
```

```shell
#比如执行如下命令标注k8s-node1是配置了SSD的节点
kubectl label node k8s-node1 disktype=ssd
```

```yaml
		spec: #定义Pod中每一个容器的属性
			containers:
			- name: nginx
              images: nginx:1.7.9
            nodeSelector: #将该Pod部署到打有ssd label的Node上
            	disktype: ssd
```

#### DaemonSet

1. 每个Node上最多只能运行一个副本。

2. 常运行:glusterd logstash Prometheus等

3. Kubernetes自己就在用DaemonSet运行系统组件

   DaemonSet kube-flannel-ds和kube-proxy分别负责在每个节点上运行flannel和kube-proxy组件

   因为flannel和kube-proxy属于系统组件，需要在命令行中通过--namespace=kube-system指定namespace kube-system。若不指定，则只返回默认namespace default中的资源。

##### kube-flannel-ds

- kube-flannel.yml

  ```yaml
  apiServer: extensions/v1beta1
  kind: DaemonSet
  metadata: 
  	name: kube-flannel-ds
  	namespace: kube-system
  	labels:
  		tier: node
  		app: flannel
  spec:
  	template:
  		metadata:
  			labels:
  				tier: node
  				app: flannel
  		spec:
  			hostNetwork: true #Node使用ubuntu宿主机网络,相当于docker run --network=hsot
  			nodeSelector:
  				beta.kubernetes.io/arch: amd64
  			containers: #flannel Pod中运行的两个容器
  			- name: kube-flannel
  			  image: quay.io/coreos/flannel:v0.8.0-amd64
  			  command: ["/apt/bin/flanneld","--ip-masq","--kube-subnet-mgr"]
  			- name: install-cni
  			  image: quay.io/coreos/flannel:v0.8.0-amd64
  			  command: ["/bin/sh","-c","set -e -x; cp -f /etc/kube-flannel/cni0conf.json"]		
  ```

##### kube-proxy

```yaml
apiServer: extensions/v1beta1
kind: DaemonSet
metadata:
	labels:
		k8s-app: kube-proxy
    name: kube-proxy
    namespace: kube-system
spec: 
	selector:
		matchLabels:
			k8s:app: kube-proxy
    template:
    	metadata:
    		labels:
    			k8s-app: kube-proxy
    	spec:
    		containers: #定义kube-proxy Pod中运行的容器
    		- image: gcr.io/google_containers/kube-proxy-amd64:1.7.4
    		  name: kube-proxy
    		  command: #容器启动命令
              - /usr/local/bin/kube-proxy
              - --kubeconfig=/var/lib/kube-proxy/kubeconfig.conf
              - --cluster-cidr=10.244.0.0/16
              volumeMounts: #将host的/proc /sys映射到容器中
              - name: proc
                mountPath: /host/proc
              - name: sys
                mountPath: /host/sys
status: # 当前daemonset运行状态
	currentNumberScheduled: 3
	desiredNumberScheduled: 3
	numberAvailable: 3
	numberMissscheduled: 0
	observedGeneration: 1
	updatedNumberScheduled: 3
```

#### Job

```yaml
apiVersion: batch/v1
kind: Job #还可以是CronJob 下面的spec : schedule: ****
metadata:
	name: myjob
spec:
	parallelism: 2 #两个Pod并行执行
	completions: 6 #直到有6个Pod完成为止
	template:
		metadata:
			name: myjob
		spec:
			containers:
			- name: hello
			  image: busybox
			  command: ["echo","hello k8s job!"]
			restartPolicy: Never #设置为Never或者Onfail,此类资源只成功运行一次就销毁
```

### Service访问Pod

```
逻辑上代表的一组Pod,具体代表哪些Pod由Label筛选出来
```

```yaml
apiVersion: v1
kind: Service
metadata:  
	name: httpd-svc #该Service的name
spec:
	selector: #挑选label标有 run: httpd 的Pod作为Service的后端服务
		run: httpd
	ports: #将Service的8080端口映射到容器的80端口,使用TCP协议
	- protocol: TCP
	  port: 8080
	  targetPort: 80
```

```
Pod的IP在容器内部配置的,httpd-svc分配到一个ClusterIP:10.99.229.179通过该IP可以访问后端的Pod
Cluster内还提供了了一个Service kebernetes通过这个IP访问k8s Api Server Pod
```

```
Cluster IP是一个虚拟IP，是由Kubernetes节点上的iptables规则管理的。
Cluster的每一个节点都配置了相同的iptables规则，这样就确保了整个Cluster都能够通过Service的Cluster IP访问Service
```

![u8rcNj.png](https://s2.ax1x.com/2019/09/29/u8rcNj.png)

#### DNS访问Service

```
kubeadm部署时会默认安装kube-dns组件，kube-dns是一个DNS服务器。每当有新的Service被创建，kubedns会添加该Service的DNS记录。Cluster中的Pod可以通过<SERVICE_NAME>.<NAMESPACE_NAME>访问Service。
```

```
例如可以用httpd-svc.default访问Service httpd-svc，同属一个namespace下的Pod通过Service DNS互访时,可以省去<NAMESPACE_NAME>
```

#### 外网访问Service

```
k8s提供了很多类型的Service : 
	ClusterIP: 只有Cluster内部的Pod和节点可以访问.
	NodePort: Service通过Cluster节点的静态端口对外提供服务。Cluster外部可以通过<NodeIP>:<NodePort>访问Service。
	LoadBalancer:Service利用云厂商特有的load balancer对外提供服务，云厂商负责将load balancer的流量导向Service。
```

##### NodePort

```yaml
apiVersion: v1
kind: Service
metadata:  
	name: httpd-svc #该Service的name
spec:
	type: NodePort
	selector: #挑选label标有 run: httpd 的Pod作为Service的后端服务
		run: httpd
	ports: #将Service的8080端口映射到容器的80端口,使用TCP协议
	- protocol: TCP
	  port: 8080
	  targetPort: 80
```

![u8gJfJ.png](https://s2.ax1x.com/2019/09/29/u8gJfJ.png)

```
k8s节点监听到来自32312端口的流量转发到httpd-svc的ClusterIP的8080端口
```

```yaml
	ports: 
	- protocol: TCP
	  nodePort: 3000 #k8s节点监听的端口
	  port: 8080 #svc的ClusterIP上监听的端口
	  targetPort: 80 #svc后端的Pod监听的端口
```

![u8hdoV.png](https://s2.ax1x.com/2019/09/29/u8hdoV.png)

```
由于Cluster所有的节点配置有相同的iptables规则,所以三个k8s节点IP:32312都可实现对httpd-svc的访问
```

```
k8sNode---iptables--- clusterIP ----iptables---- pod
k8s Node和ClusterIP在各自端口上接收到的请求都会通过iptables转发到Pod的targetPort
```

### Health Check

```
用户通过Liveness探测可以告诉Kubernetes什么时候通过重启容器实现自愈；Readiness探测则是告诉Kubernetes什么时候可以将容器加入到Service负载均衡池中，对外提供服务。
```

### Volume

```
数据外挂
```

### Secret ConfigMap

```
配置外挂
```

### Helm

- chart : 软件包
- release : 软件包运行的实例

![uGCcd0.png](https://s2.ax1x.com/2019/09/29/uGCcd0.png)

```
Tiller服务器运行在Kubernetes集群中，它会处理Helm客户端的请求，与Kubernetes API Server交互.
Helm客户端负责管理chart，Tiller服务器负责管理release。
```

### 网络

#### Pod内容器之间的通信

```
当Pod被调度到某个节点，Pod中的所有容器都在这个节点上运行，这些容器共享相同的本地文件系统、IPC和网络命名空间。
不同Pod之间不存在端口冲突的问题，因为每个Pod都有自己的IP地址。当某个容器使用localhost时，意味着使用的是容器所属Pod的地址空间。
```

#### Pod之间的通信

```
Pod的IP是集群可见的，即集群中的任何其他Pod和节点都可以通过IP直接与Pod通信，这种通信不需要借助任何网络地址转换、隧道或代理技术。Pod内部和外部使用的是同一个IP，这也意味着标准的命名服务和发现机制，比如DNS可以直接使用。
```

#### Pod与Service的 通信

```
Pod间可以直接通过IP地址通信，但前提是Pod知道对方的IP。在Kubernetes集群中，Pod可能会频繁地销毁和创建，也就是说Pod的IP不是固定的。为了解决这个问题，Service提供了访问Pod的抽象层。无论后端的Pod如何变化，Service都作为稳定的前端对外提供服务。同时，Service还提供了高可用和负载均衡功能，Service负责将请求转发给正确的Pod
```

#### 外部访问

```
无论是Pod的IP还是Service的Cluster IP，它们只能在Kubernetes集群中可见，对集群之外的世界，这些IP都是私有的。
```

```
NodePort: <NodeIP>:<NodePort> --> <ClusterIP>:<SvcPort>
LoadBalance: Cloud Provider
```

#### CNI

```
CNI规范使得Kubernetes可以灵活选择多种Plugin实现集群网络。
```

#### Network Policy

```
Network Policy赋予了Kubernetes强大的网络访问控制机制。
```

参考资料
[1]CloudMan．每天5分钟玩转Kubernetes[M]．北京：清华大学出版社，2018
