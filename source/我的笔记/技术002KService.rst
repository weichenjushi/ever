技术002KService

技术002KService
===============

Service转发的原理实现
---------------------

service 转发主要是 node 上的 kube-proxy 进程通过 watch apiserver 获取
service 对应的 endpoint，再写入 iptables 或 ipvs 规则来实现的; 对于
headless service，主要是通过 kube-dns 或 coredns 动态解析到不同 endpoint
ip 来实现的。实现 service
就近转发的关键点就在于如何将流量转发到跟当前节点在同一拓扑域的 endpoint
上，也就是会进行一次 endpoint 筛选，选出一部分符合当前节点拓扑域的
endpoint 进行转发。

%23%20%E6%8A%80%E6%9C%AF002KService%0A%5BTOC%5D%0A%0A%23%23%23%20Service%E8%BD%AC%E5%8F%91%E7%9A%84%E5%8E%9F%E7%90%86%E5%AE%9E%E7%8E%B0%0A%0Aservice%20%E8%BD%AC%E5%8F%91%E4%B8%BB%E8%A6%81%E6%98%AF%20node%20%E4%B8%8A%E7%9A%84%20kube-proxy%20%E8%BF%9B%E7%A8%8B%E9%80%9A%E8%BF%87%20watch%20apiserver%20%E8%8E%B7%E5%8F%96%20service%20%E5%AF%B9%E5%BA%94%E7%9A%84%20endpoint%EF%BC%8C%E5%86%8D%E5%86%99%E5%85%A5%20iptables%20%E6%88%96%20ipvs%20%E8%A7%84%E5%88%99%E6%9D%A5%E5%AE%9E%E7%8E%B0%E7%9A%84%3B%20%E5%AF%B9%E4%BA%8E%20headless%20service%EF%BC%8C%E4%B8%BB%E8%A6%81%E6%98%AF%E9%80%9A%E8%BF%87%20kube-dns%20%E6%88%96%20coredns%20%E5%8A%A8%E6%80%81%E8%A7%A3%E6%9E%90%E5%88%B0%E4%B8%8D%E5%90%8C%20endpoint%20ip%20%E6%9D%A5%E5%AE%9E%E7%8E%B0%E7%9A%84%E3%80%82%E5%AE%9E%E7%8E%B0%20service%20%E5%B0%B1%E8%BF%91%E8%BD%AC%E5%8F%91%E7%9A%84%E5%85%B3%E9%94%AE%E7%82%B9%E5%B0%B1%E5%9C%A8%E4%BA%8E%E5%A6%82%E4%BD%95%E5%B0%86%E6%B5%81%E9%87%8F%E8%BD%AC%E5%8F%91%E5%88%B0%E8%B7%9F%E5%BD%93%E5%89%8D%E8%8A%82%E7%82%B9%E5%9C%A8%E5%90%8C%E4%B8%80%E6%8B%93%E6%89%91%E5%9F%9F%E7%9A%84%20endpoint%20%E4%B8%8A%EF%BC%8C%E4%B9%9F%E5%B0%B1%E6%98%AF%E4%BC%9A%E8%BF%9B%E8%A1%8C%E4%B8%80%E6%AC%A1%20endpoint%20%E7%AD%9B%E9%80%89%EF%BC%8C%E9%80%89%E5%87%BA%E4%B8%80%E9%83%A8%E5%88%86%E7%AC%A6%E5%90%88%E5%BD%93%E5%89%8D%E8%8A%82%E7%82%B9%E6%8B%93%E6%89%91%E5%9F%9F%E7%9A%84%20endpoint%20%E8%BF%9B%E8%A1%8C%E8%BD%AC%E5%8F%91%E3%80%82%0A%0A
