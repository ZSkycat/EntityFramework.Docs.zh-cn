---
title: 实体框架的 EF6 的案例研究
author: divega
ms.date: 10/23/2016
ms.assetid: cd5d3ae3-717d-4095-a2ef-0e8fd72b1a2f
ms.openlocfilehash: d7982a3f89ac1e57b48039d828f287adf6dc5068
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490867"
---
# <a name="microsoft-case-studies-for-entity-framework"></a>用于实体框架的 Microsoft 案例研究
案例研究此页上突出显示采用了实体框架的几个实际生产项目。
> [!NOTE]
> 这些案例研究详细的版本不再可在 Microsoft 网站上。 因此已删除的链接。

## <a name="epicor"></a>Epicor
Epicor 是在超过 150 个国家/地区的公司开发企业资源规划 (ERP) 解决方案 （具有超过 400 个开发人员） 的大型全球软件公司。
其旗舰产品 Epicor 9 为基础上面向服务的体系结构 (SOA) 使用.NET Framework。
面对大量客户请求，以提供支持的语言集成查询 (LINQ)，并且还想要减少其后端 SQL 服务器上的负载时，团队决定升级到 Visual Studio 2010 和.NET Framework 4.0。
使用 Entity Framework 4.0，他们就能够实现这些目标，并还大大简化了开发和维护。
具体而言，实体框架的丰富的 T4 支持将允许他们进行其生成代码的完全控制和自动生成性能保存功能，例如预编译的查询和缓存中。

> "最近与现有代码，某些性能测试，我们将展开，我们就能够减少 90%的请求 SQL Server。
这是因为在 ADO.NET Entity Framework 4。" – Erik johnson 的演示，副总裁产品研究  

## <a name="veracity-solutions"></a>真实性解决方案
具有获得会难以维护和扩展通过长期的事件规划软件系统，真实性解决方案用于 Visual Studio 2010 重新编写它，如在 Silverlight 4 上构建一个功能强大且易于使用丰富 Internet 应用程序。
使用.NET RIA 服务，他们就能够快速构建实体框架，避免代码重复和常规的验证和身份验证逻辑允许跨层之上的一个服务层。  

> "我们售出了实体框架时首次引入和 Entity Framework 4 已被证明是更好。
改进的工具，并很方便地操纵定义概念模型、 存储模型和这些模型之间的映射的.edmx 文件...使用实体框架中，我可以获得一天中使用该数据访问层，并生成它出我继续操作的过程。
实体框架是我们的事实数据访问层;我不知道为什么任何人都不会使用它。" – Joe McBride 高级开发人员

## <a name="nec-display-solutions-of-america"></a>NEC 显示解决方案的 America
NEC 想要输入的解决方案获益广告商和网络所有者并提高其自己的收入数字基于位置的广告的市场。
为了做到这一点，它启动对传统的广告活动中所需的手动过程自动化的 web 应用程序。
使用 ASP.NET、 Silverlight 3、 AJAX 和 WCF，数据访问层中的实体框架，与 SQL Server 2008 以及生成站点。

> "借助 SQL Server，我们认为我们无法获取我们需要使用实时和可靠性，以帮助确保始终将提供我们的任务关键型应用程序中的信息中的信息提供广告商和网络吞吐量"-Mike Corcoran主管 IT

## <a name="darwin-dimensions"></a>达尔文维度
使用各种各样的 Microsoft 技术，Darwin 的团队着手创建 Evolver-使用者可以使用来创建令人惊叹、 逼真的头像，用于在游戏、 动画和社交网络页的联机头像门户。
使用实体框架和组件，如 Windows Workflow Foundation (WF) 和 Windows Server AppFabric （高度可缩放内存中应用程序缓存） 中拉取的高效率优点，团队就能够在 35%中提供令人惊叹的产品开发时间。
尽管团队成员拆分跨多个国家/地区，团队遵循敏捷开发流程，并且每周发布。

 > "我们尝试不想创建起见技术的技术。 为启动时，这一点至关重要，我们利用技术节省时间和资金。
 .NET 是快速、 经济高效的开发的选择。" – Zachary Olsen，架构师  

## <a name="silverware"></a>Silverware
使用超过 15 年的经验开发对于小型和中型餐馆组销售点 (POS) 解决方案，Silverware 的开发团队着手增强其产品与更多的企业级功能，为了吸引更大连锁餐馆。
使用 Microsoft 的开发工具的最新版本，他们就能够生成新的解决方案四次超乎以往的速度。
密钥的新功能，如 LINQ 和实体框架进行简化了从 Crystal Reports 迁移到 SQL Server 2008 和 SQL Server Reporting Services (SSRS) 用于其数据存储和报告需求。

> "有效的数据管理是 SilverWare – 取得成功至关重要，这是为什么我们决定使用 SQL Reporting"。 -Nicholas Romanidis，主管 IT / 软件工程
