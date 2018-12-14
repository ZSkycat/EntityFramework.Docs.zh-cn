---
title: EF Core 1.0 中的新增功能 - EF Core
author: divega
ms.date: 10/27/2016
ms.assetid: 20A25111-AEBE-4BC2-83A5-3F651952DF72
uid: core/what-is-new/ef-core-1.0
ms.openlocfilehash: 409e16d762bca3ecd083ea191ad7b42aa0a6a275
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42996845"
---
# <a name="features-included-in-ef-core-10"></a>EF Core 1.0 中包含的功能

## <a name="platforms"></a>平台
### <a name="net-framework-451"></a>.NET Framework 4.5.1
包括控制台、WPF、WinForms、ASP.NET 4 等。
### <a name="net-standard-13"></a>.NET Standard 1.3
包括面向 Windows、OSX 和 Linux 上的 .NET Framework 和 .NET Core 的 ASP.NET Core。

## <a name="modelling"></a>建模
### <a name="basic-modelling"></a>基本建模
基于具有常用标量类型（`int`、`string` 等）的 get/set 属性的 POCO 实体。
### <a name="relationships-and-navigation-properties"></a>关系和导航属性
可以在模型中根据外键指定一对多和一对零或一对一关系。 简单集合或引用类型的导航属性可以与这些关系相关联。
### <a name="built-in-conventions"></a>内置约定
这些约定基于实体类的形状构建初始模型。
### <a name="fluent-api"></a>Fluent API
允许覆盖上下文中的 `OnModelCreating` 方法，以进一步配置按约定发现的模型。
### <a name="data-annotations"></a>数据注释
可添加到实体类/属性并影响 EF 模型的属性。 例如，`[Required]` 将告知 EF 某个属性是必需的。
### <a name="relational-table-mapping"></a>关系表映射
允许实体映射到表/列。
### <a name="key-value-generation"></a>键值生成
包括客户端生成和数据库生成。
### <a name="database-generated-values"></a>数据库生成的值
允许在插入（默认值）或更新（计算列）时由数据库生成值。
### <a name="sequences-in-sql-server"></a>SQL Server 中的序列
允许在模型中定义序列对象。
### <a name="unique-constraints"></a>UNIQUE 约束
允许定义备用键以及面向该键的关系的功能。
### <a name="indexes"></a>索引
在模型中定义索引会自动在数据库中引入索引。 此外，还支持唯一索引。
### <a name="shadow-state-properties"></a>阴影状态属性
允许在未在 .NET 类中声明和存储的模型中定义属性，但可以由 EF Core 进行跟踪和更新。 不需要在对象中公开这些属性时，通常用于外键属性。
### <a name="table-per-hierarchy-inheritance-pattern"></a>“每个层次结构一张表”继承模式
允许将继承层次结构中的实体使用鉴别器列保存到单个表中，以在数据库中针对给定记录标识实体类型。
### <a name="model-validation"></a>模型验证
检测模型中的无效模式并提供有用的错误消息。

## <a name="change-tracking"></a>Change tracking
### <a name="snapshot-change-tracking"></a>快照更改跟踪
通过将当前状态与原始状态的副本（快照）进行比较，可以自动检测实体中的更改。
### <a name="notification-change-tracking"></a>通知更改追踪
修改属性值后，允许实体通知更改跟踪器。
### <a name="accessing-tracked-state"></a>访问跟踪的状态
通过 `DbContext.Entry` 和 `DbContext.ChangeTracker`。
### <a name="attaching-detached-entitiesgraphs"></a>附加分离的实体/图
新的 `DbContext.AttachGraph` API 有助于将实体重新附加到上下文，以保存新的/修改的实体。

## <a name="saving-data"></a>保存数据
### <a name="basic-save-functionality"></a>基本保存功能
允许将对实体实例的更改持久保存到数据库。
### <a name="optimistic-concurrency"></a>开放式并发
防止由于从数据库中提取数据而覆盖其他用户所做的更改。
### <a name="async-savechanges"></a>异步 SaveChanges
在数据库处理从 `SaveChanges` 发出的命令时，可以释放当前线程以处理其他请求。
### <a name="database-transactions"></a>数据库事务
表示 `SaveChanges` 始终是原子（意味着它或者完全成功，或者不对数据库进行更改）。 还存在事务相关的 API，允许在上下文实例之间共享事务等。
### <a name="relational-batching-of-statements"></a>关系：批处理语句
通过将多个 INSERT/UPDATE/DELETE 命令批量放到数据库的单个往返路线中来提供更好的性能。

## <a name="query"></a>查询
### <a name="basic-linq-support"></a>基本 LINQ 支持
提供使用 LINQ 从数据库检索数据的功能。
### <a name="mixed-clientserver-evaluation"></a>混合客户端/服务器评估
使查询能够包含无法在数据库中评估的逻辑，因此，必须在将数据检索到内存后进行评估。
### <a name="notracking"></a>NoTracking
如果上下文无需监视实体实例的变化，可加快查询执行速度（在结果只读的情况下非常有用）。
### <a name="eager-loading"></a>预先加载
提供 `Include` 和 `ThenInclude` 方法来标识在查询时也应提取的相关数据。
### <a name="async-query"></a>异步查询
当数据库处理查询时，可以释放当前线程（及其相关资源）以处理其他请求。
### <a name="raw-sql-queries"></a>原始 SQL 查询
提供 `DbSet.FromSql` 方法以使用原始 SQL 查询提取数据。 也可以使用 LINQ 编写这些查询。

## <a name="database-schema-management"></a>数据库架构管理       
### <a name="database-creationdeletion-apis"></a>数据库创建/删除 API
多数旨在测试你希望在不使用迁移的情况下快速创建/删除数据库的位置。
### <a name="relational-database-migrations"></a>关系数据库迁移
允许关系数据库架构随模型更改而演进。
### <a name="reverse-engineer-from-database"></a>从数据库反向工程
基于现有关系数据库架构搭建 EF 模型基架。

## <a name="database-providers"></a>数据库提供程序
### <a name="sql-server"></a>SQL Server
连接到 Microsoft SQL Server 2008 及以上版本。
### <a name="sqlite"></a>SQLite
连接到 SQLite 3 数据库。
### <a name="in-memory"></a>内存中
旨在实现无需连接到真实的数据库即可轻松启用测试。
### <a name="3rd-party-providers"></a>第三方提供程序
多个提供程序可用于其他数据库引擎。 有关完整的列表，请参阅[数据库提供程序](../providers/index.md)。
