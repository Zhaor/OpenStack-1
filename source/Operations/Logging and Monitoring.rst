日志与监控
========================
Chapter 14. 日志和监控

日志在哪里？

如何读取日志 

跟踪实例请求

添加自定义日志记录语句

 RabbitMQ Web管理界面或rabbitmqctl

集中管理日志 

StackTach 监控

OpenStack云是由很多不同的服务组成的，有大量的日志文件。本节旨在帮助您定位和利用日志，以及其他来跟踪您的部署的状态的方法。

 日志在哪里？

云控制器

计算节点

块存储节点

在Ubuntu上，大多数服务的日志文件写入到/va/rlog目录的子目录中

 云控制器
服务
 日志位置
 
nova-* 
 /var/log/nova 
 
glance-* 
 /var/log/glance 
 
cinder-* 
 /var/log/cinder 
 
keystone 
 /var/log/keystone 
 
horizon
 /var/log/apache2/ 
 
misc (Swift, dnsmasq)
 /var/log/syslog 
 

 计算节点
libvirt: /var/log/libvirt/libvirtd.log 

vm 实例的控制台 (启动信息) : /var/lib/nova/instances/instance-<instance id>/console.log 

 块存储节点
cinder: /var/log/cinder/cinder-volume.log 

如何读取日志
OpenStack服务使用标准的日志级别：DEBUG，INFO，AUDIT， WARNING，ERROR，CRITICAL，和 TRACE。并且只显示和记录比指定级别更"严重"的日志，或者用DEBUG 来显示所有的日志。例如，TRACE只记录如果有TRACE的日志，而INFO记录每条INFO及更严重的消息。

禁用DEBUG级别日志记录，编辑/etc/nova/nova.conf:

debug=false
Keystone 采用稍微不同的方法。为改变日志级别，编辑 /etc/keystone/logging.conf 文件并查看logger_root 和 handler_file 部分。

