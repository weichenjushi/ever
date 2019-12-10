技术002KPod

技术002KPod
===========

-  `技术002KPod <>`__

   -  `分类 <>`__

      -  `静态Pod <>`__
      -  `普通Pod <>`__

   -  `调度 <>`__
   -  `资源 <>`__
   -  `建议配置 <>`__

      -  `最大Pod数目 <>`__

   -  `特性 <>`__

      -  `Node affinity节点亲和性 <>`__
      -  `Pod间的亲和性和反亲和性 <>`__
      -  `启动策略Always/OnFailure/Never <>`__

   -  `参考文献 <>`__

分类
----

静态Pod
~~~~~~~

有人提到static
Pod，这种其实也属于节点固定，但这种Pod局限很大，比如：不能挂载configmaps和secrets等，这个由Admission
Controllers控制。

普通Pod
~~~~~~~

调度
----

-  已被调度的回调：

已被调度的pod根据FilterFunc中定义的逻辑过滤，nodeName不为空，返回true时，将会走Handler中定义的AddFunc、UpdateFunc、DeleteFunc，这个其实最终不会加入到podQueue中，但需要加入到本地缓存中，因为调度器会维护一份节点上pod列表的缓存。

-  未被调度的回调：

未被调度的pod根据FilterFunc中定义的逻辑过滤，nodeName为空且pod的SchedulerName和该调度器的名称一致时返回true；返回true时，将会走Handler中定义的AddFunc、UpdateFunc、DeleteFunc，这个最终会加入到podQueue中，kube-scheduler开始调度

资源
----

第一阶段由运行在master上的AttachDetachController负责，为这个PV完成
Attach 操作，为宿主机挂载远程磁盘；

第二阶段是运行在每个节点上kubelet组件的内部，把第一步attach的远程磁盘
mount
到宿主机目录。这个控制循环叫VolumeManagerReconciler，运行在独立的Goroutine，不会阻塞kubelet主控制循环。

完成这两步，PV对应的“持久化
Volume”就准备好了，POD可以正常启动，将“持久化
Volume”挂载在容器内指定的路径。

建议配置
--------

最大Pod数目
~~~~~~~~~~~

一个node允许最大Pod数是110，这是因为node
health机制会遍历这个node上的所有容器的状态

静态 pod指在特定的节点上直接通过
kubelet守护进程进行管理，APIServer无法管理。它没有跟任何的控制器进行关联，kubelet
守护进程对它进行监控，如果崩溃了，kubelet 守护进程会重启它。Kubelet
通过APIServer为每个静态 pod 创建 镜像 pod，这些镜像 pod 对于
APIServer是可见的（即kubectl可以查询到这些Pod），但是不受APIServer控制。

特性
----

NodeSelector->Node affinity -> Pod Affinity/AntiAffinity

Node affinity节点亲和性
~~~~~~~~~~~~~~~~~~~~~~~

两种类型 requiredDuringSchedulingIgnoredDuringExecution-》“hard”
-》必须满足
preferredDuringSchedulingIgnoredDuringExecution-》“soft”-》尽量满足
pods/pod-with-node-affinity.yaml

::

   apiVersion: v1
   kind: Pod
   metadata:
     name: with-node-affinity
   spec:
     affinity:
       nodeAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
           nodeSelectorTerms:

           - matchExpressions:
             - key: kubernetes.io/e2e-az-name

               operator: In
               values:

               - e2e-az1
               - e2e-az2

         preferredDuringSchedulingIgnoredDuringExecution:

         - weight: 1

           preference:
             matchExpressions:

             - key: another-node-label-key

               operator: In
               values:

               - another-node-label-value

     containers:

     - name: with-node-affinity

       image: k8s.gcr.io/pause:2.0

Pod间的亲和性和反亲和性
~~~~~~~~~~~~~~~~~~~~~~~

*Inter-pod affinity and anti-affinity require substantial amount of
processing which can slow down scheduling in large clusters
significantly. We do not recommend using them in clusters larger than
several hundred nodes.* 在大规模集群中会降低性能，因此，不建议使用。

*Note: Pod anti-affinity requires nodes to be consistently labelled,
i.e. every node in the cluster must have an appropriate label matching
topologyKey. If some or all nodes are missing the specified topologyKey
label, it can lead to unintended behavior.*
要求每个节点中必须存在该label标签

