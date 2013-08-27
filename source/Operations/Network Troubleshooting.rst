网络故障诊断与维护
====================================

Chapter 13. Network Troubleshooting

用ip a 命令检查接口状态

云系统里的网络流量

Finding a Failure in the Path

tcpdump

iptables

Network Configuration in the Database

Debugging DHCP Issues

Debugging DNS Issues

网络故障诊断有可能是一个非常艰难和令人困惑的过程。一个网络的问题会导致在云中的多个地方的问题。使用一个逻辑故障诊断过程可以帮助减轻混乱，和对网络问题做快速隔离。本章的目的是给你一些需要的信息。

 用’ip a’ 命令检查接口状态

在运行计算和网络服务节点，使用下面的命令查看接口信息，包括关于IPs,vlan，以及你的接口是否启动。

# ip a
如果你遇到任何形式的网络问题，一个好的初步检查是确保你的接口启动了。例如:

$ ip a | grep state
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br100 state UP qlen 1000
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
6: br100: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP
你可以完全忽略virbr0，这是QEMU创建的， openstack不用它。

 在云中的网络流量

如果你登录到一个实例，ping外部主机，例如google.com，ping包需要以下路线:



1.  虚机产生一个数据包并将其放到虚拟网口，例如虚机的eth0。

2.  数据包被传送到计算节点服务器的虚拟网口，例如vnet1。 你可以从/etc/libvirt/qemu/instance-xxxxxxxx.xml文件发现vnet 网卡。

3.  从vnet网卡，数据包被传送到计算节点的虚拟网桥，例如 br100. 

如果你运行 FlatDHCPManager, 在计算节点上只有一个网桥，如果你运行VlanManager, 每个VLAN有一个网桥。为查看数据包用的哪个网桥，可以：

$ brctl show
来查看 vnet网卡。还可以从nova.conf 查看flat_interface_bridge 选项。

4.  数据包传送到计算节点的主网卡。你可以通过brctl 查看此网卡，或在 nova.conf 文件的flat_interface选项看到它。

5.  当数据包到达物理网卡，它会被传送到缺省网关。现在这个包就失去主机的控制了。图中有一个缺省网关。但在multi-host的配置中， 计算节点就是它的缺省网关。

反方向就是ping的响应路线。

从这个路径，你可以看到一个包横跨了四个不同的网卡。如果任何这些网卡发生一个问题，网络就会发生问题。

查找路径故障

使用ping可以在网络路径上快速找到故障。在一个实例，首先看看你能不能ping到外部主机如google.com。如果你可以，那么网络应该没有问题。

如果你不能，尝试ping  托管实例的计算节点IP地址。如你能ping这个IP，那么问题是介于计算节点和计算节点的网关。如果你不能ping计算节点的 IP地址，问题是在实例和计算节点之间。这包括网桥，计算节点的物理NIC和实例的vnet NIC。

最后一个测试是建立第二个实例，看看这两个实例可以ping对方。如果可以，问题可能与计算节点上的防火墙有关。

 tcpdump

一个优秀的，非常深入的故障排除网络问题的方法是使用tcpdump。我们推荐沿着网络路径在好几个点使用tcpdump查找一个问题。如果你喜欢使用GUI，可以使用Wireshark (http://www.wireshark.org/).

例如，运行以下命令：

tcpdump -i any -n -v 'icmp[icmptype] = icmp-echoreply or icmp[icmptype] = icmp-echo' 

在以下情况下可以运行这个命令

1.  云系统外部的服务器

2.  计算节点

3.  计算节点上的实例

在这个例子中，这些节点有以下IP地址：

                            Instance  
                            10.0.2.24 
                            203.0.113.30
                            Compute Node  
                            10.0.0.42 
                            203.0.113.34
                            External Server 
                            1.2.3.4
                          
接下来对实例打开一个新的shell，然后ping运行tcpdump的外部主机。如果到外部服务器的网络路径是好的，您会看到类似于下面的:

在外部服务器上:

12:51:42.020227 IP (tos 0x0, ttl 61, id 0, offset 0, flags [DF], proto ICMP (1), length 84)
    203.0.113.30 > 1.2.3.4: ICMP echo request, id 24895, seq 1, length 64 
12:51:42.020255 IP (tos 0x0, ttl 64, id 8137, offset 0, flags [none], proto ICMP (1), length 84) 
    1.2.3.4 > 203.0.113.30: ICMP echo reply, id 24895, seq 1, length 64
在计算节点:

12:51:42.019519 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto ICMP (1), length 84)
    10.0.2.24 > 1.2.3.4: ICMP echo request, id 24895, seq 1, length 64
