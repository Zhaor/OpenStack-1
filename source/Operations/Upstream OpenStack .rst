代码回馈(Upstream) OpenStack

=======================

 
Getting Help

Reporting Bugs

Join the OpenStack Community

Features and the Development Roadmap

How to Contribute to the Documentation

Security Information

Finding Additional Information

OpenStack是建立在一个繁荣的社区，提供很多帮助，并欢迎你的贡献。本节详细介绍的一些方法，您可以与其他人参与和互动。

 得到帮助

有几个可用的途径寻求援助。最快捷的方式来帮助社区帮助你。搜索问答Q&A网站，邮件列表归档，和与你们的问题类似的bug列表。如果你什么都找不到，请按照以下部分中的说明报告bug，或使用下面的支持渠道之一。

首先要查看的应该是官方文档，在： http://docs.openstack.org/。

你可以在 ask.openstack.org网站得到一些问题的解答。

邮件列表 (https://wiki.openstack.org/wiki/Mailing_Lists) 是另一个得到帮助的好地方。wiki 页上有更多关于邮件列表的信息。作为一个操作员，主要的列表包括：

<!--[if !supportLists]-->·         <!--[endif]-->General list: openstack@lists.openstack.org. 这个列表列出现在OpenStack的状态。是一个非常高流量的列表，每天有很多，很多邮件。

<!--[if !supportLists]-->·         <!--[endif]-->Operators list: openstack-operators@lists.openstack.org. 这个列表主要是OpenStack运维人员讨论，例如你。邮件比较少，每天1，2封的样子。

<!--[if !supportLists]-->·         <!--[endif]-->Development list: openstack-dev@lists.openstack.org. 这个列表主要讨论Openstack将来的发展，每天很多封邮件。

我们建议您订阅general 列表和operator的列表，但您必须设置一个过滤器来管理general列表。你还会发现wiki页面上有相关邮件列表的归档内容，您可以搜索讨论的内容。

Multiple IRC channels (https://wiki.openstack.org/wiki/IRC)讨论一般问题和开发相关内容。这个channel的地址： #openstack on irc.freenode.net.

 提交bug

Confirming & Prioritizing

Bug Fixing

After the Change is Accepted

运维人员经常会发现你的云系统中意外的问题。OpenStack是灵活的，你可能是唯一一个提交特定问题的。每个问题都是重要的，所以必须学习如何方便地提交一个bug报告。

所有OpenStack项目使用 Launchpad 做bug跟踪。在提交bug前，你需要在launchpad上创建一个用户。 

一旦你有了一个帐户，报告bug非常简单，只需确定问题发生或导致问题的那个项目。有时这是比较难的，但这些相关人员会乐意帮助将bug转移到正确的项目上。汇报bug的地址：

<!--[if !supportLists]-->·         <!--[endif]-->Report a bug in Nova (https://bugs.launchpad.net/nova/+filebug)

<!--[if !supportLists]-->·         <!--[endif]-->Report a bug in python-novaclient (https://bugs.launchpad.net/python-novaclient/+filebug)

<!--[if !supportLists]-->·         <!--[endif]-->Report a bug in Swift (https://bugs.launchpad.net/swift/+filebug)

<!--[if !supportLists]-->·         <!--[endif]-->Report a bug in python-swiftclient (https://bugs.launchpad.net/python-swiftclient/+filebug)

<!--[if !supportLists]-->·         <!--[endif]-->Report a bug in Glance (https://bugs.launchpad.net/glance/+filebug)

<!--[if !supportLists]-->·         <!--[endif]-->Report a bug in python-glanceclient (https://bugs.launchpad.net/python-glanceclient/+filebug)

<!--[if !supportLists]-->·         <!--[endif]-->Report a bug in Keystone (https://bugs.launchpad.net/keystone/+filebug)

<!--[if !supportLists]-->·         <!--[endif]-->Report a bug in python-keystoneclient (https://bugs.launchpad.net/python-keystoneclient/+filebug)

<!--[if !supportLists]-->·         <!--[endif]-->Report a bug in Quantum (https://bugs.launchpad.net/quantum/+filebug)

<!--[if !supportLists]-->·         <!--[endif]-->Report a bug in python-quantumclient (https://bugs.launchpad.net/python-quantumclient/+filebug)

<!--[if !supportLists]-->·         <!--[endif]-->Report a bug in Cinder (https://bugs.launchpad.net/cinder/+filebug)

<!--[if !supportLists]-->·         <!--[endif]-->Report a bug in python-cinderclient (https://bugs.launchpad.net/python-cinderclient/+filebug)

<!--[if !supportLists]-->·         <!--[endif]-->Report a bug in Horizon (https://bugs.launchpad.net/horizon/+filebug)

<!--[if !supportLists]-->·         <!--[endif]-->Report a bug with the documentation (http://bugs.launchpad.net/openstack-manuals/+filebug)

<!--[if !supportLists]-->·         <!--[endif]-->Report a bug with the API documentation (http://bugs.launchpad.net/openstack-api-site/+filebug)

写一个好的bug报告，以下过程是必不可少的。首先，搜索确保没有任何基于同样的问题的bug已经被申请。如果你找到一个，一定要点击"this bug affects X people， does this bug affect you? "。如果原来没有类似问题，可以报告bug，输入细节。它至少应包括：

<!--[if !supportLists]-->·         <!--[endif]-->Rlease or milestone，或提交ID对应于您所运行的软件。

<!--[if !supportLists]-->·         <!--[endif]-->你在发现bug所运行的操作系统和版本。

<!--[if !supportLists]-->·         <!--[endif]-->重现bug的步骤，包括什么错误。

<!--[if !supportLists]-->·         <!--[endif]-->没有bug的情况下预期的结果。

<!--[if !supportLists]-->·         <!--[endif]-->日志，阅读并理解您的日志文件，只上传相关部分。

做这些后，bug被创建：

<!--[if !supportLists]-->·         <!--[endif]-->Status: New 

在bug评论栏里，你可以贡献如何修复bug的方法，并将其设置为Triaged。或者你可以直接修复：将bug 分配给自己，设置其为In progress, 并建立一个分支，部署fix，然后请求将修改部分与主程序和并。但别太操之过急了，有错误会审等环节。

 确认和优先级划分
这个阶段是关于检查一个的bug的真实性和评估其影响。这些步骤需要bug supervisor 权限(通常限于核心团队)。如果bug缺乏正确的信息来重现或重要性的评估，bug被设置为：

<!--[if !supportLists]-->·         <!--[endif]-->Status: Incomplete 

一旦重现了问题(或者是100的信心，这确实是一个有效的bug)，和有权限进行这些操作，设置：

<!--[if !supportLists]-->·         <!--[endif]-->Status: Confirmed 

核心开发者(Core developers) 会根据其影响设置优先级：

<!--[if !supportLists]-->·         <!--[endif]-->Importance:< Bug impact>

Bug的影响被设为以下几级：

<!--[if !supportLists]-->1.    <!--[endif]-->Critical  如果bug，影响到了关键功能的工作，对所有用户，或产生了数据丢失。

<!--[if !supportLists]-->2.    <!--[endif]-->High  如果bug，影响了一个关键功能正常工作，对一部分人有影响(或者有了暂时解决方案)。

<!--[if !supportLists]-->3.    <!--[endif]-->Medium如果bug，影响了辅助功能的正常工作。

<!--[if !supportLists]-->4.    <!--[endif]-->Low  如果主要是表面的问题 。

<!--[if !supportLists]-->5.    <!--[endif]-->Wishlist这不是一个真正的bug，而是一个受欢迎的修改。

如果bug已经有了解决方案或补丁，将状态设置为Triaged 

 Bug 修复
在这个阶段，开发人员对此进行修复。 在此段时间，为避免重复工作他们应该设置：

<!--[if !supportLists]-->·         <!--[endif]-->Status: In progress 

<!--[if !supportLists]-->·         <!--[endif]-->Assignee:< yourself>

当修复做好后，可以提交并得到回顾检查。

 修改被接受
修改被核查，接受，并上传到master后，bug自动变成：

<!--[if !supportLists]-->·         <!--[endif]-->Status: Fix committed 

当修复补丁被放到版本中(或计划的版本中)，bug自动变成：

<!--[if !supportLists]-->·         <!--[endif]-->Milestone: Milestone the bug was fixed in

<!--[if !supportLists]-->·         <!--[endif]-->Status: Fix released 

 加入OpenStack社区

既然这本书你看到这里，你应该考虑成为社区的正式的个人会员并加入OpenStack基金会(https://www.openstack.org/join/)。

 OpenStack基金会是一个独立的机构，提供共享资源以帮助实现目标：保护，帮助，促进OpenStack软件和相关社区的发展，包括用户、开发人员和整个生态系统。我们都有责任让这个社区成为最好的。注册成为会员是参与的第一步。例如软件，对个人会员是免费的，每个人都可以拿到。

 功能和开发路径图

OpenStack版本发布周期是六个月，通常每年4月和10月发布。在每个周期开始，社区聚集在一起进行设计峰会。在峰会上，讨论未来版本的功能，优先级和计划。下面是在峰会上的一个发布周期示例，显示里程碑版本，code freeze，string freeze等的日期。里程碑是在周期内的临时版本，软件包提供下载和测试。Code freeze是停止添加新功能。 String freeze是停止在源代码中更改任何字符串的字符。



功能需求通常从Etherpad开始，这是个协同编辑工具，用来在设计峰会上说明和协调功能。这就对特定项目的Launchpad上建立了一个蓝图，Launchpad用来描述特征时显得更加正式。项目团队成员批准蓝图，然后可以开始开发。

因此，最快让功能要求得到审议的方式是在Etherpad创建你的想法，并在设计峰会做一次介绍。如果当时没有设计峰会，你也可以直接创建蓝图。可以看这个开发者的资料：如何创建蓝图 (http://vmartinezdelacruz.com/how-to-work-with-blueprints-without-losing-your-mind/) 

下一个版本的计划路标： Releases (http://status.openstack.org/release/).

为了了解将来版本的可能的功能，或查看原来版本的功能，可以看这里： OpenStack Compute (nova) Blueprints (https://blueprints.launchpad.net/nova), OpenStack Identity (keystone) Blueprints (https://blueprints.launchpad.net/keystone) 以及release notes. 

Release notes 在这里： OpenStack wiki:

Series
 Status
 Releases
 Date
 
Grizzly
 Under development, Release schedule 
 Due
 Apr 4, 2013
 
Folsom
 Current stable release, security-supported
 2012.2 
 Sep 27, 2012
 
2012.2.1 
 Nov 29, 2012
 
2012.2.2 
 Dec 13, 2012
 
2012.2.3 
 Jan 31, 2012
 
Essex
 Community-supported, security-supported
 2012.1 
 Apr 5, 2012
 
2012.1.1 
 Jun 22, 2012
 
2012.1.2 
 Aug 10, 2012
 
2012.1.3 
 Oct 12, 2012
 
Diablo
 Community-supported
 2011.3 
 Sep 22, 2011
 
2011.3.1 
 Jan 19, 2012
 
Cactus
 Deprecated
 2011.2 
 Apr 15, 2011
 
Bexar
 Deprecated
 2011.1 
 Feb 3, 2011
 
Austin
 Deprecated
 2010.1 
 Oct 21, 2010
 

 如何在文档方面贡献

OpenStack文档的工作包括操作员和管理员文档，API文档和用户文档。

做这本书并非官方的行为，但现在这本书是在你的手中，我们希望你能为它作出贡献。OpenStack文档遵循编码迭代的工作原则，包括错误日志记录、调查和修复。

就像编码一样， docs.openstack.org 网站用Gerrit review 系统更新，源文档保存在GitHub 在openstack-manuals (http://github.com/openstack/openstack-manuals/) 代码库，以及 api-site (http://github.com/openstack/api-site/) 代码库。以文档的形式。

在文档被公布之前可以查看，到 OpenStack Gerrit 服务器 review.openstack.org 查找 project:openstack/openstack-manuals or project:openstack/api-site.

如果要了解如何文档提交的方法等信息，可以在wiki上查看： How To Contribute (https://wiki.openstack.org/wiki/How_To_Contribute) 

 安全信息

作为一个社区，我们非常重视安全，并按照特定流程报告潜在的安全问题。我们警觉地追踪，修复和定期消除安全的风险。 请通过这里的流程报告您发现的安全问题。OpenStack漏洞管理团队是一个由来自OpenStack社区的安全专家组成非常小的团队。他们的工作是促进漏洞的报告，协调安全修复程序和处理漏洞信息的披露。具体来说，团队负责下列功能：

<!--[if !supportLists]-->·         <!--[endif]-->漏洞管理：通过社区成员或用户发现的所有漏洞可以报道团队。

<!--[if !supportLists]-->·         <!--[endif]-->漏洞追踪：团队将策划一系列对漏洞相关的问题的跟踪。这些问题有些是影响私人团队或产品的，但是一旦修复到位，所有漏洞都是公开的。

<!--[if !supportLists]-->·         <!--[endif]-->负责披露：作为承诺我们工作在一个安全社区，确保功劳给予报告安全问题和安全研究的人。

取决于问题的敏感性如何，我们提供两个方法来向漏洞管理团队报告问题：

<!--[if !supportLists]-->·         <!--[endif]-->在Launchpad打开一个bug并将其标记为'安全缺陷'。这使得bug只能被漏洞管理团队看到并处理。

<!--[if !supportLists]-->·         <!--[endif]-->如果问题极为敏感，发送加密电子邮件给OpenStack安全团队成员之一。在这里拿到GPG 密钥：(http://www.openstack.org/projects/openstack-security)

你可以找到所有安全相关的组，可以加入安全组： Security Teams (http://wiki.openstack.org/SecurityTeams). 漏洞管理的流程文档全部在： Vulnerability Management (https://wiki.openstack.org/wiki/VulnerabilityManagement).

 更多信息

除了这本书，还有很多其他信息。OpenStack 官网 (http://www.openstack.org) 是一个很好的开始点，有很多 OpenStack Docs (http://docs.openstack.org) 和 OpenStack API Docs (http://api.openstack.org) ，提供技术文档。 OpenStack wiki 包括Openstack各方面的大量信息，包括一个列表： recommended tools (https://wiki.openstack.org/wiki/OperationsTools ). 最后，有一些博客，列表在这里： Planet OpenStack (http://planet.openstack.org).

