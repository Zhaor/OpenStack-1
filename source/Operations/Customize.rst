定制
===================

Chapter 16. 定制

DevStack

中间件案例

Nova Scheduler案例

Dashboard

OpenStack 不能做完所有的事情，在本章中介绍几个方式完善功能。首先，可以学习如何贡献到社区(https://wiki.openstack.org/wiki/How_To_Contribute), 按照代码回顾工作模式（https://wiki.openstack.org/wiki/GerritWorkflow)，你可以修改代码为OpenStack项目做贡献。 如果您需要的功能与现有项目的深度集成，推荐用此路径。社区始终是开放的，欢迎根据开发指南开发新的功能特性。

或者，如果你不需要功能的深度集成，还有其他方法来定制OpenStack。如果你的项目需要的功能需要使用Python Paste框架，您可以为之创建中间件并配置进来。也可以有特定的方式定制项目，比如创建一个新的OpenStack计算调度器或自定义的Dashboard。本章重点介绍自定义Openstack的第二种方法。

以这种方式自定义OpenStack，你需要一个开发环境。最好找一个快速启动和运行的环境，例如在你的云系统中运行DevStack。

 DevStack

你可以找到DevStack所有文档(http://devstack.org/)。 取决于您想自定义的项目，对象存储swift或其他项目，需要对DevStack做不同配置。对于中间件，下面的示例中，您必须安装和启用对象存储。

 

如何在一个实例中运行 DevStack Folsom 稳定版：

<!--[if !supportLists]-->1.    <!--[endif]-->从Dashboard或nova命令行启动一个实例，采用以下参数：

<!--[if !supportLists]-->·         <!--[endif]-->名字: devstack

<!--[if !supportLists]-->·         <!--[endif]-->镜像: Ubuntu 12.04 LTS

<!--[if !supportLists]-->·         <!--[endif]-->内存: 4 GB RAM (可以用 2 GB)

<!--[if !supportLists]-->·         <!--[endif]-->磁盘大小: 最小 5 GB

如果用nova客户端，启动时采用配置 --flavor 6 参数， 以确定内存硬盘的大小。

<!--[if !supportLists]-->2.    <!--[endif]-->如果您的镜像中只有root用户，您必须创建一个"stack"用户。否则如果让stack.sh 创建"stack"用户时会遇到权限问题。 如果您的镜像已经有root以外的用户，你可以跳过此步骤。 

<!--[if !supportLists]-->a.    <!--[endif]-->ssh root@<IP Address> 

<!--[if !supportLists]-->b.    <!--[endif]-->adduser --gecos "" stack 

<!--[if !supportLists]-->c.    <!--[endif]-->Enter a new password at the prompt.

<!--[if !supportLists]-->d.    <!--[endif]-->adduser stack sudo 

<!--[if !supportLists]-->e.    <!--[endif]-->grep -q "^#includedir.*/etc/sudoers.d" /etc/sudoers || echo "#includedir /etc/sudoers.d" >> /etc/sudoers 

<!--[if !supportLists]-->f.     <!--[endif]-->( umask 226 && echo "stack ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/50_stack_sh ) 

<!--[if !supportLists]-->g.    <!--[endif]-->exit 

<!--[if !supportLists]-->3.    <!--[endif]-->现在以stack用户登录并设置 DevStack.

<!--[if !supportLists]--> .     <!--[endif]-->ssh stack@<IP address> 

<!--[if !supportLists]-->a.    <!--[endif]-->At the prompt, enter the password that you created for the stack user. 

<!--[if !supportLists]-->b.    <!--[endif]-->sudo apt-get -y update 

<!--[if !supportLists]-->c.    <!--[endif]-->sudo apt-get -y install git 

<!--[if !supportLists]-->d.    <!--[endif]-->git clone https://github.com/openstack-dev/devstack.git -b stable/folsom devstack/ 

<!--[if !supportLists]-->e.    <!--[endif]-->cd devstack 

<!--[if !supportLists]-->f.     <!--[endif]-->vim localrc 

<!--[if !supportLists]-->·         <!--[endif]-->对于Swift，在Middleware 例子里，查看下面的Swift only localrc

<!--[if !supportLists]-->·         <!--[endif]-->对于其他项目，在 Nova Scheduler 例子中, 查看下面的All other projects localrc

<!--[if !supportLists]-->g.    <!--[endif]-->./stack.sh 

<!--[if !supportLists]-->h.    <!--[endif]-->screen -r stack 

[Note]
 Note
 
<!--[if !supportLists]-->·         <!--[endif]-->stack.sh 脚本的运行需要一段时间。或者可以利用这个机会加入Openstack fundation（www.openstack.org/join/).

<!--[if !supportLists]-->·         <!--[endif]-->当你运行stack.sh时, 你可能会看到一个错误信息 “ERROR: at least one RPC back-end must be enabled”.不用担心这个，swift和keystone不需要RPC (AMQP) 后端。你还可以忽略任何 ImportErrors.

<!--[if !supportLists]-->·         <!--[endif]-->Screen 是一次性查看很多相关服务的非常好的工具 。更多信息在： GNU screen quick reference. (http://aperiodic.net/screen/quick_reference)
 

Now that you have an OpenStack development environment, you're free to hack around without worrying about damaging your production deployment. Proceed to either the Middleware Example for a Swift-only environment, or the Nova Scheduler Example for all other projects.

现在，您有一个OpenStack开发环境，你可以不用担心您的生产部署环境被损坏。接下来可以只有swift的中间件实验样例，或nova scheduler 模式样例。

[1] Swift only localrc 

<!--[if !vml]-->Select Text<!--[endif]-->1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19
 ADMIN_PASSWORD=devstack

MYSQL_PASSWORD=devstack

RABBIT_PASSWORD=devstack

SERVICE_PASSWORD=devstack

SERVICE_TOKEN=devstack 

SWIFT_HASH=66a3d6b56c1f479c8b4e70ab5c2000f5

SWIFT_REPLICAS=1 

# Uncomment the BRANCHes below to use stable versions

# unified auth system (manages accounts/tokens)

KEYSTONE_BRANCH=stable/folsom

# object storage

SWIFT_BRANCH=stable/folsom

disable_all_services

enable_service key swift mysql
 

[2] All other projects localrc 

<!--[if !vml]-->Select Text<!--[endif]-->1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27
 ADMIN_PASSWORD=devstack

MYSQL_PASSWORD=devstack

RABBIT_PASSWORD=devstack

SERVICE_PASSWORD=devstack

SERVICE_TOKEN=devstack 

FLAT_INTERFACE=br100

PUBLIC_INTERFACE=eth0

VOLUME_BACKING_FILE_SIZE=20480M 

# For stable versions, look for branches named stable/[milestone]. 

# compute service

NOVA_BRANCH=stable/folsom

# volume service

CINDER_BRANCH=stable/folsom

# image catalog service

GLANCE_BRANCH=stable/folsom

# unified auth system (manages accounts/tokens)

KEYSTONE_BRANCH=stable/folsom

# django powered web control panel for openstack

HORIZON_BRANCH=stable/folsom
 

 中间件样例

大多数OpenStack项目是基于Python Paste(http://pythonpaste.org/)框架。 对其体系最好的介绍结构在： (http://pythonpaste.org/do-it-yourself-framework.html)。 由于使用这个框架，您可以将功能添加到项目中，通过把一些自定义的代码植入到项目，而不必更改任何核心代码。

为了演示如何自定义OpenStack，我们将为swift创建一个中间件，只允许一组IP地址访问容器，访问控制由容器的元数据决定。在很多情况下这样的例子是有用的。例如，您有一个容器允许从公网访问，但你需要严格限制访问者的IP，只允许白名单的IP访问。

[Warning]
 Warning
 
此示例只用于演示目的。在实际中没有做详细部署和安全测试的情况下，不能直接使用方案。
 

当在stack.sh启动时，利用命令screen –r stack加入screen会话， 会有3个screen 会话出现，在利用localrc文件只安装swift的情况下 。

0$ shell*  1$ key  2$ swift
*号表示你目前在哪个screen上。

<!--[if !supportLists]-->·         <!--[endif]-->0$ shell. 一个可以完成部分工作的shell。

<!--[if !supportLists]-->·         <!--[endif]-->1$ key. keystone 服务。

<!--[if !supportLists]-->·         <!--[endif]-->2$ swift. swift 代理服务.

 

用下面的配置来创建中间件并植入：

<!--[if !supportLists]-->1.    <!--[endif]-->所有OpenStack代码都在/opt/stack。在shell screen中，到swift目录中间件模块。

<!--[if !supportLists]-->a.    <!--[endif]-->cd /opt/stack/swift 

<!--[if !supportLists]-->b.    <!--[endif]-->vim swift/common/middleware/ip_whitelist.py 

<!--[if !supportLists]-->2.    <!--[endif]-->将以下代码加入：

# Licensed under the Apache License, Version 2.0 (the "License");

# you may not use this file except in compliance with the License.

# You may obtain a copy of the License at

#

#    http://www.apache.org/licenses/LICENSE-2.0

#

# Unless required by applicable law or agreed to in writing, software

# distributed under the License is distributed on an "AS IS" BASIS,

# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or

# implied.

# See the License for the specific language governing permissions and

# limitations under the License.

import socket

  

from swift.common.utils import get_logger

from swift.proxy.controllers.base import get_container_info

from swift.common.swob import Request, Response

  

class IPWhitelistMiddleware(object):

    """

    IP Whitelist Middleware

 

    Middleware that allows access to a container from only a set of IP

    addresses as determined by the container's metadata items that start

    with the prefix 'allow'. E.G. allow-dev=192.168.0.20

    """

 

    def __init__(self, app, conf, logger=None):

        self.app = app

 

        if logger:

            self.logger = logger

        else:

            self.logger = get_logger(conf, log_route='ip_whitelist')

 

        self.deny_message = conf.get('deny_message', "IP Denied")

        self.local_ip = socket.gethostbyname(socket.gethostname())

 

    def __call__(self, env, start_response):

        """

        WSGI entry point.

        Wraps env in swob.Request object and passes it down.

 

        :param env: WSGI environment dictionary

        :param start_response: WSGI callable

        """

        req = Request(env)

 

        try:

            version, account, container, obj = req.split_path(1, 4, True)

        except ValueError:

            return self.app(env, start_response)

 

        container_info = get_container_info(

            req.environ, self.app, swift_source='IPWhitelistMiddleware')

 

        remote_ip = env['REMOTE_ADDR']

        self.logger.debug(_("Remote IP: %(remote_ip)s"),

                          {'remote_ip': remote_ip})

 

        meta = container_info['meta']

        allow = {k:v for k,v in meta.iteritems() if k.startswith('allow')}

        allow_ips = set(allow.values())

        allow_ips.add(self.local_ip)

        self.logger.debug(_("Allow IPs: %(allow_ips)s"),

                          {'allow_ips': allow_ips})

 

        if remote_ip in allow_ips:

            return self.app(env, start_response)

        else:

            self.logger.debug(

                _("IP %(remote_ip)s denied access to Account=%(account)s "

                  "Container=%(container)s. Not in %(allow_ips)s"), locals())

            return Response(

                status=403,

                body=self.deny_message,

                request=req)(env, start_response)

 

 

def filter_factory(global_conf, **local_conf):

    """

    paste.deploy app factory for creating WSGI proxy apps.

    """

    conf = global_conf.copy()

    conf.update(local_conf)

 

    def ip_whitelist(app):

        return IPWhitelistMiddleware(app, conf)

return ip_whitelist

 

<!--[if !supportLists]-->3.    <!--[endif]-->在env和conf里有很多有用的信息，用来确定如何处理请求。为查找有哪些可用属性，可以在__init__方法增加日志(log)声明：

self.logger.debug(_("conf = %(conf)s"), locals())
<!--[if !supportLists]-->4.    <!--[endif]-->在 __call__ 方法增加log 声明：

self.logger.debug(_("env = %(env)s"), locals())
<!--[if !supportLists]-->5.    <!--[endif]-->为将中间件植入swift 需要编辑配置文件：

 vim /etc/swift/proxy-server.conf
<!--[if !supportLists]-->6.    <!--[endif]-->查找 [filter:ratelimit] 部分并复制以下配置

<!--[if !vml]-->Select Text<!--[endif]-->1

2

3

4

5

6

7

8

9
 [filter:ip_whitelist]

paste.filter_factory = swift.common.middleware.ip_whitelist:filter_factory

# You can override the default log routing for this filter here:

# set log_name = ratelimit

# set log_facility = LOG_LOCAL0

# set log_level = INFO

# set log_headers = False

# set log_address = /dev/log

deny_message = You shall not pass!
 

<!--[if !supportLists]-->7.    <!--[endif]-->查找 [pipeline:main] 部分并增加ip_whitelist 到下述列表中：

<!--[if !vml]-->Select Text<!--[endif]-->1

2
 [pipeline:main]

pipeline = catch_errors healthcheck cache ratelimit ip_whitelist authtoken keystoneauth proxy-logging proxy-server
 

<!--[if !supportLists]-->8.    <!--[endif]-->重新启动swift 代理(proxy)服务来使之使用中间件。切换到swift screen启动：

<!--[if !supportLists]-->a.    <!--[endif]-->按Ctrl-A 然后按 2, 2是screen的编号。或者可以Ctrl-A 后按n切换到下一个屏幕。

<!--[if !supportLists]-->b.    <!--[endif]-->Ctrl-C 停止服务

<!--[if !supportLists]-->c.    <!--[endif]-->按 Up 键重复最后一个命令，回车

<!--[if !supportLists]-->9.    <!--[endif]-->使用Swift CLI命令行测试中间件。切换到shell screen 并在swift screen 查看输出。

<!--[if !supportLists]-->a.    <!--[endif]-->按 Ctrl-A 接着按0

<!--[if !supportLists]-->b.    <!--[endif]-->cd ~/devstack 

<!--[if !supportLists]-->c.    <!--[endif]-->source openrc 

<!--[if !supportLists]-->d.    <!--[endif]-->swift post middleware-test 

<!--[if !supportLists]-->e.    <!--[endif]-->按Ctrl-A 并接着按 2

<!--[if !supportLists]-->10.<!--[endif]-->多行输出后会看到下面信息：

proxy-server ... IPWhitelistMiddleware
proxy-server Remote IP: 203.0.113.68 (txn: ...)
proxy-server Allow IPs: set(['203.0.113.68']) (txn: ...)
前三个语句基本上是关于中间件与其他的swift服务交互时不需要再进行身份验证。最后2个语句是由中间件产生，表明从DevStack实例发送的请求获得批准。

 
从外部一个能连接到 DevStack 的机器测试中间件：

<!--[if !supportLists]-->a.    <!--[endif]-->swift --os-auth-url=http://203.0.113.68:5000/v2.0/ --os-region-name=RegionOne --os-username=demo:demo --os-password=devstack list middleware-test 

<!--[if !supportLists]-->b.    <!--[endif]-->Container GET failed: http://203.0.113.68:8080/v1/AUTH_.../middleware-test?format=json 403 Forbidden   你通过测试了！

 

<!--[if !supportLists]-->12.<!--[endif]-->检查Swift 日志声明，应该能看到这些：

proxy-server Invalid user token - deferring reject downstream
proxy-server Authorizing from an overriding middleware (i.e: tempurl) (txn: ...)
proxy-server ... IPWhitelistMiddleware
proxy-server Remote IP: 198.51.100.12 (txn: ...)
proxy-server Allow IPs: set(['203.0.113.68']) (txn: ...)
proxy-server IP 198.51.100.12 denied access to Account=AUTH_... Container=None. Not in set(['203.0.113.68']) (txn: ...)
这里能看到请求被拒绝，因为ip地址没有被放到允许列表中。

<!--[if !supportLists]-->13.<!--[endif]-->在DevStack 实例上增加一些元数据配置，允许远程的主机访问。

<!--[if !supportLists]-->a.    <!--[endif]-->先按Ctrl-A 再按 0

<!--[if !supportLists]-->b.    <!--[endif]-->swift post --meta allow-dev:198.51.100.12 middleware-test 

<!--[if !supportLists]-->14.<!--[endif]-->再试一遍第9步的命令，应该就可以了。

功能测试不能代替单元和集成测试，可以作为初步测试的一部分。

采用Python Paste 框架，一种样式可以被所有的项目采用，只需要创建一个中间件并植入。中间件作为项目pipeline的一部分，可以被其他服务调用，但不影响到项目核心代码。查看 pipeline 的值，在项目的 conf 或 ini 配置文件（在 /etc/<project> 目录）。 

当你做好一个中间件，我们鼓励你开源，在OpenStack邮件列表上让社区知道。也许其他人需要相同的功能。 他们可以使用您的代码，提供反馈，并可能做出贡献。如果存在足够的支持的话，也许你可以建议它被添加到swift官方的中间件(https://github.com/openstack/swift/tree/master/swift/common/middleware).

 

 Nova Scheduler(调度器)例子

许多OpenStack项目允许自定义的特定功能使用驱动的体系结构。您可以编写一个驱动程序，符合某个特定接口，并将通过配置植入。例如，您可以轻松插入一个新的nova调度器。

现有的nova调度器功能全面，并有很好的文档 (http://docs.openstack.org/folsom/openstack-compute/admin/content/ch_scheduling.html). 

然而，根据用户的使用情况，现有的调度程序可能不能满足您的要求。您可能需要创建一个新的调度器。

要创建一个调度程序必须从nova.scheduler.driver.Scheduler类继承。可以重写下面五种方法，必须重写带*两个方法：

<!--[if !supportLists]-->·         <!--[endif]-->update_service_capabilities 

<!--[if !supportLists]-->·         <!--[endif]-->hosts_up 

<!--[if !supportLists]-->·         <!--[endif]-->schedule_live_migration 

<!--[if !supportLists]-->·         <!--[endif]-->* schedule_prep_resize 

<!--[if !supportLists]-->·         <!--[endif]-->* schedule_run_instance 

为演示自定义OpenStack，我们将创建一个新的调度器，随机的将一个实例根据初始IP地址和名字的前缀放置于一个主机上。当你有一组用户在某一个子网上，希望所有实例在这个子网内的主机启动，这样的例子可能是有用的。

[Note]
 Note
 
此示例只用于说明目的。没有进一步的开发和测试它不应被用来作为实际的调度程序。
 

当你加入用screen –r stack 启动加入stack.sh屏幕会话时，会得到很多屏幕会话：

0$ shell*  1$ key  2$ g-reg  3$ g-api  4$ n-api  5$ n-cpu  6$ n-crt  7$ n-net  8-$ n-sch ...
<!--[if !supportLists]-->·         <!--[endif]-->shell . 能完成一些工作的shell

<!--[if !supportLists]-->·         <!--[endif]-->key . keystone服务

<!--[if !supportLists]-->·         <!--[endif]-->g-* . glance服务

<!--[if !supportLists]-->·         <!--[endif]-->n-* . nova服务

<!--[if !supportLists]-->·         <!--[endif]-->n-sch . nova scheduler调度服务

 

创建一个调度器并通过配置植入：

<!--[if !supportLists]-->1.    <!--[endif]-->OpenStack 代码在 /opt/stack目录，因此到这个目录并编辑调度器模块：

<!--[if !supportLists]-->a.    <!--[endif]-->cd /opt/stack/nova 

<!--[if !supportLists]-->b.    <!--[endif]-->vim nova/scheduler/ip_scheduler.py 

<!--[if !supportLists]-->2.    <!--[endif]-->复制以下代码：

# vim: tabstop=4 shiftwidth=4 softtabstop=4

# Copyright (c) 2013 OpenStack Foundation

# All Rights Reserved.

#

#    Licensed under the Apache License, Version 2.0 (the "License"); you may

#    not use this file except in compliance with the License. You may obtain

#    a copy of the License at

#

#   http://www.apache.org/licenses/LICENSE-2.0

#

#    Unless required by applicable law or agreed to in writing, software

#    distributed under the License is distributed on an "AS IS" BASIS, WITHOUT

#    WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the

#    License for the specific language governing permissions and limitations

#    under the License.

 

"""

IP Scheduler implementation

"""

 

import random

 

from nova import exception

from nova.openstack.common import log as logging

from nova import flags

from nova.scheduler import driver

 

FLAGS = flags.FLAGS

LOG = logging.getLogger(__name__)

 

 

class IPScheduler(driver.Scheduler):

    """

    Implements Scheduler as a random node selector based on

    IP address and hostname prefix.

    """

 

    def _filter_hosts(self, hosts, hostname_prefix):

        """Filter a list of hosts based on hostname prefix."""

 

        hosts = [host for host in hosts if host.startswith(hostname_prefix)]

        return hosts

 

    def _schedule(self, context, topic, request_spec, filter_properties):

        """

        Picks a host that is up at random based on

        IP address and hostname prefix.

        """

 

        elevated = context.elevated()

        hosts = self.hosts_up(elevated, topic)

 

        if not hosts:

            msg = _("Is the appropriate service running?")

            raise exception.NoValidHost(reason=msg)

 

        remote_ip = context.remote_address

 

        if remote_ip.startswith('10.1'):

            hostname_prefix = 'doc'

        elif remote_ip.startswith('10.2'):

            hostname_prefix = 'ops'

        else:

            hostname_prefix = 'dev'

 

        hosts = self._filter_hosts(hosts, hostname_prefix)

        host = hosts[int(random.random() * len(hosts))]

        LOG.debug(_("Request from %(remote_ip)s scheduled to %(host)s")

                  % locals())

        return host

    def schedule_run_instance(self, context, request_spec,

                              admin_password, injected_files,

                              requested_networks, is_first_time,

                              filter_properties):

        """Attempts to run the instance"""

        instance_uuids = request_spec.get('instance_uuids')

        for num, instance_uuid in enumerate(instance_uuids):

            request_spec['instance_properties']['launch_index'] = num

            try:

                host = self._schedule(context, 'compute', request_spec,

                                      filter_properties)

                updated_instance = driver.instance_update_db(context,

                                                             instance_uuid)

                self.compute_rpcapi.run_instance(context,

                                                 instance=updated_instance, host=host,

                                                 requested_networks=requested_networks,

                                                 injected_files=injected_files,

                                                 admin_password=admin_password,

                                                 is_first_time=is_first_time,

                                                 request_spec=request_spec,

                                                 filter_properties=filter_properties)

            except Exception as ex:

                # NOTE(vish): we don't reraise the exception here to make sure

                # that all instances in the request get set to

                # error properly

                driver.handle_schedule_error(context, ex, instance_uuid,

                                         request_spec)

 

    def schedule_prep_resize(self, context, image, request_spec,

                             filter_properties, instance, instance_type,

                             reservations):

        """Select a target for resize."""

        host = self._schedule(context, 'compute', request_spec,

                              filter_properties)

        self.compute_rpcapi.prep_resize(context, image, instance,instance_type,host,reservations)

<!--[if !supportLists]-->3.    <!--[endif]-->在上下文中有很多有用的信息，request_spec，filter_properties，您可以用来决定从哪里调度实例。更多了解哪些属性可用，您可以将下列日志声明插入调度器的schedule_run_instance。

<!--[if !supportLists]-->4.  <!--[endif]-->LOG.debug(_("context = %(context)s") % {'context': context.__dict__})LOG.debug(_("request_spec = %(request_spec)s") % locals())LOG.debug(_("filter_properties = %(filter_properties)s") % locals())
<!--[if !supportLists]-->5.    <!--[endif]-->为将写好的调度器植入Nova ，修改配置文件：

LOG$ vim /etc/nova/nova.conf
<!--[if !supportLists]-->6.    <!--[endif]-->将 compute_scheduler_driver 改为：

LOGcompute_scheduler_driver=nova.scheduler.ip_scheduler.IPScheduler
<!--[if !supportLists]-->7.    <!--[endif]-->重新启动Nova scheduler 服务以实用新的调度器。启动并切换到n-sch screen：

<!--[if !supportLists]-->a.    <!--[endif]-->按 Ctrl-A 再按8

<!--[if !supportLists]-->b.    <!--[endif]-->按 Ctrl-C 以停掉服务

<!--[if !supportLists]-->c.    <!--[endif]-->重新启动服务

<!--[if !supportLists]-->8.    <!--[endif]-->Test your scheduler with the Nova CLI. Start by switching to the shell screen and finish by switching back to the n-sch screen to check the log output.

<!--[if !supportLists]-->9.    <!--[endif]-->用nova CLI测试调度程序。切换到shell screen执行，最后到n-sch屏查看输出。

<!--[if !supportLists]-->a.    <!--[endif]-->Ctrl-A， 0

<!--[if !supportLists]-->b.    <!--[endif]-->cd ~/devstack 

<!--[if !supportLists]-->c.    <!--[endif]-->source openrc 

<!--[if !supportLists]-->d.    <!--[endif]-->IMAGE_ID=`nova image-list | egrep cirros | egrep -v "kernel|ramdisk" | awk '{print $2}'` 

<!--[if !supportLists]-->e.    <!--[endif]-->nova boot --flavor 1 --image $IMAGE_ID scheduler-test 

<!--[if !supportLists]-->f.     <!--[endif]-->Ctrl-A ， 8

<!--[if !supportLists]-->10.<!--[endif]-->在日志中可以看到以下内容：

LOG2013-02-27 17:39:31 DEBUG nova.scheduler.ip_scheduler [req-... demo demo] Request from 50.56.172.78 scheduled to
devstack-nova from (pid=4118) _schedule /opt/stack/nova/nova/scheduler/ip_scheduler.py:73
这样的功能测试不能代替单元测试和集成测试，但它是一个很好的开始。

采用驱动器架构，一个类似的样式可以被所有的项目采用，只需要创建一个适合驱动器的模块和类并配置植入。你的代码在功能被使用时被其他服务调用，但不影响到项目核心代码。为分辨一个项目是否采用驱动器架构，可以通过在项目的/ect/<project>目录下的 conf配置文件中查找’driver’ 值。

当你的调度器做好，我们鼓励你开源，发在OpenStack邮件列表上让社区知道。也许其他人需要相同的功能。 他们可以使用您的代码，提供反馈，并可能做出贡献。如果存在足够的支持的话，也许你可以建议它被添加到nova官方的中间件： schedulers (https://github.com/openstack/nova/tree/master/nova/scheduler).

 

 Dashboard

Dashboard 是基于python的 Django (https://www.djangoproject.com/) web 应用框架。最好的指南在 Build on Horizon (http://docs.openstack.org/developer/horizon/topics/tutorial.html).

