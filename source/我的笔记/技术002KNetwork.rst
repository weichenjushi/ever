技术002KNetwork

技术002KNetwork
===============

-  `技术002KNetwork <>`__

   -  `基础知识网络 <>`__
   -  `流量控制 <>`__

      -  `传统的流量控制是如何实现的？实现的原理的是什么？ <>`__
      -  `k8s中的流量控制 <>`__

   -  `Kube-Proxy <>`__

      -  `ipvs <>`__

   -  `网络常用工具 <>`__

      -  `tcpdump <>`__
      -  `iperf <>`__
      -  `iproute <>`__
      -  `iptables <>`__
      -  `route <>`__

   -  `网络方案 <>`__
   -  `参考文档 <>`__

基础知识网络
------------

`命名空间 <https://mp.weixin.qq.com/s/lxIy8PqVckFS_npD2vvBYwhttps://docker-k8s-lab.readthedocs.io/en/latest/docker/netns.html>`__

`TCP/IP/ARP/ICMP协议 <https://mp.weixin.qq.com/s/-70w949-R87RSli_j_981A>`__

-  MB/S与MBIT/S概念

Mbit/s意思是 兆比特/秒，俗称:小b,
是指每秒传输的比特位数，即家里使用的10M或50M宽带或者speedtest测速结果再或者Cacti监控看到的带宽峰值就是这个小b的概念。

MB/s意思是
兆字节/秒,俗称:大B，是指每秒传输的字节数量，也是实际下载文件看到的网络速度。
8Mbit/s(运营商网络带宽)=1MB/s(实际文件下载速度)

流量控制
--------

传统的流量控制是如何实现的？实现的原理的是什么？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`Linux服务器
使用tc命令对网卡流量上下行限速 <https://blog.csdn.net/jack170601/article/details/78840403>`__

`iproute(网卡限速)–使用方法 <https://blog.51cto.com/liuzhengwei521/2083704>`__
`Docker网络隔离初步设想 <https://toutiao.io/posts/gvxrgc/preview>`__
`Tc流量控制 <https://www.jianshu.com/p/4b5cc3845f2c>`__

k8s中的流量控制
~~~~~~~~~~~~~~~

-  network
   policy用来控制pod和pod中的流量的流向，分为同一个namespace和不同的namespace

https://console.bluemix.net/docs/containers/cs_network_policy.html#isolate_services

IBM公有云用的是Calico

-  Calico网络控制方案pre-DNAT

https://console.bluemix.net/docs/containers/cs_network_policy.html#block_ingress

-  方案1:`实现K8S中Pod带宽限制
   Calico/Flanne通过cni的bandwitch来限制 <https://zhuanlan.zhihu.com/p/54988169>`__
-  方案2:`阿里云的插件容器服务里限制应用的带宽 <https://yq.aliyun.com/articles/388097>`__

Kube-Proxy
----------

kube-proxy 是 Kubernetes 部署的关键组件。它的作用是监听 API server 中
service 和 endpoint
的变化情况，并为服务配置负载均衡。它可以以三种模式运行：userspace、iptables
和 IPVS。在这篇文章中，作者将对 iptables 和 IPVS
进行比较，衡量它们在真实微服务环境中的表现，并总结最佳选择策略。https://www.projectcalico.org/comparing-kube-proxy-modes-iptables-or-ipvs/?utm_sq=g4rzfsszs8##

ipvs
~~~~

-  ipvs转发模式 三种转发模式性能从高到低：DR > NAT >FULLNATipvs
-  负载均衡器常用调度算法

● 轮询（Round Robin） ● 加权轮询（Weighted Round Robin） ● 最少连接调度
● 加权最少连接调度

网络常用工具
------------

tcpdump
~~~~~~~

iperf
~~~~~

iproute
~~~~~~~

iptables
~~~~~~~~

route
~~~~~

