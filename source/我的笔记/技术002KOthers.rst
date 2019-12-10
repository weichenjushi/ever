技术002KOthers

技术002KOthers
==============

-  `技术002KOthers <>`__

   -  `Docker hub <>`__
   -  `gcr.io k8s.gcr.io quay.io镜像拉取 <>`__
   -  `CLI 瑞士军刀 <>`__
   -  `为什么k8s是系统级别的保障 or k8s的设计理念 <>`__

Docker hub
----------

mirrorgooglecontainers https://hub.docker.com/u/mirrorgooglecontainers/
anjia0532 https://hub.docker.com/u/anjia0532/

gcr.io k8s.gcr.io quay.io镜像拉取
---------------------------------

`镜像拉取 <https://www.ilanni.com/?p=14534>`__ 1. docker官方仓库 docker
pull xxx:yyy 中科大 docker pull
docker.mirrors.ustc.edu.cn/library/xxx:yyy azure docker pull
dockerhub.azk8s.cn/library/xxx:yyy 2. gcr.io镜像加速 docker pull
gcr.io/xxx/yyy:zzz 中科大 docker pull
gcr.mirrors.ustc.edu.cn/xxx/yyy:zzz azure docker pull docker pull
gcr.azk8s.cn/xxx/yyy:zzz 3. k8s.gcr.io镜像加速 docker pull
k8s.gcr.io/xxx:yyy <=> docker pull gcr.io/google-containers/xxx:yyy
azure docker pull gcr.azk8s.cn/google-containers/xxx:yyy (中科大 docker
pull gcr.mirrors.ustc.edu.cn/google-containers/xxx:yyy)
4.quay.io镜像加速 docker pull quay.io/xxx/yyy:zzz 中科大 docker pull
quay.mirrors.ustc.edu.cn/xxx/yyy:zzz Azure中国 docker pull
quay.azk8s.cn/xxx/yyy:zzz

CLI 瑞士军刀
------------

k9s kubectx kubens kube-ps1 popeye stern kubectl-bindrole帮助查找
Kubernetes 集群中指定 SA/Group/User 的权限等信息。

为什么k8s是系统级别的保障 or k8s的设计理念
------------------------------------------

https://hackernoon.com/level-triggering-and-reconciliation-in-kubernetes-1f17fe30333d

Kubernetes being self-healing

-  两个信号

k8s通过观察2个信号，一个是desired state(理想)状态一个是actual
state(实际状态)

-  安全

与传统的YARN相比较，安全性更高

-  集群规模

到 1.9 版本就已可以稳定支持 5000 个节点、15 万个 Pod 和 30
万个容器的规模，覆盖绝大部分用户场景，从此彻底消除业界对 Kubernetes
规模的顾虑。

-  自愈/自我调度/自动更新、回滚/水平扩展和负载均衡

