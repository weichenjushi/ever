技术002KNToolsTCPDUMP

tcpdump
http://bencane.com/2014/10/13/quick-and-practical-reference-for-tcpdump/
A Quick and Practical Reference for tcpdump tcpdump -n 不带hostname
tcpdump -vvv -vvv显示更多的ip packets信息 tcpdump -vvvv -c 1
-c显示的行数 tcpdump -i eth0 -i 指定监控的网卡信息，默认是取eth0，
tcpdump -i any any是监控服务器上的所有的网卡 tcpdump -w /path/to/file
-w保存到文件中 tcpdump -r /path/to/file -r从文件中读取 tcpdump -s 100
-s指定packet的长度，默认是65535 bytes filter tcpdump -nvvv -i any -c 3
host 10.0.3.1 过滤host=10.0.3.1的流量 tcpdump -nvvv -i any -c 3 src host
10.0.3.1 只显示source host的流量

tcpdump -nvvv -i any -c 3 port 22 and port 60738
比较有用，只抓取port22和60738之间的流量，或者tcpdump -nvvv -i any -c 3
‘port 22 && port 60738’

tcpdump -nvvv -i any -c 20 ‘port 80 or port 443’ 或 tcpdump -nvvv -i any
-c 20 ‘port 80 or port 443’

tcpdump -nvvv -i any -c 20 ‘(port 80 or port 443) and host 10.0.3.169’

tcpdump -nvvv -i any -c 20 ‘((port 80 or port 443) and (host 10.0.3.169
or host 10.0.3.1)) and dst host 10.0.3.246’ 通过括号来实现运算的优先级别

抓取其他协议ICMP, UDP, and ARP 的包 观察 1.srcip和port dstip和port

10.0.3.246.56894 > 192.168.0.92.22: Flags [S], cksum 0xcf28 (incorrect
-> 0x0388), seq 682725222, win 29200, options [mss 1460,sackOK,TS val
619989005 ecr 0,nop,wscale 7], length 0

Given the above output we can see that the source ip is 10.0.3.246 the
source port is 56894 and the destination ip is 192.168.0.92 with a
destination port of 22. This is pretty easy to identify once you
understand the format of tcpdump. If you haven’t guessed the format yet
you can break it down as follows src-ip.src-port > dest-ip.dest-port:
Flags[S] the source is in front of the > and the destination is behind.
You can think of the > as an arrow pointing to the destination.

2.packet的类型 以及packet观察

10.0.3.246.56894 > 192.168.0.92.22: Flags [S], cksum 0xcf28 (incorrect
-> 0x0388), seq 682725222, win 29200, options [mss 1460,sackOK,TS val
619989005 ecr 0,nop,wscale 7], length 0

From the sample above we can tell that the packet is a single SYN
packet. We can identify this by the Flags [S] section of the tcpdump
output, different types of packets have different types of flags.
Without going too deep into what types of packets exist within TCP you
can use the below as a cheat sheet for identifying packet types.

[S] - SYN (Start Connection) [.] - No Flag Set [P] - PSH (Push Data) [F]
- FIN (Finish Connection) [R] - RST (Reset Connection)

Depending on the version and output of tcpdump you may also see flags
such as [S.] this is used to indicate a SYN-ACK packet.

An unhealthy example

15:15:43.323412 IP (tos 0x0, ttl 64, id 51051, offset 0, flags [DF],
proto TCP (6), length 60)

10.0.3.246.56894 > 192.168.0.92.22: Flags [S], cksum 0xcf28 (incorrect
-> 0x0388), seq 682725222, win 29200, options [mss 1460,sackOK,TS val
619989005 ecr 0,nop,wscale 7], length 0

15:15:44.321444 IP (tos 0x0, ttl 64, id 51052, offset 0, flags [DF],
proto TCP (6), length 60)

10.0.3.246.56894 > 192.168.0.92.22: Flags [S], cksum 0xcf28 (incorrect
-> 0x028e), seq 682725222, win 29200, options [mss 1460,sackOK,TS val
619989255 ecr 0,nop,wscale 7], length 0

15:15:46.321610 IP (tos 0x0, ttl 64, id 51053, offset 0, flags [DF],
proto TCP (6), length 60)

10.0.3.246.56894 > 192.168.0.92.22: Flags [S], cksum 0xcf28 (incorrect
-> 0x009a), seq 682725

The above sampling shows an example of an unhealthy exchange, and by
unhealthy exchange for this example that means no exchange. In the above
sample we can see that 10.0.3.246 is sending a SYN packet to host
192.168.0.92 however we never see a response from host 192.168.0.92.

A healthy example

15:18:25.716453 IP (tos 0x10, ttl 64, id 53344, offset 0, flags [DF],
proto TCP (6), length 60)

10.0.3.246.34908 > 192.168.0.110.22: Flags [S], cksum 0xcf3a (incorrect
-> 0xc838), seq 1943877315, win 29200, options [mss 1460,sackOK,TS val
620029603 ecr 0,nop,wscale 7], length 0

15:18:25.716777 IP (tos 0x0, ttl 63, id 0, offset 0, flags [DF], proto
TCP (6), length 60)