常用命令：添加路由ip route add 172.16.0.0/12 via 10.136.44.254 dev
eth0https://blog.csdn.net/yuanchao99/article/details/18992567以机器10.103.17.235(默认路由是公网地址)和10.120.4.16(默认路由是内外地址)为例来判断路由的问题：[wangluhui@103-17-235-sh-100-k10
~]$ route -nKernel IP routing
tableDestination     Gateway         Genmask         Flags Metric
Ref    Use
Iface0.0.0.0         220.181.86.126  0.0.0.0         UG    0      0        0
eth110.0.0.0        10.103.23.254   255.0.0.0       UG    0      0        0
eth010.103.16.0     0.0.0.0         255.255.248.0   U     0      0        0
eth0124.243.223.0   0.0.0.0         255.255.255.0   U     0      0        0
eth4169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0
eth0172.16.0.0      10.103.23.254   255.240.0.0     UG    0      0        0
eth0172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0
docker0220.181.86.0    0.0.0.0         255.255.255.128
U     0      0        0
eth1路由匹配该路由表，如果都匹配不到，那么会取第一条数据：0.0.0.0         220.181.86.126  0.0.0.0         UG    0      0        0
eth1，可以看到第一条数据的网关是个外网地址，所以，需要在文件/etc/sysconfig/network-scripts/route-eth0中添加记录172.16.0.0/12
via
10.103.23.254，其中网关10.103.23.254是物理机所在的网关10.120.4.16机器上[wangluhui@120-4-16-SH-1037-B10
~]$ ip routedefault via 10.120.4.254 dev eth010.0.0.0/8 via 10.120.4.254
dev eth010.120.4.0/24 dev eth0  proto kernel  scope link  src
10.120.4.16169.254.0.0/16 dev eth0  scope link  metric 1002

查看网卡是千兆还是万兆ethtool eth0 scp限制速度scp -l
2000这里2000是2M的意思 Route:route -n 查看路由表\ |image0|

根据路由信息，ping
192.168.190.203，会匹配到第一条。第一条路由的意思是：去往任何网段的数据包都发往网管169.254.1.1，然后从eth0网卡发送出去。

路由表中Flags标志的含义：U up表示当前为启动状态H
host表示该路由为一个主机，多为达到数据包的路由G Gateway
表示该路由是一个网关，如果没有说明目的地是直连的D Dynamicaly
表示该路由是重定向报文修改M 表示该路由已被重定向报文修改

网络方案
--------

`k8s
cni <https://thenewstack.io/hackers-guide-kubernetes-networking/>`__

Kubelet invokes the CNI plug-in with environment variables containing
command parameters (CNI_ARGS, CNI_COMMAND, CNI_IFNAME, CNI_NETNS,
CNI_CONTAINERID, CNI_PATH) and streams the json.conf file through stdin.
The plug-in responds with json output text, describing the results and
status. See more detailed explanation and examples here. It’s relatively
simple to develop your own CNI plug-in if you know the Go programming
language, as the framework does much of the magic and you can use or
extend one of the existing plug-ins here.Kubelet will pass the POD name
and namespace as part of the CNI_ARGS variable (for
example  “K8S_POD_NAMESPACE=default;K8S_POD_NAME=mytests-1227152546-vq7kw;”
). We can use this to customize the network configuration per POD or POD
namespace (e.g. put every namespace in a different subnet). Future
Kubernetes versions will treat networks as equal citizens and include
network configuration as part of the POD or namespace spec just like
memory, CPUs and volumes. For the time being, we can use annotations to
store configuration or record POD networking data/state.

参考文档
--------

1.破案：Kubernetes/Docker 上无法解释的连接超时.网络问题排查示例

2.\ `网络篇 Kubernetes
网络故障常见排查方法 <https://mp.weixin.qq.com/s/TrQBrbGZnB4Hus55UZUgxg>`__

3.\ `k8s网络方案比较 <https://itnext.io/benchmark-results-of-kubernetes-network-plugins-cni-over-10gbit-s-network-36475925a560>`__
安装维护/安全/性能/资源消耗