12:51:42.019519 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto ICMP (1), length 84)
    10.0.2.24 > 1.2.3.4: ICMP echo request, id 24895, seq 1, length 64
12:51:42.019545 IP (tos 0x0, ttl 63, id 0, offset 0, flags [DF], proto ICMP (1), length 84)
    203.0.113.30 > 1.2.3.4: ICMP echo request, id 24895, seq 1, length 64
12:51:42.019780 IP (tos 0x0, ttl 62, id 8137, offset 0, flags [none], proto ICMP (1), length 84)
    1.2.3.4 > 203.0.113.30: ICMP echo reply, id 24895, seq 1, length 64
12:51:42.019801 IP (tos 0x0, ttl 61, id 8137, offset 0, flags [none], proto ICMP (1), length 84)
    1.2.3.4 > 10.0.2.24: ICMP echo reply, id 24895, seq 1, length 64
12:51:42.019807 IP (tos 0x0, ttl 61, id 8137, offset 0, flags [none], proto ICMP (1), length 84)
    1.2.3.4 > 10.0.2.24: ICMP echo reply, id 24895, seq 1, length 64
在实例上:

12:51:42.020974 IP (tos 0x0, ttl 61, id 8137, offset 0, flags [none], proto ICMP (1), length 84)
 1.2.3.4 > 10.0.2.24: ICMP echo reply, id 24895, seq 1, length 64
在这里,外部服务器收到请求和发送ping回应包。在计算节点，您可以看到两个ping和ping回复成功通过。在计算节点你也可能看到重复的包，如上图所示，由于tcpdump捕获的数据包包括网桥和输出接口的。

 iptables

Nova 自动管理iptables，包括从一个计算节点的实例双向转发数据包，，转发浮动IP流量，和管理安全组规则。运行以下命令来查看当前iptables配置

# iptables-save
[Note]
 Note
 
如果你修改配置，下次重启nova网络时生效。你必须使用OpenStack来管理iptables。
 

 网络配置在数据库中

手动去关联一个floatingIP

在nova数据库中有几个表保存网络相关信息：

·         fixed_ips: 包括nova 里的每一个IP 地址。这个表通过fixed_ips.instalce_uuid 字段关联实例instance表。

·         floating_ips: 包括nova里每一个floating IP 地址。这个表通过floating_ips.fixed_ip_id字段关联到fixed_ips表。

·         instances: 没有网络方面的属性，但包含有实例的信息，这些实例使用了fixed_ip和floating_ip。

从这些表中您可以看到，一个浮动IP技术上从来没有直接关联到一个实例，它必须通过一个固定的IP。

 手动去关联一个floatingIP 
有时一个实例终止了，但floating IP没有被释放，因为数据库是处于不一致的状态，通常的去关联工具没有正常工作。为了解决这个问题，必须手动更新数据库。

首先查找出问题的实例的UUID: 

mysql> select uuid from instances where hostname = 'hostname';
接着查找这个UUID所关联的 Fixed IP:

mysql> select * from fixed_ips where instance_uuid = '<uuid>';
这时就能找到相关Floating IP 条目:

mysql> select * from floating_ips where fixed_ip_id = '<fixed_ip_id>';
最后，可以去关联 Floating IP:

mysql> update floating_ips set fixed_ip_id = NULL, host = NULL where fixed_ip_id = '<fixed_ip_id>';
还可以将此IP 移出用户地址池:

mysql> update floating_ips set project_id = NULL where fixed_ip_id = '<fixed_ip_id>';
 查找 DHCP 相关问题：

一个常见的网络问题是一个实例的靴子成功但无法访问，因为它未能从dnsmasq，发起的nova网络服务的DHCP服务器获得IP地址。

识别这个问题最简单的方法是查看您的实例控制台输出。如果DHCP失败，您可以检索控制台日志: 

$ nova console-log <instance name or uuid>
如果实例未能通过DHCP获得一个IP，控制台应该出现一些消息。例如对于Cirros 镜像，你看到输出像这样:

