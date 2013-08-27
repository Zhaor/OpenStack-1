维护、故障和调试
================================


云控制器和存储代理的故障维护

计算节点故障维护

存储节点故障维护

处理一个系统整体故障

配置管理

硬件相关任务

数据库

HDWMY

故障查找

云运行过程中的停机时间，不管计划内或计划外，有时是必然的。本章的目的是提供有用的信息来主动处理，或被动式处理这些事件。

云控制器和存储代理的故障维护

计划内维护

重启云控制器或存储代理

云控制器或存储代理重启后的工作

云控制器失效

云控制器和存储代理在预期和非预期的停机对系统的影响类似。在云系统中通常每种服务器有一台，因此如果停机会造成严重影响。

对云控制器，好消息是如果你的云是使用FlatDHCP multi-host 网络模式，当云控制器是离线，现有实例和卷将会继续运行。然而对于存储代理，在它恢复运行之前，不可能有存储数据流量。

 计划维护停机
计划停机的一种方法是在非工作时间做，例如凌晨1或2点，这样会影响更少的用户。如果你的云控制器或存储代理太重要了，已经无法在任何时间点停机，你必须考虑高可用性选项。

 重启一个云控制器或存储代理
总体来说，就是运行"reboot"，命令。操作系统会关闭服务并重新启动。如果想非常小心，可以在启动前备份你的工作。

 云控制器或存储代理重启后的工作
云控制器或存储代理重启后，要确认所有的服务已经启动：

# ps aux | grep nova-
# grep AMQP /var/log/nova/nova-*.log
# ps aux | grep glance-
# ps aux | grep keystone
# ps aux | grep cinder
并确保各服务功能正常:

# source openrc
# glance index
# nova list
# keystone tenant-list
对于存储代理，保证对象存储服务启动：

# ps aux | grep swift
检查其功能:

# swift stat
 云控制器失效
这是一个严重问题。云控制器是非常重要的。如果只有一个云控制器而且失效，很多服务会丢失。

