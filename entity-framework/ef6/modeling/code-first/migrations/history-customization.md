---
title: 自定义迁移历史记录表的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: ed5518f0-a9a6-454e-9e98-a4fa7748c8d0
ms.openlocfilehash: e3faefc4b812ec4bc440ed2bb48747053d8cb1b3
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283688"
---
# <a name="customizing-the-migrations-history-table"></a>自定义迁移历史记录表
> [!NOTE]
> **仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。 如果使用的是早期版本，则部分或全部信息不适用。

> [!NOTE]
> 本文假定你知道如何在基本方案中使用 Code First 迁移。 如果不这样做，则你将需要读取[Code First 迁移](~/ef6/modeling/code-first/migrations/index.md)然后再继续。

## <a name="what-is-migrations-history-table"></a>迁移历史记录表是什么？

迁移历史记录表是用于存储有关迁移应用到数据库的详细信息的 Code First 迁移的表。 默认情况下在数据库中的表的名称是\_\_应用第一个迁移执行操作数据库时，将创建 MigrationHistory 和它。 在 Entity Framework 5 此表是系统表，如果应用程序使用 Microsoft Sql Server 数据库。 但是这已在 Entity Framework 6 和迁移历史记录表不再标记系统表。

## <a name="why-customize-migrations-history-table"></a>为什么要自定义迁移历史记录表？

迁移历史记录表是应仅由 Code First 迁移一起，并手动更改可能会中断迁移。 但是，有时默认配置不适合和表需要进行自定义，例如：

-   您需要更改名称和/或列以启用 3 方面<sup>rd</sup>参与方迁移提供程序
-   你想要更改的表的名称
-   您需要使用的非默认架构\_ \_MigrationHistory 表
-   若要将上下文的给定版本的其他数据，因此你需向表中添加一个额外的列

## <a name="words-of-precaution"></a>预防措施的单词

更改迁移历史记录表是功能强大，但您需要注意不过度编写。 EF 运行时当前不会检查自定义的迁移历史记录表是否与运行时兼容。 如果不是你的应用程序可能会导致运行时或不可预知的行为。 这一点尤为重要，如果使用每个数据库的多个上下文中这种情况下多个上下文可以使用相同的迁移历史记录表以存储有关迁移信息。

## <a name="how-to-customize-migrations-history-table"></a>如何自定义迁移历史记录表？

在开始之前需要知道，仅在应用第一次迁移之前，您可以自定义迁移历史记录表。 现在，对代码。

首先，需要创建一个 System.Data.Entity.Migrations.History.HistoryContext 类派生的类。 HistoryContext 类派生自 DbContext 类中，因此配置迁移历史记录表是非常类似于使用 fluent API 配置 EF 模型。 只需重写 OnModelCreating 方法并使用 fluent API 来配置表。

>[!NOTE]
> 配置 EF 模型时通常不需要调用基。重写 OnModelCreating 方法由于 DbContext.OnModelCreating() 具有空白正文从 onmodelcreating （)。 配置迁移历史记录表时，这不是这种情况。 在此情况下第一个需要在 onmodelcreating （） 重写中执行操作是实际调用基。Onmodelcreating （)。 这会将迁移历史记录表配置的默认方式，其中，然后调整中重写方法中。

让我们假设你想要重命名的迁移历史记录表，并将其放到名为"admin"的自定义架构。 除了数据库管理员希望您 MigrationId 列重命名为迁移\_id。  可以通过创建从 HistoryContext 派生的以下类实现此目的：

``` csharp
    using System.Data.Common;
    using System.Data.Entity;
    using System.Data.Entity.Migrations.History;

    namespace CustomizableMigrationsHistoryTableSample
    {
        public class MyHistoryContext : HistoryContext
        {
            public MyHistoryContext(DbConnection dbConnection, string defaultSchema)
                : base(dbConnection, defaultSchema)
            {
            }

            protected override void OnModelCreating(DbModelBuilder modelBuilder)
            {
                base.OnModelCreating(modelBuilder);
                modelBuilder.Entity<HistoryRow>().ToTable(tableName: "MigrationHistory", schemaName: "admin");
                modelBuilder.Entity<HistoryRow>().Property(p => p.MigrationId).HasColumnName("Migration_ID");
            }
        }
    }
```

自定义 HistoryContext 准备就绪后，您需要通过注册它通过使 EF 意识到这一点[基于代码的配置](https://msdn.com/data/jj680699):

``` csharp
    using System.Data.Entity;

    namespace CustomizableMigrationsHistoryTableSample
    {
        public class ModelConfiguration : DbConfiguration
        {
            public ModelConfiguration()
            {
                this.SetHistoryContext("System.Data.SqlClient",
                    (connection, defaultSchema) => new MyHistoryContext(connection, defaultSchema));
            }
        }
    }
```

这就是这么多了。 现在可以转到包管理器控制台中，Enable-migrations，添加迁移和最后更新数据库。 这会导致将添加到迁移历史记录表配置根据 HistoryContext 派生类中指定的详细信息的数据库。

![数据库](~/ef6/media/database.png)
