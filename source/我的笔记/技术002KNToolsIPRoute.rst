技术002KNToolsIPRoute

安装
====

yum install iproute

常用命令
========

ip addr ip addr show eth0 ip link ip link show eth0 ip -s link show eth0
统计网卡的输入和输出 ip route show default via 107.170.58.1 dev eth0
metric 100 107.170.58.0/24 dev eth0 proto kernel scope link src
107.170.58.162

This shows us that the default route to the greater internet is
available through the eth0 interface and the address 107.170.58.1. We
can access this server through that interface, where our own interface
address is 107.170.58.162.

重启网卡 ip link set eth1 up ip link set eth1 down 广播 ip link set eth1
multicast on ip link set eth1 multicast off
调整最大传输数据包mtu和队列大小 ip link set eth1 mtu 1500 ip link set
eth1 txqueuelen 1000 在网卡down的状态下，修改网卡的名字和arp状态 ip link
set eth1 name eth10 ip link set eth1 arp on 添加device的地址 ip addr add
ip_address/net_prefix brd + dev interface 删除device的地址 ip addr del
ip_address/net_prefix dev interface ip rule show

%23%23%20%E5%AE%89%E8%A3%85%0Ayum%20install%20iproute%0A%0A%23%23%20%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4%0Aip%20addr%0Aip%20addr%20show%20eth0%0Aip%20link%0Aip%20link%20show%20eth0%0Aip%20-s%20link%20show%20eth0%20%E7%BB%9F%E8%AE%A1%E7%BD%91%E5%8D%A1%E7%9A%84%E8%BE%93%E5%85%A5%E5%92%8C%E8%BE%93%E5%87%BA%0Aip%20route%20show%0Adefault%20via%20107.170.58.1%20dev%20eth0%20%20metric%20100%0A107.170.58.0%2F24%20dev%20eth0%20%20proto%20kernel%20%20scope%20link%20%20src%20107.170.58.162%20%0AThis%20shows%20us%20that%20the%20default%20route%20to%20the%20greater%20internet%20is%20available%20through%20the%20eth0%20interface%20and%20the%20address%20107.170.58.1.%20We%20can%20access%20this%20server%20through%20that%20interface%2C%20where%20our%20own%20interface%20address%20is%20107.170.58.162.%0A%E9%87%8D%E5%90%AF%E7%BD%91%E5%8D%A1%0Aip%20link%20set%20eth1%20up%0Aip%20link%20set%20eth1%20down%0A%E5%B9%BF%E6%92%AD%0Aip%20link%20set%20eth1%20multicast%20on%0Aip%20link%20set%20eth1%20multicast%20off%0A%E8%B0%83%E6%95%B4%E6%9C%80%E5%A4%A7%E4%BC%A0%E8%BE%93%E6%95%B0%E6%8D%AE%E5%8C%85mtu%E5%92%8C%E9%98%9F%E5%88%97%E5%A4%A7%E5%B0%8F%0Aip%20link%20set%20eth1%20mtu%201500%0Aip%20link%20set%20eth1%20txqueuelen%201000%0A%E5%9C%A8%E7%BD%91%E5%8D%A1down%E7%9A%84%E7%8A%B6%E6%80%81%E4%B8%8B%EF%BC%8C%E4%BF%AE%E6%94%B9%E7%BD%91%E5%8D%A1%E7%9A%84%E5%90%8D%E5%AD%97%E5%92%8Carp%E7%8A%B6%E6%80%81%0Aip%20link%20set%20eth1%20name%20eth10%0Aip%20link%20set%20eth1%20arp%20on%0A%E6%B7%BB%E5%8A%A0device%E7%9A%84%E5%9C%B0%E5%9D%80%0Aip%20addr%20add%20ip_address%2Fnet_prefix%20brd%20%2B%20dev%20interface%0A%E5%88%A0%E9%99%A4device%E7%9A%84%E5%9C%B0%E5%9D%80%0Aip%20addr%20del%20ip_address%2Fnet_prefix%20dev%20interface%0Aip%20rule%20show%0A