udhcpc (v1.17.2) started
Sending discover...
Sending discover...
Sending discover...
No lease, forking to background
starting DHCP forEthernet interface eth0 [ [1;32mOK[0;39m ]
cloud-setup: checking http://169.254.169.254/2009-04-04/meta-data/instance-id
wget: can't connect to remote host (169.254.169.254): Network is unreachable
在你建立实例的实例正常启动后，任务是找出问题在哪里。 一个DHCP问题可能是由于一个不正常的dnsmasq过程。首先，通过检查日志调试，然后重新只对该项目(租户)启动dnsmasq进程。在VLAN模式，对于每个租户有一个dnsmasq过程。一旦你重启目标dnsmasq进程，最简单的排除dnsmasq原因方法是杀死所有的dnsmasq进程，重启nova-network。作为最后的手段:

# killall dnsmasq
# restart nova-network
等待几分钟，你应该能看到新的dnsmasq进程在运行：

# ps aux | grep dnsmasq
nobody 3735 0.0 0.0 27540 1044 ? S 15:40 0:00 /usr/sbin/dnsmasq --strict-order --bind-interfaces --conf-file= 
    --domain=novalocal --pid-file=/var/lib/nova/networks/nova-br100.pid --listen-address=192.168.100.1 
    --except-interface=lo --dhcp-range=set:'novanetwork',192.168.100.2,static,120s --dhcp-lease-max=256
    --dhcp-hostsfile=/var/lib/nova/networks/nova-br100.conf --dhcp-script=/usr/bin/nova-dhcpbridge --leasefile-ro
root 3736 0.0 0.0 27512 444 ? S 15:40 0:00 /usr/sbin/dnsmasq --strict-order --bind-interfaces --conf-file= 
     --domain=novalocal --pid-file=/var/lib/nova/networks/nova-br100.pid --listen-address=192.168.100.1 
     --except-interface=lo --dhcp-range=set:'novanetwork',192.168.100.2,static,120s --dhcp-lease-max=256
     --dhcp-hostsfile=/var/lib/nova/networks/nova-br100.conf --dhcp-script=/usr/bin/nova-dhcpbridge --leasefile-ro
如果你的实例仍不能够获得IP地址，接下来要检查dnsmasq是否看到了从实例发出的DHCP请求。在运行dnsmasq进程的机器上（如果运行在mulit-host模式下就是计算主机）查看/ var / log / syslog来检查。如果是看到了正确dnsmasq请求， 并且分发了IP， 输出就类似这样：

Feb 27 22:01:36 mynode dnsmasq-dhcp[2438]: DHCPDISCOVER(br100) fa:16:3e:56:0b:6f 
Feb 27 22:01:36 mynode dnsmasq-dhcp[2438]: DHCPOFFER(br100) 192.168.100.3 fa:16:3e:56:0b:6f 
Feb 27 22:01:36 mynode dnsmasq-dhcp[2438]: DHCPREQUEST(br100) 192.168.100.3 fa:16:3e:56:0b:6f 
Feb 27 22:01:36 mynode dnsmasq-dhcp[2438]: DHCPACK(br100) 192.168.100.3 fa:16:3e:56:0b:6f test
如果你没有看到DHCP DISCOVER，那问题存在于从实例到运行dnsmasq的机器。如果你看到上面所有的输出，实例都还不能够获得IP地址，那就是包能从实例到dnsmasq主机，但没能回来。

如果你看到其他信息，例如：

Feb 27 22:01:36 mynode dnsmasq-dhcp[25435]: DHCPDISCOVER(br100) fa:16:3e:78:44:84 no address available
这有可能是dnsmasq 和/或 nova-network 相关的问题。 (对于上面的例子，问题很可能是dnsmasq 已经没有多余的fixed ip地址可分配了)。

如果dnsmasq日志不是很正常，用命令行方式查看一下dnsmasq参数，看看是否正确：

$ ps aux | grep dnsmasq
输出应该类似这样：

108 1695 0.0 0.0 25972 1000 ? S Feb26 0:00 /usr/sbin/dnsmasq -u libvirt-dnsmasq --strict-order --bind-interfaces
 --pid-file=/var/run/libvirt/network/default.pid --conf-file= --except-interface lo --listen-address 192.168.122.1
 --dhcp-range 192.168.122.2,192.168.122.254 --dhcp-leasefile=/var/lib/libvirt/dnsmasq/default.leases
 --dhcp-lease-max=253 --dhcp-no-override
nobody 2438 0.0 0.0 27540 1096 ? S Feb26 0:00 /usr/sbin/dnsmasq --strict-order --bind-interfaces --conf-file=
 --domain=novalocal --pid-file=/var/lib/nova/networks/nova-br100.pid --listen-address=192.168.100.1
 --except-interface=lo --dhcp-range=set:'novanetwork',192.168.100.2,static,120s --dhcp-lease-max=256 
 --dhcp-hostsfile=/var/lib/nova/networks/nova-br100.conf --dhcp-script=/usr/bin/nova-dhcpbridge --leasefile-ro
root 2439 0.0 0.0 27512 472 ? S Feb26 0:00 /usr/sbin/dnsmasq --strict-order --bind-interfaces --conf-file= 
 --domain=novalocal --pid-file=/var/lib/nova/networks/nova-br100.pid --listen-address=192.168.100.1 
 --except-interface=lo --dhcp-range=set:'novanetwork',192.168.100.2,static,120s --dhcp-lease-max=256 
 --dhcp-hostsfile=/var/lib/nova/networks/nova-br100.conf --dhcp-script=/usr/bin/nova-dhcpbridge --leasefile-ro
如果这个问题似乎并不与dnsmasq有关，则可使用tcpdump接口的方式来确定数据包在哪里丢失。

DHCP使用UDP协议。客户端从服务器端口68 发送到端口67上。尝试启动一个新实例，然后系统地在网卡上监控，直到你确定在哪里流量消失。使用tcpdump监听br100端口67和68： 

# tcpdump -i br100 -n port 67 or port 68
应该在接口上使用命令如“ip a ”和“brctl show“做健康检查，确保接口是开启的，并且配置的方式和你想要的一样。

调试DNS问题

如果你能够ssh连接到一个实例，但需要很长时间(例如一分钟)才出现登陆提示，那么可能是DNS问题。DNS问题可能让ssh服务器做反向DNS查找来确定你的接入IP地址。如果DNS在你的实例上不工作，必须等待DNS反向查找超时后， ssh登录流程才得以完成。

调试DNS问题，首先确保实例相关的dnsmasq能够正确解析。如果主机无法解析，实例也不行。

一个快速的方法来检查DNS是否工作的方法是， 在实例中用host名利解析一个主机名。如果DNS正常， 你应该看到：

$ host openstack.org
openstack.org has address 174.143.194.225
openstack.org mail is handled by 10 mx1.emailsrvr.com.
openstack.org mail is handled by 20 mx2.emailsrvr.com.
如果运行的是 Cirros 镜像，没有‘host’命令，可以用ping 主机名的方式查看是否能解析。如果DNS正常：

$ ping openstack.org
PING openstack.org (174.143.194.225): 56 data bytes
如果不正常：

$ ping openstack.org
ping: bad address 'openstack.org'
在一个OpenStack云，dnsmasq过程除了担当DHCP服务器外还作为实例的DNS服务器。一个不正常的dnsmasq过程可能会造成实例的DNS相关问题。如上一节中提到的，排除dnsmasq过程故障的简单方法是  kill dnsmasq过程，重启nova-network。然而，要知道这个命令对计算节点上所有人都有影响，要作为最后的手段。命令：

# killall dnsmasq
# restart nova-network
重启后检查DNS是否工作。 如果重启后还是不行，可能需要用tcpdump来追踪哪里出了问题。DNS服务器监听UDP 53端口。在bri100上应该能看到DNS请求。启动tcpdump: 

# tcpdump -i br100 -n -v udp port 53
tcpdump: listening on br100, link-type EN10MB (Ethernet), capture size 65535 bytes
然后到实例上 ping openstack.org, 应该能看到以下信息：

16:36:18.807518 IP (tos 0x0, ttl 64, id 56057, offset 0, flags [DF], proto UDP (17), length 59)
 192.168.100.4.54244 > 192.168.100.1.53: 2+ A? openstack.org. (31)
16:36:18.808285 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 75)
 192.168.100.1.53 > 192.168.100.4.54244: 2 1/0/0 openstack.org. A 174.143.194.225 (47)
