技术002KIngress

-  `技术002KIngress <>`__

   -  `Ingress <>`__

      -  `Nginx-Ingress <>`__

   -  `其他服务暴露方式 <>`__

      -  `CLUSTERIP <>`__
      -  `NODEPORT <>`__
      -  `haproxy <>`__

-  `参考文献 <>`__

技术002KIngress
===============

Ingress
-------

一个集群可以配置多个Ingress。创建Ingress时，可以通过注解ingress.class表示那种ingress。

-  性能优化

由于NginxIngress
Controller要监听物理机上的80端口，我们最初的做法是给他配置了hosrtport，但当大量业务上线时，我们发现QPS超过500/s就会出现无法转发数据包的情况。经过排查发现，系统软中断占用的CPU特别高，hostport会使用iptables进行数据包的转发，后来将Ingress
Controller修改为hostnetwork模式，直接使用Docker的host模式，性能得到提升，QPS可以达到5k以上。

-  Nginx配置优化

Nginx
IngressController大致的工作流程是先通过监听Service、Ingress等资源的变化然后根据Service、Ingress的信息以及nginx.temple文件，将每个service对应的endpoint填入模板中生成最终的Nginx配置。但是很多情况下模板中默认的配置参数并不满足我们的需求，这时需要通过kubernetes中ConfigMap机制基于Nginx
Ingress Controller使用我们定制化的模板。

-  日志回滚

默认情况下Docker会将日志记录在系统的/var/lib/docker/container/xxxx下面的文件里，但是前端日志量是非常大的，很容易就会将系统盘写满，通过配置ConfigMap的方式，可以将日志目录改到主机上，通过配置logrotate服务可以实现日志的定时回滚、压缩等操作。

-  服务应急

当线上服务出现不可用的情况时，我们会准备一套应急的服务作为备用，一但服务出现问题，我们可以将流量切换到应急的服务上去。在k8s上，这一系列操作变得更加简单，这需再准备一套ingress规则，将生产环境的Servuce改为应急的Service，切换的时候通过kubectl
replace -f xxx.yaml 将相应的Ingress替换，即可实现服务的无感知切换。

Nginx-Ingress
~~~~~~~~~~~~~

-  nginx-ingress方案测试通过

（1）根据自己的需要创建ingress的deploy

kubectl apply -f
https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml

（2）根据自己的需要创建svc

kubectl apply -f
https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/service-nodeport.yaml

（3）参考文档3创建两个deploy，测试通过 参考文档

文档1 NGINX Ingress Controller
https://kubernetes.github.io/ingress-nginx/deploy/#prerequisite-generic-deployment-command

文档2 ingress
https://kubernetes.io/docs/concepts/services-networking/ingress/

文档3 Set up Ingress on Minikube with the NGINX Ingress Controller
https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/

-  示例配置文件

   apiVersion: v1 items:

   -  apiVersion: extensions/v1beta1

      kind: Ingress metadata: annotations:
      ingress.kubernetes.io/force-ssl-redirect: “true”
      ingress.kubernetes.io/rewrite-target: /
      kubernetes.io/ingress.class: nginx creationTimestamp:
      2019-05-10T05:29:28Z generation: 1 name: audio-callback-api
      namespace: default resourceVersion: “386759969” selfLink:
      /apis/extensions/v1beta1/namespaces/default/ingresses/audio-callback-api
      uid: 94dcfb45-72e4-11e9-804a-6c92bf4758bd spec: rules:

      -  host: cl-k8s.yidianzixun.com

         http: paths:

         -  backend:

            serviceName: audio-callback-api servicePort: http path:
            /audio status: loadBalancer: ingress:

         -  {}

   -  apiVersion: extensions/v1beta1

      kind: Ingress metadata: annotations:
      ingress.kubernetes.io/grpc-backend: “true”
      ingress.kubernetes.io/proxy-body-size: 16m
      ingress.kubernetes.io/ssl-redirect: “true”
      kubernetes.io/ingress.class: nginx creationTimestamp:
      2019-03-28T02:25:53Z generation: 1 name: compete-feedback-grpc
      namespace: default resourceVersion: “386759940” selfLink:
      /apis/extensions/v1beta1/namespaces/default/ingresses/compete-feedback-grpc
      uid: cf931119-5100-11e9-9b8a-5cb9019cfa9c spec: rules:

      -  http:

         paths:

         -  backend:

            serviceName: compete-feedback servicePort: grpc path:
            /feed.FeedBack status: loadBalancer: ingress:

         -  {}

