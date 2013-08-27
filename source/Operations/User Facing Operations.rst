面向用户的操作
===========================


本指南是面向操作员的，并非为了给用户提供详尽的参考，但作为一个操作员你需要对如何使用云系统有基本的了解。 本章从一个普通用户的角度来看Openstack，这帮助操作员了解客户的需求，并且在接到报故障的时候知道是用户的问题还是系统本身的问题。主要的概念覆盖了软件镜像，类型模板(flavor)，安全组，块存储和实例。


镜像


OpenStack 镜像通常可以被理解为“虚机模板”。镜像也可以被认为是标准安装介质例如ISO 镜像. 基本上，它们都含有能启动实例的启动系统文件。
 
增加镜像

有几种预制作好的镜像可以被很简单的导入镜像服务。一个最通常被加入的镜像就是CirrOS 镜像，非常小，被用来作为测试。为增加这种镜像，只需要：
# wget https://launchpad.net/cirros/trunk/0.3.0/+download/cirros-0.3.0-x86_64-disk.img 
# glance image-create --name='cirros image' --is-public=true --container-format=bare --disk-format=qcow2 < cirros-0.3.0-x86_64-disk.img
glance image-create 命令有很多选项，例如  min-disk选项对启动分区有大小要求的镜像(象windows需要比较大的分区)非常有用。为查看这些选项：
$ glance help image-create
location 选项需要特别注意。它并不复制整个镜像到Glance，而是提供镜像的原始路径。当启动一个实例的时候，Glance会到该路径加载镜像。
copy-from 选项从指定路径复制镜像到 /var/lib/glance/images 。在例子中使用STDIN 重定向也完成相同任务。
运行下述命令来查看已有镜像的详细信息：
$ glance details
 删除镜像
为了删除一个镜像，用以下命令：
$ glance image-delete <image uuid>
	注意
	删除镜像不影响基于此镜像的虚机实例或快照。
 其他命令行选项：