192.168.0.110.22 > 10.0.3.246.34908: Flags [S.], cksum 0x594a (correct),
seq 4001145915, ack 1943877316, win 5792, options [mss 1460,sackOK,TS
val 18495104 ecr 620029603,nop,wscale 2], length 0

15:18:25.716899 IP (tos 0x10, ttl 64, id 53345, offset 0, flags [DF],
proto TCP (6), length 52)

10.0.3.246.34908 > 192.168.0.110.22: Flags [.], cksum 0xcf32 (incorrect
-> 0x9dcc), ack 1, win 229, options [nop,nop,TS val 620029603 ecr
18495104], length 0

A healthy example would look like the above, in the above we can see a
standard TCP 3-way handshake. The first packet above is a SYN packet
from host 10.0.3.246 to host 192.168.0.110, the second packet is a
SYN-ACK from host 192.168.0.110 acknowledging the SYN. The final packet
is a ACK or rather a SYN-ACK-ACK from host 10.0.3.246 acknowledging that
it has received the SYN-ACK. From this point on there is an established
TCP/IP connection.

3.用Hex和ASCII 查看数据包

tcpdump -nvvv -i any -c 1 -XX ‘port 80 and host 10.0.3.1’ -XX flag to
print the packet data in hex and ascii

tcpdump -nvvv -i any -c 1 -A ‘port 80 and host 10.0.3.1’ -A Printing
packet data in ASCII only

通过这种人可读取的格式方便排查问题，如果对于加密的数据，
可以通过ssldump和wireshark。However, if you use have the certificates in
use you could use commands such as ssldump or even wireshark.

tcpdump
http://bencane.com/2014/10/13/quick-and-practical-reference-for-tcpdump/
A Quick and Practical Reference for tcpdump tcpdump -n 不带hostname
tcpdump -vvv -vvv显示更多的ip packets信息 tcpdump -vvvv -c 1
-c显示的行数 tcpdump -i eth0 -i 指定监控的网卡信息，默认是取eth0，
tcpdump -i any any是监控服务器上的所有的网卡 tcpdump -w /path/to/file
-w保存到文件中 tcpdump -r /path/to/file -r从文件中读取 tcpdump -s 100
-s指定packet的长度，默认是65535 bytes filter tcpdump -nvvv -i any -c 3
host 10.0.3.1 过滤host=10.0.3.1的流量 tcpdump -nvvv -i any -c 3 src host
10.0.3.1 只显示source host的流量

tcpdump -nvvv -i any -c 3 port 22 and port 60738
比较有用，只抓取port22和60738之间的流量，或者tcpdump -nvvv -i any -c 3
‘port 22 && port 60738’

tcpdump -nvvv -i any -c 20 ‘port 80 or port 443’ 或 tcpdump -nvvv -i any
-c 20 ‘port 80 or port 443’

tcpdump -nvvv -i any -c 20 ‘(port 80 or port 443) and host 10.0.3.169’

tcpdump -nvvv -i any -c 20 ‘((port 80 or port 443) and (host 10.0.3.169
or host 10.0.3.1)) and dst host 10.0.3.246’ 通过括号来实现运算的优先级别

抓取其他协议ICMP, UDP, and ARP 的包 观察 1.srcip和port dstip和port

10.0.3.246.56894 > 192.168.0.92.22: Flags [S], cksum 0xcf28 (incorrect
-> 0x0388), seq 682725222, win 29200, options [mss 1460,sackOK,TS val
619989005 ecr 0,nop,wscale 7], length 0

Given the above output we can see that the source ip is 10.0.3.246 the
source port is 56894 and the destination ip is 192.168.0.92 with a
destination port of 22. This is pretty easy to identify once you
understand the format of tcpdump. If you haven’t guessed the format yet
you can break it down as follows src-ip.src-port > dest-ip.dest-port:
Flags[S] the source is in front of the > and the destination is behind.
You can think of the > as an arrow pointing to the destination.

2.packet的类型 以及packet观察

10.0.3.246.56894 > 192.168.0.92.22: Flags [S], cksum 0xcf28 (incorrect
-> 0x0388), seq 682725222, win 29200, options [mss 1460,sackOK,TS val
619989005 ecr 0,nop,wscale 7], length 0

From the sample above we can tell that the packet is a single SYN
packet. We can identify this by the Flags [S] section of the tcpdump
output, different types of packets have different types of flags.
Without going too deep into what types of packets exist within TCP you
can use the below as a cheat sheet for identifying packet types.

[S] - SYN (Start Connection) [.] - No Flag Set [P] - PSH (Push Data) [F]
- FIN (Finish Connection) [R] - RST (Reset Connection)

Depending on the version and output of tcpdump you may also see flags
such as [S.] this is used to indicate a SYN-ACK packet.

An unhealthy example

15:15:43.323412 IP (tos 0x0, ttl 64, id 51051, offset 0, flags [DF],
proto TCP (6), length 60)

10.0.3.246.56894 > 192.168.0.92.22: Flags [S], cksum 0xcf28 (incorrect
-> 0x0388), seq 682725222, win 29200, options [mss 1460,sackOK,TS val
619989005 ecr 0,nop,wscale 7], length 0

15:15:44.321444 IP (tos 0x0, ttl 64, id 51052, offset 0, flags [DF],
proto TCP (6), length 60)