Horizon的日志配置在 /etc/openstack_dashboard/local_settings.py. 因为Horizon是一个Django web 框架应用，它沿用 Django Logging (https://docs.djangoproject.com/en/dev/topics/logging/) 规则。

找到错误源的第一步通常是要搜索日志中最新的CRITICAL,TRACE, 或ERROR消息。

一个例子如下，出现了 CRITICAL 日志消息，带有相关的 TRACE (Python traceback) 信息:

2013-02-25 21:05:51 17409 CRITICAL cinder [-] Bad or unexpected response from the storage volume backend API: volume group 
 cinder-volumes doesn't exist
2013-02-25 21:05:51 17409 TRACE cinder Traceback (most recent call last):
2013-02-25 21:05:51 17409 TRACE cinder File "/usr/bin/cinder-volume", line 48, in <module>
2013-02-25 21:05:51 17409 TRACE cinder service.wait()
2013-02-25 21:05:51 17409 TRACE cinder File "/usr/lib/python2.7/dist-packages/cinder/service.py", line 422, in wait
2013-02-25 21:05:51 17409 TRACE cinder _launcher.wait()
2013-02-25 21:05:51 17409 TRACE cinder File "/usr/lib/python2.7/dist-packages/cinder/service.py", line 127, in wait
2013-02-25 21:05:51 17409 TRACE cinder service.wait()
2013-02-25 21:05:51 17409 TRACE cinder File "/usr/lib/python2.7/dist-packages/eventlet/greenthread.py", line 166, in wait
2013-02-25 21:05:51 17409 TRACE cinder return self._exit_event.wait()
2013-02-25 21:05:51 17409 TRACE cinder File "/usr/lib/python2.7/dist-packages/eventlet/event.py", line 116, in wait
2013-02-25 21:05:51 17409 TRACE cinder return hubs.get_hub().switch()
2013-02-25 21:05:51 17409 TRACE cinder File "/usr/lib/python2.7/dist-packages/eventlet/hubs/hub.py", line 177, in switch
2013-02-25 21:05:51 17409 TRACE cinder return self.greenlet.switch()
2013-02-25 21:05:51 17409 TRACE cinder File "/usr/lib/python2.7/dist-packages/eventlet/greenthread.py", line 192, in main
2013-02-25 21:05:51 17409 TRACE cinder result = function(*args, **kwargs)
2013-02-25 21:05:51 17409 TRACE cinder File "/usr/lib/python2.7/dist-packages/cinder/service.py", line 88, in run_server
2013-02-25 21:05:51 17409 TRACE cinder server.start()
2013-02-25 21:05:51 17409 TRACE cinder File "/usr/lib/python2.7/dist-packages/cinder/service.py", line 159, in start
2013-02-25 21:05:51 17409 TRACE cinder self.manager.init_host()
2013-02-25 21:05:51 17409 TRACE cinder File "/usr/lib/python2.7/dist-packages/cinder/volume/manager.py", line 95, 
 in init_host
2013-02-25 21:05:51 17409 TRACE cinder self.driver.check_for_setup_error()
2013-02-25 21:05:51 17409 TRACE cinder File "/usr/lib/python2.7/dist-packages/cinder/volume/driver.py", line 116, 
 in check_for_setup_error
2013-02-25 21:05:51 17409 TRACE cinder raise exception.VolumeBackendAPIException(data=exception_message)
2013-02-25 21:05:51 17409 TRACE cinder VolumeBackendAPIException: Bad or unexpected response from the storage volume 
 backend API: volume group cinder-volumes doesn't exist
2013-02-25 21:05:51 17409 TRACE cinder
在这个例子中，cinder-volumes启动失败，并提供stack跟踪，因为它的volume后台已无法设置存储卷--可能是因为预计中配置的LVM卷不存在。

一个错误信息例子:

2013-02-25 20:26:33 6619 ERROR nova.openstack.common.rpc.common [-] AMQP server on localhost:5672 is unreachable:
 [Errno 111] ECONNREFUSED. Trying again in 23 seconds.
这个例子中, nova 服务不能连接到 RabbitMQ 服务器，因为一个链接被拒绝的错误。

 跟踪实例的请求

当一个实例出现错误时，通常可以跟踪与该实例关联的的nova-* 日志文件，并查看相关云控制器和计算节点。

一般来说跟踪日志文件中实例的UUID相关内容。下面例子：

ubuntu@initial:~$ nova list
+--------------------------------------+--------+--------+---------------------------+
| ID                                   | Name   | Status | Networks                  |
+--------------------------------------+--------+--------+---------------------------+
| faf7ded8-4a46-413b-b113-f19590746ffe | cirros | ACTIVE | novanetwork=192.168.100.3 |
+--------------------------------------+--------+--------+---------------------------+
实例的UUID是 faf7ded8-4a46-413b-b113-f19590746ffe。如果在云控制器中的/var/log/nova-*.log文件搜索此字符串，一般出现在 nova-api.log， nova-scheduler.log文件中。如果在计算节点的/var/log/nova-*.log 中搜索，一般在 nova-network.log 和nova-compute.log文件中。如果没有出现任何ERROR或CRITICAL消息，就查看最新的日志项报告，这可能会提供一些线索。

添加自定义日志记录声明

如果在日志中没有足够信心，你需要在nova-* 日志中添加自定义的日志记录声明。

源文件在 /usr/lib/python2.7/dist-packages/nova 

添加日志记录声明。下面一行应该在文件的顶部。对于大多数文件，这行应该已经有了：

from nova.openstack.common import log as logging
LOG = logging.getLogger(__name__)
添加一个DEBUG 日志声明：

LOG.debug("This is a custom debugging statement")
你可能注意到，所有现有的日志消息前面加一个下划线和括号包围，例如：

LOG.debug(_("Logging statement appears here"))
这是为了支持能把日志信息通过翻译成不同语言，利用多语言库gettext (http://docs.python.org/2/library/gettext.html)。 你不需要对您自己的自定义日志消息做这个。然而，如果你想为OpenStack项目贡献代码，包括日志语句，您必须用下划线和括号包括你的日志消息。

 RabbitMQ Web管理界面或 rabbitmqctl

除了连接失败，RabbitMQ日志文件通常不用于调试OpenStack相关问题。相反，我们建议您使用RabbitMQ web管理界面。在云控制器上启用它：

# /usr/lib/rabbitmq/bin/rabbitmq-plugins enable rabbitmq_management
# service rabbitmq-server restart
在云控制器上访问RabbitMQ web管理界面：  http://localhost:55672.

[Note]
 Note
 
ubuntu 12.04安装RabbitMQ版本2.7.1，使用端口55672。RabbitMQ 3.0及以上版本使用端口15672。您可以检查该版本RabbitMQ你有做你的本地机器上运行：

$ dpkg -s rabbitmq-server | grep "Version:"
Version: 2.7.1-0ubuntu4
 

启用RabbitMQ Web管理界面的另一种方式是使用rabbitmqctl命令。例如， rabbitmqctl list_queues| grep cinder显示任何留在队列中的消息。如果有，可能表示cinder务没有正确连接到rabbitmq，可能需要重新启动。

Items to monitor for RabbitMQ include the number of items in each of the queues and the processing time statistics for the server.

监视RabbitMQ项目包括在每个队列的消息数量和服务器处理时间的统计数据。

 集中管理日志

rsyslog 客户端配置

rsyslog 服务器端配置

因为云一般由多个服务器组成，确定一起事件可能需要检查多个服务器。更好的解决方案是将所有服务器的日志发送到中央位置，放在一个地方被访问。

Ubuntu的默认的日志记录服务用rsyslog。 它本身可以将日志发送到远程位置，所以启用此功能不需要任何安装，只需要修改配置文件。要这样做，可以考虑将日志在管理网络内运行，或使用一个加密的VPN来提高安全性。

 rsyslog客户端配置
首先，除了标准的日志文件位置，将所有OpenStack组件日志配置为记录到syslog。并且配置每个组件登录到不同的syslog facility。这样便于在中央服务器上将记录按组件拆分。配置以下文件：

nova.conf:

use_syslog=True
syslog_log_facility=LOG_LOCAL0
glance-api.conf and glance-registry.conf:

use_syslog=True
syslog_log_facility=LOG_LOCAL1
cinder.conf:

use_syslog=True
syslog_log_facility=LOG_LOCAL2
keystone.conf:

use_syslog=True
syslog_log_facility=LOG_LOCAL3
对于Swift，缺省的日志就是syslog.

接着创建 /etc/rsyslog.d/client.conf：

*.* @192.168.1.10
这将指示rsyslog发送的所有日志到相关IP。在这个例子中，IP是云控制器。

 

 rsyslog 服务器端配置
指定一个中央服务器作为日志服务器。最好的做法是选择一个专用服务器。创建一个文件： /etc/rsyslog.d/server.conf，带以下内容：

# Enable UDP 
$ModLoad imudp 
# Listen on 192.168.1.10 only 
$UDPServerAddress 192.168.1.10
# Port 514 
$UDPServerRun 514  
      
# Create logging templates for nova
$template NovaFile,"/var/log/rsyslog/%HOSTNAME%/nova.log" 
$template NovaAll,"/var/log/rsyslog/nova.log"
      
 
      
# Log everything else to syslog.log 
$template DynFile,"/var/log/rsyslog/%HOSTNAME%/syslog.log"
*.* ?DynFile
      
 
      
# Log various openstack components to their own individual file
local0.* ?NovaFile 
local0.* ?NovaAll 
& ~
上面的示例配置仅处理nova服务。它首先配置rsyslog充当服务器，运行在端口512上。接下来，它创造了一系列记录的模板。记录模板控制接收日志存储的位置。使用上面的示例，一个从c01.example.com 的nova日志转到以下位置：

<!--[if !supportLists]-->·         <!--[endif]-->/var/log/rsyslog/c01.example.com/nova.log 

<!--[if !supportLists]-->·         <!--[endif]-->/var/log/rsyslog/nova.log 

也指定从 c02.example.com 发来的日志存到：

<!--[if !supportLists]-->·         <!--[endif]-->/var/log/rsyslog/c02.example.com/nova.log 

<!--[if !supportLists]-->·         <!--[endif]-->/var/log/rsyslog/nova.log 

因此你有了对每个计算节点各自的日志，也包含从所有节点的汇集的日志文件。

 StackTach

StackTach是由Rackspace创建的nova 通知信息（notification）的收集和报告的工具。通知信息和日志基本相同，但可以更详细。通知信息的参考资料在：System Usage Data (https://wiki.openstack.org/wiki/SystemUsageData)。

启动通知信息可以在 nova.conf文件中配置：

notification_topics=monitor 
notification_driver=nova.openstack.common.notifier.rabbit_notifier
当 nova能发送信息通知时，安装和配置 StackTach. 由于StackTach相对较新而且经常变动，安装帮助文档的版本很快变旧，因此到在线的 StackTach GitHub repo (https://github.com/rackerlabs/stacktach) 来查看。

 监控

处理监控

资源告警

OpenStack资源

智能告警

趋势

有两种类型的监控：看问题，或看使用趋势。前确保所有服务都运行起来，创建一个功能正常的云。后者包括监视资源使用情况，以对潜在的瓶颈和升级作出明智的决定。

 处理监控
告警监控的基本类型是简单地检查，看看所需的进程是否正在运行。例如，确保nova-api服务是否在云控制器上运行：

[ root@cloud ~ ] # ps aux | grep nova-api
nova 12786 0.0 0.0 37952 1312 ? Ss Feb11 0:00 su -s /bin/sh -c exec nova-api --config-file=/etc/nova/nova.conf nova
nova 12787 0.0 0.1 135764 57400 ? S Feb11 0:01 /usr/bin/python /usr/bin/nova-api --config-file=/etc/nova/nova.conf
nova 12792 0.0 0.0 96052 22856 ? S Feb11 0:01 /usr/bin/python /usr/bin/nova-api --config-file=/etc/nova/nova.conf
nova 12793 0.0 0.3 290688 115516 ? S Feb11 1:23 /usr/bin/python /usr/bin/nova-api --config-file=/etc/nova/nova.conf
nova 12794 0.0 0.2 248636 77068 ? S Feb11 0:04 /usr/bin/python /usr/bin/nova-api --config-file=/etc/nova/nova.conf
root 24121 0.0 0.0 11688 912 pts/5 S+ 13:07 0:00 grep nova-api
通过使用Nagios和NRPE软件您可以为一些关键进程创建自动报警的。例如，要确保nova-compute进程在计算节点上正常运行，可以在Nagios服务器上创建一个告警，像这样：

define service { 
    host_name c01.example.com 
    check_command check_nrpe_1arg!check_nova-compute 
    use generic-service 
    notification_period 24x7 
    contact_groups sysadmins 
    service_description nova-compute 
}
然后在真实的计算节点上创建以下 NRPE 配置：

command[check_nova-compute]=/usr/lib/nagios/plugins/check_procs -c 1: -a nova-compute
Nagios会一直检查nova-compute服务。 

 资源告警
资源告警在一个或多个资源非常低的时候告警。监视阈值应调到您的特定OpenStack环境，监控资源工具不需要OpenStack专用的，–任何普通类型的告警系统都可以很好地工作。

你希望监控的部分资源如下：

<!--[if !supportLists]-->·         <!--[endif]-->磁盘利用

<!--[if !supportLists]-->·         <!--[endif]-->服务器负载

<!--[if !supportLists]-->·         <!--[endif]-->内存使用

<!--[if !supportLists]-->·         <!--[endif]-->网络 IO

<!--[if !supportLists]-->·         <!--[endif]-->可用的 vCPU

例如，使用Nagios在计算节点监视磁盘容量，将下面的代码添加到Nagios配置：

define service { 
    host_name c01.example.com 
    check_command check_nrpe!check_all_disks!20% 10% 
    use generic-service 
    contact_groups sysadmins 
    service_description Disk 
}
在计算节点，增加以下 NRPE 配置：

command[check_all_disks]=/usr/lib/nagios/plugins/check_disk -w $ARG1$ -c $ARG2$ -e
Nagios 会发出 WARNING 信息，当计算节点磁盘达到80%用量，在达到90%用量发出 CRITICAL告警。

 OpenStack-特定的资源
.

资源如内存、磁盘和CPU等是通用的，即使是对非OpenStack服务器的整体健康都很重要。对于OpenStack，这些资源是非常重要的另一个原因是：确保足够的资源可用来启动实例。有几个方法可以看到OpenStack资源的使用。

第一种是使用 nova 命令：

# nova usage-list
此命令显示某租户有多少已运行实例，和少量的综合统计信息。此命令快速概述您的云的状况，没有很多细节。

其次，nova 数据库包括3个表，保存使用量信息。表nova.quotas 和 nova.quota_usages保存配额信息。如果一个租户的配合和缺省配额不一样，配额会被存入nova.quotas ，例如：

mysql> select project_id, resource, hard_limit from quotas; 
+----------------------------------+-----------------------------+------------+
| project_id                       | resource                    | hard_limit |
+----------------------------------+-----------------------------+------------+
| 628df59f091142399e0689a2696f5baa | metadata_items              | 128        |
| 628df59f091142399e0689a2696f5baa | injected_file_content_bytes | 10240      |
| 628df59f091142399e0689a2696f5baa | injected_files              | 5          |
| 628df59f091142399e0689a2696f5baa | gigabytes                   | 1000       |
| 628df59f091142399e0689a2696f5baa | ram                         | 51200      |
| 628df59f091142399e0689a2696f5baa | floating_ips                | 10         |
| 628df59f091142399e0689a2696f5baa | instances                   | 10         |
| 628df59f091142399e0689a2696f5baa | volumes                     | 10         |
| 628df59f091142399e0689a2696f5baa | cores                       | 20         |
+----------------------------------+-----------------------------+------------+ 
nova.quota_usages 表跟踪租户目前的资源使用状况：

mysql> select project_id, resource, in_use from quota_usages where project_id like '628%';
+----------------------------------+--------------+--------+ 
| project_id                       | resource     | in_use | 
+----------------------------------+--------------+--------+ 
| 628df59f091142399e0689a2696f5baa | instances    | 1      |
| 628df59f091142399e0689a2696f5baa | ram          | 512    | 
| 628df59f091142399e0689a2696f5baa | cores        | 1      | 
| 628df59f091142399e0689a2696f5baa | floating_ips | 1      | 
| 628df59f091142399e0689a2696f5baa | volumes      | 2      | 
| 628df59f091142399e0689a2696f5baa | gigabytes    | 12     | 
| 628df59f091142399e0689a2696f5baa | images       | 1      | 
+----------------------------------+--------------+--------+
有了租户的配额和资源的使用量，能算出一个使用百分比。例如，如果该租户使用了1个浮动IP，配额是10个，即表明他使用了10%的浮动Ip配额。可以把这个过程变成一个格式化的报告：

+-----------------------------------+------------+-------------+---------------+ 
| some_tenant                                                                  | 
+-----------------------------------+------------+-------------+---------------+ 
| Resource                          | Used       | Limit       |               | 
+-----------------------------------+------------+-------------+---------------+ 
| cores                             | 1          | 20          |           5 % | 
| floating_ips                      | 1          | 10          |          10 % | 
| gigabytes                         | 12         | 1000        |           1 % | 
| images                            | 1          | 4           |          25 % | 
| injected_file_content_bytes       | 0          | 10240       |           0 % | 
| injected_file_path_bytes          | 0          | 255         |           0 % | 
| injected_files                    | 0          | 5           |           0 % | 
| instances                         | 1          | 10          |          10 % | 
| key_pairs                         | 0          | 100         |           0 % | 
| metadata_items                    | 0          | 128         |           0 % | 
| ram                               | 512        | 51200       |           1 % | 
| reservation_expire                | 0          | 86400       |           0 % | 
| security_group_rules              | 0          | 20          |           0 % | 
| security_groups                   | 0          | 10          |           0 % | 
| volumes                           | 2          | 10          |          20 % | 
+-----------------------------------+------------+-------------+---------------+
上表可以用脚本定制，脚本在 GitHub (https://github.com/cybera/novac/blob/dev/libexec/novac-quota-report). 

[Note]
 Note
 
此脚本是特定于某个OpenStack安装，必须修改以适合您的环境。然而，基本逻辑是相通的。
 

 智能告警
智能报警可以被认为是一个持续集成的操作形式。例如，您可以查看glance-api端口9292， 了解glance-api和glance-registry 进程运行情况，从而很容易地检查Glance是否已启动并运行。

但如何知道软件镜像已经被成功地加载到软件镜像服务中？也许软件镜像服务存储的磁盘已满或S3后端宕机。可以通过做一个快速镜像上传来检查：

#!/bin/bash 
# 
# assumes that resonable credentials have been stored at 
# /root/auth 
 
      
. /root/openrc 
wget https://launchpad.net/cirros/trunk/0.3.0/+download/cirros-0.3.0-x86_64-disk.img  
glance image-create --name='cirros image' --is-public=true --container-format=bare --disk-format=qcow2 < cirros-0.3.0-x8
6_64-disk.img
将此脚本部署到监控告警系统(如Nagios)，你现在有一种自动化的方式确保镜像上传工作正常。

[Note]
 Note
 
在每个测试之后，您必须删除镜像。 更好的是，测试你是否可以成功地从镜像服务中删除镜像。
 

智能告警比普通告警需要相当多的时间来做计划和实施。一个好的实现思路：

<!--[if !supportLists]-->·         <!--[endif]-->列出你的云系统工作的每个动作。

<!--[if !supportLists]-->·         <!--[endif]-->对每个动作创建自动的测试的方法。

<!--[if !supportLists]-->·         <!--[endif]-->将测试方法部署到告警系统中。

 

其他智能告警的例子包括：

<!--[if !supportLists]-->·         <!--[endif]-->实例能被创建或销毁么？

<!--[if !supportLists]-->·         <!--[endif]-->用户能被创建么？

<!--[if !supportLists]-->·         <!--[endif]-->对象能被存储或删除么？

<!--[if !supportLists]-->·         <!--[endif]-->卷能被创建或删除么？

 趋势
趋势可以帮助洞察你的云每一天是如何运行的。例如，系统繁忙是否经常出现，或从趋势上看你是否需要增加新的计算节点。

趋势采取了与告警稍稍不同的方法。告警是对结果检查，是成功还是失败，而趋势则是记录当前状态以及在每个时间点发生了什么。一旦记录了足够的时间点，可以看到相关的值随时间的变化趋势。

所有前面提到的告警类型也可以用于趋势报告。其他一些趋势的例子包括：

<!--[if !supportLists]-->·         <!--[endif]-->每个计算节点上的实例数

<!--[if !supportLists]-->·         <!--[endif]-->使用的类型统计 

<!--[if !supportLists]-->·         <!--[endif]-->使用的卷的数量

<!--[if !supportLists]-->·         <!--[endif]-->每小时对象存储请求的数目

<!--[if !supportLists]-->·         <!--[endif]-->每小时Nova-api请求的数量

<!--[if !supportLists]-->·         <!--[endif]-->数据存储服务IO统计

As an example, recording nova-api usage can allow you to track the need to scale your cloud controller. By keeping an eye on nova-api requests, you can determine if you need to spawn more nova-api processes or go as far as introducing an entirely new server to run nova-api. To get an approximate count of the requests, look for standard INFO messages in

例如，记录nova-api的使用，可以跟踪您的云控制器是否需要扩展。通过监视nova-api请求，可以确定是否需要更多的nova-api进程或引入一个全新的服务器来运行nova-api。为得到这个请求数量，在/var/log/nova/nova-api.log中查看标准INFO信息：

# grep INFO /var/log/nova/nova-api.log | wc

通过查看成功的请求的数量可以得到更多统计信息：

# grep " 200 " /var/log/nova/nova-api.log | wc

定期运行此命令的结果，记录这些结果，您可以得到一个随着时间的推移的趋势报告，显示您的nova-api是使用增加、减少或保持稳定。

有些工具如collectd可以用来存储这些信息。一开始，可以使用collectd作为COUNTER类型数据的结果的存储。更多信息可以在collectd的文档中找到（https://collectd.orgwikiindex.phpdata_source/）。

 