全部选项可以用以下命令查看：
$ glance help
或： OpenStack Image Service 命令行指南。 (http://docs.openstack.org/cli/quick-start/content/glance-cli-reference.html)
 镜像服务和数据库
唯一不被Glance 数据库存储的是镜像本身。Glance数据库有两个主要的表：
images
image_properties
通过直接对数据库的操作，SQL查询等可以得到定制化的Glance镜像的列表和报告。 技术上，可以通过操作数据库来更新镜像的属性，虽然这不是推荐的做法。
 镜像数据库查询示例 
一个有趣的例子是修改镜像列表和镜像拥有者。这可以简单地通过查找拥有者的ID来实现。以下的例子做得更多的事-会显示拥有者的名字：
$ mysql> select glance.images.id, glance.images.name, keystone.tenant.name, is_public from glance. Images inner join keystone.tenant on glance. images.owner=keystone.tenant.id;

另一个例子，显示某个镜像的详细信息:

$ mysql> select name, value from image_properties where id = <image_id>


类型模板(flavor)
在Openstack中，虚机硬件模板被称为类型模板(flavor)，包括RAM和硬盘大小，CPU核数等。标准安装后有5个缺省的类型。类型模板可以被有管理员权限的用户修改(修改的权限也可以被编辑，通过在nova-api 服务器上的/etc/nova/policy.json 文件中修改访问控制：compute_extension:flavormanage )。在系统上查看可用的类型模板：
$ nova flavor-list
+----+-----------+-----------+------+-----------+\+-------+-\+-------------+
| ID | Name      | Memory_MB | Disk | Ephemeral |/| VCPUs | /| extra_specs |
+----+-----------+-----------+------+-----------+\+-------+-\+-------------+
| 1  | m1.tiny   | 512       | 0    | 0         |/| 1     | /| {}          |
| 2  | m1.small  | 2048      | 10   | 20        |\| 1     | \| {}          |
| 3  | m1.medium | 4096      | 10   | 40        |/| 2     | /| {}          |
| 4  | m1.large  | 8192      | 10   | 80        |\| 4     | \| {}          |
| 5  | m1.xlarge | 16384     | 10   | 160       |/| 8     | /| {}          |
+----+-----------+-----------+------+-----------+\+-------+-\+-------------+
nova flavor-create 命令可以让经过授权的用户创建新类型模板。其他控制功能可以通过以下命令查看：
1	$ nova help | grep flavor.
类型模板定义了以下元素:
列	描述
ID	一个唯一的数字ID 
Name	描述性的名字。xx.size_name通常方式是不需要的，虽然有些第三方工具可能需要这么设置
Memory_MB	虚机内存(MB)
Disk	虚拟启动硬盘的大小（GB）。这是个装载启动软件的非持久化的硬盘。当从一个持久化硬盘启动的时候就不需要了。 大小为 "0" 是一个特殊的大小，表示采用和启动软件镜像相同的大小。
Ephemeral	指定第二个非持久化硬盘的大小。这是一个空的，没有被格式化的硬盘，只在虚机存在的时候存在。
Swap	虚机的可选的交换分区空间
VCPUs	虚机中虚拟CPU的核数
RXTX_Factor	此可选属性让被创建的服务器有和其带有的网络硬件有不同的带宽。这个可变因子定义RXTX(输入输出)与网络硬件带宽的比例。 缺省值是1.0，也就是说，和硬件带宽相同。
Is_Public	布尔值，类型模板是只给租户内的用户用还是可以给其他租户使用（公开）。缺省为真，即公开。
extra_specs	附加的可选项，限制哪台主机可以运行某种类型模板。采用key/value值得方式，只有有相同key/value值得主机才能运行相关类型模板。可以用来处理在特殊情况下部署，例如： 有些类型模板只能在有GPU的主机上运行。
 如何修改一个已有的类型模板?
不幸的是，OpenStack没有提供修改模板的接口，只有增加和删除。Dashboard里的模板修改的工作模式其实是删除旧模板并增加一个同名模板。

安全组
对新用户，Openstack最常见的问题是当启动一个实例，未能设置适当的安全组，之后后无法访问网络上的实例。
安全组是一组应用于一个实例的网络的IP过滤规则，是基于具体项目的，项目成员可以编辑默认的规则，或对他们组添加新规则。所有的项目如果没有其他安全组定义，都有一个“默认”安全组，并将其应用于实例。（除非改变了这个安全组，阻挡所有的进入流量？）。
nova.conf 文件中的选项 allow_same_net_traffic (缺省为true) 是在全局范围内，控制规则是否适用于共享一个网络的主机群。当设置为true，在同一子网主机之间可以相互传送所有类型数据，没有经过过滤。在Flat模式的网络，这使得所有项目中的所有实例可以未过滤的相互通信。在VLAN模式网络,在同一个项目允许实例相互访问。如果 allow_same_net_traffic 被设置为false, 安全组被部署到所有的连接，在这种情况下，还是可以模拟出 为true的效果，那就是配置缺省安全组为全部通过。
当前项目的安全组在dashboard的“访问与安全”部分找到。 为查看一个安全组的细节，在安全组下选择“编辑”。显然，可以从这个界面修改安全组。另外在主访问&安全页面有一个“创建安全组”按钮来创建新组。-- 我们这里讨论中使用的术语,与命令行的术语相同。
以下命令显示在当前项目中的安全组列表：
$ nova secgroup-list
+---------+-------------+
| Name    | Description |
+---------+-------------+
| default | default     |
| open    | all ports   |
+---------+-------------+

查看一个叫‘open’的安全组:
$ nova secgroup-list-rules open
+-------------+-----------+---------+-----------+--------------+ 
 | IP Protocol | From Port | To Port | IP Range  | Source Group | 
 +-------------+-----------+---------+-----------+--------------+ 
 | icmp        | -1        | 255     | 0.0.0.0/0 |              | 
 | tcp         | 1         | 65535   | 0.0.0.0/0 |              | 
 | udp         | 1         | 65535   | 0.0.0.0/0 |              | 
 +-------------+-----------+---------+-----------+--------------+ 

解释：所有的规则都是“允许(allow)”，因为缺省都是“拒绝(deny)”。第一列是IP协议 (icmp, tcp, 或 udp) 第二列和第三列描述端口号范围。第四列用CIDR格式描述IP地址范围。这个例子中描述允许所有IP的所有端口。
在前面章节中介绍过，规则的条数是quota_security_group_rules被控制的，而每个项目的安全组数是被quota_security_groups quota 控制的。
当增加一个安全组时，应该用一个简单又有解释性的名字。因为名字会在被使用的实例中显示但附带说明不会显示。比如一个安全组名字为‘http’ 会比较好，但‘张三的组’或‘组1’则不好理解。
作为例子，我们创建一个安全组，允许所有的从任何地方的web 流量能连接internet。 称之为 "global_http"，意思为任何地方倒internet的web流量，容易理解。 
+-------------+-------------------------------------+
| Name        | Description                         |
+-------------+-------------------------------------+
| global_http | allow web traffic from the internet |
+-------------+-------------------------------------+
用以下命令增加规则：
$ nova secgroup-add-rule <secgroup> <ip-proto> <from-port> <to-port>
                              <cidr>
$ nova secgroup-add-rule global_http tcp 80 80 0.0.0.0/0
+-------------+-----------+---------+-----------+--------------+
| IP Protocol | From Port | To Port | IP Range  | Source Group |
+-------------+-----------+---------+-----------+--------------+
| tcp         | 80        | 80      | 0.0.0.0/0 |              |
+-------------+-----------+---------+-----------+--------------+
注意：这里的‘from-port’ 和 ‘to-port’并非源端口和目的端口，而是端口的范围。复杂的规则可以通过多条规则实现。例如：如果想允许http 和 https 流量：
$ nova secgroup-add-rule global_http tcp 443 443 0.0.0.0/0
+-------------+-----------+---------+-----------+--------------+
| IP Protocol | From Port | To Port | IP Range  | Source Group |
+-------------+-----------+---------+-----------+--------------+
| tcp         | 443       | 443     | 0.0.0.0/0 |              |
+-------------+-----------+---------+-----------+--------------+
增加上一条之后，规则变为:
$ nova secgroup-list-rules global_http
+-------------+-----------+---------+-----------+--------------+
| IP Protocol | From Port | To Port | IP Range  | Source Group |
+-------------+-----------+---------+-----------+--------------+
| tcp         | 80        | 80      | 0.0.0.0/0 |              |
| tcp         | 443       | 443     | 0.0.0.0/0 |              |
+-------------+-----------+---------+-----------+--------------+
反方向操作是secgroup-delete-rule，同样的格式。如果想删除整个安全组可以用：secgroup-delete.
为一个实例的集群创建安全组：
SourceGroups 采用动态的模式定义CIDR 可访问资源。一个用户设置一个SourceGroup (有安全组名字)， 所有此用户在其他的实例上可以动态选择使用这个SourceGroup。 这种方式减轻了每个新用户需要一个新的规则的麻烦。使用方式： 
usage: nova secgroup-add-group-rule <secgroup> <source-group> <ip-proto> <from-port> <to-port>
$ nova secgroup-add-group-rule cluster global-http tcp 22 22
"cluster" 规则允许所有使用‘global-http’的实例能通过ssh 访问。

块存储
连接： 块存储创建失败
OpenStack卷是持久的块存储设备，可以附加到实例或分离，但同时只能连接到一个实例，类似于一个外部硬盘，不象网络文件系统或对象存储那样提供共享存储方式。块存储让实例中的操作系统把文件系统加载和安装到块设备。
类似于其他可移动磁盘技术，重要的是操作系统不能利用磁盘，然后马上移掉它，数据容易出问题。在Linux实例中任何文件系统在硬件移除前需要从卷中卸载。OpenStack volume 服务不知道从一个实例中移除卷是否安全，因此只能根据指令去做。如果用户告诉volume服务从一个实例卸载卷，而这个卷正在被写入，可以想象文件系统肯定会有某种程度的损坏，不管是什么进程在用这个卷。
OpenStack没有涉及实例操作系统在访问块设备时需要的步骤等等相关规则。 所涉及到的只有如何创建卷并将其挂载到实例或卸载。这些操作可以在dashboard的‘卷（volume）’页面中找到，或用cinder命令行客户端。
为了增加一个卷只需要名字和卷大小（GB），将其输入到‘创建卷’菜单，或者用命令行方式：
$ cinder create --display-name test-volume 10
创建了一个人名为‘test-volume’的卷，10GB。列出已存在的卷和所连接的实例(如果有)：
$ cinder list
+------------+---------+--------------------+------+-------------+-------------+
|     ID     | Status  |    Display Name    | Size | Volume Type | Attached to |
+------------+---------+--------------------+------+-------------+-------------+
| 0821...19f |  active |    test-volume     |  10  |     None    |             |
+------------+---------+--------------------+------+-------------+-------------+
块存储服务还允许创建快照。记住这是块级别的快照，最好在卷没有连接到实例的时候进行快照，或者卷没有被加载或使用的时候做。 如果在卷被频繁使用的时候做，可能得到的是一个不一致的文件系统。 实际上，缺省情况下，volume卷服务在卷被加载时不做快照，--你可以强迫它做。为了创建快照，一种方式是在dashboard的卷页面选择“创建快照”，或使用命令行：
usage: cinder snapshot-create [--force <True|False>]
[--display-name <display-name>]
[--display-description <display-description>]
<volume-id>

Positional arguments:  <volume-id>           ID of the volume to snapshot
Optional arguments:  --force <True|False>  Optional flag to indicate whether to snapshot a volume                        even if its attached to an instance. (Default=False)  --display-name <display-name>                        Optional snapshot name. (Default=None)
--display-description <display-description>
Optional snapshot description. (Default=None)
 块存储创建失败
如果用户试图创建卷时马上进入一个错误状态，查错最好的办法是基于卷的UUID 来查询(grep) cinder的log文件。首先尝试云控制器上的日志文件，然后试所要创建卷的存储节点: 
# grep 903b85d0-bacc-4855-a261-10843fc2d65b /var/log/cinder/*.log 

实例
启动实例
实例启动失败
实例特性数据
实例是在一个OpenStack云运行的虚拟机。本节讨论如何操作实例，相关镜像，网络特性，以及如何在数据库中显示的。
启动实例
为了发起一个实例，需要选择一个镜像，一个类型模型，和一个名字。 名字不一定需要唯一，但如果是唯一的话，你的日子会好过很多，因为很多工具用到名字代替UUID。 发起实例可以通过dashboard的“实例”页面的“发起实例(launch instance)”按钮，然后到选择镜像和快照页面。
命令行模式:
$ nova boot --flavor <flavor> --image <image> <name>
这里有些选项。在启动实例之前最好读完本节，这是最基本的命令。
$ nova delete <instance-uuid>
要注意， 将一个实例关闭电源(关机)并不代表这个实例在openstack中被去除。
 实例启动失败
如果一个实例不能启动并马上进入"Error" 状态，有几种方式来排查故障。有些只需普通用户权限有些需要能login到log服务器或计算节点。
不能启动最通常的原因， 是对配额的分配使系统中没有合适的计算节点满足实例要求。这种情况下可以用nova show 命令来查看，错误信息如下：
$ nova show test-instance
+------------------------+--------------------------------------------------------------\
| Property               | Value                                                        /
+------------------------+--------------------------------------------------------------\
| OS-DCF:diskConfig      | MANUAL                                                       /
| OS-EXT-STS:power_state | 0                                                            \
| OS-EXT-STS:task_state  | None                                                         /
| OS-EXT-STS:vm_state    | error                                                        \
| accessIPv4             |                                                              /
| accessIPv6             |                                                              \
| config_drive           |                                                              /
| created                | 2013-03-01T19:28:24Z                                         \
| fault                  | {u'message': u'NoValidHost', u'code': 500, u'created': u'2013/
| flavor                 | xxl.super (11)                                               \
| hostId                 |                                                              /
| id                     | 940f3b2f-bd74-45ad-bee7-eb0a7318aa84                         \
| image                  | quantal-test (65b4f432-7375-42b6-a9b8-7f654a1e676e)          /
| key_name               | None                                                         \
| metadata               | {}                                                           /
| name                   | test-instance                                                \
| security_groups        | [{u'name': u'default'}]                                      /
| status                 | ERROR                                                        \
| tenant_id              | 98333a1a28e746fa8c629c83a818ad57                             /
| updated                | 2013-03-01T19:28:26Z                                         \
| user_id                | a1ef823458d24a68955fec6f3d390019                             /
+------------------------+--------------------------------------------------------------\   
在本例子中看到错误信息显示为 NoValidHost 表明调度器无法满足实例的需求。
如果nova show 显示的信息不够，可以搜索相关计算节点的nova-compute.log文件中的内容，用实例的UUID检索。或者调度服务器上的nova-scheduler.log，这些都能提供底层的错误信息。
作为管理员用 nova show 命令会显示实例所在的计算节点的HostID 。但如果实例没有被调度成功，则没有这个ID。
 实例特性数据
有各种各样的方法来注入定制化的数据包括用户数据，元数据服务，授权密钥(authorized_keys)注入，和文件注入。
澄清用户数据与元数据的区别： “用户数据”是一部分数据，在实例没有运行时设置。这个用户数据可以在实例运行中存取和使用。人们使用这个用户数据来存储配置，脚本,或其他的数据，如果租户想要的话。
而元数据，是与实例相关联的一组 键/值（key/value）。在实例存在期间，当用户通过Compute API 发出指令时，nova-compute从实例的内部或外部来读写这些键/值。但是，你不能通过与EC2元数据服务兼容的方式来直接查询元数据。 
用户可以通过nova命令生成和注册ssh 密钥：
$ nova keypair-add mykey > mykey.pem
这生成了一个名字为mykey的密钥，可以关联到实例上。mykey.pem 文件是私钥，需要保存到一个安全的地方，因为它允许以root的用户访问与其关联的实例。
你可以用以下命令注册一个公钥：
$ nova keypair-add --pub-key mykey.pub mykey
你必须有相对应的私钥来访问和此公钥相关联的实例。
在一个实例启动时关联密钥：在命令行增加 --key_name mykey :
$ nova boot --image ubuntu-cloudimage --flavor 1 --key_name mykey
当启动一个服务器时，可以加上元数据，这样就能更容易地分辨运行的实例。 用 --meta 选项，带一个 key=value 对, 并确定这个键值对的定义。例如，可以增加一个描述：
$ nova boot --image=test-image --flavor=1 smallimage --meta description='Small test image'
在实例信息中可以看到元数据的信息：
$ nova show smallimage
+------------------------+-----------------------------------------+
|     Property           |                   Value                 |
+------------------------+-----------------------------------------+
|   OS-DCF:diskConfig    |               MANUAL                    |
| OS-EXT-STS:power_state |                 1                       |
| OS-EXT-STS:task_state  |                None                     |
|  OS-EXT-STS:vm_state   |               active                    |
|    accessIPv4          |                                         |
|    accessIPv6          |                                         |
|      config_drive      |                                         |
|     created            |            2012-05-16T20:48:23Z         |
|      flavor            |              m1.small                   |
|      hostId            |             de0...487                   |
|        id              |             8ec...f915                  |
|      image             |             natty-image                 |
|     key_name           |                                         |
|     metadata           | {u'description': u'Small test image'}   |
|       name             |             smallimage2                 |
|    private network     |            172.16.101.11                |
|     progress           |                 0                       |
|     public network     |             10.4.113.11                 |
|      status            |               ACTIVE                    |
|    tenant_id           |             e83...482                   |
|     updated            |            2012-05-16T20:48:35Z         |
|     user_id            |          de3...0a9                      |
+------------------------+-----------------------------------------+
用户数据是元数据服务里一个特殊的键，它持有一个文件，在本实例上的云应用能访问。例如  cloudinit (https://help.ubuntu.com/community/CloudInit) 是一个开源软件包，在实例启动的时候可以使用这些用户数据。
用户数据的生成：可以在本地创建一个文件然后传送给实例，在实例创建的时候，增加选项： --user-data <user-data-file> 。 举例:
$ nova boot --image ubuntu-cloudimage --flavor 1 --user-data mydata.file 
任意文件可以被放到实例的文件系统，采用 --file <dst-path=src-path> 选项。 最多可以存放5个文件。 例如有一个特殊授权密钥文件名字为：special_authorized_keysfile ， 要放到实例中代替通常的ssh密钥注入，可以用以下名利：
$ nova boot --image ubuntu-cloudimage --flavor 1 --file /root/.ssh/authorized_keys=special_authorized_keysfile

关联安全组
如前面所讲，如果允许网络流量到实例，安全组的配置是需要的，除非缺省安全组已经被配置了允许流量通过。
添加安全组通常在实例启动时做。在dashboard上这部分在“访问与安全”选项卡的“启动实例”对话框。在命令行启动方式下， 增加安全组用  --security-groups 选项， 并用逗号分隔一组安全组。
还可以在实例运行的时候增加安全组。目前此功能只能在命令行方式下做。
$ nova add-secgroup <server> <securitygroup>
$ nova remove-secgroup <server> <securitygroup>

Floating IPs
前面介绍过，项目有一个配额来控制floating ip的数量，但是这些在可以使用之前需要被一个用户分配好。分配一个floating IP到项目是通过dashboard “访问与安全”页面里的 “分配IP项目” 按钮， 或在命令行上使用: 
$ nova floating-ip-create
一旦被分配， floating ip 可以被设置到运行实例中，通过dashboard， 一种方式是在“访问与安全”页面里的IP 操作的 “设置IP项目” 按钮，或者在“实例”页面中对应实例的相关按钮。相反操作“floating ip去关联” , 只在“访问与安全”页面里可以操作。
命令行的方式中，下列命令完成以上任务：
$ nova add-floating-ip <server> <address>
$ nova remove-floating-ip <server> <address>

加载块存储
可以在dashboard的“卷”页面上为实例加载块存储。点击“编辑加载”选择加载哪个卷。 在命令行中，可以：
$ nova volume-attach <server> <volume> 
还可以用命令行在实例启动的时候指定块设备，命令如下：
--block-device-mapping <dev-name=mapping> 
块设备 mapping的格式是： <dev-name=<id>:<type>:<size(GB)>:<delete-on-terminate>,这里:
dev-name	卷所加载的设备名称：/dev/dev_name .
id			卷启动的ID ， 会在 nova volume-list 的输出中看到。
Type		或是 snap, 表示卷是从一个快照建立的，或其他类型(空字符串都可以)。在上面例子中，卷并非由快照建立的，因此下面的显示为空。
size (GB)		卷大小（GB）。也可以不填，让计算服务决定大小。
delete-on-terminate	布尔值， 表明在实例被删除后卷是否被删除。 True 为1， False为0。
如果原来就有一个块存储上有启动文件的，甚至可以直接从块存储上启动实例。下面例子中会尝试从ID=13的卷中启动，这个卷不会被删除。 --key-name 要改为正确的密钥。
$ nova boot --flavor 2 --key-name mykey --block-device-mapping vda=13:::0 boot-from-vol-test
因为bug 1163566 (https://bugs.launchpad.net/nova/+bug/1163566) 在horizon中你必须选择一个镜像来启动，虽然这个镜像你没有用。 
为了正常从一个镜像启动并加载块存储，不要将设备映射到vda。
制作快照
保证快照的一致性
OpenStack的快照机制允许你从一台运行的实例中创建镜像。这使得对基础镜像的升级或做定制修改变得非常容易。 为一个正在运行的实例做镜像可以用命令：
$ nova image-create <instance name or uuid> <name of new image>
Dashboard上面关于镜像的接口有些清晰，因为它们将快照页面分在三个部分：
镜像
实例快照
卷快照
但是， 实例快照是一个镜像。快照镜像和从网上下载的原始镜像的唯一区别就是快照镜像在glance数控库中有附加的属性，这些属性可以在image_properties 表中找到，包括：
属性	值
image_type	快照
instance_uuid	<被快照的实例的uuid >
base_image_ref	<被快照的卷的uuid >
image_location	快照
 保证快照一致性
本节内容来自 Sébastien Han's OpenStack: Perform Consistent Snapshots blog entry (http://www.sebastien-han.fr/blog/2012/12/10/openstack-perform-consistent-snapshots/)
快照抓取的是文件系统的状态，而非内存的状态。因此为了保证快照包括所需要的信息，要：
运行的程序将数据写到磁盘上。
文件系统不能有‘脏’缓存数据：当程序要求写磁盘时，系统还没写完。
为了确保重要内容写到磁盘(例如,数据库)，我们建议您阅读对于那些应用程序的命令文档来确定他们的内容同步到磁盘。如果你不确定如何做到这一点，最安全的方法是简单地正常停止这些服务运行。
为解决‘脏’缓存数据问题，我们还建议在做快照前用sync命令。
# sync
运行sync写脏缓冲区(缓冲块已修改但还没有写到磁盘块)到磁盘。
只运行sync命令还不足以保证文件系统的一致性。建议用fsfreeze 工具， 可以阻止对文件系统的新的访问，以建立一个适合做镜像的文件系统。fsfreeze 支持几种文件系统, ext3, ext4, 和 XFS. 如果你的系统是在ubuntu上的，安装 util-linux 包以得到 fsfreeze:
# apt-get install util-linux
如果你的操作系统没有一个版本的fsfreeze可用，您可以使用xfs_freez代替，可以在Ubuntu的xfsprogs包找到。尽管是“xfs”的名字，xfs_freez也适用在ext3和ext4，如果您使用的是Linux内核版本2.6.29或更新的情况下。Xfs_freez 支持相同的命令行参数fsfreeze一样。
例如需要对一个持久的块存储卷做快照，客户机操作系统探测到的/ dev /vdb 挂载到 /mnt。fsfreeze命令接受2个参数: 
-f: freeze the system
-u: un-freeze the system
为了冻结一个卷，作为root用户:
# fsfreeze -f /mnt
卷必须被挂载才能被冻结。
当“fsfreeze - f”命令发出，在文件系统所有正在进行中的事务被允许完成，新写系统的调用被阻止，以及其他修改文件系统的调用都停止了。最重要的是，所有的脏数据、元数据和日志信息被写入到磁盘。
一旦卷已被冻结,不要试图读或写，因为这些操作会挂起。 操作系统停止每个I / O操作和任何I / O的尝试被延迟，直到文件系统被解冻。
一旦你已经执行了fsfreeze命令，就可以安全的执行快照。例如,如果您的实例被命名为mon-instance，你用快照得到一个镜像，名字叫mon-snapshot，命令如下:
$ nova image-create mon-instance mon-snapshot
当快照已经完成，可以用下面的命令解冻文件系统，在实例中用root用户：
# fsfreeze -u /mnt
如果你想备份根文件系统，您不能简单地执行上面的命令，因为它会冻结提示。而是运行下面的命令：
# fsfreeze -f / && sleep 30 && fsfreeze -u /

实例在数据库中的信息
实例信息存储在几个数据库表中，表操作者最可能需要查看的是‘实例instanse’表。
实例表 带有正在运行或被删除的实例的几乎所有的信息。如果详细地看，它有很复杂的字段，都我我我在是操作者查询信息所需要的。
实例已被删除时“delete”字段设置为“1”， 如果没有删除为NULL。这对于实例的查询是非常重要的。
“uuid”字段是实例的uuid，作为外键用于在整个数据库中其他的表。这个id还在日志、dashboard和命令行工具等里面来唯一地标识一个实例。
通过一组外键可以找到实例之间的关系。最有用的是“user_id”和“项目id”，它们是创建实例的用户的和其所在项目的uuid。
"hostname" 字段表示实例是在哪个计算节点上运行。"display-name"与"host"名字相同，可以用nova rename命令修改。
还有一些时间相关字段，可以在一个实例状态发生改变时用于跟踪：
created_at
updated_at
deleted_at
scheduled_at
launched_at
terminated_at