10.0.3.246.56894 > 192.168.0.92.22: Flags [S], cksum 0xcf28 (incorrect
-> 0x028e), seq 682725222, win 29200, options [mss 1460,sackOK,TS val
619989255 ecr 0,nop,wscale 7], length 0

15:15:46.321610 IP (tos 0x0, ttl 64, id 51053, offset 0, flags [DF],
proto TCP (6), length 60)

10.0.3.246.56894 > 192.168.0.92.22: Flags [S], cksum 0xcf28 (incorrect
-> 0x009a), seq 682725

The above sampling shows an example of an unhealthy exchange, and by
unhealthy exchange for this example that means no exchange. In the above
sample we can see that 10.0.3.246 is sending a SYN packet to host
192.168.0.92 however we never see a response from host 192.168.0.92.

A healthy example

15:18:25.716453 IP (tos 0x10, ttl 64, id 53344, offset 0, flags [DF],
proto TCP (6), length 60)

10.0.3.246.34908 > 192.168.0.110.22: Flags [S], cksum 0xcf3a (incorrect
-> 0xc838), seq 1943877315, win 29200, options [mss 1460,sackOK,TS val
620029603 ecr 0,nop,wscale 7], length 0

15:18:25.716777 IP (tos 0x0, ttl 63, id 0, offset 0, flags [DF], proto
TCP (6), length 60)

192.168.0.110.22 > 10.0.3.246.34908: Flags [S.], cksum 0x594a (correct),
seq 4001145915, ack 1943877316, win 5792, options [mss 1460,sackOK,TS
val 18495104 ecr 620029603,nop,wscale 2], length 0

15:18:25.716899 IP (tos 0x10, ttl 64, id 53345, offset 0, flags [DF],
proto TCP (6), length 52)

10.0.3.246.34908 > 192.168.0.110.22: Flags [.], cksum 0xcf32 (incorrect
-> 0x9dcc), ack 1, win 229, options [nop,nop,TS val 620029603 ecr
18495104], length 0

A healthy example would look like the above, in the above we can see a
standard TCP 3-way handshake. The first packet above is a SYN packet
from host 10.0.3.246 to host 192.168.0.110, the second packet is a
SYN-ACK from host 192.168.0.110 acknowledging the SYN. The final packet
is a ACK or rather a SYN-ACK-ACK from host 10.0.3.246 acknowledging that
it has received the SYN-ACK. From this point on there is an established
TCP/IP connection.

3.用Hex和ASCII 查看数据包

tcpdump -nvvv -i any -c 1 -XX ‘port 80 and host 10.0.3.1’ -XX flag to
print the packet data in hex and ascii

tcpdump -nvvv -i any -c 1 -A ‘port 80 and host 10.0.3.1’ -A Printing
packet data in ASCII only

通过这种人可读取的格式方便排查问题，如果对于加密的数据，
可以通过ssldump和wireshark。However, if you use have the certificates in
use you could use commands such as ssldump or even wireshark.