topologyKey代表节点 label的Key，默认的标签有 kubernetes.io/hostname
failure-domain.beta.kubernetes.io/zone
failure-domain.beta.kubernetes.io/region
beta.kubernetes.io/instance-type kubernetes.io/os kubernetes.io/arch
PodAntiAffinity配置，保证该该服务的两个实例不会共存在一个节点上。

::

   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: redis-cache
   spec:
     selector:
       matchLabels:
         app: store
     replicas: 3
     template:
       metadata:
         labels:
           app: store
       spec:
         affinity:
           podAntiAffinity:
             requiredDuringSchedulingIgnoredDuringExecution:

             - labelSelector:

                 matchExpressions:

                 - key: app

                   operator: In
                   values:

                   - store

               topologyKey: "kubernetes.io/hostname"
         containers:

         - name: redis-server

           image: redis:3.2-alpine

保证web-server的每个实例不会共存在同一个节点，尽量和标签为app=store的pod共存在同一台机器上

::

   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: web-server
   spec:
     selector:
       matchLabels:
         app: web-store
     replicas: 3
     template:
       metadata:
         labels:
           app: web-store
       spec:
         affinity:
           podAntiAffinity:
             requiredDuringSchedulingIgnoredDuringExecution:

             - labelSelector:

                 matchExpressions:

                 - key: app

                   operator: In
                   values:

                   - web-store

               topologyKey: "kubernetes.io/hostname"
           podAffinity:
             requiredDuringSchedulingIgnoredDuringExecution:

             - labelSelector:

                 matchExpressions:

                 - key: app

                   operator: In
                   values:

                   - store

               topologyKey: "kubernetes.io/hostname"
         containers:

         - name: web-app

           image: nginx:1.12-alpine

启动策略Always/OnFailure/Never
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Daemonset里的pod
   Template下必须有RestartPolicy，如果没指定，会默认为Always
-  另外Deployment、Statefulset的restartPolicy也必须为Always，保证pod异常退出，或者健康检查
   livenessProbe失败后由kubelet重启容器。https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment
-  Job和CronJob是运行一次的pod，restartPolicy只能为OnFailure或Never，确保容器执行完成后不再重启。https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/

参考文献
--------

`Pod调度源码分析 <https://mp.weixin.qq.com/s/UcpP4koV1tTRxfM7Vmi0Ug>`__