为了防止此问题发生，可以建立一个云控制器的集群。相关资料可以在这里找到： OpenStack High Availability Guide (http://docs.openstack.org/trunk/openstack-ha/content/ch-intro.html).

第二种办法是用一个配置管理工具例如Puppet，来自动创建一个云控制器。如果有现成的硬件，这需要不到十五分钟的时间。在云控制器被创建后，回复备份的数据。 (请查看Backup and Recovery 一章)。

在实践中，云控制器经过长时间的离线再启动，有时nova计算服务计算节点在连接rabbitmq时不太顺利。这时可以重启相关nova服务计算节点。

 计算节点故障维护

计划维护停机  

计算节点重启后工作

实例

从一个失效实例中检查和恢复数据

卷

计算节点完全失效

/var/lib/nova/instances

有时一个计算节点要么意外崩溃或需要重启来维护。

 计划维护停机
如果你由于计划维护(如软件或硬件升级)需要重新启动一个计算节点，首先确保所有托管实例被移除此节点。如果你的云是用共享存储，使用nova live-migration 命令。首先,获得一组需要移动的实例:

# nova list --host c01.example.com --all-tenants
接着一个一个地迁移:

# nova live-migration <uuid> c02.example.com
如果你没有使用共享存储，您可以使用： --block 迁移选项：

# nova live-migration --block-migrate <uuid> c02.example.com
迁移所有的实例后，确保nova-compute 计算服务已停止:

# stop nova-compute
如果你使用一个配置管理系统，如puppet，这保证了nova-compute计算服务持续运行，你可以暂时移走init文件：

# mkdir /root/tmp
# mv /etc/init/nova-compute.conf /root/tmp
# mv /etc/init.d/nova-compute /root/tmp
然后，将计算节点shutdown，做维护工作，然后重新启动。你可以重新启动nova-compute 服务，先将文件复原：

# mv /root/tmp/nova-compute.conf /etc/init
# mv /root/tmp/nova-compute /etc/init.d/
然后启动nova-compute:

# start nova-compute
你还可以将计算实例迁移回原来的计算节点。

 计算节点启动后
当你重新启动一个计算节点,首先确认它引导成功。这包括确保nova-compute 计算服务正在运行：

# ps aux | grep nova-compute
# status nova-compute
并确认其连接到了AMQP server:

# grep AMQP /var/log/nova/nova-compute
2013-02-26 09:51:31 12427 INFO nova.openstack.common.rpc.common [-] Connected to AMQP server on 199.116.232.36:5672
在计算节点成功运行后，你必须使上面的实例运行。根据服务等级协议，你要启动每个实例，并保证它们都正常运行。

 实例
以下命令列出在某个计算节点的所有实例:

# nova list --host c01.example.com --all-tenants
有了实例的列表后，可以一个个启动：

# nova reboot <uuid>
 Note
 
当实例被不正常关机后，重启时可能出现问题。这时可以使用dashboard的vnc 界面来做故障排查。
 

如果一个实例无法启动， 用 virsh list 命令无法看到，可以采用以下命令：

# tail -f /var/log/nova/nova-compute.log
这时重新运行nova reboot，你可能发现实例不能启动的错误信息。

很多时候，这个错误主要是 libvirt's XML file (/etc/libvirt/qemu/instance-xxxxxxxx.xml) 消失了。你可以重新创建XML文件并重启实例： 

# nova reboot --hard <uuid>
从故障的实例检查和恢复数据
某些情况下，实例在运行，但无法通过ssh连接，不能对命令做出相应。VNC 界面可以显示其为启动失败或内核挂起的错误信息。 这可能是实例的文件系统被破坏。如果你需要做恢复，或检查实例的内容，可以用 qemu-nbd 来挂载硬盘。

 Note
 
如果你想查看别人的数据信息，需要征得别人的同意！
 

为了访问实例的硬盘(/var/lib/nova/instances/instance-xxxxxx/disk), 须采用以下步骤:

1.   使用virsh命令暂停该实例

2.   连接设备到磁盘qemu nbd

3.   挂载qemu nbd设备

4.   检查后卸载qemu nbd设备设备

5.   断开qemu nbd

6.   恢复实例

如果你不做步骤4-6, OpenStack 将无法继续管理这个实例。它将对所有的命令不响应，并被标为关机。

 

当硬盘文件被加载，你可以访问它并将其作为一个普通的文件夹，内有文件的结构。但是，我们不建议修改任何文件，因为这会改掉acl， 造成实例无法启动。 

1.  将实例关闭，并记下内部ID.

2.  root@compute-node:~# virsh list
3.  Id Name                 State
4.  ----------------------------------
5.  1 instance-00000981    running
6.  2 instance-000009f5    running 
7.  30 instance-0000274a    running
8.                    
9.  root@compute-node:~# virsh suspend 30
Domain 30 suspended
10. 连接 qemu-nbd 设备到硬盘

11. root@compute-node:/var/lib/nova/instances/instance-0000274a# ls -lh
12. total 33M
13. -rw-rw---- 1 libvirt-qemu kvm  6.3K Oct 15 11:31 console.log
14. -rw-r--r-- 1 libvirt-qemu kvm   33M Oct 15 22:06 disk
15. -rw-r--r-- 1 libvirt-qemu kvm  384K Oct 15 22:06 disk.local
16. -rw-rw-r-- 1 nova         nova 1.7K Oct 15 11:30 libvirt.xml
root@compute-node:/var/lib/nova/instances/instance-0000274a# qemu-nbd -c /dev/nbd0 `pwd`/disk             
17. 挂载 qemu-nbd 设备.

qemu-nbd设备会将实例硬盘的不同分区export成不同的硬盘设备。例如， 如果vda 硬盘上，vda1是启动分区，那么 qemu-nbd 将他们exports 成： /dev/nbd0，/dev/nbd0p1.

#mount the root partition of the device
root@compute-node:/var/lib/nova/instances/instance-0000274a# mount /dev/nbd0p1 /mnt/
# List the directories of mnt, and the vm's folder is display
# You can inspect the folders and access the /var/log/ files
为了检查其他的临时硬盘分区，用另外的挂载点来挂载不同分区。

# umount /mnt
# qemu-nbd -c /dev/nbd1 `pwd`/disk.local
# mount /dev/nbd1 /mnt/
root@compute-node:/var/lib/nova/instances/instance-0000274a# ls -lh /mnt/
total 76K
lrwxrwxrwx.  1 root root    7 Oct 15 00:44 bin -> usr/bin
dr-xr-xr-x.  4 root root 4.0K Oct 15 01:07 boot
drwxr-xr-x.  2 root root 4.0K Oct 15 00:42 dev
drwxr-xr-x. 70 root root 4.0K Oct 15 11:31 etc
drwxr-xr-x.  3 root root 4.0K Oct 15 01:07 home
lrwxrwxrwx.  1 root root    7 Oct 15 00:44 lib -> usr/lib
lrwxrwxrwx.  1 root root    9 Oct 15 00:44 lib64 -> usr/lib64
drwx------.  2 root root  16K Oct 15 00:42 lost+found
drwxr-xr-x.  2 root root 4.0K Feb  3  2012 media
drwxr-xr-x.  2 root root 4.0K Feb  3  2012 mnt
drwxr-xr-x.  2 root root 4.0K Feb  3  2012 opt
drwxr-xr-x.  2 root root 4.0K Oct 15 00:42 proc
dr-xr-x---.  3 root root 4.0K Oct 15 21:56 root
drwxr-xr-x. 14 root root 4.0K Oct 15 01:07 run
lrwxrwxrwx.  1 root root    8 Oct 15 00:44 sbin -> usr/sbin
drwxr-xr-x.  2 root root 4.0K Feb  3  2012 srv
drwxr-xr-x.  2 root root 4.0K Oct 15 00:42 sys
drwxrwxrwt.  9 root root 4.0K Oct 15 16:29 tmp
drwxr-xr-x. 13 root root 4.0K Oct 15 00:44 usr
drwxr-xr-x. 17 root root 4.0K Oct 15 00:44 var
18. 当做完检查，要卸载硬盘并释放qemu-nbd设备

19. root@compute-node:/var/lib/nova/instances/instance-0000274a# umount /mnt
20. root@compute-node:/var/lib/nova/instances/instance-0000274a# qemu-nbd -d /dev/nbd0
/dev/nbd0 disconnected
21. 重启实例

22. root@compute-node:/var/lib/nova/instances/instance-0000274a# virsh list
23. Id Name                 State
24. ----------------------------------
25. 1 instance-00000981    running
26. 2 instance-000009f5    running
27. 30 instance-0000274a    paused
28.                   
29. root@compute-node:/var/lib/nova/instances/instance-0000274a# virsh resume 30
Domain 30 resumed
 卷
如果相关实例有附加卷，首先生成列表和卷UUID：

mysql> select nova.instances.uuid as instance_uuid, cinder.volumes.id as volume_uuid, cinder.volumes.status, 
cinder.volumes.attach_status, cinder.volumes.mountpoint, cinder.volumes.display_name from cinder.volumes
inner join nova.instances on cinder.volumes.instance_uuid=nova.instances.uuid 
 where nova.instances.host = 'c01.example.com';
结果类似这样:

+---------------+-------------+--------+---------------+------------+--------------+ 
| instance_uuid | volume_uuid | status | attach_status | mountpoint | display_name | 
+---------------+-------------+--------+---------------+------------+--------------+ 
| 9b969a05      | 1f0fbf36    | in-use | attached      | /dev/vdc   | test         | 
+---------------+-------------+--------+---------------+------------+--------------+ 
1 row in set (0.00 sec)
接着，手动分离和重新连接卷：

# nova volume-detach <instance_uuid> <volume_uuid>
# nova volume-attach <instance_uuid> <volume_uuid> /dev/vdX
上面的X是正确的挂载点。在做以上动作前保证实例能成功启动并能login。

 计算节点完全失效
如果一个计算节点出现故障而无法复原，可以重新启动相关实例，如果实例是存储在共享存储中： /var/lib/nova/instances.

首先通过以下命令生成一个在故障节点上的实例的列表包括UUID：

mysql> select uuid from instances where host = 'c01.example.com' and deleted = 0;
接着，告诉nova，所有原来在01.example.com 上的实例现在转移到：c02.example.com:

mysql> update instances set host = 'c02.example.com' where host = 'c01.example.com' and deleted = 0;
然后，用Nova 命令启动原来c01.example.com上的所有实例，同时重新生成相关XML文件:

# nova reboot --hard <uuid>
最后，利用在 After a compute node Reboots.一节中的命令重新挂载相关卷。

 /var/lib/nova/instances
在谈及计算节点故障时这个目录需要提一下。这个目录包含在此主机上的实例的libvirt KVM 文件类型的硬盘镜像。如果你没有使用共享存储，这个目录在所有计算节点上是相同的。

/var/lib/nova/instances 包括两类子目录：

第一类是 _base 目录。这里包括所有的在这个计算节点的实例从glance缓存过来的镜像文件。文件名最后为_20（或其他数字）的是临时性基础镜像。

另一个目录是 instance-xxxxxxxx. 这些目录对应在本计算节点上运行的实例。里面的文件和_base目录下的文件对应。 它们基本上就是_base目录里文件的修改。

在/var/lib/nova/instances 里面的字目录和文件都是唯一命名的。在 _base 目录里的文件名字是基于glance镜像，而目录名instance-xxxxxxxx 则针对于每个实例。例如，如果将/var/lib/nova/instances 目录下数据copy到另外一台计算节点，你不用担心任何文件被覆盖损坏，因为名字一样的文件其实就是同一个文件。

虽然这个方法没有文档支持，你可以用，在计算节点完全失效的情况下，如果实例是在本地保存的。

 存储节点故障和维护

重新启动存储节点

关闭存储节点

更换Swift硬盘

由于对象存储的容余高可用性，因此修复存储的问题要比计算节点的问题容易。

 重启一个存储节点
If a storage node requires a reboot, simply reboot it. Requests for data hosted on that node are redirected to other copies while the server is rebooting.

 Shutting Down a Storage Node
I 如果您需要关闭一个存储节点持续一段时间(1 +天)，请考虑删除此存储节点。例如:

# swift-ring-builder account.builder remove <ip address of storage node>
# swift-ring-builder container.builder remove <ip address of storage node>
# swift-ring-builder object.builder remove <ip address of storage node>
# swift-ring-builder account.builder rebalance
# swift-ring-builder container.builder rebalance
# swift-ring-builder object.builder rebalance   
然后，重新分布ring文件到其他的节点： 

# for i in s01.example.com s02.example.com s03.example.com
> do
> scp *.ring.gz $i:/etc/swift
> done    
这样就将受影响的存储节点移出集群。

当这个节点被修复能加入集群，只需要将其加入到ring中。采用swift-ring-builder语法的细节需要参考在建立集群的时候的选项。

 更换一个swift硬盘
If a hard drive fails in a Object Storage node, replacing it is relatively easy. This assumes that your Object Storage environment is configured correctly where the data that is stored on the failed drive is also replicated to other drives in the Object Storage environment. 

如果在一个对象存储节点的一个硬盘出现故障，取代它的是相对容易的。这个假设你的对象存储环境是正确配置数据存储在故障硬盘，数据也被复制到对象存储环境的其他驱动器。

假设 /dev/sdb 故障.

首先，卸载硬盘：

# umount /dev/sdb
接着，物理移除硬盘，并更换为一个好的硬盘。

确保操作系统认识这个新加硬盘：

# dmesg | tail
应该能看到关于 /dev/sdb的信息.

因为在swift中不建议使用分区，因此只需要格式化整个硬盘：

# mkfs.xfs -i size=1024 /dev/sdb
最后挂载硬盘:

# mount -a
Swift能发现新的硬盘出现了，并自动将数据复制到本硬盘上来。

 处理一个全面故障

对于整体全面的故障，例如数据中心电源坏了，通常的方法是对每一个服务做出优先级，并按优先级的次序恢复。

 

Table 12.1. 服务恢复优先级列表（示例）
 
1
 内部网络连接
 
2
 存储设备恢复
 
3
 虚机的公网连接恢复
 
4
 Nova-compute, nova-network, cinder hosts
 
5
 使用虚机
 
10
 消息队列和数据库服务
 
15
 Keystone services
 
20
 cinder-scheduler
 
21
 Image Catalogue and Delivery services
 
22
 nova-scheduler services
 
98
 Cinder-api
 
99
 Nova-api services
 
100
 Dashboard node
 

使用这个例子优先级列表以确保用户受影响的服务是尽快恢复，但要在有一个稳定的环境之后。当然,尽管被列为一个行式项目，每一步都需要大量的工作。例如，刚启动数据库，您应该检查其完整性，或开始nova服务后，您应该将验证管理程序匹配数据库并修复任何不匹配的地方。

配置管理

维护一个OpenStack云要求您管理多个物理服务器，这个数字可能会随着时间的推移而增长。因为手工管理节点很容易出错，我们强烈建议您使用一个配置管理工具。这些工具自动化过程,确保你所有的节点都配置正确，鼓励你保持你的配置信息(如包和配置选项)在一个版本控制存储库。

很多配置管理工具都是可用的，并且本指南不推荐一个特定的。在OpenStack社区两个最受欢迎的是puppet(https://puppetlabs.com/)加上OpenStack Puppet模块(http://github.com/puppetlabs/puppetlabs-openstack)，和Chef(http://opscode.com/chef)加OpenStack Chef模块(https://github.com/opscode/openstack-chef-repo)。其他新配置工具包括Juju(https://juju.ubuntu.com/)Ansible(http://ansible.cc)和Salt (http://saltstack.com)，和更成熟的配置管理工具包括CFEngine(http://cfengine.com)和Bcfg2(http://bcfg2.org)。

 硬件相关工作

增加一个计算节点

增加一个对象存储节点

更换组件

类似于你的初始部署，应该确保所有的硬件在加入到生产之前是适当地作了配置。最大限度地利用硬件。

 增加一个计算节点
如果你发现你已经或即将到达你的计算资源容量限制，你应该计划添加额外的计算节点。添加更多的节点是相当容易的。添加节点的过程与当初始计算节点被部署到您的云一样：使用一个自动化部署系统引导的操作系统服务器与裸机，然后有一个配置管理系统安装和配置OpenStack计算服务。一旦计算服务已经以其他计算节点同样的方式作安装和配置，它自动加入云。云控制器通知新节点，并开始调度实例启动。

如果你的OpenStack块存储节点独立于你的计算节点，同样的程序仍然适用，因为用于服务的消息队列和轮询系统都相同。

我们建议新增计算和块存储节点时使用相同的硬件。至少,确保计算节点cpu相似，以保证能实时迁移。

 

 添加一个对象存储节点
添加一个新的对象存储节点与添加计算或块存储节点是不同的。你仍然想用自动部署和配置管理系统来做服务器最初配置。完成之后,您需要添加本地磁盘存储节点到对象存储ring。确切的命令是和用于添加初始磁盘到环相同的。在对象存储代理服务器的所有磁盘存储节点简单地重新运行这个命令。一旦完成了，平衡ring ,并复制ring文件其他存储节点。

 

[Note]
 注意：
 
如果你的新对象存储节点有不同数量的磁盘比原始节点,命令来添加新的节点是不同的比原来的命令。这些参数不同环境环境。
 

 更换组件
硬件故障在大规模部署,如云基础设施中是常见的。对此需要考虑时间节省和可用性的平衡。例如，一个对象存储集群中有坏硬盘时，可以轻松地正常运行，如果在一段时间它有足够的余量。或者,如果你计算能力没有用完，你可以考虑将实例从内存失败的主机迁移，直到你有时间去处理这个问题。

 数据库

数据库连接

性能优化

几乎所有的OpenStack组件都有一个底层数据库存储持久化信息。通常这个数据库是MySQL。正常的MySQL管理适用于这些数据库。OpenStack不对数据库做特殊的配置。基本的管理包括性能调整、高可用性、备份、恢复和修复。有关更多信息,请参见标准MySQL管理指南。

你可以使用一些技巧，例如更快速检索数据库信息，或解决数据不一致的错误。例如，一个实例终止但状态没有在数据库中更新。这些技巧在本书中讨论。

 数据库连接
搜索配置文件以查看每一个OpenStack组件所对应的数据库信息。利用 ‘sql_connection’ 或 ‘connection’:

# grep -hE "connection ?=" /etc/nova/nova.conf /etc/glance/glance-*.conf 
/etc/cinder/cinder.conf /etc/keystone/keystone.conf
    sql_connection = mysql://nova:nova@cloud.alberta.sandbox.cybera.ca/nova
    sql_connection = mysql://glance:password@cloud.example.com/glance 
    sql_connection = mysql://glance:password@cloud.example.com/glance 
    sql_connection=mysql://cinder:password@cloud.example.com/cinder 
    connection = mysql://keystone_admin:password@cloud.example.com/keystone
连接语法是 :

mysql:// <username> : <password> @ <hostname> / <database name>
 性能优化
当你的云系统增长时，MySQL的使用率也增长。如果你担心MySQL变成了性能瓶颈，就需要做优化。请参考MySQL Optimization Overview (http://dev.mysql.com/doc/refman/5.5/en/optimize-overview.html).

 HDWMY

Hourly

Daily

Weekly

Monthly

Quarterly

Semi-Annually

这里有一个在每小时，天，星期，月，年要做的事情的列表，要注意这些任务只是建议，并非强制。

 Hourly
·         检查监控系统，发现问题后修复他们。

·         检查你的要做工作的列表。

 Daily
·         检查实例的状态是否异常并研究为什麽出现异常。

·         检查是否有安全补丁并在需要时打补丁。

 Weekly
·         检查云系统利用率: 

o    用户配额

o    硬盘空间

o    镜像利用率

o    超大实例

o    网络利用率（带宽，IP）

·         检查你的报警系统是否工作正常。

 Monthly
·         检查上个月利用率的趋势。

·         检查某些用户是否要去掉。

·         检查某些运营账号是否要去掉。

 Quarterly
·         检查上个季度的利用率趋势

·         做季度利用率报告。

·         回顾和计划云系统增加组件。

·         计划做一次openstack版本升级。

 Semi-Annually
·         OpenStack升级.

·         OpenStack 升级后相关工作(服务的更新等?)

 确定哪些组件出现故障

检查新增的log

命令行方式运行

复杂状况案例

升级

OpenStack's collection of different components interact with each other strongly. For example, uploading an image requires interaction from nova-api, glance-api, glance-registry, Keystone, and potentially swift-proxy. As a result, it is sometimes difficult to determine exactly where problems lie. Assisting in this is the purpose of this section. 

OpenStack的不同组件间的交互非常多。例如，上传一个镜像需要 nova-api，glance-api,glance-registry, keystone, swift-proxy。作为结果，有些时候很难发现问题究竟在哪里。难题解决是本节的目的。

 检查新增Log
首先要看的是你正试图运行的命令相关日志文件。例如，如果nova-list命令失败，尝试查看log并重新执行命令：

终端 1:

# tail -f /var/log/nova/nova-api.log
终端 2:

# nova list
在日志文件中寻找任何错误痕迹。有关更多信息，请参见本章的日志和监控部分。

如果信息表明错误是由另一个组件引起的，就转到相关组件的log文件。例如，如果nova不能访问glance，查看glance-api的 log: 

终端 1:

# tail -f /var/log/glance/api.log
终端 2:

# nova list
不断反复检查直到发现问题的核心 。

 使用命令行运行
有些时候错误信息不一定在log文件中出现。这种情况下，采用不同的方式，或者用命令行运行这些服务。例如，如果glance-api服务不能启动，可以尝试用命令行： 

# sudo -u glance -H glance-api
这时有可能会通过输出发现错误。

 Note
 
 -H 标志 是需要的，因为用sudo时 ，某些服务会写文件到用户的home 目录，如果不用-H，则可能无法写入。
 

 负责状况的例子
One morning, a compute node failed to run any instances. The log files were a bit vague, claiming that a certain instance was unable to be started. This ended up being a red herring because the instance was simply the first instance in alphabetical order, so it was the first instance that nova-compute would touch. 

一天早上,一个计算节点不能运行任何实例。日志文件是有点模糊，声称一个特定实例不能被启动。这其实是一个转移视线的信息，因为其实这个实例是名字排一个的实例。

进一步的故障诊断显示，libvirt是根本没有运行。如果libvirt没有运行，那么没有实例可以通过KVM进行虚拟化。在试图开始libvirt时，它会默默地停止运行。libvirt的日志没有解释为什么。

接下来，在命令行中运行libvirtd守护进程。终于出现一个有用的错误信息： 它不能连接到d - bus。像它听起来那么荒谬，libvirt，nova计算，依赖于d – bus，但不知何故d – bus停止了。只是重启d – bus，整个系统回到正轨，很快一切都恢复运转。

 升级
除了对象存储，从OpenStack一个版本升级到另一个需要大量的工作。

升级过程一般遵循以下步骤:

1.  阅读release notes和其他文档。

2.  查看不同版本的不一致性。

3.  计划一个升级的时间表，并在测试集群上测试。

4.  运行升级。

你可以在用户实例运行时进行升级。然而，这种策略可以是危险的。别忘了适当的通知你的用户，和备份。

似乎是最成功的一般顺序是:

1.  升级Openstack认证服务(keystone).

2.  升级Openstack镜像服务(glance).

3.  升级Openstack计算服务(nova) 

4.  升级Openstack块存储服务(cinder) 

对于其中的每个步骤,完成下面的子步骤：

1.  停止服务.

2.  对数据库和配置文件作一个备份.

3.  用发行版的包管理器来更新包.

4.  根据release notes更新配置文件.

5.  数据库更新.

6.  重新启动服务.

7.  检查每个组件是否工作.

可能最重要的一步，都是升级前的测试。特别是如果发布一个新版本后你立即升级，未发现的bug可能会阻碍升级过程。一些部署人员更愿意等到维护版本出现后再升级。然而，如果你有一个重大部署，您可能遵循的开发和测试的过程，从而确保在你的环境下的bug被修复。

完成升级的计算，同时保持实例运行OpenStack，您应该能够使用动态迁移，对一个节点执行更新时移动实例到另外节点。然后搬回来。然而,重要的是确保数据库更改成功，否则可能出现集群状态的不一致。