tcpdump%20%0Ahttp%3A%2F%2Fbencane.com%2F2014%2F10%2F13%2Fquick-and-practical-reference-for-tcpdump%2F%0AA%20Quick%20and%20Practical%20Reference%20for%20tcpdump%0Atcpdump%20-n%20%E4%B8%8D%E5%B8%A6hostname%0Atcpdump%20-vvv%20%20%20%20%20%20%20-vvv%E6%98%BE%E7%A4%BA%E6%9B%B4%E5%A4%9A%E7%9A%84ip%20packets%E4%BF%A1%E6%81%AF%0Atcpdump%20-vvvv%20-c%201%20%20-c%E6%98%BE%E7%A4%BA%E7%9A%84%E8%A1%8C%E6%95%B0%0Atcpdump%20-i%20eth0%20%20%20%20%20-i%20%E6%8C%87%E5%AE%9A%E7%9B%91%E6%8E%A7%E7%9A%84%E7%BD%91%E5%8D%A1%E4%BF%A1%E6%81%AF%EF%BC%8C%E9%BB%98%E8%AE%A4%E6%98%AF%E5%8F%96eth0%EF%BC%8C%20tcpdump%20-i%20any%20%20any%E6%98%AF%E7%9B%91%E6%8E%A7%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84%E6%89%80%E6%9C%89%E7%9A%84%E7%BD%91%E5%8D%A1%0Atcpdump%20-w%20%2Fpath%2Fto%2Ffile%20%20-w%E4%BF%9D%E5%AD%98%E5%88%B0%E6%96%87%E4%BB%B6%E4%B8%AD%0Atcpdump%20-r%20%2Fpath%2Fto%2Ffile%20%20-r%E4%BB%8E%E6%96%87%E4%BB%B6%E4%B8%AD%E8%AF%BB%E5%8F%96%0Atcpdump%20-s%20100%20-s%E6%8C%87%E5%AE%9Apacket%E7%9A%84%E9%95%BF%E5%BA%A6%EF%BC%8C%E9%BB%98%E8%AE%A4%E6%98%AF65535%20bytes%0Afilter%0Atcpdump%20-nvvv%20-i%20any%20-c%203%20host%2010.0.3.1%20%20%E8%BF%87%E6%BB%A4host%3D10.0.3.1%E7%9A%84%E6%B5%81%E9%87%8F%0Atcpdump%20-nvvv%20-i%20any%20-c%203%20src%20host%2010.0.3.1%20%E5%8F%AA%E6%98%BE%E7%A4%BAsource%20host%E7%9A%84%E6%B5%81%E9%87%8F%0Atcpdump%20-nvvv%20-i%20any%20-c%203%20port%2022%20and%20port%2060738%20%E6%AF%94%E8%BE%83%E6%9C%89%E7%94%A8%EF%BC%8C%E5%8F%AA%E6%8A%93%E5%8F%96port22%E5%92%8C60738%E4%B9%8B%E9%97%B4%E7%9A%84%E6%B5%81%E9%87%8F%EF%BC%8C%E6%88%96%E8%80%85tcpdump%20-nvvv%20-i%20any%20-c%203%20’port%2022%20%26%26%20port%2060738’%0Atcpdump%20-nvvv%20-i%20any%20-c%2020%20’port%2080%20or%20port%20443’%20%E6%88%96%20%20tcpdump%20-nvvv%20-i%20any%20-c%2020%20’port%2080%20or%20port%20443’%0Atcpdump%20-nvvv%20-i%20any%20-c%2020%20’(port%2080%20or%20port%20443)%20and%20host%2010.0.3.169’%0Atcpdump%20-nvvv%20-i%20any%20-c%2020%20’((port%2080%20or%20port%20443)%20and%20(host%2010.0.3.169%20or%20host%2010.0.3.1))%20and%20dst%20host%2010.0.3.246’%20%E9%80%9A%E8%BF%87%E6%8B%AC%E5%8F%B7%E6%9D%A5%E5%AE%9E%E7%8E%B0%E8%BF%90%E7%AE%97%E7%9A%84%E4%BC%98%E5%85%88%E7%BA%A7%E5%88%AB%0A%E6%8A%93%E5%8F%96%E5%85%B6%E4%BB%96%E5%8D%8F%E8%AE%AEICMP%2C%20UDP%2C%20and%20ARP%20%E7%9A%84%E5%8C%85%0A%E8%A7%82%E5%AF%9F%0A1.srcip%E5%92%8Cport%20%20%20dstip%E5%92%8Cport%0A10.0.3.246.56894%20%3E%20192.168.0.92.22%3A%20Flags%20%5BS%5D%2C%20cksum%200xcf28%20(incorrect%20-%3E%200x0388)%2C%20seq%20682725222%2C%20win%2029200%2C%20options%20%5Bmss%201460%2CsackOK%2CTS%20val%20619989005%20ecr%200%2Cnop%2Cwscale%207%5D%2C%20length%200%0AGiven%20the%20above%20output%20we%20can%20see%20that%20the%20source%20ip%20is%2010.0.3.246%20the%20source%20port%20is%2056894%20and%20the%20destination%20ip%20is%20192.168.0.92%20with%20a%20destination%20port%20of%2022.%20This%20is%20pretty%20easy%20to%20identify%20once%20you%20understand%20the%20format%20of%20tcpdump.%20If%20you%20haven%E2%80%99t%20guessed%20the%20format%20yet%20you%20can%20break%20it%20down%20as%20follows%20src-ip.src-port%20%3E%20dest-ip.dest-port%3A%20Flags%5BS%5D%20the%20source%20is%20in%20front%20of%20the%20%3E%20and%20the%20destination%20is%20behind.%20You%20can%20think%20of%20the%20%3E%20as%20an%20arrow%20pointing%20to%20the%20destination.%0A2.packet%E7%9A%84%E7%B1%BB%E5%9E%8B%20%E4%BB%A5%E5%8F%8Apacket%E8%A7%82%E5%AF%9F%0A10.0.3.246.56894%20%3E%20192.168.0.92.22%3A%20Flags%20%5BS%5D%2C%20cksum%200xcf28%20(incorrect%20-%3E%200x0388)%2C%20seq%20682725222%2C%20win%2029200%2C%20options%20%5Bmss%201460%2CsackOK%2CTS%20val%20619989005%20ecr%200%2Cnop%2Cwscale%207%5D%2C%20length%200%0AFrom%20the%20sample%20above%20we%20can%20tell%20that%20the%20packet%20is%20a%20single%20SYN%20packet.%20We%20can%20identify%20this%20by%20the%20Flags%20%5BS%5D%20section%20of%20the%20tcpdump%20output%2C%20different%20types%20of%20packets%20have%20different%20types%20of%20flags.%20Without%20going%20too%20deep%20into%20what%20types%20of%20packets%20exist%20within%20TCP%20you%20can%20use%20the%20below%20as%20a%20cheat%20sheet%20for%20identifying%20packet%20types.%0A%5BS%5D%20-%20SYN%20(Start%20Connection)%0A%5B.%5D%20-%20No%20Flag%20Set%0A%5BP%5D%20-%20PSH%20(Push%20Data)%0A%5BF%5D%20-%20FIN%20(Finish%20Connection)%0A%5BR%5D%20-%20RST%20(Reset%20Connection)%0ADepending%20on%20the%20version%20and%20output%20of%20tcpdump%20you%20may%20also%20see%20flags%20such%20as%20%5BS.%5D%20this%20is%20used%20to%20indicate%20a%20SYN-ACK%20packet.%0AAn%20unhealthy%20example%0A15%3A15%3A43.323412%20IP%20(tos%200x0%2C%20ttl%2064%2C%20id%2051051%2C%20offset%200%2C%20flags%20%5BDF%5D%2C%20proto%20TCP%20(6)%2C%20length%2060)%0A%20%20%20%2010.0.3.246.56894%20%3E%20192.168.0.92.22%3A%20Flags%20%5BS%5D%2C%20cksum%200xcf28%20(incorrect%20-%3E%200x0388)%2C%20seq%20682725222%2C%20win%2029200%2C%20options%20%5Bmss%201460%2CsackOK%2CTS%20val%20619989005%20ecr%200%2Cnop%2Cwscale%207%5D%2C%20length%200%0A15%3A15%3A44.321444%20IP%20(tos%200x0%2C%20ttl%2064%2C%20id%2051052%2C%20offset%200%2C%20flags%20%5BDF%5D%2C%20proto%20TCP%20(6)%2C%20length%2060)%0A%20%20%20%2010.0.3.246.56894%20%3E%20192.168.0.92.22%3A%20Flags%20%5BS%5D%2C%20cksum%200xcf28%20(incorrect%20-%3E%200x028e)%2C%20seq%20682725222%2C%20win%2029200%2C%20options%20%5Bmss%201460%2CsackOK%2CTS%20val%20619989255%20ecr%200%2Cnop%2Cwscale%207%5D%2C%20length%200%0A15%3A15%3A46.321610%20IP%20(tos%200x0%2C%20ttl%2064%2C%20id%2051053%2C%20offset%200%2C%20flags%20%5BDF%5D%2C%20proto%20TCP%20(6)%2C%20length%2060)%0A%20%20%20%2010.0.3.246.56894%20%3E%20192.168.0.92.22%3A%20Flags%20%5BS%5D%2C%20cksum%200xcf28%20(incorrect%20-%3E%200x009a)%2C%20seq%20682725%0AThe%20above%20sampling%20shows%20an%20example%20of%20an%20unhealthy%20exchange%2C%20and%20by%20unhealthy%20exchange%20for%20this%20example%20that%20means%20no%20exchange.%20In%20the%20above%20sample%20we%20can%20see%20that%2010.0.3.246%20is%20sending%20a%20SYN%20packet%20to%20host%20192.168.0.92%20however%20we%20never%20see%20a%20response%20from%20host%20192.168.0.92.%0AA%20healthy%20example%0A15%3A18%3A25.716453%20IP%20(tos%200x10%2C%20ttl%2064%2C%20id%2053344%2C%20offset%200%2C%20flags%20%5BDF%5D%2C%20proto%20TCP%20(6)%2C%20length%2060)%0A%20%20%20%2010.0.3.246.34908%20%3E%20192.168.0.110.22%3A%20Flags%20%5BS%5D%2C%20cksum%200xcf3a%20(incorrect%20-%3E%200xc838)%2C%20seq%201943877315%2C%20win%2029200%2C%20options%20%5Bmss%201460%2CsackOK%2CTS%20val%20620029603%20ecr%200%2Cnop%2Cwscale%207%5D%2C%20length%200%0A15%3A18%3A25.716777%20IP%20(tos%200x0%2C%20ttl%2063%2C%20id%200%2C%20offset%200%2C%20flags%20%5BDF%5D%2C%20proto%20TCP%20(6)%2C%20length%2060)%0A%20%20%20%20192.168.0.110.22%20%3E%2010.0.3.246.34908%3A%20Flags%20%5BS.%5D%2C%20cksum%200x594a%20(correct)%2C%20seq%204001145915%2C%20ack%201943877316%2C%20win%205792%2C%20options%20%5Bmss%201460%2CsackOK%2CTS%20val%2018495104%20ecr%20620029603%2Cnop%2Cwscale%202%5D%2C%20length%200%0A15%3A18%3A25.716899%20IP%20(tos%200x10%2C%20ttl%2064%2C%20id%2053345%2C%20offset%200%2C%20flags%20%5BDF%5D%2C%20proto%20TCP%20(6)%2C%20length%2052)%0A%20%20%20%2010.0.3.246.34908%20%3E%20192.168.0.110.22%3A%20Flags%20%5B.%5D%2C%20cksum%200xcf32%20(incorrect%20-%3E%200x9dcc)%2C%20ack%201%2C%20win%20229%2C%20options%20%5Bnop%2Cnop%2CTS%20val%20620029603%20ecr%2018495104%5D%2C%20length%200%0AA%20healthy%20example%20would%20look%20like%20the%20above%2C%20in%20the%20above%20we%20can%20see%20a%20standard%20TCP%203-way%20handshake.%20The%20first%20packet%20above%20is%20a%20SYN%20packet%20from%20host%2010.0.3.246%20to%20host%20192.168.0.110%2C%20the%20second%20packet%20is%20a%20SYN-ACK%20from%20host%20192.168.0.110%20acknowledging%20the%20SYN.%20The%20final%20packet%20is%20a%20ACK%20or%20rather%20a%20SYN-ACK-ACK%20from%20host%2010.0.3.246%20acknowledging%20that%20it%20has%20received%20the%20SYN-ACK.%20From%20this%20point%20on%20there%20is%20an%20established%20TCP%2FIP%20connection.%0A3.%E7%94%A8Hex%E5%92%8CASCII%20%E6%9F%A5%E7%9C%8B%E6%95%B0%E6%8D%AE%E5%8C%85%20%0Atcpdump%20-nvvv%20-i%20any%20-c%201%20-XX%20’port%2080%20and%20host%2010.0.3.1’%20%20%20-XX%20flag%20to%20print%20the%20packet%20data%20in%20hex%20and%20ascii%0Atcpdump%20-nvvv%20-i%20any%20-c%201%20-A%20’port%2080%20and%20host%2010.0.3.1’%20-A%20Printing%20packet%20data%20in%20ASCII%20only%0A%E9%80%9A%E8%BF%87%E8%BF%99%E7%A7%8D%E4%BA%BA%E5%8F%AF%E8%AF%BB%E5%8F%96%E7%9A%84%E6%A0%BC%E5%BC%8F%E6%96%B9%E4%BE%BF%E6%8E%92%E6%9F%A5%E9%97%AE%E9%A2%98%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%AF%B9%E4%BA%8E%E5%8A%A0%E5%AF%86%E7%9A%84%E6%95%B0%E6%8D%AE%EF%BC%8C%20%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87ssldump%E5%92%8Cwireshark%E3%80%82However%2C%20if%20you%20use%20have%20the%20certificates%20in%20use%20you%20could%20use%20commands%20such%20as%20ssldump%20or%20even%20wireshark.%0Atcpdump%20%0Ahttp%3A%2F%2Fbencane.com%2F2014%2F10%2F13%2Fquick-and-practical-reference-for-tcpdump%2F%0AA%20Quick%20and%20Practical%20Reference%20for%20tcpdump%0Atcpdump%20-n%20%E4%B8%8D%E5%B8%A6hostname%0Atcpdump%20-vvv%20%20%20%20%20%20%20-vvv%E6%98%BE%E7%A4%BA%E6%9B%B4%E5%A4%9A%E7%9A%84ip%20packets%E4%BF%A1%E6%81%AF%0Atcpdump%20-vvvv%20-c%201%20%20-c%E6%98%BE%E7%A4%BA%E7%9A%84%E8%A1%8C%E6%95%B0%0Atcpdump%20-i%20eth0%20%20%20%20%20-i%20%E6%8C%87%E5%AE%9A%E7%9B%91%E6%8E%A7%E7%9A%84%E7%BD%91%E5%8D%A1%E4%BF%A1%E6%81%AF%EF%BC%8C%E9%BB%98%E8%AE%A4%E6%98%AF%E5%8F%96eth0%EF%BC%8C%20tcpdump%20-i%20any%20%20any%E6%98%AF%E7%9B%91%E6%8E%A7%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84%E6%89%80%E6%9C%89%E7%9A%84%E7%BD%91%E5%8D%A1%0Atcpdump%20-w%20%2Fpath%2Fto%2Ffile%20%20-w%E4%BF%9D%E5%AD%98%E5%88%B0%E6%96%87%E4%BB%B6%E4%B8%AD%0Atcpdump%20-r%20%2Fpath%2Fto%2Ffile%20%20-r%E4%BB%8E%E6%96%87%E4%BB%B6%E4%B8%AD%E8%AF%BB%E5%8F%96%0Atcpdump%20-s%20100%20-s%E6%8C%87%E5%AE%9Apacket%E7%9A%84%E9%95%BF%E5%BA%A6%EF%BC%8C%E9%BB%98%E8%AE%A4%E6%98%AF65535%20bytes%0Afilter%0Atcpdump%20-nvvv%20-i%20any%20-c%203%20host%2010.0.3.1%20%20%E8%BF%87%E6%BB%A4host%3D10.0.3.1%E7%9A%84%E6%B5%81%E9%87%8F%0Atcpdump%20-nvvv%20-i%20any%20-c%203%20src%20host%2010.0.3.1%20%E5%8F%AA%E6%98%BE%E7%A4%BAsource%20host%E7%9A%84%E6%B5%81%E9%87%8F%0Atcpdump%20-nvvv%20-i%20any%20-c%203%20port%2022%20and%20port%2060738%20%E6%AF%94%E8%BE%83%E6%9C%89%E7%94%A8%EF%BC%8C%E5%8F%AA%E6%8A%93%E5%8F%96port22%E5%92%8C60738%E4%B9%8B%E9%97%B4%E7%9A%84%E6%B5%81%E9%87%8F%EF%BC%8C%E6%88%96%E8%80%85tcpdump%20-nvvv%20-i%20any%20-c%203%20’port%2022%20%26%26%20port%2060738’%0Atcpdump%20-nvvv%20-i%20any%20-c%2020%20’port%2080%20or%20port%20443’%20%E6%88%96%20%20tcpdump%20-nvvv%20-i%20any%20-c%2020%20’port%2080%20or%20port%20443’%0Atcpdump%20-nvvv%20-i%20any%20-c%2020%20’(port%2080%20or%20port%20443)%20and%20host%2010.0.3.169’%0Atcpdump%20-nvvv%20-i%20any%20-c%2020%20’((port%2080%20or%20port%20443)%20and%20(host%2010.0.3.169%20or%20host%2010.0.3.1))%20and%20dst%20host%2010.0.3.246’%20%E9%80%9A%E8%BF%87%E6%8B%AC%E5%8F%B7%E6%9D%A5%E5%AE%9E%E7%8E%B0%E8%BF%90%E7%AE%97%E7%9A%84%E4%BC%98%E5%85%88%E7%BA%A7%E5%88%AB%0A%E6%8A%93%E5%8F%96%E5%85%B6%E4%BB%96%E5%8D%8F%E8%AE%AEICMP%2C%20UDP%2C%20and%20ARP%20%E7%9A%84%E5%8C%85%0A%E8%A7%82%E5%AF%9F%0A1.srcip%E5%92%8Cport%20%20%20dstip%E5%92%8Cport%0A10.0.3.246.56894%20%3E%20192.168.0.92.22%3A%20Flags%20%5BS%5D%2C%20cksum%200xcf28%20(incorrect%20-%3E%200x0388)%2C%20seq%20682725222%2C%20win%2029200%2C%20options%20%5Bmss%201460%2CsackOK%2CTS%20val%20619989005%20ecr%200%2Cnop%2Cwscale%207%5D%2C%20length%200%0AGiven%20the%20above%20output%20we%20can%20see%20that%20the%20source%20ip%20is%2010.0.3.246%20the%20source%20port%20is%2056894%20and%20the%20destination%20ip%20is%20192.168.0.92%20with%20a%20destination%20port%20of%2022.%20This%20is%20pretty%20easy%20to%20identify%20once%20you%20understand%20the%20format%20of%20tcpdump.%20If%20you%20haven%E2%80%99t%20guessed%20the%20format%20yet%20you%20can%20break%20it%20down%20as%20follows%20src-ip.src-port%20%3E%20dest-ip.dest-port%3A%20Flags%5BS%5D%20the%20source%20is%20in%20front%20of%20the%20%3E%20and%20the%20destination%20is%20behind.%20You%20can%20think%20of%20the%20%3E%20as%20an%20arrow%20pointing%20to%20the%20destination.%0A2.packet%E7%9A%84%E7%B1%BB%E5%9E%8B%20%E4%BB%A5%E5%8F%8Apacket%E8%A7%82%E5%AF%9F%0A10.0.3.246.56894%20%3E%20192.168.0.92.22%3A%20Flags%20%5BS%5D%2C%20cksum%200xcf28%20(incorrect%20-%3E%200x0388)%2C%20seq%20682725222%2C%20win%2029200%2C%20options%20%5Bmss%201460%2CsackOK%2CTS%20val%20619989005%20ecr%200%2Cnop%2Cwscale%207%5D%2C%20length%200%0AFrom%20the%20sample%20above%20we%20can%20tell%20that%20the%20packet%20is%20a%20single%20SYN%20packet.%20We%20can%20identify%20this%20by%20the%20Flags%20%5BS%5D%20section%20of%20the%20tcpdump%20output%2C%20different%20types%20of%20packets%20have%20different%20types%20of%20flags.%20Without%20going%20too%20deep%20into%20what%20types%20of%20packets%20exist%20within%20TCP%20you%20can%20use%20the%20below%20as%20a%20cheat%20sheet%20for%20identifying%20packet%20types.%0A%5BS%5D%20-%20SYN%20(Start%20Connection)%0A%5B.%5D%20-%20No%20Flag%20Set%0A%5BP%5D%20-%20PSH%20(Push%20Data)%0A%5BF%5D%20-%20FIN%20(Finish%20Connection)%0A%5BR%5D%20-%20RST%20(Reset%20Connection)%0ADepending%20on%20the%20version%20and%20output%20of%20tcpdump%20you%20may%20also%20see%20flags%20such%20as%20%5BS.%5D%20this%20is%20used%20to%20indicate%20a%20SYN-ACK%20packet.%0AAn%20unhealthy%20example%0A15%3A15%3A43.323412%20IP%20(tos%200x0%2C%20ttl%2064%2C%20id%2051051%2C%20offset%200%2C%20flags%20%5BDF%5D%2C%20proto%20TCP%20(6)%2C%20length%2060)%0A%20%20%20%2010.0.3.246.56894%20%3E%20192.168.0.92.22%3A%20Flags%20%5BS%5D%2C%20cksum%200xcf28%20(incorrect%20-%3E%200x0388)%2C%20seq%20682725222%2C%20win%2029200%2C%20options%20%5Bmss%201460%2CsackOK%2CTS%20val%20619989005%20ecr%200%2Cnop%2Cwscale%207%5D%2C%20length%200%0A15%3A15%3A44.321444%20IP%20(tos%200x0%2C%20ttl%2064%2C%20id%2051052%2C%20offset%200%2C%20flags%20%5BDF%5D%2C%20proto%20TCP%20(6)%2C%20length%2060)%0A%20%20%20%2010.0.3.246.56894%20%3E%20192.168.0.92.22%3A%20Flags%20%5BS%5D%2C%20cksum%200xcf28%20(incorrect%20-%3E%200x028e)%2C%20seq%20682725222%2C%20win%2029200%2C%20options%20%5Bmss%201460%2CsackOK%2CTS%20val%20619989255%20ecr%200%2Cnop%2Cwscale%207%5D%2C%20length%200%0A15%3A15%3A46.321610%20IP%20(tos%200x0%2C%20ttl%2064%2C%20id%2051053%2C%20offset%200%2C%20flags%20%5BDF%5D%2C%20proto%20TCP%20(6)%2C%20length%2060)%0A%20%20%20%2010.0.3.246.56894%20%3E%20192.168.0.92.22%3A%20Flags%20%5BS%5D%2C%20cksum%200xcf28%20(incorrect%20-%3E%200x009a)%2C%20seq%20682725%0AThe%20above%20sampling%20shows%20an%20example%20of%20an%20unhealthy%20exchange%2C%20and%20by%20unhealthy%20exchange%20for%20this%20example%20that%20means%20no%20exchange.%20In%20the%20above%20sample%20we%20can%20see%20that%2010.0.3.246%20is%20sending%20a%20SYN%20packet%20to%20host%20192.168.0.92%20however%20we%20never%20see%20a%20response%20from%20host%20192.168.0.92.%0AA%20healthy%20example%0A15%3A18%3A25.716453%20IP%20(tos%200x10%2C%20ttl%2064%2C%20id%2053344%2C%20offset%200%2C%20flags%20%5BDF%5D%2C%20proto%20TCP%20(6)%2C%20length%2060)%0A%20%20%20%2010.0.3.246.34908%20%3E%20192.168.0.110.22%3A%20Flags%20%5BS%5D%2C%20cksum%200xcf3a%20(incorrect%20-%3E%200xc838)%2C%20seq%201943877315%2C%20win%2029200%2C%20options%20%5Bmss%201460%2CsackOK%2CTS%20val%20620029603%20ecr%200%2Cnop%2Cwscale%207%5D%2C%20length%200%0A15%3A18%3A25.716777%20IP%20(tos%200x0%2C%20ttl%2063%2C%20id%200%2C%20offset%200%2C%20flags%20%5BDF%5D%2C%20proto%20TCP%20(6)%2C%20length%2060)%0A%20%20%20%20192.168.0.110.22%20%3E%2010.0.3.246.34908%3A%20Flags%20%5BS.%5D%2C%20cksum%200x594a%20(correct)%2C%20seq%204001145915%2C%20ack%201943877316%2C%20win%205792%2C%20options%20%5Bmss%201460%2CsackOK%2CTS%20val%2018495104%20ecr%20620029603%2Cnop%2Cwscale%202%5D%2C%20length%200%0A15%3A18%3A25.716899%20IP%20(tos%200x10%2C%20ttl%2064%2C%20id%2053345%2C%20offset%200%2C%20flags%20%5BDF%5D%2C%20proto%20TCP%20(6)%2C%20length%2052)%0A%20%20%20%2010.0.3.246.34908%20%3E%20192.168.0.110.22%3A%20Flags%20%5B.%5D%2C%20cksum%200xcf32%20(incorrect%20-%3E%200x9dcc)%2C%20ack%201%2C%20win%20229%2C%20options%20%5Bnop%2Cnop%2CTS%20val%20620029603%20ecr%2018495104%5D%2C%20length%200%0AA%20healthy%20example%20would%20look%20like%20the%20above%2C%20in%20the%20above%20we%20can%20see%20a%20standard%20TCP%203-way%20handshake.%20The%20first%20packet%20above%20is%20a%20SYN%20packet%20from%20host%2010.0.3.246%20to%20host%20192.168.0.110%2C%20the%20second%20packet%20is%20a%20SYN-ACK%20from%20host%20192.168.0.110%20acknowledging%20the%20SYN.%20The%20final%20packet%20is%20a%20ACK%20or%20rather%20a%20SYN-ACK-ACK%20from%20host%2010.0.3.246%20acknowledging%20that%20it%20has%20received%20the%20SYN-ACK.%20From%20this%20point%20on%20there%20is%20an%20established%20TCP%2FIP%20connection.%0A3.%E7%94%A8Hex%E5%92%8CASCII%20%E6%9F%A5%E7%9C%8B%E6%95%B0%E6%8D%AE%E5%8C%85%20%0Atcpdump%20-nvvv%20-i%20any%20-c%201%20-XX%20’port%2080%20and%20host%2010.0.3.1’%20%20%20-XX%20flag%20to%20print%20the%20packet%20data%20in%20hex%20and%20ascii%0Atcpdump%20-nvvv%20-i%20any%20-c%201%20-A%20’port%2080%20and%20host%2010.0.3.1’%20-A%20Printing%20packet%20data%20in%20ASCII%20only%0A%E9%80%9A%E8%BF%87%E8%BF%99%E7%A7%8D%E4%BA%BA%E5%8F%AF%E8%AF%BB%E5%8F%96%E7%9A%84%E6%A0%BC%E5%BC%8F%E6%96%B9%E4%BE%BF%E6%8E%92%E6%9F%A5%E9%97%AE%E9%A2%98%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%AF%B9%E4%BA%8E%E5%8A%A0%E5%AF%86%E7%9A%84%E6%95%B0%E6%8D%AE%EF%BC%8C%20%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87ssldump%E5%92%8Cwireshark%E3%80%82However%2C%20if%20you%20use%20have%20the%20certificates%20in%20use%20you%20could%20use%20commands%20such%20as%20ssldump%20or%20even%20wireshark.%0A
