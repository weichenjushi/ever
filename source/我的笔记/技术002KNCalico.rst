技术002KNCalico

-  `技术002KNCalico <>`__

   -  `Requirements <>`__

      -  `Basic <>`__
      -  `IP pool <>`__

技术002KNCalico
===============

Requirements
------------

Basic
~~~~~

Calico can run on any Kubernetes cluster which meets the following
criteria.

-  The kubelet must be configured to use CNI network plugins (e.g
   –network-plugin=cni).
-  The kube-proxy must be started in iptables proxy mode. This is the
   default as of Kubernetes v1.2.0.
-  The kube-proxy must be started without the –masquerade-all flag,
   which conflicts with Calico policy.
-  The Kubernetes NetworkPolicy API requires at least Kubernetes version
   v1.3.0.

IP pool
~~~~~~~

calico网段默认是

in Kubernetes, all three of the following arguments must be equal to, or
contain, the Calico IP pool CIDRs:

kube-apiserver: –pod-network-cidr kube-proxy: –cluster-cidr
kube-controller-manager: –cluster-cidr

%5BTOC%5D%0A%0A%23%20%E6%8A%80%E6%9C%AF002KNCalico%0A%23%23%20Requirements%0A%23%23%23%20Basic%0ACalico%20can%20run%20on%20any%20Kubernetes%20cluster%20which%20meets%20the%20following%20criteria.%0A%0A-%20The%20kubelet%20must%20be%20configured%20to%20use%20CNI%20network%20plugins%20(e.g%20–network-plugin%3Dcni).%0A-%20The%20kube-proxy%20must%20be%20started%20in%20iptables%20proxy%20mode.%20This%20is%20the%20default%20as%20of%20Kubernetes%20v1.2.0.%0A-%20The%20kube-proxy%20must%20be%20started%20without%20the%20–masquerade-all%20flag%2C%20which%20conflicts%20with%20Calico%20policy.%0A-%20The%20Kubernetes%20NetworkPolicy%20API%20requires%20at%20least%20Kubernetes%20version%20v1.3.0.%0A%23%23%23%20IP%20pool%0Acalico%E7%BD%91%E6%AE%B5%E9%BB%98%E8%AE%A4%E6%98%AF%0Ain%20Kubernetes%2C%20all%20three%20of%20the%20following%20arguments%20must%20be%20equal%20to%2C%20or%20contain%2C%20the%20Calico%20IP%20pool%20CIDRs%3A%0A%0Akube-apiserver%3A%20–pod-network-cidr%0Akube-proxy%3A%20–cluster-cidr%0Akube-controller-manager%3A%20–cluster-cidr%0A%0A%0A%0A%0A%0A