其他服务暴露方式
----------------

CLUSTERIP
~~~~~~~~~

-  ClusterIP是通过每个节点的kuber-proxy进程修改本地的iptables，使用DNAT的方式将ClusterIP转换为实际的endpoint地址。

NODEPORT
~~~~~~~~

-  NodePort是为了Kubernetes集群外部的应用方便访问kubernetes的服务而提供的一种方案，它会在每个机器上。

-  

haproxy
~~~~~~~

百分点采用的Loadbalancer负载均衡器是基于haproxy，通过watcher
Kubernetes-apiserver中service以及endpoint信息，\ **动态修改haproxy转发规则**\ 来实现的。

参考文献
========

`K8s 工程师必懂的 10 种 Ingress
控制器 <https://mp.weixin.qq.com/s/ooPkJUgtwpdYGwpDjuyrdA>`__

`你想要的百分点大规模Kubernetes集群应用实践来了 <https://yq.aliyun.com/articles/680184>`__

%5BTOC%5D%0A%23%20%E6%8A%80%E6%9C%AF002KIngress%0A%0A%23%23%20Ingress%0A%0A%E4%B8%80%E4%B8%AA%E9%9B%86%E7%BE%A4%E5%8F%AF%E4%BB%A5%E9%85%8D%E7%BD%AE%E5%A4%9A%E4%B8%AAIngress%E3%80%82%E5%88%9B%E5%BB%BAIngress%E6%97%B6%EF%BC%8C%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87%E6%B3%A8%E8%A7%A3ingress.class%E8%A1%A8%E7%A4%BA%E9%82%A3%E7%A7%8Dingress%E3%80%82%0A%0A-%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%0A%0A%E7%94%B1%E4%BA%8ENginxIngress%20Controller%E8%A6%81%E7%9B%91%E5%90%AC%E7%89%A9%E7%90%86%E6%9C%BA%E4%B8%8A%E7%9A%8480%E7%AB%AF%E5%8F%A3%EF%BC%8C%E6%88%91%E4%BB%AC%E6%9C%80%E5%88%9D%E7%9A%84%E5%81%9A%E6%B3%95%E6%98%AF%E7%BB%99%E4%BB%96%E9%85%8D%E7%BD%AE%E4%BA%86hosrtport%EF%BC%8C%E4%BD%86%E5%BD%93%E5%A4%A7%E9%87%8F%E4%B8%9A%E5%8A%A1%E4%B8%8A%E7%BA%BF%E6%97%B6%EF%BC%8C%E6%88%91%E4%BB%AC%E5%8F%91%E7%8E%B0QPS%E8%B6%85%E8%BF%87500%2Fs%E5%B0%B1%E4%BC%9A%E5%87%BA%E7%8E%B0%E6%97%A0%E6%B3%95%E8%BD%AC%E5%8F%91%E6%95%B0%E6%8D%AE%E5%8C%85%E7%9A%84%E6%83%85%E5%86%B5%E3%80%82%E7%BB%8F%E8%BF%87%E6%8E%92%E6%9F%A5%E5%8F%91%E7%8E%B0%EF%BC%8C%E7%B3%BB%E7%BB%9F%E8%BD%AF%E4%B8%AD%E6%96%AD%E5%8D%A0%E7%94%A8%E7%9A%84CPU%E7%89%B9%E5%88%AB%E9%AB%98%EF%BC%8Chostport%E4%BC%9A%E4%BD%BF%E7%94%A8iptables%E8%BF%9B%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%8C%85%E7%9A%84%E8%BD%AC%E5%8F%91%EF%BC%8C%E5%90%8E%E6%9D%A5%E5%B0%86Ingress%20Controller%E4%BF%AE%E6%94%B9%E4%B8%BAhostnetwork%E6%A8%A1%E5%BC%8F%EF%BC%8C%E7%9B%B4%E6%8E%A5%E4%BD%BF%E7%94%A8Docker%E7%9A%84host%E6%A8%A1%E5%BC%8F%EF%BC%8C%E6%80%A7%E8%83%BD%E5%BE%97%E5%88%B0%E6%8F%90%E5%8D%87%EF%BC%8CQPS%E5%8F%AF%E4%BB%A5%E8%BE%BE%E5%88%B05k%E4%BB%A5%E4%B8%8A%E3%80%82%0A%0A-%20Nginx%E9%85%8D%E7%BD%AE%E4%BC%98%E5%8C%96%0A%0ANginx%20IngressController%E5%A4%A7%E8%87%B4%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E6%98%AF%E5%85%88%E9%80%9A%E8%BF%87%E7%9B%91%E5%90%ACService%E3%80%81Ingress%E7%AD%89%E8%B5%84%E6%BA%90%E7%9A%84%E5%8F%98%E5%8C%96%E7%84%B6%E5%90%8E%E6%A0%B9%E6%8D%AEService%E3%80%81Ingress%E7%9A%84%E4%BF%A1%E6%81%AF%E4%BB%A5%E5%8F%8Anginx.temple%E6%96%87%E4%BB%B6%EF%BC%8C%E5%B0%86%E6%AF%8F%E4%B8%AAservice%E5%AF%B9%E5%BA%94%E7%9A%84endpoint%E5%A1%AB%E5%85%A5%E6%A8%A1%E6%9D%BF%E4%B8%AD%E7%94%9F%E6%88%90%E6%9C%80%E7%BB%88%E7%9A%84Nginx%E9%85%8D%E7%BD%AE%E3%80%82%E4%BD%86%E6%98%AF%E5%BE%88%E5%A4%9A%E6%83%85%E5%86%B5%E4%B8%8B%E6%A8%A1%E6%9D%BF%E4%B8%AD%E9%BB%98%E8%AE%A4%E7%9A%84%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E5%B9%B6%E4%B8%8D%E6%BB%A1%E8%B6%B3%E6%88%91%E4%BB%AC%E7%9A%84%E9%9C%80%E6%B1%82%EF%BC%8C%E8%BF%99%E6%97%B6%E9%9C%80%E8%A6%81%E9%80%9A%E8%BF%87kubernetes%E4%B8%ADConfigMap%E6%9C%BA%E5%88%B6%E5%9F%BA%E4%BA%8ENginx%20Ingress%20Controller%E4%BD%BF%E7%94%A8%E6%88%91%E4%BB%AC%E5%AE%9A%E5%88%B6%E5%8C%96%E7%9A%84%E6%A8%A1%E6%9D%BF%E3%80%82%0A%0A-%20%E6%97%A5%E5%BF%97%E5%9B%9E%E6%BB%9A%0A%0A%E9%BB%98%E8%AE%A4%E6%83%85%E5%86%B5%E4%B8%8BDocker%E4%BC%9A%E5%B0%86%E6%97%A5%E5%BF%97%E8%AE%B0%E5%BD%95%E5%9C%A8%E7%B3%BB%E7%BB%9F%E7%9A%84%2Fvar%2Flib%2Fdocker%2Fcontainer%2Fxxxx%E4%B8%8B%E9%9D%A2%E7%9A%84%E6%96%87%E4%BB%B6%E9%87%8C%EF%BC%8C%E4%BD%86%E6%98%AF%E5%89%8D%E7%AB%AF%E6%97%A5%E5%BF%97%E9%87%8F%E6%98%AF%E9%9D%9E%E5%B8%B8%E5%A4%A7%E7%9A%84%EF%BC%8C%E5%BE%88%E5%AE%B9%E6%98%93%E5%B0%B1%E4%BC%9A%E5%B0%86%E7%B3%BB%E7%BB%9F%E7%9B%98%E5%86%99%E6%BB%A1%EF%BC%8C%E9%80%9A%E8%BF%87%E9%85%8D%E7%BD%AEConfigMap%E7%9A%84%E6%96%B9%E5%BC%8F%EF%BC%8C%E5%8F%AF%E4%BB%A5%E5%B0%86%E6%97%A5%E5%BF%97%E7%9B%AE%E5%BD%95%E6%94%B9%E5%88%B0%E4%B8%BB%E6%9C%BA%E4%B8%8A%EF%BC%8C%E9%80%9A%E8%BF%87%E9%85%8D%E7%BD%AElogrotate%E6%9C%8D%E5%8A%A1%E5%8F%AF%E4%BB%A5%E5%AE%9E%E7%8E%B0%E6%97%A5%E5%BF%97%E7%9A%84%E5%AE%9A%E6%97%B6%E5%9B%9E%E6%BB%9A%E3%80%81%E5%8E%8B%E7%BC%A9%E7%AD%89%E6%93%8D%E4%BD%9C%E3%80%82%0A%0A-%20%E6%9C%8D%E5%8A%A1%E5%BA%94%E6%80%A5%0A%0A%E5%BD%93%E7%BA%BF%E4%B8%8A%E6%9C%8D%E5%8A%A1%E5%87%BA%E7%8E%B0%E4%B8%8D%E5%8F%AF%E7%94%A8%E7%9A%84%E6%83%85%E5%86%B5%E6%97%B6%EF%BC%8C%E6%88%91%E4%BB%AC%E4%BC%9A%E5%87%86%E5%A4%87%E4%B8%80%E5%A5%97%E5%BA%94%E6%80%A5%E7%9A%84%E6%9C%8D%E5%8A%A1%E4%BD%9C%E4%B8%BA%E5%A4%87%E7%94%A8%EF%BC%8C%E4%B8%80%E4%BD%86%E6%9C%8D%E5%8A%A1%E5%87%BA%E7%8E%B0%E9%97%AE%E9%A2%98%EF%BC%8C%E6%88%91%E4%BB%AC%E5%8F%AF%E4%BB%A5%E5%B0%86%E6%B5%81%E9%87%8F%E5%88%87%E6%8D%A2%E5%88%B0%E5%BA%94%E6%80%A5%E7%9A%84%E6%9C%8D%E5%8A%A1%E4%B8%8A%E5%8E%BB%E3%80%82%E5%9C%A8k8s%E4%B8%8A%EF%BC%8C%E8%BF%99%E4%B8%80%E7%B3%BB%E5%88%97%E6%93%8D%E4%BD%9C%E5%8F%98%E5%BE%97%E6%9B%B4%E5%8A%A0%E7%AE%80%E5%8D%95%EF%BC%8C%E8%BF%99%E9%9C%80%E5%86%8D%E5%87%86%E5%A4%87%E4%B8%80%E5%A5%97ingress%E8%A7%84%E5%88%99%EF%BC%8C%E5%B0%86%E7%94%9F%E4%BA%A7%E7%8E%AF%E5%A2%83%E7%9A%84Servuce%E6%94%B9%E4%B8%BA%E5%BA%94%E6%80%A5%E7%9A%84Service%EF%BC%8C%E5%88%87%E6%8D%A2%E7%9A%84%E6%97%B6%E5%80%99%E9%80%9A%E8%BF%87kubectl%20replace%20-f%20xxx.yaml%20%E5%B0%86%E7%9B%B8%E5%BA%94%E7%9A%84Ingress%E6%9B%BF%E6%8D%A2%EF%BC%8C%E5%8D%B3%E5%8F%AF%E5%AE%9E%E7%8E%B0%E6%9C%8D%E5%8A%A1%E7%9A%84%E6%97%A0%E6%84%9F%E7%9F%A5%E5%88%87%E6%8D%A2%E3%80%82%0A%0A%23%23%23%20Nginx-Ingress%0A-%20nginx-ingress%E6%96%B9%E6%A1%88%E6%B5%8B%E8%AF%95%E9%80%9A%E8%BF%87%0A%0A%EF%BC%881%EF%BC%89%E6%A0%B9%E6%8D%AE%E8%87%AA%E5%B7%B1%E7%9A%84%E9%9C%80%E8%A6%81%E5%88%9B%E5%BB%BAingress%E7%9A%84deploy%20%0Akubectl%20apply%20-f%20https%3A%2F%2Fraw.githubusercontent.com%2Fkubernetes%2Fingress-nginx%2Fmaster%2Fdeploy%2Fstatic%2Fmandatory.yaml%0A%EF%BC%882%EF%BC%89%E6%A0%B9%E6%8D%AE%E8%87%AA%E5%B7%B1%E7%9A%84%E9%9C%80%E8%A6%81%E5%88%9B%E5%BB%BAsvc%0Akubectl%20apply%20-f%20https%3A%2F%2Fraw.githubusercontent.com%2Fkubernetes%2Fingress-nginx%2Fmaster%2Fdeploy%2Fstatic%2Fprovider%2Fbaremetal%2Fservice-nodeport.yaml%0A%EF%BC%883%EF%BC%89%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A33%E5%88%9B%E5%BB%BA%E4%B8%A4%E4%B8%AAdeploy%EF%BC%8C%E6%B5%8B%E8%AF%95%E9%80%9A%E8%BF%87%0A%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%0A%E6%96%87%E6%A1%A31%20NGINX%20Ingress%20Controller%20https%3A%2F%2Fkubernetes.github.io%2Fingress-nginx%2Fdeploy%2F%23prerequisite-generic-deployment-command%0A%E6%96%87%E6%A1%A32%20ingress%20https%3A%2F%2Fkubernetes.io%2Fdocs%2Fconcepts%2Fservices-networking%2Fingress%2F%20%0A%E6%96%87%E6%A1%A33%20Set%20up%20Ingress%20on%20Minikube%20with%20the%20NGINX%20Ingress%20Controller%20https%3A%2F%2Fkubernetes.io%2Fdocs%2Ftasks%2Faccess-application-cluster%2Fingress-minikube%2F%0A%0A-%20%E7%A4%BA%E4%BE%8B%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%0A%60%60%60%0AapiVersion%3A%20v1%0Aitems%3A%0A-%20apiVersion%3A%20extensions%2Fv1beta1%0A%20%20kind%3A%20Ingress%0A%20%20metadata%3A%0A%20%20%20%20annotations%3A%0A%20%20%20%20%20%20ingress.kubernetes.io%2Fforce-ssl-redirect%3A%20%22true%22%0A%20%20%20%20%20%20ingress.kubernetes.io%2Frewrite-target%3A%20%2F%0A%20%20%20%20%20%20kubernetes.io%2Fingress.class%3A%20nginx%0A%20%20%20%20creationTimestamp%3A%202019-05-10T05%3A29%3A28Z%0A%20%20%20%20generation%3A%201%0A%20%20%20%20name%3A%20audio-callback-api%0A%20%20%20%20namespace%3A%20default%0A%20%20%20%20resourceVersion%3A%20%22386759969%22%0A%20%20%20%20selfLink%3A%20%2Fapis%2Fextensions%2Fv1beta1%2Fnamespaces%2Fdefault%2Fingresses%2Faudio-callback-api%0A%20%20%20%20uid%3A%2094dcfb45-72e4-11e9-804a-6c92bf4758bd%0A%20%20spec%3A%0A%20%20%20%20rules%3A%0A%20%20%20%20-%20host%3A%20cl-k8s.yidianzixun.com%0A%20%20%20%20%20%20http%3A%0A%20%20%20%20%20%20%20%20paths%3A%0A%20%20%20%20%20%20%20%20-%20backend%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20serviceName%3A%20audio-callback-api%0A%20%20%20%20%20%20%20%20%20%20%20%20servicePort%3A%20http%0A%20%20%20%20%20%20%20%20%20%20path%3A%20%2Faudio%0A%20%20status%3A%0A%20%20%20%20loadBalancer%3A%0A%20%20%20%20%20%20ingress%3A%0A%20%20%20%20%20%20-%20%7B%7D%0A%60%60%60%0A%0A%60%60%60%0A-%20apiVersion%3A%20extensions%2Fv1beta1%0A%20%20kind%3A%20Ingress%0A%20%20metadata%3A%0A%20%20%20%20annotations%3A%0A%20%20%20%20%20%20ingress.kubernetes.io%2Fgrpc-backend%3A%20%22true%22%0A%20%20%20%20%20%20ingress.kubernetes.io%2Fproxy-body-size%3A%2016m%0A%20%20%20%20%20%20ingress.kubernetes.io%2Fssl-redirect%3A%20%22true%22%0A%20%20%20%20%20%20kubernetes.io%2Fingress.class%3A%20nginx%0A%20%20%20%20creationTimestamp%3A%202019-03-28T02%3A25%3A53Z%0A%20%20%20%20generation%3A%201%0A%20%20%20%20name%3A%20compete-feedback-grpc%0A%20%20%20%20namespace%3A%20default%0A%20%20%20%20resourceVersion%3A%20%22386759940%22%0A%20%20%20%20selfLink%3A%20%2Fapis%2Fextensions%2Fv1beta1%2Fnamespaces%2Fdefault%2Fingresses%2Fcompete-feedback-grpc%0A%20%20%20%20uid%3A%20cf931119-5100-11e9-9b8a-5cb9019cfa9c%0A%20%20spec%3A%0A%20%20%20%20rules%3A%0A%20%20%20%20-%20http%3A%0A%20%20%20%20%20%20%20%20paths%3A%0A%20%20%20%20%20%20%20%20-%20backend%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20serviceName%3A%20compete-feedback%0A%20%20%20%20%20%20%20%20%20%20%20%20servicePort%3A%20grpc%0A%20%20%20%20%20%20%20%20%20%20path%3A%20%2Ffeed.FeedBack%0A%20%20status%3A%0A%20%20%20%20loadBalancer%3A%0A%20%20%20%20%20%20ingress%3A%0A%20%20%20%20%20%20-%20%7B%7D%0A%60%60%60%0A%0A%0A%23%23%20%E5%85%B6%E4%BB%96%E6%9C%8D%E5%8A%A1%E6%9A%B4%E9%9C%B2%E6%96%B9%E5%BC%8F%0A%23%23%23%20CLUSTERIP%0A-%20ClusterIP%E6%98%AF%E9%80%9A%E8%BF%87%E6%AF%8F%E4%B8%AA%E8%8A%82%E7%82%B9%E7%9A%84kuber-proxy%E8%BF%9B%E7%A8%8B%E4%BF%AE%E6%94%B9%E6%9C%AC%E5%9C%B0%E7%9A%84iptables%EF%BC%8C%E4%BD%BF%E7%94%A8DNAT%E7%9A%84%E6%96%B9%E5%BC%8F%E5%B0%86ClusterIP%E8%BD%AC%E6%8D%A2%E4%B8%BA%E5%AE%9E%E9%99%85%E7%9A%84endpoint%E5%9C%B0%E5%9D%80%E3%80%82%0A%0A%23%23%23%20NODEPORT%0A-%20NodePort%E6%98%AF%E4%B8%BA%E4%BA%86Kubernetes%E9%9B%86%E7%BE%A4%E5%A4%96%E9%83%A8%E7%9A%84%E5%BA%94%E7%94%A8%E6%96%B9%E4%BE%BF%E8%AE%BF%E9%97%AEkubernetes%E7%9A%84%E6%9C%8D%E5%8A%A1%E8%80%8C%E6%8F%90%E4%BE%9B%E7%9A%84%E4%B8%80%E7%A7%8D%E6%96%B9%E6%A1%88%EF%BC%8C%E5%AE%83%E4%BC%9A%E5%9C%A8%E6%AF%8F%E4%B8%AA%E6%9C%BA%E5%99%A8%E4%B8%8A%E3%80%82%0A-%20%0A%0A%23%23%23%20haproxy%0A%E7%99%BE%E5%88%86%E7%82%B9%E9%87%87%E7%94%A8%E7%9A%84Loadbalancer%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E5%99%A8%E6%98%AF%E5%9F%BA%E4%BA%8Ehaproxy%EF%BC%8C%E9%80%9A%E8%BF%87watcher%20Kubernetes-apiserver%E4%B8%ADservice%E4%BB%A5%E5%8F%8Aendpoint%E4%BF%A1%E6%81%AF%EF%BC%8C\ **%E5%8A%A8%E6%80%81%E4%BF%AE%E6%94%B9haproxy%E8%BD%AC%E5%8F%91%E8%A7%84%E5%88%99**\ %E6%9D%A5%E5%AE%9E%E7%8E%B0%E7%9A%84%E3%80%82%0A%0A%0A%23%23%20%E5%8F%82%E8%80%83%E6%96%87%E7%8C%AE%0A%0A%5BK8s%20%E5%B7%A5%E7%A8%8B%E5%B8%88%E5%BF%85%E6%87%82%E7%9A%84%2010%20%E7%A7%8D%20Ingress%20%E6%8E%A7%E5%88%B6%E5%99%A8%5D(https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FooPkJUgtwpdYGwpDjuyrdA)%0A%0A%5B%E4%BD%A0%E6%83%B3%E8%A6%81%E7%9A%84%E7%99%BE%E5%88%86%E7%82%B9%E5%A4%A7%E8%A7%84%E6%A8%A1Kubernetes%E9%9B%86%E7%BE%A4%E5%BA%94%E7%94%A8%E5%AE%9E%E8%B7%B5%E6%9D%A5%E4%BA%86%5D(https%3A%2F%2Fyq.aliyun.com%2Farticles%2F680184)%0A%0A
