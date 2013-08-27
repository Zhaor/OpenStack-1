备份和恢复
====================================

Chapter 15. 备份和恢复

要备份什么 

数据库备份 

文件系统备份

备份的恢复

在创建OpenStack备份策略时可以参考标准备份的最佳做法。例如，备份数据的频率是和你希望多快将数据恢复紧密相关的。

[Note]
 Note
 
如果不希望有任何数据丢失，应同时考虑高可用性和备份。
 

其他对备份需要考量的因素有：

<!--[if !supportLists]-->·         <!--[endif]-->需要多少份备份？ 

<!--[if !supportLists]-->·         <!--[endif]-->需要异地备份保存吗？

<!--[if !supportLists]-->·         <!--[endif]-->每隔多长时间需要备份测试？

与备份策略同样重要的是恢复策略(或至少是恢复测试)。

 要备份什么

OpenStack是由许多组件和可拆卸部件组成，备份关键数据是非常简单的。

本章仅介绍如何备份OpenStack组件运行需要的各种配置文件和各种数据库。本章不介绍如何备份在块存储设备中保存的对象或数据。一般来说，这些需要用户自己备份。

 数据库备份

在示例中的OpenStack架构，云控制器也作为MySQL服务器。这个MySQL服务器保存Nova, Glance, Cinder, Keystone的数据。所有这些数据库在一个地方，因此很容易创建数据库备份：

<!--[if !vml]-->Select Text<!--[endif]-->1

2
 # mysqldump --opt --all-databases >

openstack.sql
 

如果你只想备份一个库，例如nova，可以用：

<!--[if !vml]-->Select Text<!--[endif]-->1
 # mysqldump --opt nova > nova.sql
 

您可以轻松地创建一个cron作业来自动执行此过程，每天运行以下脚本：

<!--[if !vml]-->Select Text<!--[endif]-->1

2

3

4

5

6

7
 #!/bin/bash

backup_dir="/var/lib/backups/mysql"

filename="${backup_dir}/mysql-`hostname`-`eval date +%Y%m%d`.sql.gz"

# Dump the entire MySQL database

/usr/bin/mysqldump --opt --all-databases | gzip > $filename 

# Delete backups older than 7 days

find $backup_dir -ctime +7 -type f -delete
 

此脚本复制整个MySQL数据库，并删除任何超过7天的备份。

 文件系统备份

计算

镜像与交付列表

认证信息

块存储

对象存储

 

本节讨论应定期备份哪些文件和目录，根据不同服务分类。

 计算
需要对云控制器和计算节点的 /etc/nova 目录做定期的备份。

日志需要做备份。但如果做了日志集中管理， 那么 /var/log/nova 目录不需要做备份。

/var/lib/nova是另一个需要备份的重要目录。一个例外的是计算节点的/var/lib/nova/instances子目录。此目录包含正在运行的实例的KVM虚机镜像。如果你需要对实例备份，可以备份此目录。当然是否需要备份要看具体情况，在大多数情况下，您不需要这样做。还要注意如果是对运行中的KVM实例备份，从备份恢复后可能启动不正常。

 镜像列表和交付
对 /etc/glance and /var/log/glance 备份也采用类似的规则。

/var/lib/glance 目录需要被备份。要注意 /var/lib/glance/images目录。如果用的是基于文件的Glance后端，/var/lib/glance/images 是存储软件镜像的。

有两种方法来确保这个目录的稳定性。首先是确保在RAID阵列上运行此目录，在一个磁盘发生故障时目录还可用。第二种方法是使用工具，如rsync将镜像复制到另一台服务器：

# rsync -az --progress /var/lib/glance/images backup-server:/var/lib/glance/images/

 认证信息
/etc/keystone and /var/log/keystone 的备份采用和其他组件同样的规则。

/var/lib/keystone目录一般应该没有任何数据，也可以备份。 

 块存储
/etc/cinder and /var/log/cinder的备份采用和其他组件同样的规则。 

/var/lib/cinder 也可以被备份

 Object Storage对象存储
/etc/swift 的备份非常重要。这个目录包括Swift配置文件和ring文件，及ring builder文件，丢失这些文件，集群中的数据将不能被访问。一个好方法是复制 builder文件和ring文件一起到每个节点，保存多个副本在你的存储集群中。

 备份的恢复

恢复备份是一个相当简单的过程。首先，请确保您正在恢复的服务没有运行。例如，要做一个完整的云控制器上恢复nova，首先停止所有nova服务：

<!--[if !vml]-->Select Text<!--[endif]-->1

2

3

4

5

6
 # stop nova-api

# stop nova-cert

# stop nova-consoleauth

# stop nova-novncproxy

# stop nova-objectstore

# stop nova-scheduler
 

做完后停止 MySQL:

<!--[if !vml]-->Select Text<!--[endif]-->1
 # stop mysql
 

现在可以引入一个备份的mysql库:

<!--[if !vml]-->Select Text<!--[endif]-->1
 # mysql nova < nova.sql
 

恢复备份的目录：

<!--[if !vml]-->Select Text<!--[endif]-->1

2
 # mv /etc/nova{,.orig}

# cp -a /path/to/backup/nova /etc/
 

文件恢复后，重启服务：

<!--[if !vml]-->Select Text<!--[endif]-->1

2

3

4

5
 # start mysql

# for i in nova-api nova-cert nova-consoleauth nova-novncproxy nova-objectstore nova-scheduler

> do

> start $i

> done
 

其他服务遵循相同的过程，对应各自的目录和数据库。