%23%20%E6%8A%80%E6%9C%AF002KNetwork%0A%5BTOC%5D%0A%0A%23%23%23%20%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E7%BD%91%E7%BB%9C%0A%5B%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4%5D(https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FlxIy8PqVckFS_npD2vvBYwhttps%3A%2F%2Fdocker-k8s-lab.readthedocs.io%2Fen%2Flatest%2Fdocker%2Fnetns.html)%0A%5BTCP%2FIP%2FARP%2FICMP%E5%8D%8F%E8%AE%AE%5D(https%3A%2F%2Fmp.weixin.qq.com%2Fs%2F-70w949-R87RSli_j_981A)%0A%0A-%20MB%2FS%E4%B8%8EMBIT%2FS%E6%A6%82%E5%BF%B5%0AMbit%2Fs%E6%84%8F%E6%80%9D%E6%98%AF%20%E5%85%86%E6%AF%94%E7%89%B9%2F%E7%A7%92%EF%BC%8C%E4%BF%97%E7%A7%B0%3A%E5%B0%8Fb%2C%20%E6%98%AF%E6%8C%87%E6%AF%8F%E7%A7%92%E4%BC%A0%E8%BE%93%E7%9A%84%E6%AF%94%E7%89%B9%E4%BD%8D%E6%95%B0%EF%BC%8C%E5%8D%B3%E5%AE%B6%E9%87%8C%E4%BD%BF%E7%94%A8%E7%9A%8410M%E6%88%9650M%E5%AE%BD%E5%B8%A6%E6%88%96%E8%80%85speedtest%E6%B5%8B%E9%80%9F%E7%BB%93%E6%9E%9C%E5%86%8D%E6%88%96%E8%80%85Cacti%E7%9B%91%E6%8E%A7%E7%9C%8B%E5%88%B0%E7%9A%84%E5%B8%A6%E5%AE%BD%E5%B3%B0%E5%80%BC%E5%B0%B1%E6%98%AF%E8%BF%99%E4%B8%AA%E5%B0%8Fb%E7%9A%84%E6%A6%82%E5%BF%B5%E3%80%82%0A%0AMB%2Fs%E6%84%8F%E6%80%9D%E6%98%AF%20%E5%85%86%E5%AD%97%E8%8A%82%2F%E7%A7%92%2C%E4%BF%97%E7%A7%B0%3A%E5%A4%A7B%EF%BC%8C%E6%98%AF%E6%8C%87%E6%AF%8F%E7%A7%92%E4%BC%A0%E8%BE%93%E7%9A%84%E5%AD%97%E8%8A%82%E6%95%B0%E9%87%8F%EF%BC%8C%E4%B9%9F%E6%98%AF%E5%AE%9E%E9%99%85%E4%B8%8B%E8%BD%BD%E6%96%87%E4%BB%B6%E7%9C%8B%E5%88%B0%E7%9A%84%E7%BD%91%E7%BB%9C%E9%80%9F%E5%BA%A6%E3%80%82%0A%0A8Mbit%2Fs(%E8%BF%90%E8%90%A5%E5%95%86%E7%BD%91%E7%BB%9C%E5%B8%A6%E5%AE%BD)%3D1MB%2Fs(%E5%AE%9E%E9%99%85%E6%96%87%E4%BB%B6%E4%B8%8B%E8%BD%BD%E9%80%9F%E5%BA%A6)%0A%0A%23%23%23%20%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6%0A%0A%23%23%23%23%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6%E6%98%AF%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E7%9A%84%EF%BC%9F%E5%AE%9E%E7%8E%B0%E7%9A%84%E5%8E%9F%E7%90%86%E7%9A%84%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F%0A%0A%5BLinux%E6%9C%8D%E5%8A%A1%E5%99%A8%20%E4%BD%BF%E7%94%A8tc%E5%91%BD%E4%BB%A4%E5%AF%B9%E7%BD%91%E5%8D%A1%E6%B5%81%E9%87%8F%E4%B8%8A%E4%B8%8B%E8%A1%8C%E9%99%90%E9%80%9F%5D(https%3A%2F%2Fblog.csdn.net%2Fjack170601%2Farticle%2Fdetails%2F78840403)%0A%5Biproute(%E7%BD%91%E5%8D%A1%E9%99%90%E9%80%9F)–%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%5D(https%3A%2F%2Fblog.51cto.com%2Fliuzhengwei521%2F2083704)%0A%5BDocker%E7%BD%91%E7%BB%9C%E9%9A%94%E7%A6%BB%E5%88%9D%E6%AD%A5%E8%AE%BE%E6%83%B3%5D(https%3A%2F%2Ftoutiao.io%2Fposts%2Fgvxrgc%2Fpreview)%0A%5BTc%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6%5D(https%3A%2F%2Fwww.jianshu.com%2Fp%2F4b5cc3845f2c)%0A%0A%23%23%23%23%20k8s%E4%B8%AD%E7%9A%84%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6%0A-%20network%20policy%E7%94%A8%E6%9D%A5%E6%8E%A7%E5%88%B6pod%E5%92%8Cpod%E4%B8%AD%E7%9A%84%E6%B5%81%E9%87%8F%E7%9A%84%E6%B5%81%E5%90%91%EF%BC%8C%E5%88%86%E4%B8%BA%E5%90%8C%E4%B8%80%E4%B8%AAnamespace%E5%92%8C%E4%B8%8D%E5%90%8C%E7%9A%84namespace%0A%0Ahttps%3A%2F%2Fconsole.bluemix.net%2Fdocs%2Fcontainers%2Fcs_network_policy.html%23isolate_services%0AIBM%E5%85%AC%E6%9C%89%E4%BA%91%E7%94%A8%E7%9A%84%E6%98%AFCalico%0A-%20Calico%E7%BD%91%E7%BB%9C%E6%8E%A7%E5%88%B6%E6%96%B9%E6%A1%88pre-DNAT%0A%0Ahttps%3A%2F%2Fconsole.bluemix.net%2Fdocs%2Fcontainers%2Fcs_network_policy.html%23block_ingress%0A%0A-%20%E6%96%B9%E6%A1%881%3A%5B%E5%AE%9E%E7%8E%B0K8S%E4%B8%ADPod%E5%B8%A6%E5%AE%BD%E9%99%90%E5%88%B6%20Calico%2FFlanne%E9%80%9A%E8%BF%87cni%E7%9A%84bandwitch%E6%9D%A5%E9%99%90%E5%88%B6%5D(https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F54988169)%0A-%20%E6%96%B9%E6%A1%882%3A%5B%E9%98%BF%E9%87%8C%E4%BA%91%E7%9A%84%E6%8F%92%E4%BB%B6%E5%AE%B9%E5%99%A8%E6%9C%8D%E5%8A%A1%E9%87%8C%E9%99%90%E5%88%B6%E5%BA%94%E7%94%A8%E7%9A%84%E5%B8%A6%E5%AE%BD%5D(https%3A%2F%2Fyq.aliyun.com%2Farticles%2F388097)%0A%0A%23%23%23%20Kube-Proxy%0Akube-proxy%20%E6%98%AF%20Kubernetes%20%E9%83%A8%E7%BD%B2%E7%9A%84%E5%85%B3%E9%94%AE%E7%BB%84%E4%BB%B6%E3%80%82%E5%AE%83%E7%9A%84%E4%BD%9C%E7%94%A8%E6%98%AF%E7%9B%91%E5%90%AC%20API%20server%20%E4%B8%AD%20service%20%E5%92%8C%20endpoint%20%E7%9A%84%E5%8F%98%E5%8C%96%E6%83%85%E5%86%B5%EF%BC%8C%E5%B9%B6%E4%B8%BA%E6%9C%8D%E5%8A%A1%E9%85%8D%E7%BD%AE%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E3%80%82%E5%AE%83%E5%8F%AF%E4%BB%A5%E4%BB%A5%E4%B8%89%E7%A7%8D%E6%A8%A1%E5%BC%8F%E8%BF%90%E8%A1%8C%EF%BC%9Auserspace%E3%80%81iptables%20%E5%92%8C%20IPVS%E3%80%82%E5%9C%A8%E8%BF%99%E7%AF%87%E6%96%87%E7%AB%A0%E4%B8%AD%EF%BC%8C%E4%BD%9C%E8%80%85%E5%B0%86%E5%AF%B9%20iptables%20%E5%92%8C%20IPVS%20%E8%BF%9B%E8%A1%8C%E6%AF%94%E8%BE%83%EF%BC%8C%E8%A1%A1%E9%87%8F%E5%AE%83%E4%BB%AC%E5%9C%A8%E7%9C%9F%E5%AE%9E%E5%BE%AE%E6%9C%8D%E5%8A%A1%E7%8E%AF%E5%A2%83%E4%B8%AD%E7%9A%84%E8%A1%A8%E7%8E%B0%EF%BC%8C%E5%B9%B6%E6%80%BB%E7%BB%93%E6%9C%80%E4%BD%B3%E9%80%89%E6%8B%A9%E7%AD%96%E7%95%A5%E3%80%82https%3A%2F%2Fwww.projectcalico.org%2Fcomparing-kube-proxy-modes-iptables-or-ipvs%2F%3Futm_sq%3Dg4rzfsszs8%23%23%20%0A%23%23%23%23%20ipvs%0A-%20ipvs%E8%BD%AC%E5%8F%91%E6%A8%A1%E5%BC%8F%20%E4%B8%89%E7%A7%8D%E8%BD%AC%E5%8F%91%E6%A8%A1%E5%BC%8F%E6%80%A7%E8%83%BD%E4%BB%8E%E9%AB%98%E5%88%B0%E4%BD%8E%EF%BC%9ADR%20%26gt%3B%20NAT%20%26gt%3BFULLNATipvs%20%0A-%20%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E5%99%A8%E5%B8%B8%E7%94%A8%E8%B0%83%E5%BA%A6%E7%AE%97%E6%B3%95%20%0A%E2%97%8F%20%E8%BD%AE%E8%AF%A2%EF%BC%88Round%20Robin%EF%BC%89%0A%E2%97%8F%20%E5%8A%A0%E6%9D%83%E8%BD%AE%E8%AF%A2%EF%BC%88Weighted%20Round%20Robin%EF%BC%89%0A%E2%97%8F%20%E6%9C%80%E5%B0%91%E8%BF%9E%E6%8E%A5%E8%B0%83%E5%BA%A6%0A%E2%97%8F%20%E5%8A%A0%E6%9D%83%E6%9C%80%E5%B0%91%E8%BF%9E%E6%8E%A5%E8%B0%83%E5%BA%A6%0A%23%23%23%20%E7%BD%91%E7%BB%9C%E5%B8%B8%E7%94%A8%E5%B7%A5%E5%85%B7%0A%23%23%23%23%20tcpdump%0A%23%23%23%23%20iperf%0A%23%23%23%23%20iproute%0A%23%23%23%23%20iptables%0A%23%23%23%23%20route%0A%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4%EF%BC%9A%E6%B7%BB%E5%8A%A0%E8%B7%AF%E7%94%B1ip%20route%20add%20172.16.0.0%2F12%20via%2010.136.44.254%20dev%20eth0https%3A%2F%2Fblog.csdn.net%2Fyuanchao99%2Farticle%2Fdetails%2F18992567%E4%BB%A5%E6%9C%BA%E5%99%A810.103.17.235(%E9%BB%98%E8%AE%A4%E8%B7%AF%E7%94%B1%E6%98%AF%E5%85%AC%E7%BD%91%E5%9C%B0%E5%9D%80)%E5%92%8C10.120.4.16(%E9%BB%98%E8%AE%A4%E8%B7%AF%E7%94%B1%E6%98%AF%E5%86%85%E5%A4%96%E5%9C%B0%E5%9D%80)%E4%B8%BA%E4%BE%8B%E6%9D%A5%E5%88%A4%E6%96%AD%E8%B7%AF%E7%94%B1%E7%9A%84%E9%97%AE%E9%A2%98%EF%BC%9A%5Bwangluhui%40103-17-235-sh-100-k10%20\ :sub:`%5D%24%20route%20-nKernel%20IP%20routing%20tableDestination%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3BGateway%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3BGenmask%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3BFlags%20Metric%20Ref%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3BUse%20Iface0.0.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B220.181.86.126%26nbsp%3B%26nbsp%3B0.0.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3BUG%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%20eth110.0.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B10.103.23.254%26nbsp%3B%26nbsp%3B%26nbsp%3B255.0.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3BUG%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%20eth010.103.16.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0.0.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B255.255.248.0%26nbsp%3B%26nbsp%3B%26nbsp%3BU%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%20eth0124.243.223.0%26nbsp%3B%26nbsp%3B%26nbsp%3B0.0.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B255.255.255.0%26nbsp%3B%26nbsp%3B%26nbsp%3BU%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%20eth4169.254.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0.0.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B255.255.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3BU%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B1002%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%20eth0172.16.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B10.103.23.254%26nbsp%3B%26nbsp%3B%26nbsp%3B255.240.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3BUG%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%20eth0172.17.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0.0.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B255.255.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3BU%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%20docker0220.181.86.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0.0.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B255.255.255.128%20U%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%20eth1%E8%B7%AF%E7%94%B1%E5%8C%B9%E9%85%8D%E8%AF%A5%E8%B7%AF%E7%94%B1%E8%A1%A8%EF%BC%8C%E5%A6%82%E6%9E%9C%E9%83%BD%E5%8C%B9%E9%85%8D%E4%B8%8D%E5%88%B0%EF%BC%8C%E9%82%A3%E4%B9%88%E4%BC%9A%E5%8F%96%E7%AC%AC%E4%B8%80%E6%9D%A1%E6%95%B0%E6%8D%AE%EF%BC%9A0.0.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B220.181.86.126%26nbsp%3B%26nbsp%3B0.0.0.0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3BUG%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B%26nbsp%3B0%20eth1%EF%BC%8C%E5%8F%AF%E4%BB%A5%E7%9C%8B%E5%88%B0%E7%AC%AC%E4%B8%80%E6%9D%A1%E6%95%B0%E6%8D%AE%E7%9A%84%E7%BD%91%E5%85%B3%E6%98%AF%E4%B8%AA%E5%A4%96%E7%BD%91%E5%9C%B0%E5%9D%80%EF%BC%8C%E6%89%80%E4%BB%A5%EF%BC%8C%E9%9C%80%E8%A6%81%E5%9C%A8%E6%96%87%E4%BB%B6%2Fetc%2Fsysconfig%2Fnetwork-scripts%2Froute-eth0%E4%B8%AD%E6%B7%BB%E5%8A%A0%E8%AE%B0%E5%BD%95172.16.0.0%2F12%20via%2010.103.23.254%EF%BC%8C%E5%85%B6%E4%B8%AD%E7%BD%91%E5%85%B310.103.23.254%E6%98%AF%E7%89%A9%E7%90%86%E6%9C%BA%E6%89%80%E5%9C%A8%E7%9A%84%E7%BD%91%E5%85%B310.120.4.16%E6%9C%BA%E5%99%A8%E4%B8%8A%5Bwangluhui%40120-4-16-SH-1037-B10%20`\ %5D%24%20ip%20routedefault%20via%2010.120.4.254%20dev%20eth010.0.0.0%2F8%20via%2010.120.4.254%20dev%20eth010.120.4.0%2F24%20dev%20eth0%26nbsp%3B%26nbsp%3Bproto%20kernel%26nbsp%3B%26nbsp%3Bscope%20link%26nbsp%3B%26nbsp%3Bsrc%2010.120.4.16169.254.0.0%2F16%20dev%20eth0%26nbsp%3B%26nbsp%3Bscope%20link%26nbsp%3B%26nbsp%3Bmetric%201002%0A%0A%E6%9F%A5%E7%9C%8B%E7%BD%91%E5%8D%A1%E6%98%AF%E5%8D%83%E5%85%86%E8%BF%98%E6%98%AF%E4%B8%87%E5%85%86ethtool%20eth0%0A%0Ascp%E9%99%90%E5%88%B6%E9%80%9F%E5%BA%A6scp%20-l%202000%E8%BF%99%E9%87%8C2000%E6%98%AF2M%E7%9A%84%E6%84%8F%E6%80%9D%0A%0ARoute%3Aroute%20-n%20%E6%9F%A5%E7%9C%8B%E8%B7%AF%E7%94%B1%E8%A1%A8!%5B256b630d66f2542a3e6b29c398251401.png%5D(evernotecid%3A%2F%2F48E6E93E-A436-4E8F-9B4B-205CB9D86842%2Fappyinxiangcom%2F23555478%2FENResource%2Fp1103)%0A%E6%A0%B9%E6%8D%AE%E8%B7%AF%E7%94%B1%E4%BF%A1%E6%81%AF%EF%BC%8Cping%20192.168.190.203%EF%BC%8C%E4%BC%9A%E5%8C%B9%E9%85%8D%E5%88%B0%E7%AC%AC%E4%B8%80%E6%9D%A1%E3%80%82%E7%AC%AC%E4%B8%80%E6%9D%A1%E8%B7%AF%E7%94%B1%E7%9A%84%E6%84%8F%E6%80%9D%E6%98%AF%EF%BC%9A%E5%8E%BB%E5%BE%80%E4%BB%BB%E4%BD%95%E7%BD%91%E6%AE%B5%E7%9A%84%E6%95%B0%E6%8D%AE%E5%8C%85%E9%83%BD%E5%8F%91%E5%BE%80%E7%BD%91%E7%AE%A1169.254.1.1%EF%BC%8C%E7%84%B6%E5%90%8E%E4%BB%8Eeth0%E7%BD%91%E5%8D%A1%E5%8F%91%E9%80%81%E5%87%BA%E5%8E%BB%E3%80%82%0A%0A%E8%B7%AF%E7%94%B1%E8%A1%A8%E4%B8%ADFlags%E6%A0%87%E5%BF%97%E7%9A%84%E5%90%AB%E4%B9%89%EF%BC%9AU%20up%E8%A1%A8%E7%A4%BA%E5%BD%93%E5%89%8D%E4%B8%BA%E5%90%AF%E5%8A%A8%E7%8A%B6%E6%80%81H%20host%E8%A1%A8%E7%A4%BA%E8%AF%A5%E8%B7%AF%E7%94%B1%E4%B8%BA%E4%B8%80%E4%B8%AA%E4%B8%BB%E6%9C%BA%EF%BC%8C%E5%A4%9A%E4%B8%BA%E8%BE%BE%E5%88%B0%E6%95%B0%E6%8D%AE%E5%8C%85%E7%9A%84%E8%B7%AF%E7%94%B1G%20Gateway%20%E8%A1%A8%E7%A4%BA%E8%AF%A5%E8%B7%AF%E7%94%B1%E6%98%AF%E4%B8%80%E4%B8%AA%E7%BD%91%E5%85%B3%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%B2%A1%E6%9C%89%E8%AF%B4%E6%98%8E%E7%9B%AE%E7%9A%84%E5%9C%B0%E6%98%AF%E7%9B%B4%E8%BF%9E%E7%9A%84D%20Dynamicaly%20%E8%A1%A8%E7%A4%BA%E8%AF%A5%E8%B7%AF%E7%94%B1%E6%98%AF%E9%87%8D%E5%AE%9A%E5%90%91%E6%8A%A5%E6%96%87%E4%BF%AE%E6%94%B9M%20%E8%A1%A8%E7%A4%BA%E8%AF%A5%E8%B7%AF%E7%94%B1%E5%B7%B2%E8%A2%AB%E9%87%8D%E5%AE%9A%E5%90%91%E6%8A%A5%E6%96%87%E4%BF%AE%E6%94%B9%0A%23%23%23%20%E7%BD%91%E7%BB%9C%E6%96%B9%E6%A1%88%0A%5Bk8s%20cni%5D(https%3A%2F%2Fthenewstack.io%2Fhackers-guide-kubernetes-networking%2F)%0AKubelet%20invokes%20the%20CNI%20plug-in%20with%20environment%20variables%20containing%20command%20parameters%20(CNI_ARGS%2C%20CNI_COMMAND%2C%20CNI_IFNAME%2C%20CNI_NETNS%2C%20CNI_CONTAINERID%2C%20CNI_PATH)%20and%20streams%20the%20json.conf%20file%20through%20stdin.%20The%20plug-in%20responds%20with%20json%20output%20text%2C%20describing%20the%20results%20and%20status.%20See%20more%20detailed%20explanation%20and%20examples%20here.%20It%E2%80%99s%20relatively%20simple%20to%20develop%20your%20own%20CNI%20plug-in%20if%20you%20know%20the%20Go%20programming%20language%2C%20as%20the%20framework%20does%20much%20of%20the%20magic%20and%20you%20can%20use%20or%20extend%20one%20of%20the%20existing%20plug-ins%20here.Kubelet%20will%20pass%20the%20POD%20name%20and%20namespace%20as%20part%20of%20the%20CNI_ARGS%20variable%20(for%20example%26nbsp%3B%26nbsp%3B%E2%80%9CK8S_POD_NAMESPACE%3Ddefault%3BK8S_POD_NAME%3Dmytests-1227152546-vq7kw%3B%E2%80%9D%20).%20We%20can%20use%20this%20to%20customize%20the%20network%20configuration%20per%20POD%20or%20POD%20namespace%20(e.g.%20put%20every%20namespace%20in%20a%20different%20subnet).%20Future%20Kubernetes%20versions%20will%20treat%20networks%20as%20equal%20citizens%20and%20include%20network%20configuration%20as%20part%20of%20the%20POD%20or%20namespace%20spec%20just%20like%20memory%2C%20CPUs%20and%20volumes.%20For%20the%20time%20being%2C%20we%20can%20use%20annotations%20to%20store%20configuration%20or%20record%20POD%20networking%20data%2Fstate.%0A%0A%23%23%23%20%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%0A1.%E7%A0%B4%E6%A1%88%EF%BC%9AKubernetes%2FDocker%20%E4%B8%8A%E6%97%A0%E6%B3%95%E8%A7%A3%E9%87%8A%E7%9A%84%E8%BF%9E%E6%8E%A5%E8%B6%85%E6%97%B6.%E7%BD%91%E7%BB%9C%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5%E7%A4%BA%E4%BE%8B%0A2.%5B%E7%BD%91%E7%BB%9C%E7%AF%87%20Kubernetes%20%E7%BD%91%E7%BB%9C%E6%95%85%E9%9A%9C%E5%B8%B8%E8%A7%81%E6%8E%92%E6%9F%A5%E6%96%B9%E6%B3%95%5D(https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FTrQBrbGZnB4Hus55UZUgxg)%0A3.%5Bk8s%E7%BD%91%E7%BB%9C%E6%96%B9%E6%A1%88%E6%AF%94%E8%BE%83%5D(https%3A%2F%2Fitnext.io%2Fbenchmark-results-of-kubernetes-network-plugins-cni-over-10gbit-s-network-36475925a560)%20%E5%AE%89%E8%A3%85%E7%BB%B4%E6%8A%A4%2F%E5%AE%89%E5%85%A8%2F%E6%80%A7%E8%83%BD%2F%E8%B5%84%E6%BA%90%E6%B6%88%E8%80%97

.. |image0| image:: ../_resources/256b630d66f2542a3e6b29c398251401.png