%23%20%E6%8A%80%E6%9C%AF002KOthers%0A%5BTOC%5D%0A%0A%23%23%23%20Docker%20hub%0Amirrorgooglecontainers%20https%3A%2F%2Fhub.docker.com%2Fu%2Fmirrorgooglecontainers%2F%0Aanjia0532%20https%3A%2F%2Fhub.docker.com%2Fu%2Fanjia0532%2F%0A%0A%23%23%23%20gcr.io%20k8s.gcr.io%20quay.io%E9%95%9C%E5%83%8F%E6%8B%89%E5%8F%96%0A%5B%E9%95%9C%E5%83%8F%E6%8B%89%E5%8F%96%5D(https%3A%2F%2Fwww.ilanni.com%2F%3Fp%3D14534)%0A%0A1.%20docker%E5%AE%98%E6%96%B9%E4%BB%93%E5%BA%93%0Adocker%20pull%20xxx%3Ayyy%0A%E4%B8%AD%E7%A7%91%E5%A4%A7%20docker%20pull%20docker.mirrors.ustc.edu.cn%2Flibrary%2Fxxx%3Ayyy%0Aazure%20docker%20pull%20dockerhub.azk8s.cn%2Flibrary%2Fxxx%3Ayyy%0A2.%20gcr.io%E9%95%9C%E5%83%8F%E5%8A%A0%E9%80%9F%0Adocker%20pull%20gcr.io%2Fxxx%2Fyyy%3Azzz%0A%E4%B8%AD%E7%A7%91%E5%A4%A7%20%20docker%20pull%20gcr.mirrors.ustc.edu.cn%2Fxxx%2Fyyy%3Azzz%0Aazure%20docker%20pull%20docker%20pull%20gcr.azk8s.cn%2Fxxx%2Fyyy%3Azzz%0A3.%20k8s.gcr.io%E9%95%9C%E5%83%8F%E5%8A%A0%E9%80%9F%0Adocker%20pull%20k8s.gcr.io%2Fxxx%3Ayyy%20%3C%3D%3E%0Adocker%20pull%20gcr.io%2Fgoogle-containers%2Fxxx%3Ayyy%0Aazure%20%0Adocker%20pull%20gcr.azk8s.cn%2Fgoogle-containers%2Fxxx%3Ayyy%0A\ [STRIKEOUT:%E4%B8%AD%E7%A7%91%E5%A4%A7%20%0Adocker%20pull%20gcr.mirrors.ustc.edu.cn%2Fgoogle-containers%2Fxxx%3Ayyy]\ %0A%0A4.quay.io%E9%95%9C%E5%83%8F%E5%8A%A0%E9%80%9F%0Adocker%20pull%20quay.io%2Fxxx%2Fyyy%3Azzz%0A%E4%B8%AD%E7%A7%91%E5%A4%A7%0Adocker%20pull%20quay.mirrors.ustc.edu.cn%2Fxxx%2Fyyy%3Azzz%0AAzure%E4%B8%AD%E5%9B%BD%0Adocker%20pull%20quay.azk8s.cn%2Fxxx%2Fyyy%3Azzz%0A%0A%0A%0A%23%23%23%20CLI%20%E7%91%9E%E5%A3%AB%E5%86%9B%E5%88%80%0Ak9s%0Akubectx%0Akubens%0Akube-ps1%0Apopeye%0Astern%0Akubectl-bindrole%E5%B8%AE%E5%8A%A9%E6%9F%A5%E6%89%BE%20Kubernetes%20%E9%9B%86%E7%BE%A4%E4%B8%AD%E6%8C%87%E5%AE%9A%20SA%2FGroup%2FUser%20%E7%9A%84%E6%9D%83%E9%99%90%E7%AD%89%E4%BF%A1%E6%81%AF%E3%80%82%0A%0A%23%23%23%20%E4%B8%BA%E4%BB%80%E4%B9%88k8s%E6%98%AF%E7%B3%BB%E7%BB%9F%E7%BA%A7%E5%88%AB%E7%9A%84%E4%BF%9D%E9%9A%9C%20or%20k8s%E7%9A%84%E8%AE%BE%E8%AE%A1%E7%90%86%E5%BF%B5%0Ahttps%3A%2F%2Fhackernoon.com%2Flevel-triggering-and-reconciliation-in-kubernetes-1f17fe30333d%0AKubernetes%20being%20self-healing%0A-%20%E4%B8%A4%E4%B8%AA%E4%BF%A1%E5%8F%B7%0Ak8s%E9%80%9A%E8%BF%87%E8%A7%82%E5%AF%9F2%E4%B8%AA%E4%BF%A1%E5%8F%B7%EF%BC%8C%E4%B8%80%E4%B8%AA%E6%98%AFdesired%20state(%E7%90%86%E6%83%B3)%E7%8A%B6%E6%80%81%E4%B8%80%E4%B8%AA%E6%98%AFactual%20state(%E5%AE%9E%E9%99%85%E7%8A%B6%E6%80%81)%0A-%20%E5%AE%89%E5%85%A8%0A%E4%B8%8E%E4%BC%A0%E7%BB%9F%E7%9A%84YARN%E7%9B%B8%E6%AF%94%E8%BE%83%EF%BC%8C%E5%AE%89%E5%85%A8%E6%80%A7%E6%9B%B4%E9%AB%98%0A-%20%E9%9B%86%E7%BE%A4%E8%A7%84%E6%A8%A1%0A%E5%88%B0%201.9%20%E7%89%88%E6%9C%AC%E5%B0%B1%E5%B7%B2%E5%8F%AF%E4%BB%A5%E7%A8%B3%E5%AE%9A%E6%94%AF%E6%8C%81%205000%20%E4%B8%AA%E8%8A%82%E7%82%B9%E3%80%8115%20%E4%B8%87%E4%B8%AA%20Pod%20%E5%92%8C%2030%20%E4%B8%87%E4%B8%AA%E5%AE%B9%E5%99%A8%E7%9A%84%E8%A7%84%E6%A8%A1%EF%BC%8C%E8%A6%86%E7%9B%96%E7%BB%9D%E5%A4%A7%E9%83%A8%E5%88%86%E7%94%A8%E6%88%B7%E5%9C%BA%E6%99%AF%EF%BC%8C%E4%BB%8E%E6%AD%A4%E5%BD%BB%E5%BA%95%E6%B6%88%E9%99%A4%E4%B8%9A%E7%95%8C%E5%AF%B9%20Kubernetes%20%E8%A7%84%E6%A8%A1%E7%9A%84%E9%A1%BE%E8%99%91%E3%80%82%0A-%20%E8%87%AA%E6%84%88%2F%E8%87%AA%E6%88%91%E8%B0%83%E5%BA%A6%2F%E8%87%AA%E5%8A%A8%E6%9B%B4%E6%96%B0%E3%80%81%E5%9B%9E%E6%BB%9A%2F%E6%B0%B4%E5%B9%B3%E6%89%A9%E5%B1%95%E5%92%8C%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1
