技术002KNode

技术002KNode
============

-  `技术002KNode <>`__

   -  `影响k8s集群节点数规模的因素 <>`__
   -  `集群支持的节点数目 <>`__

      -  `v1.16 - 5000 <>`__

   -  `集群的Node选择大资源配置机器还是小资源配置机器 <>`__

      -  `大资源节点 <>`__
      -  `小资源节点 <>`__

   -  `集群到底用大资源节点还是小资源节点-总结 <>`__
   -  `参考文献 <>`__

影响k8s集群节点数规模的因素
---------------------------

-  etcd

如果我们继续使用 etcd v2 的话，是无法负载 5000 节点的集群规模的。etcd v2
中的 watch 实现是一个主要障碍。在 5000
节点规模的集群中，我们每秒需要向同一个 watcher 发送至少 500 个 watch
事件，这在 v2 中是不可能的。

集群支持的节点数目
------------------

v1.16 - 5000
~~~~~~~~~~~~

At v1.16, Kubernetes supports clusters with up to 5000 nodes. More
specifically, we support configurations that meet all of the following
criteria:

前提条件：

-  No more than 5000 nodes
-  No more than 150000(15w) total pods
-  No more than 300000(30w) total containers
-  No more than 100 pods per node

集群的Node选择大资源配置机器还是小资源配置机器
----------------------------------------------

大资源节点
~~~~~~~~~~

-  优点

1 Less management overhead 降低管理成本 2 Lower costs per node
针对于裸机，每个节点更少的费用，因为资源和节点数目不是线性增加的
针对于公有云，使用大机器不会节省费用 3.Allows running resource-hungry
applications

-  缺点

1 Large number of pods per node 2 Limited replication 3 Higher blast
radius 4 Large scaling increments

小资源节点
~~~~~~~~~~

-  优点

1 Reduced blast radius 机器的资源越小，宕机的话影响的范围越小 2 Allows
high replication 将副本数打的更散一点儿，服务保障行更好

-  缺点

1 Large number of nodes But large numbers of nodes can be a challenge
for the Kubernetes control plane. 1)节点之间互相通信以节点的平方增加
2）controller manager组件挨个遍历每个节点的健康情况，增加了该组件的负担
3）etcd的负担，kubelet通过api-server watch object的变化
官方支持5009节点，实际上500个节点就有点问题了
更多的node节点就需要配置更好的master。

As you can see, for 500 worker nodes, the used master nodes have 32 and
36 CPU cores and 120 GB and 60 GB of memory, respectively.

\**So, if you intend to use a large number of small nodes, there are two
things you need to keep in mind

1.The more worker nodes you have, the more performant master nodes you
need

2.If you plan to use more than 500 nodes, you can expect to hit some
performance bottlenecks that require some effort to solve*\*

2 More system overhead

For example, imagine that all system daemons of a single node together
use 0.1 CPU cores and 0.1 GB of memory.

If you have a single node of 10 CPU cores and 10 GB of memory, then the
daemons consume 1% of your cluster’s capacity.

如果物理机10c，10G，那么daemons守护进程将会占用1/100的资源，那么1/100的资源账单将会用来运行系统进程。

On the other hand, if you have 10 nodes of 1 CPU core and 1 GB of
memory, then the daemons consume 10% of your cluster’s capacity.

Thus, in the second case, 10% of your bill is for running the system,
whereas in the first case, it’s only 1%.

3 Lower resource utilisation 4 Pod limits on small nodes

集群到底用大资源节点还是小资源节点-总结
---------------------------------------

-  根据你的应用来调整你的集群
-  集群的节点的配置多样化

参考文献
--------

`not-one-size-fits-all-how-to-size-kubernetes-clusters.pdf <https://static.sched.com/hosted_files/kccnceu18/4e/kubecon2018-not-one-size-fits-all-how-to-size-kubernetes-clusters.pdf>`__

`Building large
clusters <https://kubernetes.io/docs/setup/best-practices/cluster-large/>`__

Architecting Kubernetes clusters — choosing a worker node size