%23%20%E6%8A%80%E6%9C%AF002KPod%0A%0A%5BTOC%5D%0A%0A%23%23%23%20%E5%88%86%E7%B1%BB%0A%23%23%23%23%20%E9%9D%99%E6%80%81Pod%0A%E6%9C%89%E4%BA%BA%E6%8F%90%E5%88%B0static%20Pod%EF%BC%8C%E8%BF%99%E7%A7%8D%E5%85%B6%E5%AE%9E%E4%B9%9F%E5%B1%9E%E4%BA%8E%E8%8A%82%E7%82%B9%E5%9B%BA%E5%AE%9A%EF%BC%8C%E4%BD%86%E8%BF%99%E7%A7%8DPod%E5%B1%80%E9%99%90%E5%BE%88%E5%A4%A7%EF%BC%8C%E6%AF%94%E5%A6%82%EF%BC%9A%E4%B8%8D%E8%83%BD%E6%8C%82%E8%BD%BDconfigmaps%E5%92%8Csecrets%E7%AD%89%EF%BC%8C%E8%BF%99%E4%B8%AA%E7%94%B1Admission%20Controllers%E6%8E%A7%E5%88%B6%E3%80%82%0A%23%23%23%23%20%E6%99%AE%E9%80%9APod%0A%0A%23%23%23%20%E8%B0%83%E5%BA%A6%0A-%20%E5%B7%B2%E8%A2%AB%E8%B0%83%E5%BA%A6%E7%9A%84%E5%9B%9E%E8%B0%83%EF%BC%9A%0A%E5%B7%B2%E8%A2%AB%E8%B0%83%E5%BA%A6%E7%9A%84pod%E6%A0%B9%E6%8D%AEFilterFunc%E4%B8%AD%E5%AE%9A%E4%B9%89%E7%9A%84%E9%80%BB%E8%BE%91%E8%BF%87%E6%BB%A4%EF%BC%8CnodeName%E4%B8%8D%E4%B8%BA%E7%A9%BA%EF%BC%8C%E8%BF%94%E5%9B%9Etrue%E6%97%B6%EF%BC%8C%E5%B0%86%E4%BC%9A%E8%B5%B0Handler%E4%B8%AD%E5%AE%9A%E4%B9%89%E7%9A%84AddFunc%E3%80%81UpdateFunc%E3%80%81DeleteFunc%EF%BC%8C%E8%BF%99%E4%B8%AA%E5%85%B6%E5%AE%9E%E6%9C%80%E7%BB%88%E4%B8%8D%E4%BC%9A%E5%8A%A0%E5%85%A5%E5%88%B0podQueue%E4%B8%AD%EF%BC%8C%E4%BD%86%E9%9C%80%E8%A6%81%E5%8A%A0%E5%85%A5%E5%88%B0%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E4%B8%AD%EF%BC%8C%E5%9B%A0%E4%B8%BA%E8%B0%83%E5%BA%A6%E5%99%A8%E4%BC%9A%E7%BB%B4%E6%8A%A4%E4%B8%80%E4%BB%BD%E8%8A%82%E7%82%B9%E4%B8%8Apod%E5%88%97%E8%A1%A8%E7%9A%84%E7%BC%93%E5%AD%98%E3%80%82%0A-%20%E6%9C%AA%E8%A2%AB%E8%B0%83%E5%BA%A6%E7%9A%84%E5%9B%9E%E8%B0%83%EF%BC%9A%0A%E6%9C%AA%E8%A2%AB%E8%B0%83%E5%BA%A6%E7%9A%84pod%E6%A0%B9%E6%8D%AEFilterFunc%E4%B8%AD%E5%AE%9A%E4%B9%89%E7%9A%84%E9%80%BB%E8%BE%91%E8%BF%87%E6%BB%A4%EF%BC%8CnodeName%E4%B8%BA%E7%A9%BA%E4%B8%94pod%E7%9A%84SchedulerName%E5%92%8C%E8%AF%A5%E8%B0%83%E5%BA%A6%E5%99%A8%E7%9A%84%E5%90%8D%E7%A7%B0%E4%B8%80%E8%87%B4%E6%97%B6%E8%BF%94%E5%9B%9Etrue%EF%BC%9B%E8%BF%94%E5%9B%9Etrue%E6%97%B6%EF%BC%8C%E5%B0%86%E4%BC%9A%E8%B5%B0Handler%E4%B8%AD%E5%AE%9A%E4%B9%89%E7%9A%84AddFunc%E3%80%81UpdateFunc%E3%80%81DeleteFunc%EF%BC%8C%E8%BF%99%E4%B8%AA%E6%9C%80%E7%BB%88%E4%BC%9A%E5%8A%A0%E5%85%A5%E5%88%B0podQueue%E4%B8%AD%EF%BC%8Ckube-scheduler%E5%BC%80%E5%A7%8B%E8%B0%83%E5%BA%A6%0A%0A%23%23%23%20%E8%B5%84%E6%BA%90%0A%E7%AC%AC%E4%B8%80%E9%98%B6%E6%AE%B5%E7%94%B1%E8%BF%90%E8%A1%8C%E5%9C%A8master%E4%B8%8A%E7%9A%84AttachDetachController%E8%B4%9F%E8%B4%A3%EF%BC%8C%E4%B8%BA%E8%BF%99%E4%B8%AAPV%E5%AE%8C%E6%88%90%20Attach%20%E6%93%8D%E4%BD%9C%EF%BC%8C%E4%B8%BA%E5%AE%BF%E4%B8%BB%E6%9C%BA%E6%8C%82%E8%BD%BD%E8%BF%9C%E7%A8%8B%E7%A3%81%E7%9B%98%EF%BC%9B%0A%0A%E7%AC%AC%E4%BA%8C%E9%98%B6%E6%AE%B5%E6%98%AF%E8%BF%90%E8%A1%8C%E5%9C%A8%E6%AF%8F%E4%B8%AA%E8%8A%82%E7%82%B9%E4%B8%8Akubelet%E7%BB%84%E4%BB%B6%E7%9A%84%E5%86%85%E9%83%A8%EF%BC%8C%E6%8A%8A%E7%AC%AC%E4%B8%80%E6%AD%A5attach%E7%9A%84%E8%BF%9C%E7%A8%8B%E7%A3%81%E7%9B%98%20mount%20%E5%88%B0%E5%AE%BF%E4%B8%BB%E6%9C%BA%E7%9B%AE%E5%BD%95%E3%80%82%E8%BF%99%E4%B8%AA%E6%8E%A7%E5%88%B6%E5%BE%AA%E7%8E%AF%E5%8F%ABVolumeManagerReconciler%EF%BC%8C%E8%BF%90%E8%A1%8C%E5%9C%A8%E7%8B%AC%E7%AB%8B%E7%9A%84Goroutine%EF%BC%8C%E4%B8%8D%E4%BC%9A%E9%98%BB%E5%A1%9Ekubelet%E4%B8%BB%E6%8E%A7%E5%88%B6%E5%BE%AA%E7%8E%AF%E3%80%82%0A%0A%E5%AE%8C%E6%88%90%E8%BF%99%E4%B8%A4%E6%AD%A5%EF%BC%8CPV%E5%AF%B9%E5%BA%94%E7%9A%84%E2%80%9C%E6%8C%81%E4%B9%85%E5%8C%96%20Volume%E2%80%9D%E5%B0%B1%E5%87%86%E5%A4%87%E5%A5%BD%E4%BA%86%EF%BC%8CPOD%E5%8F%AF%E4%BB%A5%E6%AD%A3%E5%B8%B8%E5%90%AF%E5%8A%A8%EF%BC%8C%E5%B0%86%E2%80%9C%E6%8C%81%E4%B9%85%E5%8C%96%20Volume%E2%80%9D%E6%8C%82%E8%BD%BD%E5%9C%A8%E5%AE%B9%E5%99%A8%E5%86%85%E6%8C%87%E5%AE%9A%E7%9A%84%E8%B7%AF%E5%BE%84%E3%80%82%0A%0A%0A%23%23%23%20%E5%BB%BA%E8%AE%AE%E9%85%8D%E7%BD%AE%0A%23%23%23%23%20%E6%9C%80%E5%A4%A7Pod%E6%95%B0%E7%9B%AE%0A%E4%B8%80%E4%B8%AAnode%E5%85%81%E8%AE%B8%E6%9C%80%E5%A4%A7Pod%E6%95%B0%E6%98%AF110%EF%BC%8C%E8%BF%99%E6%98%AF%E5%9B%A0%E4%B8%BAnode%20health%E6%9C%BA%E5%88%B6%E4%BC%9A%E9%81%8D%E5%8E%86%E8%BF%99%E4%B8%AAnode%E4%B8%8A%E7%9A%84%E6%89%80%E6%9C%89%E5%AE%B9%E5%99%A8%E7%9A%84%E7%8A%B6%E6%80%81%0A%0A%E9%9D%99%E6%80%81%20pod%E6%8C%87%E5%9C%A8%E7%89%B9%E5%AE%9A%E7%9A%84%E8%8A%82%E7%82%B9%E4%B8%8A%E7%9B%B4%E6%8E%A5%E9%80%9A%E8%BF%87%20kubelet%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B%E8%BF%9B%E8%A1%8C%E7%AE%A1%E7%90%86%EF%BC%8CAPIServer%E6%97%A0%E6%B3%95%E7%AE%A1%E7%90%86%E3%80%82%E5%AE%83%E6%B2%A1%E6%9C%89%E8%B7%9F%E4%BB%BB%E4%BD%95%E7%9A%84%E6%8E%A7%E5%88%B6%E5%99%A8%E8%BF%9B%E8%A1%8C%E5%85%B3%E8%81%94%EF%BC%8Ckubelet%20%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B%E5%AF%B9%E5%AE%83%E8%BF%9B%E8%A1%8C%E7%9B%91%E6%8E%A7%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%B4%A9%E6%BA%83%E4%BA%86%EF%BC%8Ckubelet%20%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B%E4%BC%9A%E9%87%8D%E5%90%AF%E5%AE%83%E3%80%82Kubelet%20%E9%80%9A%E8%BF%87APIServer%E4%B8%BA%E6%AF%8F%E4%B8%AA%E9%9D%99%E6%80%81%20pod%20%E5%88%9B%E5%BB%BA%20%E9%95%9C%E5%83%8F%20pod%EF%BC%8C%E8%BF%99%E4%BA%9B%E9%95%9C%E5%83%8F%20pod%20%E5%AF%B9%E4%BA%8E%20APIServer%E6%98%AF%E5%8F%AF%E8%A7%81%E7%9A%84%EF%BC%88%E5%8D%B3kubectl%E5%8F%AF%E4%BB%A5%E6%9F%A5%E8%AF%A2%E5%88%B0%E8%BF%99%E4%BA%9BPod%EF%BC%89%EF%BC%8C%E4%BD%86%E6%98%AF%E4%B8%8D%E5%8F%97APIServer%E6%8E%A7%E5%88%B6%E3%80%82%0A%0A%23%23%23%20%E7%89%B9%E6%80%A7%0ANodeSelector-%3ENode%20affinity%20-%3E%20Pod%20Affinity%2FAntiAffinity%0A%23%23%23%23%20Node%20affinity%E8%8A%82%E7%82%B9%E4%BA%B2%E5%92%8C%E6%80%A7%0A%E4%B8%A4%E7%A7%8D%E7%B1%BB%E5%9E%8B%0ArequiredDuringSchedulingIgnoredDuringExecution-%E3%80%8B%E2%80%9Chard%E2%80%9D%20-%E3%80%8B%E5%BF%85%E9%A1%BB%E6%BB%A1%E8%B6%B3%0A%0ApreferredDuringSchedulingIgnoredDuringExecution-%E3%80%8B%E2%80%9Csoft%E2%80%9D-%E3%80%8B%E5%B0%BD%E9%87%8F%E6%BB%A1%E8%B6%B3%0Apods%2Fpod-with-node-affinity.yaml%20%0A%0A%60%60%60%0AapiVersion%3A%20v1%0Akind%3A%20Pod%0Ametadata%3A%0A%20%20name%3A%20with-node-affinity%0Aspec%3A%0A%20%20affinity%3A%0A%20%20%20%20nodeAffinity%3A%0A%20%20%20%20%20%20requiredDuringSchedulingIgnoredDuringExecution%3A%0A%20%20%20%20%20%20%20%20nodeSelectorTerms%3A%0A%20%20%20%20%20%20%20%20-%20matchExpressions%3A%0A%20%20%20%20%20%20%20%20%20%20-%20key%3A%20kubernetes.io%2Fe2e-az-name%0A%20%20%20%20%20%20%20%20%20%20%20%20operator%3A%20In%0A%20%20%20%20%20%20%20%20%20%20%20%20values%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20-%20e2e-az1%0A%20%20%20%20%20%20%20%20%20%20%20%20-%20e2e-az2%0A%20%20%20%20%20%20preferredDuringSchedulingIgnoredDuringExecution%3A%0A%20%20%20%20%20%20-%20weight%3A%201%0A%20%20%20%20%20%20%20%20preference%3A%0A%20%20%20%20%20%20%20%20%20%20matchExpressions%3A%0A%20%20%20%20%20%20%20%20%20%20-%20key%3A%20another-node-label-key%0A%20%20%20%20%20%20%20%20%20%20%20%20operator%3A%20In%0A%20%20%20%20%20%20%20%20%20%20%20%20values%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20-%20another-node-label-value%0A%20%20containers%3A%0A%20%20-%20name%3A%20with-node-affinity%0A%20%20%20%20image%3A%20k8s.gcr.io%2Fpause%3A2.0%0A%60%60%60%0A%0A%23%23%23%23%20Pod%E9%97%B4%E7%9A%84%E4%BA%B2%E5%92%8C%E6%80%A7%E5%92%8C%E5%8F%8D%E4%BA%B2%E5%92%8C%E6%80%A7%0A\ *Inter-pod%20affinity%20and%20anti-affinity%20require%20substantial%20amount%20of%20processing%20which%20can%20slow%20down%20scheduling%20in%20large%20clusters%20significantly.%20We%20do%20not%20recommend%20using%20them%20in%20clusters%20larger%20than%20several%20hundred%20nodes.*\ %20%E5%9C%A8%E5%A4%A7%E8%A7%84%E6%A8%A1%E9%9B%86%E7%BE%A4%E4%B8%AD%E4%BC%9A%E9%99%8D%E4%BD%8E%E6%80%A7%E8%83%BD%EF%BC%8C%E5%9B%A0%E6%AD%A4%EF%BC%8C%E4%B8%8D%E5%BB%BA%E8%AE%AE%E4%BD%BF%E7%94%A8%E3%80%82%0A%0A\ *Note%3A%20Pod%20anti-affinity%20requires%20nodes%20to%20be%20consistently%20labelled%2C%20i.e.%20every%20node%20in%20the%20cluster%20must%20have%20an%20appropriate%20label%20matching%20topologyKey.%20If%20some%20or%20all%20nodes%20are%20missing%20the%20specified%20topologyKey%20label%2C%20it%20can%20lead%20to%20unintended%20behavior.*\ %20%E8%A6%81%E6%B1%82%E6%AF%8F%E4%B8%AA%E8%8A%82%E7%82%B9%E4%B8%AD%E5%BF%85%E9%A1%BB%E5%AD%98%E5%9C%A8%E8%AF%A5label%E6%A0%87%E7%AD%BE%0A%0AtopologyKey%E4%BB%A3%E8%A1%A8%E8%8A%82%E7%82%B9%20label%E7%9A%84Key%EF%BC%8C%E9%BB%98%E8%AE%A4%E7%9A%84%E6%A0%87%E7%AD%BE%E6%9C%89%0Akubernetes.io%2Fhostname%0Afailure-domain.beta.kubernetes.io%2Fzone%0Afailure-domain.beta.kubernetes.io%2Fregion%0Abeta.kubernetes.io%2Finstance-type%0Akubernetes.io%2Fos%0Akubernetes.io%2Farch%0A%0APodAntiAffinity%E9%85%8D%E7%BD%AE%EF%BC%8C%E4%BF%9D%E8%AF%81%E8%AF%A5%E8%AF%A5%E6%9C%8D%E5%8A%A1%E7%9A%84%E4%B8%A4%E4%B8%AA%E5%AE%9E%E4%BE%8B%E4%B8%8D%E4%BC%9A%E5%85%B1%E5%AD%98%E5%9C%A8%E4%B8%80%E4%B8%AA%E8%8A%82%E7%82%B9%E4%B8%8A%E3%80%82%0A%60%60%60%0AapiVersion%3A%20apps%2Fv1%0Akind%3A%20Deployment%0Ametadata%3A%0A%20%20name%3A%20redis-cache%0Aspec%3A%0A%20%20selector%3A%0A%20%20%20%20matchLabels%3A%0A%20%20%20%20%20%20app%3A%20store%0A%20%20replicas%3A%203%0A%20%20template%3A%0A%20%20%20%20metadata%3A%0A%20%20%20%20%20%20labels%3A%0A%20%20%20%20%20%20%20%20app%3A%20store%0A%20%20%20%20spec%3A%0A%20%20%20%20%20%20affinity%3A%0A%20%20%20%20%20%20%20%20podAntiAffinity%3A%0A%20%20%20%20%20%20%20%20%20%20requiredDuringSchedulingIgnoredDuringExecution%3A%0A%20%20%20%20%20%20%20%20%20%20-%20labelSelector%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20matchExpressions%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20-%20key%3A%20app%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20operator%3A%20In%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20values%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20-%20store%0A%20%20%20%20%20%20%20%20%20%20%20%20topologyKey%3A%20%22kubernetes.io%2Fhostname%22%0A%20%20%20%20%20%20containers%3A%0A%20%20%20%20%20%20-%20name%3A%20redis-server%0A%20%20%20%20%20%20%20%20image%3A%20redis%3A3.2-alpine%0A%60%60%60%0A%0A%E4%BF%9D%E8%AF%81web-server%E7%9A%84%E6%AF%8F%E4%B8%AA%E5%AE%9E%E4%BE%8B%E4%B8%8D%E4%BC%9A%E5%85%B1%E5%AD%98%E5%9C%A8%E5%90%8C%E4%B8%80%E4%B8%AA%E8%8A%82%E7%82%B9%EF%BC%8C%E5%B0%BD%E9%87%8F%E5%92%8C%E6%A0%87%E7%AD%BE%E4%B8%BAapp%3Dstore%E7%9A%84pod%E5%85%B1%E5%AD%98%E5%9C%A8%E5%90%8C%E4%B8%80%E5%8F%B0%E6%9C%BA%E5%99%A8%E4%B8%8A%0A%0A%60%60%60%0AapiVersion%3A%20apps%2Fv1%0Akind%3A%20Deployment%0Ametadata%3A%0A%20%20name%3A%20web-server%0Aspec%3A%0A%20%20selector%3A%0A%20%20%20%20matchLabels%3A%0A%20%20%20%20%20%20app%3A%20web-store%0A%20%20replicas%3A%203%0A%20%20template%3A%0A%20%20%20%20metadata%3A%0A%20%20%20%20%20%20labels%3A%0A%20%20%20%20%20%20%20%20app%3A%20web-store%0A%20%20%20%20spec%3A%0A%20%20%20%20%20%20affinity%3A%0A%20%20%20%20%20%20%20%20podAntiAffinity%3A%0A%20%20%20%20%20%20%20%20%20%20requiredDuringSchedulingIgnoredDuringExecution%3A%0A%20%20%20%20%20%20%20%20%20%20-%20labelSelector%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20matchExpressions%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20-%20key%3A%20app%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20operator%3A%20In%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20values%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20-%20web-store%0A%20%20%20%20%20%20%20%20%20%20%20%20topologyKey%3A%20%22kubernetes.io%2Fhostname%22%0A%20%20%20%20%20%20%20%20podAffinity%3A%0A%20%20%20%20%20%20%20%20%20%20requiredDuringSchedulingIgnoredDuringExecution%3A%0A%20%20%20%20%20%20%20%20%20%20-%20labelSelector%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20matchExpressions%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20-%20key%3A%20app%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20operator%3A%20In%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20values%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20-%20store%0A%20%20%20%20%20%20%20%20%20%20%20%20topologyKey%3A%20%22kubernetes.io%2Fhostname%22%0A%20%20%20%20%20%20containers%3A%0A%20%20%20%20%20%20-%20name%3A%20web-app%0A%20%20%20%20%20%20%20%20image%3A%20nginx%3A1.12-alpine%0A%60%60%60%0A%0A%23%23%23%23%20%E5%90%AF%E5%8A%A8%E7%AD%96%E7%95%A5Always%2FOnFailure%2FNever%0A-%20Daemonset%E9%87%8C%E7%9A%84pod%20Template%E4%B8%8B%E5%BF%85%E9%A1%BB%E6%9C%89RestartPolicy%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%B2%A1%E6%8C%87%E5%AE%9A%EF%BC%8C%E4%BC%9A%E9%BB%98%E8%AE%A4%E4%B8%BAAlways%0A-%20%E5%8F%A6%E5%A4%96Deployment%E3%80%81Statefulset%E7%9A%84restartPolicy%E4%B9%9F%E5%BF%85%E9%A1%BB%E4%B8%BAAlways%EF%BC%8C%E4%BF%9D%E8%AF%81pod%E5%BC%82%E5%B8%B8%E9%80%80%E5%87%BA%EF%BC%8C%E6%88%96%E8%80%85%E5%81%A5%E5%BA%B7%E6%A3%80%E6%9F%A5%20livenessProbe%E5%A4%B1%E8%B4%A5%E5%90%8E%E7%94%B1kubelet%E9%87%8D%E5%90%AF%E5%AE%B9%E5%99%A8%E3%80%82https%3A%2F%2Fkubernetes.io%2Fzh%2Fdocs%2Fconcepts%2Fworkloads%2Fcontrollers%2Fdeployment%0A-%20Job%E5%92%8CCronJob%E6%98%AF%E8%BF%90%E8%A1%8C%E4%B8%80%E6%AC%A1%E7%9A%84pod%EF%BC%8CrestartPolicy%E5%8F%AA%E8%83%BD%E4%B8%BAOnFailure%E6%88%96Never%EF%BC%8C%E7%A1%AE%E4%BF%9D%E5%AE%B9%E5%99%A8%E6%89%A7%E8%A1%8C%E5%AE%8C%E6%88%90%E5%90%8E%E4%B8%8D%E5%86%8D%E9%87%8D%E5%90%AF%E3%80%82https%3A%2F%2Fkubernetes.io%2Fdocs%2Fconcepts%2Fworkloads%2Fcontrollers%2Fjobs-run-to-completion%2F%0A%0A%23%23%23%20%E5%8F%82%E8%80%83%E6%96%87%E7%8C%AE%0A%5BPod%E8%B0%83%E5%BA%A6%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%5D(https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FUcpP4koV1tTRxfM7Vmi0Ug)%0A%0A
