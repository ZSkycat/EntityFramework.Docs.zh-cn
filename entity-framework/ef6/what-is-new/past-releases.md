---
title: 以前的版本的 Entity Framework 的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 1060bb99-765f-4f32-aaeb-d6635d3dbd3e
ms.openlocfilehash: 4fa27f8259ecc011d9d30080aee3c44353ef533d
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283987"
---
# <a name="past-releases-of-entity-framework"></a>实体框架的旧版本

实体框架的第一个版本已于 2008 年，作为.NET Framework 3.5 SP1 和 Visual Studio 2008 SP1 的一部分发布。

从 EF4.1 版本具有附带开始[EntityFramework NuGet 包](https://www.nuget.org/packages/EntityFramework/)-当前有一个在 NuGet.org 上最受欢迎的包。

版本 4.1 和之间 5.0，EntityFramework NuGet 包扩展 EF 库与随.NET Framework 的一部分。   

从版本 6 开始，EF 成为一个开源项目，并还脱离完全外窗体.NET Framework。
这意味着，当将 EntityFramework 版本 6 NuGet 包添加到应用程序，会获取不依赖于.NET Framework 的一部分发布的 EF 位 EF 库的完整副本。
这有助于加快开发速度和新功能的交付的某种程度上。

2016 年 6 月，我们发布了 EF Core 1.0。 EF Core 基于新的基本代码，专为更轻型且可扩展的 EF 版本。
当前 EF Core 是开发的 Microsoft 的实体框架团队的重点。
这意味着没有新的主要功能，计划于 EF6。 但是 EF6 仍然作为一个开放源代码项目和受支持的 Microsoft 产品进行维护。

这是过去版本的列表中按时间顺序逆序，与每个版本中引入了新功能的信息。

## <a name="ef-613"></a>EF 6.1.3
EF 6.1.3 运行时已于 2015 年 10 月发布到 NuGet。
此版本包含优先级较高的缺陷和回归 6.1.2 上报告的唯一修补程序版本。
修复包括：

- 查询： EF 6.1.2 中回归： OUTER APPLY 引入和更复杂的查询 1 对 1 关系和"let"子句
- TPT 问题隐藏基类中继承的类的属性
- DbMigration.Sql 失败时的文本中包含单词 go
- 为 UnionAll 和 Intersect 修整支持创建兼容性标志
- 使用多个包括查询不适用于 6.1.2 （使用 6.1.1）
- 从 EF 6.1.1 到 6.1.2 升级后的"您有 SQL 语法错误"异常

## <a name="ef-612"></a>EF 6.1.2
EF 6.1.2 运行时在 2014 年 12 月发布到 NuGet。
此版本是主要是关于 bug 修复。 我们也可以接受一些来自社区的成员的值得注意的内容更改：
- **查询缓存参数可以配置从 app/web.configuration 文件**
    ``` xml
    <entityFramework>
      <queryCache size='1000' cleaningIntervalInSeconds='-1'/>
    </entityFramework>
    ```
- **SqlFile 和 SqlResource 方法 DbMigration**允许您运行 SQL 脚本作为文件存储或嵌入的资源。

## <a name="ef-611"></a>EF 6.1.1
EF 6.1.1 运行时在 2014 年 6 月发布到 NuGet。
此版本包含很多人遇到的问题的修补程序。 等：
- 打开具有在 EF6 设计器中的小数精度 EF5 edmx 设计器： 错误
- 对 LocalDB 的默认实例检测逻辑不使用 SQL Server 2014

## <a name="ef-610"></a>EF 6.1.0
EF 6.1.0 运行时在 2014 年 3 月发布到 NuGet。
此项次要更新，包括大量的新功能：

- **工具合并**提供一致的方法来创建新的 EF 模型。 此功能[扩展 ADO.NET 实体数据模型向导，以支持创建 Code First 模型](~/ef6/modeling/code-first/workflows/existing-database.md)，包括从现有数据库反向工程。 这些功能都在 EF Power Tools Beta 质量以前不可用。
- **[处理的事务提交失败](~/ef6/fundamentals/connection-resiliency/commit-failures.md)** 提供了该协议使用新引入的功能以截获事务操作 CommitFailureHandler。 CommitFailureHandler 允许连接故障自动恢复，同时提交事务。
- **[IndexAttribute](~/ef6/modeling/code-first/data-annotations.md)** 允许通过将指定的索引`[Index]`Code First 模型中属性的属性 （或属性）。 代码首先将然后对应的索引在数据库中创建。
- **公共映射 API**提供对 EF 对如何属性和类型映射到数据库中列和表的信息的访问。 在以前的版本此 API 的内部。
- **[能够配置通过 App/Web.config 文件侦听器](~/ef6/fundamentals/configuring/config-file.md)** 允许无需重新编译应用程序添加的侦听器。
- **System.Data.Entity.Infrastructure.Interception.DatabaseLogger**是一个新的侦听器，它可以更轻松地记录到文件中的所有数据库操作。 与以前的功能组合，此操作可以轻松地[的部署的应用程序的数据库操作的日志记录切换](~/ef6/fundamentals/configuring/config-file.md)，而无需重新编译。
- **迁移模型更改检测**已得到改进，以便更准确; 更改检测进程的性能已得到了增强已搭建基架的迁移。
- **性能改进**包括在初始化过程中减少的数据库操作，在 LINQ 查询中的 null 相等比较的优化更快地查看生成 （模型创建） 中更多方案，和更高效具有多个关联的被跟踪实体的具体化。

## <a name="ef-602"></a>EF 6.0.2
EF 6.0.2 运行时已于 2013 年 12 月发布到 NuGet。
此修补程序版本仅限于解决问题，引入了 EF6 版本 （回归性能/行为自 EF5） 中。

## <a name="ef-601"></a>EF 6.0.1
在 EF 6.0.1 运行时发布到 NuGet 同时使用 EF 6.0.0 的 2013 年 10 月因为后者已嵌入了锁定下之前的几个月的 Visual Studio 的版本中。
此修补程序版本仅限于解决问题，引入了 EF6 版本 （回归性能/行为自 EF5） 中。
最显著的更改是在 EF 模型的预热期间解决一些性能问题。
这是非常重要，因为预热性能已在 EF6 和这些问题的一个方面抵消一些在 EF6 中所做的其他性能提升。

## <a name="ef-60"></a>EF 6.0
EF 6.0.0 运行时已于 2013 年 10 月发布到 NuGet。
这是在其完整的 EF 运行时中包含的第一个版本[EntityFramework NuGet 包](https://www.nuget.org/packages/EntityFramework/)这不依赖于 EF 位的.NET Framework 的一部分。
将运行时的其余部分移动到 NuGet 包所需中断现有代码的更改的数。
请参阅章节[升级到 Entity Framework 6](upgrading-to-ef6.md)有关升级所需的手动步骤的详细信息。

此版本包括了许多新功能。
以下功能适用于使用 Code First 或 EF 设计器创建的模型：

- **[异步查询和保存](~/ef6/fundamentals/async.md)** .NET 4.5 中添加了对引入的基于任务的异步模式支持。
- **[连接复原](~/ef6/fundamentals/connection-resiliency/retry-logic.md)** 启用从暂时性连接故障自动恢复。
- **[基于代码的配置](~/ef6/fundamentals/configuring/code-based.md)** 提供选项来在代码中执行配置 – 在配置文件 – 传统上执行。
- **[依赖项解析](~/ef6/fundamentals/configuring/dependency-resolution.md)** 引入了支持的服务定位器模式，我们已将它们分离出来可以替换为自定义实现的功能的一些项。
- **[拦截/SQL 日志记录](~/ef6/fundamentals/logging-and-interception.md)** 提供低级别构建基块，用于构建在最前面的简单 SQL 日志记录的 EF 操作的拦截。
- **可测试性改进**更加轻松地创建 test double 的 DbContext 和 DbSet 时[使用的模拟框架](~/ef6/fundamentals/testing/mocking.md)或[编写你自己 test double](~/ef6/fundamentals/testing/writing-test-doubles.md)。
- **[现在可以使用已打开 DbConnection 创建 DbContext](~/ef6/fundamentals/connection-management.md)** 这样位置是非常有帮助如果创建上下文时，可以打开该连接的方案 (例如，共享组件之间的连接位置您可以保证连接的状态）。
- **[改进的事务支持](~/ef6/saving/transactions.md)** framework 以及创建框架内的事务的改进的方法的外部事务提供支持。
- **枚举、 空间和更好的性能，在.NET 4.0** -通过移动用于到 EF NuGet 包，.NET Framework 中是我们现在将能够从 EF5.net 提供枚举支持、 空间数据类型和性能改进的核心组件4.0。
- **改进了在 LINQ 查询中的 Enumerable.Contains 性能**。
- **改进了的预热时间 （视图生成）**，尤其是对于大型模型。
- **可插入复数&amp;单数形式服务**。
- **等于或 GetHashCode 的自定义实现**实体上现在支持类。
- **DbSet.AddRange/RemoveRange**提供以优化的方式添加或删除多个实体集中。
- **DbChangeTracker.HasChanges**提供简单且高效的方式来查看是否有任何挂起的更改保存到数据库。
- **Sqlcefunctions 与**提供 SQL Compact 等效于 SqlFunctions。

以下功能将仅应用于代码优先：

- **[自定义代码优先约定](~/ef6/modeling/code-first/conventions/custom.md)** 允许编写您自己的约定来帮助避免重复的配置。 我们提供适用于轻量级约定，以及一些更复杂的构建基块的简单 API，以便您可以编写更复杂的约定。
- **[第一个映射到 Insert/Update/Delete 存储过程的代码](~/ef6/modeling/code-first/fluent/cud-stored-procedures.md)** 现在支持。
- **[幂等迁移脚本](~/ef6/modeling/code-first/migrations/index.md)** ，可以生成可以升级到最新版本的任何版本的数据库的 SQL 脚本。
- **[可配置的迁移历史记录表](~/ef6/modeling/code-first/migrations/history-customization.md)** 可用于自定义迁移历史记录表的定义。 这是对于需要适当的数据类型等来为要正常工作的迁移历史记录表指定的数据库提供程序特别有用。
- **每个数据库的多个上下文**中删除一个 Code First 模型，每个数据库时使用迁移或 Code First 自动创建数据库的上一个限制。
- **[DbModelBuilder.HasDefaultSchema](~/ef6/modeling/code-first/fluent/types-and-properties.md)** 是一个新的代码第一个 API，允许在一个位置配置的 Code First 模型的默认数据库架构。 以前的代码优先的默认架构是硬编码到&quot;dbo&quot;和以配置表所属的架构的唯一方式是通过 ToTable API。
- **DbModelBuilder.Configurations.AddFromAssembly 方法**允许您轻松地添加与 Code First Fluent API 使用配置类时的程序集中定义的所有配置类。
- **[自定义迁移操作](http://romiller.com/2013/02/27/ef6-writing-your-own-code-first-migration-operations/)** 可让你将添加要在基于代码的迁移过程中使用的其他操作。
- **默认事务隔离级别更改为 READ_COMMITTED_SNAPSHOT**创建使用 Code First，从而获得更多的可伸缩性和更少的死锁的数据库。
- **实体和复杂类型现在可以是 nestedinside 类**。 |

## <a name="ef-50"></a>EF 5.0
EF 5.0.0 运行时已于 2012 年 8 月发布到 NuGet。
此版本引入了一些新功能，包括枚举支持、 表值函数、 空间数据类型和各种性能改进。

Visual Studio 2012 中的实体框架设计器还引入了支持每个模型，多个关系图上的存储过程设计面和批处理导入的形状的颜色设置。

下面是我们整理了专用于 EF 5 发布内容的列表。

-   [EF 5 发布文章](https://blogs.msdn.com/b/adonet/archive/2012/08/15/ef5-released.aspx)
-   EF5 中的新增功能
    -   [枚举支持中的代码首先](~/ef6/modeling/code-first/data-types/enums.md)
    -   [在 EF 设计器中的枚举支持](~/ef6/modeling/designer/data-types/enums.md)
    -   [在代码中首先空间数据类型。](~/ef6/modeling/code-first/data-types/spatial.md)
    -   [在 EF 设计器中的空间数据类型](~/ef6/modeling/designer/data-types/spatial.md)
    -   [针对空间类型的提供程序支持](~/ef6/fundamentals/providers/spatial-support.md)
    -   [表值函数](~/ef6/modeling/designer/advanced/tvfs.md)
    -   [每个模型的多个关系图](~/ef6/modeling/designer/multiple-diagrams.md)
-   设置您的模型
    -   [创建模型](~/ef6/modeling/index.md)
    -   [连接和模型](~/ef6/fundamentals/configuring/connection-strings.md)
    -   [性能注意事项](~/ef6/fundamentals/performance/perf-whitepaper.md)
    -   [使用 Microsoft SQL Azure](~/ef6/fundamentals/connection-resiliency/retry-logic.md)
    -   [配置文件设置](~/ef6/fundamentals/configuring/config-file.md)
    -   [术语表](~/ef6/resources/glossary.md)
    -   Code First
        -   [代码优先到新的数据库 （演练和视频）](~/ef6/modeling/code-first/workflows/new-database.md)
        -   [Code First 到现有数据库 （演练和视频）](~/ef6/modeling/code-first/workflows/existing-database.md)
        -   [约定](~/ef6/modeling/code-first/conventions/built-in.md)
        -   [数据注释](~/ef6/modeling/code-first/data-annotations.md)
        -   [Fluent API 的配置/映射属性和类型](~/ef6/modeling/code-first/fluent/types-and-properties.md)
        -   [Fluent API-配置关系](~/ef6/modeling/code-first/fluent/relationships.md)
        -   [使用 VB.NET Fluent API](~/ef6/modeling/code-first/fluent/vb.md)
        -   [Code First 迁移](~/ef6/modeling/code-first/migrations/index.md)
        -   [自动 Code First 迁移](~/ef6/modeling/code-first/migrations/automatic.md)
        -   [Migrate.exe](~/ef6/modeling/code-first/migrations/migrate-exe.md)
        -   [定义 Dbset](~/ef6/modeling/code-first/dbsets.md)
    -   EF 设计器
        -   [模型的第一个 （演练和视频）](~/ef6/modeling/designer/workflows/model-first.md)
        -   [数据库优先 （演练和视频）](~/ef6/modeling/designer/workflows/database-first.md)
        -   [复杂类型](~/ef6/modeling/designer/data-types/complex-types.md)
        -   [关联/关系](~/ef6/modeling/designer/relationships.md)
        -   [TPT 继承模式](~/ef6/modeling/designer/inheritance/tpt.md)
        -   [TPH 继承模式](~/ef6/modeling/designer/inheritance/tph.md)
        -   [使用存储过程的查询](~/ef6/modeling/designer/stored-procedures/query.md)
        -   [具有多个结果集的存储的过程](~/ef6/modeling/designer/advanced/multiple-result-sets.md)
        -   [插入、 更新和删除的存储过程](~/ef6/modeling/designer/stored-procedures/cud.md)
        -   [将一个实体映射到多个表 （实体拆分）](~/ef6/modeling/designer/entity-splitting.md)
        -   [将多个实体映射到一个表 （表拆分）](~/ef6/modeling/designer/table-splitting.md)
        -   [定义查询](~/ef6/modeling/designer/advanced/defining-query.md)
        -   [代码生成模板](~/ef6/modeling/designer/codegen/index.md)
        -   [恢复到 ObjectContext](~/ef6/modeling/designer/codegen/legacy-objectcontext.md)
-   使用您的模型
    -   [使用 DbContext](~/ef6/fundamentals/working-with-dbcontext.md)
    -   [查询/查找实体](~/ef6/querying/index.md)
    -   [使用关系](~/ef6/fundamentals/relationships.md)
    -   [加载相关的实体](~/ef6/querying/related-data.md)
    -   [使用本地数据](~/ef6/querying/local-data.md)
    -   [N 层应用程序](~/ef6/fundamentals/disconnected-entities/index.md)
    -   [原始 SQL 查询](~/ef6/querying/raw-sql.md)
    -   [乐观并发模式](~/ef6/saving/concurrency.md)
    -   [使用代理](~/ef6/fundamentals/proxies.md)
    -   [自动检测更改](~/ef6/saving/change-tracking/auto-detect-changes.md)
    -   [非跟踪查询](~/ef6/querying/no-tracking.md)
    -   [加载方法](~/ef6/querying/load-method.md)
    -   [添加 / 附加和实体状态](~/ef6/saving/change-tracking/entity-state.md)
    -   [使用属性值](~/ef6/saving/change-tracking/property-values.md)
    -   [数据绑定与 WPF (Windows Presentation Foundation)](~/ef6/fundamentals/databinding/wpf.md)
    -   [数据绑定与 WinForms （Windows 窗体）](~/ef6/fundamentals/databinding/winforms.md)

## <a name="ef-431"></a>EF 4.3.1
EF 4.3.1 运行时在不久之后 EF 4.3.0 2012 年 2 月发布到 NuGet。
此修补程序版本包含到 EF 4.3 版本一些 bug 修复，并引入了对使用 Visual Studio 2012 使用 EF 4.3 客户更好地 LocalDB 支持。

下面是我们整理了专门的 EF 4.3.1 版本的内容的列表，大部分为 EF 4.1 提供的内容仍也适用于 EF 4.3。

-   [EF 4.3.1 发布博客文章](https://blogs.msdn.com/b/adonet/archive/2012/02/29/ef4-3-1-and-ef5-beta-1-available-on-nuget.aspx)

## <a name="ef-43"></a>EF 4.3
EF 4.3.0 运行时已于 2012 年 2 月发布到 NuGet。
此版本中提供了新的 Code First 迁移功能，允许由 Code First 来以增量方式更改，因为 Code First 模型发展后创建的数据库。

下面是我们整理了专用于 EF 4.3 版本的内容的列表，大部分的 EF 4.1 提供的内容仍适用于 EF 4.3 以及：
-   [EF 4.3 发布文章](https://blogs.msdn.com/b/adonet/archive/2012/02/09/ef-4-3-released.aspx)
-   [EF 4.3 基于代码的迁移演练](https://blogs.msdn.com/b/adonet/archive/2012/02/09/ef-4-3-code-based-migrations-walkthrough.aspx)
-   [EF 4.3 自动迁移演练](https://blogs.msdn.com/b/adonet/archive/2012/02/09/ef-4-3-automatic-migrations-walkthrough.aspx)

## <a name="ef-42"></a>EF 4.2
EF 4.2.0 运行时已于 2011 年 11 月发布到 NuGet。
此版本包括对 EF 4.1.1 的 bug 修复版本。
因为此版本仅包含 bug 修复它可能是 EF 4.1.2 修补版本，但我们选择移动到 4.2 可使我们能够释放 4.1.x 中所使用的修补程序版本号，并采用基于日期离开移动[语义 Versionsing](https://semver.org)语义化版本控制的标准。

下面是我们整理了专用于 EF 4.2 版本的内容的列表，EF 4.1 提供的内容仍也适用于 EF 4.2。

-   [EF 4.2 发布文章](https://blogs.msdn.com/b/adonet/archive/2011/11/01/ef-4-2-released.aspx)
-   [代码优先演练](https://blogs.msdn.com/b/adonet/archive/2011/09/28/ef-4-2-code-first-walkthrough.aspx)
-   [模型和数据库的第一个演练](https://blogs.msdn.com/b/adonet/archive/2011/09/28/ef-4-2-model-amp-database-first-walkthrough.aspx)

## <a name="ef-411"></a>EF 4.1.1
EF 4.1.10715 运行时已于 2011 年 7 月发布到 NuGet。
除了 bug 修复此修补程序版本引入了一些组件，以方便您的设计时工具可使用 Code First 模型。
这些组件使用 Code First 迁移 （包括在 EF 4.3） 和 EF Power Tools。

您会注意到奇怪的版本编号 4.1.10715 包。
要使用基于日期的修补程序版本，我们决定采用之前，我们使用[语义化版本控制](https://semver.org)。
将此与 EF 4.1 修补程序 1 （或 EF 4.1.1） 的版本。

下面是我们整理了有关 4.1.1 的内容的列表，版本：

-   [EF 4.1.1 发布帖子](https://blogs.msdn.com/b/adonet/archive/2011/07/25/ef-4-1-update-1-released.aspx)

## <a name="ef-41"></a>EF 4.1
EF 4.1.10331 运行时才能在 NuGet 上发布在 2011 年 4 月的第一个。
此版本中提供了简化的 DbContext API 和 Code First 工作流。

你会注意到奇怪的版本号，4.1.10331，实际上应已 4.1。 此外还有 4.1.10311 版本应已 4.1.0-rc （'rc' 代表 release candidate）。
要使用基于日期的修补程序版本，我们决定采用之前，我们使用[语义化版本控制](https://semver.org)。

下面是内容的我们整理了 4.1 版本的列表。 大部分内容仍适用于更高版本的 Entity Framework:

-   [EF 4.1 发布文章](https://blogs.msdn.com/b/adonet/archive/2011/04/11/ef-4-1-released.aspx)
-   [代码优先演练](https://blogs.msdn.com/b/adonet/archive/2011/03/15/ef-4-1-code-first-walkthrough.aspx)
-   [模型和数据库的第一个演练](https://blogs.msdn.com/b/adonet/archive/2011/03/15/ef-4-1-model-amp-database-first-walkthrough.aspx)
-   [SQL Azure 联合和 Entity Framework](https://blogs.msdn.com/b/adonet/archive/2012/01/10/sql-azure-federations-and-the-entity-framework.aspx)

## <a name="ef-40"></a>EF 4.0
此版本包含在.NET Framework 4 和 Visual Studio 2010 中，在 2010 年 4 月。
此版本中的重要新功能包括 POCO 支持、 外键映射、 延迟加载、 可测试性改进，可自定义代码生成和模型优先工作流。

虽然它已实体框架的第二个版本，它名为 EF 4，以符合其附带的.NET Framework 版本。
在此版本中，我们将开始使实体框架在 NuGet 上可用和采用语义化版本控制，因为我们已无法再绑定到.NET Framework 版本。

请注意，某些后续版本的.NET Framework 附带的包含 EF 位对重要更新。
事实上，许多 EF 5.0 的新功能实现为这些位上的改进。
但是，ef 合理化版本控制，以便我们继续来指代作为 EF 4.0 运行时，是.NET Framework 的一部分而包含所有较新版本的 EF 位[EntityFramework NuGet 包](https://www.nuget.org/packages/EntityFramework/)。         

## <a name="ef-35"></a>EF 3.5
实体框架的初始版本包含在.NET 3.5 Service Pack 1 和 Visual Studio 2008 SP1，在 2008 年 8 月发布。
此版本中提供基本的 O/RM 支持使用 Database First 工作流。