%23%20%E6%8A%80%E6%9C%AF002KNode%0A%5BTOC%5D%0A%0A%23%23%23%20%E5%BD%B1%E5%93%8Dk8s%E9%9B%86%E7%BE%A4%E8%8A%82%E7%82%B9%E6%95%B0%E8%A7%84%E6%A8%A1%E7%9A%84%E5%9B%A0%E7%B4%A0%0A-%20etcd%0A%E5%A6%82%E6%9E%9C%E6%88%91%E4%BB%AC%E7%BB%A7%E7%BB%AD%E4%BD%BF%E7%94%A8%20etcd%20v2%20%E7%9A%84%E8%AF%9D%EF%BC%8C%E6%98%AF%E6%97%A0%E6%B3%95%E8%B4%9F%E8%BD%BD%205000%20%E8%8A%82%E7%82%B9%E7%9A%84%E9%9B%86%E7%BE%A4%E8%A7%84%E6%A8%A1%E7%9A%84%E3%80%82etcd%20v2%20%E4%B8%AD%E7%9A%84%20watch%20%E5%AE%9E%E7%8E%B0%E6%98%AF%E4%B8%80%E4%B8%AA%E4%B8%BB%E8%A6%81%E9%9A%9C%E7%A2%8D%E3%80%82%E5%9C%A8%205000%20%E8%8A%82%E7%82%B9%E8%A7%84%E6%A8%A1%E7%9A%84%E9%9B%86%E7%BE%A4%E4%B8%AD%EF%BC%8C%E6%88%91%E4%BB%AC%E6%AF%8F%E7%A7%92%E9%9C%80%E8%A6%81%E5%90%91%E5%90%8C%E4%B8%80%E4%B8%AA%20watcher%20%E5%8F%91%E9%80%81%E8%87%B3%E5%B0%91%20500%20%E4%B8%AA%20watch%20%E4%BA%8B%E4%BB%B6%EF%BC%8C%E8%BF%99%E5%9C%A8%20v2%20%E4%B8%AD%E6%98%AF%E4%B8%8D%E5%8F%AF%E8%83%BD%E7%9A%84%E3%80%82%0A%0A%0A%23%23%23%20%E9%9B%86%E7%BE%A4%E6%94%AF%E6%8C%81%E7%9A%84%E8%8A%82%E7%82%B9%E6%95%B0%E7%9B%AE%0A%0A%23%23%23%23%20%20v1.16%20-%205000%0AAt%20v1.16%2C%20Kubernetes%20supports%20clusters%20with%20up%20to%205000%20nodes.%20More%20specifically%2C%20we%20support%20configurations%20that%20meet%20all%20of%20the%20following%20criteria%3A%0A%E5%89%8D%E6%8F%90%E6%9D%A1%E4%BB%B6%EF%BC%9A%0A-%20No%20more%20than%205000%20nodes%0A-%20No%20more%20than%20150000(15w)%20total%20pods%0A-%20No%20more%20than%20300000(30w)%20total%20containers%0A-%20No%20more%20than%20100%20pods%20per%20node%0A%0A%0A%0A%23%23%23%20%E9%9B%86%E7%BE%A4%E7%9A%84Node%E9%80%89%E6%8B%A9%E5%A4%A7%E8%B5%84%E6%BA%90%E9%85%8D%E7%BD%AE%E6%9C%BA%E5%99%A8%E8%BF%98%E6%98%AF%E5%B0%8F%E8%B5%84%E6%BA%90%E9%85%8D%E7%BD%AE%E6%9C%BA%E5%99%A8%0A%23%23%23%23%20%E5%A4%A7%E8%B5%84%E6%BA%90%E8%8A%82%E7%82%B9%0A-%20%E4%BC%98%E7%82%B9%0A1%20Less%20management%20overhead%0A%E9%99%8D%E4%BD%8E%E7%AE%A1%E7%90%86%E6%88%90%E6%9C%AC%0A2%20Lower%20costs%20per%20node%0A%E9%92%88%E5%AF%B9%E4%BA%8E%E8%A3%B8%E6%9C%BA%EF%BC%8C%E6%AF%8F%E4%B8%AA%E8%8A%82%E7%82%B9%E6%9B%B4%E5%B0%91%E7%9A%84%E8%B4%B9%E7%94%A8%EF%BC%8C%E5%9B%A0%E4%B8%BA%E8%B5%84%E6%BA%90%E5%92%8C%E8%8A%82%E7%82%B9%E6%95%B0%E7%9B%AE%E4%B8%8D%E6%98%AF%E7%BA%BF%E6%80%A7%E5%A2%9E%E5%8A%A0%E7%9A%84%0A%E9%92%88%E5%AF%B9%E4%BA%8E%E5%85%AC%E6%9C%89%E4%BA%91%EF%BC%8C%E4%BD%BF%E7%94%A8%E5%A4%A7%E6%9C%BA%E5%99%A8%E4%B8%8D%E4%BC%9A%E8%8A%82%E7%9C%81%E8%B4%B9%E7%94%A8%0A3.Allows%20running%20resource-hungry%20applications%0A-%20%E7%BC%BA%E7%82%B9%0A1%20Large%20number%20of%20pods%20per%20node%0A2%20Limited%20replication%0A3%20Higher%20blast%20radius%0A4%20Large%20scaling%20increments%0A%0A%0A%23%23%23%23%20%E5%B0%8F%E8%B5%84%E6%BA%90%E8%8A%82%E7%82%B9%0A-%20%E4%BC%98%E7%82%B9%0A1%20Reduced%20blast%20radius%20%0A%E6%9C%BA%E5%99%A8%E7%9A%84%E8%B5%84%E6%BA%90%E8%B6%8A%E5%B0%8F%EF%BC%8C%E5%AE%95%E6%9C%BA%E7%9A%84%E8%AF%9D%E5%BD%B1%E5%93%8D%E7%9A%84%E8%8C%83%E5%9B%B4%E8%B6%8A%E5%B0%8F%0A2%20Allows%20high%20replication%0A%E5%B0%86%E5%89%AF%E6%9C%AC%E6%95%B0%E6%89%93%E7%9A%84%E6%9B%B4%E6%95%A3%E4%B8%80%E7%82%B9%E5%84%BF%EF%BC%8C%E6%9C%8D%E5%8A%A1%E4%BF%9D%E9%9A%9C%E8%A1%8C%E6%9B%B4%E5%A5%BD%0A-%20%E7%BC%BA%E7%82%B9%0A1%20Large%20number%20of%20nodes%0A%0ABut%20large%20numbers%20of%20nodes%20can%20be%20a%20challenge%20for%20the%20Kubernetes%20control%20plane.%0A1)%E8%8A%82%E7%82%B9%E4%B9%8B%E9%97%B4%E4%BA%92%E7%9B%B8%E9%80%9A%E4%BF%A1%E4%BB%A5%E8%8A%82%E7%82%B9%E7%9A%84%E5%B9%B3%E6%96%B9%E5%A2%9E%E5%8A%A0%0A2%EF%BC%89controller%20manager%E7%BB%84%E4%BB%B6%E6%8C%A8%E4%B8%AA%E9%81%8D%E5%8E%86%E6%AF%8F%E4%B8%AA%E8%8A%82%E7%82%B9%E7%9A%84%E5%81%A5%E5%BA%B7%E6%83%85%E5%86%B5%EF%BC%8C%E5%A2%9E%E5%8A%A0%E4%BA%86%E8%AF%A5%E7%BB%84%E4%BB%B6%E7%9A%84%E8%B4%9F%E6%8B%85%0A3%EF%BC%89etcd%E7%9A%84%E8%B4%9F%E6%8B%85%EF%BC%8Ckubelet%E9%80%9A%E8%BF%87api-server%20watch%20object%E7%9A%84%E5%8F%98%E5%8C%96%0A%0A%E5%AE%98%E6%96%B9%E6%94%AF%E6%8C%815009%E8%8A%82%E7%82%B9%EF%BC%8C%E5%AE%9E%E9%99%85%E4%B8%8A500%E4%B8%AA%E8%8A%82%E7%82%B9%E5%B0%B1%E6%9C%89%E7%82%B9%E9%97%AE%E9%A2%98%E4%BA%86%0A%E6%9B%B4%E5%A4%9A%E7%9A%84node%E8%8A%82%E7%82%B9%E5%B0%B1%E9%9C%80%E8%A6%81%E9%85%8D%E7%BD%AE%E6%9B%B4%E5%A5%BD%E7%9A%84master%E3%80%82%0AAs%20you%20can%20see%2C%20for%20500%20worker%20nodes%2C%20the%20used%20master%20nodes%20have%2032%20and%2036%20CPU%20cores%20and%20120%20GB%20and%2060%20GB%20of%20memory%2C%20respectively.%0A%0A%0A\ **So%2C%20if%20you%20intend%20to%20use%20a%20large%20number%20of%20small%20nodes%2C%20there%20are%20two%20things%20you%20need%20to%20keep%20in%20mind%0A1.The%20more%20worker%20nodes%20you%20have%2C%20the%20more%20performant%20master%20nodes%20you%20need%0A2.If%20you%20plan%20to%20use%20more%20than%20500%20nodes%2C%20you%20can%20expect%20to%20hit%20some%20performance%20bottlenecks%20that%20require%20some%20effort%20to%20solve**\ %0A%0A2%20More%20system%20overhead%0A%0AFor%20example%2C%20imagine%20that%20all%20system%20daemons%20of%20a%20single%20node%20together%20use%200.1%20CPU%20cores%20and%200.1%20GB%20of%20memory.%0A%0AIf%20you%20have%20a%20single%20node%20of%2010%20CPU%20cores%20and%2010%20GB%20of%20memory%2C%20then%20the%20daemons%20consume%201%25%20of%20your%20cluster’s%20capacity.%0A%E5%A6%82%E6%9E%9C%E7%89%A9%E7%90%86%E6%9C%BA10c%EF%BC%8C10G%EF%BC%8C%E9%82%A3%E4%B9%88daemons%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B%E5%B0%86%E4%BC%9A%E5%8D%A0%E7%94%A81%2F100%E7%9A%84%E8%B5%84%E6%BA%90%EF%BC%8C%E9%82%A3%E4%B9%881%2F100%E7%9A%84%E8%B5%84%E6%BA%90%E8%B4%A6%E5%8D%95%E5%B0%86%E4%BC%9A%E7%94%A8%E6%9D%A5%E8%BF%90%E8%A1%8C%E7%B3%BB%E7%BB%9F%E8%BF%9B%E7%A8%8B%E3%80%82%0A%0AOn%20the%20other%20hand%2C%20if%20you%20have%2010%20nodes%20of%201%20CPU%20core%20and%201%20GB%20of%20memory%2C%20then%20the%20daemons%20consume%2010%25%20of%20your%20cluster’s%20capacity.%0A%0AThus%2C%20in%20the%20second%20case%2C%2010%25%20of%20your%20bill%20is%20for%20running%20the%20system%2C%20whereas%20in%20the%20first%20case%2C%20it’s%20only%201%25.%0A%0A3%20Lower%20resource%20utilisation%0A%0A4%20Pod%20limits%20on%20small%20nodes%0A%0A%23%23%23%20%E9%9B%86%E7%BE%A4%E5%88%B0%E5%BA%95%E7%94%A8%E5%A4%A7%E8%B5%84%E6%BA%90%E8%8A%82%E7%82%B9%E8%BF%98%E6%98%AF%E5%B0%8F%E8%B5%84%E6%BA%90%E8%8A%82%E7%82%B9-%E6%80%BB%E7%BB%93%0A-%20%E6%A0%B9%E6%8D%AE%E4%BD%A0%E7%9A%84%E5%BA%94%E7%94%A8%E6%9D%A5%E8%B0%83%E6%95%B4%E4%BD%A0%E7%9A%84%E9%9B%86%E7%BE%A4%0A-%20%E9%9B%86%E7%BE%A4%E7%9A%84%E8%8A%82%E7%82%B9%E7%9A%84%E9%85%8D%E7%BD%AE%E5%A4%9A%E6%A0%B7%E5%8C%96%0A%0A%23%23%20%E5%8F%82%E8%80%83%E6%96%87%E7%8C%AE%0A%5Bnot-one-size-fits-all-how-to-size-kubernetes-clusters.pdf%5D(https%3A%2F%2Fstatic.sched.com%2Fhosted_files%2Fkccnceu18%2F4e%2Fkubecon2018-not-one-size-fits-all-how-to-size-kubernetes-clusters.pdf)%0A%5BBuilding%20large%20clusters%0A%5D(https%3A%2F%2Fkubernetes.io%2Fdocs%2Fsetup%2Fbest-practices%2Fcluster-large%2F)%0AArchitecting%20Kubernetes%20clusters%20%E2%80%94%20choosing%20a%20worker%20node%20size
