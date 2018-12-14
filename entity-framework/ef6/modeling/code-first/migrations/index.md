---
title: Code First 迁移 - EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 36591d8f-36e1-4835-8a51-90f34f633d1e
ms.openlocfilehash: f408ef861a2992783142fa1483d1433ca710399a
ms.sourcegitcommit: 15022dd06d919c29b1189c82611ea32f9fdc6617
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2018
ms.locfileid: "47415791"
---
# <a name="code-first-migrations"></a>Code First 迁移
如果使用的是 Code First 工作流，推荐使用 Code First 迁移改进应用程序的数据库架构。 迁移提供一组允许以下操作的工具：

1. 创建可用于 EF 模型的初始数据库
2. 生成迁移以跟踪对 EF 模型所做的更改
2. 使数据库随时掌握这些更改

下方演练将概述实体框架中的 Code First 迁移。 可以完成整个演练或跳到感兴趣的主题。 包含以下主题：

## <a name="building-an-initial-model--database"></a>生成初始模型和数据库

开始使用迁移之前，需要会用到项目和 Code First 模型。 对于此演练，我们将使用规范的“博客”和“帖子”模型。

-   创建新的 MigrationsDemo 控制台应用程序
-   将最新版本的 EntityFramework NuGet 包添加到项目中
    -   “工具”–&gt;“库包管理器”–&gt;“包管理器控制台”
    -   运行 Install-Package EntityFramework 命令
-   添加 Model.cs 文件，其代码如下所示。 此代码定义了构成域模型的单个“博客”类和 EF Code First 上下文 BlogContext 类

  ``` csharp
      using System.Data.Entity;
      using System.Collections.Generic;
      using System.ComponentModel.DataAnnotations;
      using System.Data.Entity.Infrastructure;

      namespace MigrationsDemo
      {
          public class BlogContext : DbContext
          {
              public DbSet<Blog> Blogs { get; set; }
          }

          public class Blog
          {
              public int BlogId { get; set; }
              public string Name { get; set; }
          }
      }
  ```

-   现在我们拥有一个模型，可用它执行数据访问操作。 更新 Program.cs 文件，其代码如下所示。

  ``` csharp
      using System;
      using System.Collections.Generic;
      using System.Linq;
      using System.Text;

      namespace MigrationsDemo
      {
          class Program
          {
              static void Main(string[] args)
              {
                  using (var db = new BlogContext())
                  {
                      db.Blogs.Add(new Blog { Name = "Another Blog " });
                      db.SaveChanges();

                      foreach (var blog in db.Blogs)
                      {
                          Console.WriteLine(blog.Name);
                      }
                  }

                  Console.WriteLine("Press any key to exit...");
                  Console.ReadKey();
              }
          }
      }
  ```

-   运行应用程序，随即会创建 MigrationsCodeDemo.BlogContext 数据库。

    ![数据库  LocalDB](~/ef6/media/databaselocaldb.png)

## <a name="enabling-migrations"></a>启用迁移

现可对模型进行更多更改。

-   将 Url 属性引入“博客”类。

``` csharp
    public string Url { get; set; }
```

如果要再次运行应用程序，则会收到一个 InvalidOperationException，指出“创建数据库后，支持‘BlogContext’上下文的模型已发生变化。请考虑使用 Code First 迁移更新数据库 ([http://go.microsoft.com/fwlink/?LinkId=238269](https://go.microsoft.com/fwlink/?LinkId=238269))”。

如异常情况所述，可开始使用 Code First 迁移。 第一步是启用上下文迁移。

-   在包管理器控制台中运行 Enable-Migrations 命令

    此命令已将“迁移”文件夹添加到项目中。 此新文件夹包含两个文件：

-   **配置类。** 此类允许配置迁移对上下文的行为方式。 对于此演练，将只使用默认配置。
    由于项目中只有一个 Code First 上下文，因此 Enable-Migrations 已自动填充此配置适用的上下文类型。
-   **InitialCreate 迁移**。 之所以生成此迁移，是因为在启用迁移之前，我们已使用 Code First 创建了数据库。 已构建的迁移中的代码表示已在数据库中创建的对象。 在本演练中，即为具有 BlogId 和“名称”列的 Blog 表。 文件名包含时间戳，这样有助于排序。
    如果尚未创建数据库，则不会将此 InitialCreate 迁移添加到项目中。相反，第一次调用 Add-Migration 时，会将创建这些表的代码构建到新的迁移中。

### <a name="multiple-models-targeting-the-same-database"></a>针对同一数据库的多个模型

使用 EF6 之前的版本时，只能使用一个 Code First 模型生成/管理数据库的架构。 这是因为每个数据库的单个 \_\_MigrationsHistory 表无法识别哪些项属于哪个模型。

从 EF6 开始，配置类中包括 ContextKey 属性。 该属性充当每个 Code First 模型的唯一标识符。 \_\_MigrationsHistory 表中相应的列允许来自多个模型的项共享表。 默认情况下，此属性设置为上下文的完全限定名称。

## <a name="generating--running-migrations"></a>生成和运行迁移

Code First 迁移具有两个需要用户了解的主要命令。

-   Add-Migration 将基于自上次迁移创建以来对模型所做的更改来构建下一次迁移
-   Update-Database 将对数据库应用任意挂起的迁移

我们需要构建迁移来处理添加的新 Url 属性。 Add-Migration 命令可为这些迁移命名，仅需调用 AddBlogUrl。

-   在包管理器控制台中运行 Add-Migration AddBlogUrl 命令
-   现“迁移”文件夹中具有新的 AddBlogUrl 迁移。 迁移文件名以时间戳作为前缀，这样有助于排序

``` csharp
    namespace MigrationsDemo.Migrations
    {
        using System;
        using System.Data.Entity.Migrations;

        public partial class AddBlogUrl : DbMigration
        {
            public override void Up()
            {
                AddColumn("dbo.Blogs", "Url", c => c.String());
            }

            public override void Down()
            {
                DropColumn("dbo.Blogs", "Url");
            }
        }
    }
```

现可编辑或添加到此迁移，但所有内容看起来都很合适。 使用 Update-Database 将此迁移应用到数据库。

-   在包管理器控制台运行 Update-Database 命令
-   Code First 迁移将比较“迁移”文件夹中的迁移和已应用于数据库的迁移。 迁移会发现需应用 AddBlogUrl 迁移，并运行它。

MigrationsDemo.BlogContext 数据库现已更新，其中包含“博客”表中的 Url 列。

## <a name="customizing-migrations"></a>自定义迁移

到目前为止，我们已在未进行任何更改的情况下生成并运行了迁移。 现在我们来看看如何编辑默认生成的代码。

-   现在可对模型进行更多更改，我们将新的 Rating 属性添加到“博客”类

``` csharp
    public int Rating { get; set; }
```

-   同时添加一个新的“帖子”类

``` csharp
    public class Post
    {
        public int PostId { get; set; }
        [MaxLength(200)]
        public string Title { get; set; }
        public string Content { get; set; }

        public int BlogId { get; set; }
        public Blog Blog { get; set; }
    }
```

-   此外，再将“帖子”集合添加到“博客”类，以形成“博客”和“帖子”之间的另一层关系

``` csharp
    public virtual List<Post> Posts { get; set; }
```

使用 Add-Migration 命令使 Code First 迁移提供对迁移的最佳猜测 我们将调用此迁移 AddPostClass。

-   在包管理器控制台中运行 Add-Migration AddPostClass 命令。

Code First 迁移出色的构建了这些更改，但我们可能还需要做出一些更改：

1.  首先，将唯一索引添加到 Posts.Title 列（添加在以下代码的 22 和 29 行）。
2.  同时添加不可为 NULL 的 Blogs.Rating 列。 如果表中存在任何现有数据，则会为数据分配新列数据类型的 CLR 默认值（分级是整数，因此将为 0）。 但我们要将默认值指定为 3，以便“博客”表中的现有行以合适的分级开始。
    （可以看到以下代码的第 24 行指定的默认值）

``` csharp
    namespace MigrationsDemo.Migrations
    {
        using System;
        using System.Data.Entity.Migrations;

        public partial class AddPostClass : DbMigration
        {
            public override void Up()
            {
                CreateTable(
                    "dbo.Posts",
                    c => new
                        {
                            PostId = c.Int(nullable: false, identity: true),
                            Title = c.String(maxLength: 200),
                            Content = c.String(),
                            BlogId = c.Int(nullable: false),
                        })
                    .PrimaryKey(t => t.PostId)
                    .ForeignKey("dbo.Blogs", t => t.BlogId, cascadeDelete: true)
                    .Index(t => t.BlogId)
                    .Index(p => p.Title, unique: true);

                AddColumn("dbo.Blogs", "Rating", c => c.Int(nullable: false, defaultValue: 3));
            }

            public override void Down()
            {
                DropIndex("dbo.Posts", new[] { "Title" });
                DropIndex("dbo.Posts", new[] { "BlogId" });
                DropForeignKey("dbo.Posts", "BlogId", "dbo.Blogs");
                DropColumn("dbo.Blogs", "Rating");
                DropTable("dbo.Posts");
            }
        }
    }
```

已编辑的迁移准备就绪，所以我们使用 Update-Database 来更新数据库。 这次指定 –Verbose 标志，以便可以看到 Code First 迁移正在运行的 SQL。

-   在包管理器控制台中运行 Update-Database –Verbose 命令。

## <a name="data-motion--custom-sql"></a>数据移动/自定义 SQL

到目前为止，我们已经介绍了不更改或移动任何数据的迁移操作，现在来看看需要移动数据的操作。 目前还没有对数据移动的原生支持，但我们可以在脚本中的任何位置运行一些任意 SQL 命令。

-   将 Post.Abstract 属性添加到模型中。 稍后，我们将使用“内容”列开头的一些文本预填充现有帖子的“摘要”。

``` csharp
    public string Abstract { get; set; }
```

使用 Add-Migration 命令使 Code First 迁移提供对迁移的最佳猜测

-   在包管理器控制台中运行 Add-Migration AddPostAbstract 命令。
-   生成的迁移会处理架构更改，但我们还是希望使用每个帖子内容的前 100 个字符预填充“摘要”列。 要执行此操作，可在添加列之后下拉到 SQL 并运行 UPDATE 语句。
    （添加在以下代码的第 12 行中）

``` csharp
    namespace MigrationsDemo.Migrations
    {
        using System;
        using System.Data.Entity.Migrations;

        public partial class AddPostAbstract : DbMigration
        {
            public override void Up()
            {
                AddColumn("dbo.Posts", "Abstract", c => c.String());

                Sql("UPDATE dbo.Posts SET Abstract = LEFT(Content, 100) WHERE Abstract IS NULL");
            }

            public override void Down()
            {
                DropColumn("dbo.Posts", "Abstract");
            }
        }
    }
```

已编辑的迁移一切正常，所以我们可使用 Update-Database 来更新数据库。 我们将指定 –Verbose 标志，以便可以看到针对数据库运行的 SQL。

-   在包管理器控制台中运行 Update-Database –Verbose 命令。

## <a name="migrate-to-a-specific-version-including-downgrade"></a>迁移到特定版本（包括降级）

到目前为止，我们一直在升级到最新迁移，但用户有时可能希望升级/降级到特定迁移。

假设想在运行 AddBlogUrl 迁移后将数据库迁移到其之前的状态。 此时可使用 –TargetMigration 切换为降级到此迁移。

-   在包管理器控制台中运行 Update-Database –TargetMigration: AddBlogUrl 命令。

此命令将为 AddBlogAbstract 和 AddPostClass 迁移运行 Down 脚本。

如果想要一直回退到空数据库，可使用 Update-Database –TargetMigration: $InitialDatabase 命令。

## <a name="getting-a-sql-script"></a>获取 SQL 脚本

如果另一位开发人员希望在其计算机上进行这些更改，则只需在我们将更改签入源代码管理之后进行同步即可。 在获得我们的新迁移后，他们只需运行 Update-database 命令即可在本地应用更改。 但是，如果想将这些更改推送到测试服务器以及最终的产品，则可能需要一个可以传递给 DBA 的 SQL 脚本。

-   运行 Update-Database 命令，但是这次需指定 –Script 标志，以便将更改写入脚本，而不是应用更改。 我们还将指定要为其生成脚本的源和目标迁移。 我们希望脚本从空数据库 ($InitialDatabase) 转为最新版本（迁移 AddPostAbstract）。
    如果未指定目标迁移，迁移将使用最新的迁移作为目标。如果未指定源迁移，迁移将使用数据库的当前状态。
-   在包管理器控制台中运行 Update-Database -Script -SourceMigration: $InitialDatabase -TargetMigration: AddPostAbstract 命令

Code First 迁移将运行迁移管道，但并非是应用更改，而是将更改写入到 .sql 文件。 生成脚本后，将在 Visual Studio 中打开，以供查看或保存。

### <a name="generating-idempotent-scripts"></a>生成幂等脚本

从 EF6 开始，如果指定 –SourceMigration $InitialDatabase，则生成的脚本将为“幂等”。 幂等脚本可以将当前任何版本的数据库升级到最新版本（或使用 – TargetMigration 升级到指定版本）。 生成的脚本包括检查 \_\_MigrationsHistory 表的逻辑，并且仅应用以前未应用的更改。

## <a name="automatically-upgrading-on-application-startup-migratedatabasetolatestversion-initializer"></a>应用程序启动时自动升级（MigrateDatabaseToLatestVersion 初始值设定项）

如果要部署应用程序，则可能希望应用程序在启动时自动升级数据库（通过应用各种挂起的迁移）。 可以通过注册 MigrateDatabaseToLatestVersion 数据库初始值设定项来执行此操作。 数据库初始值设定项仅包含一些用于确保正确设置数据库的逻辑。 第一次在应用程序进程中使用上下文时，会运行此逻辑 (AppDomain)。

如下所示，可以更新 Program.cs 文件，以在使用上下文（第 14 行）之前，为 BlogContext 设置 MigrateDatabaseToLatestVersion 初始化值设定项。 请注意，还需要为 System.Data.Entity 命名空间（第 5 行）添加 using 语句。

创建此初始值设定项的实例时，需要指定上下文类型 (BlogContext) 和迁移配置（配置）- 迁移配置是启用迁移时添加到“迁移”文件夹的类**。

``` csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Data.Entity;
    using MigrationsDemo.Migrations;

    namespace MigrationsDemo
    {
        class Program
        {
            static void Main(string[] args)
            {
                Database.SetInitializer(new MigrateDatabaseToLatestVersion\<BlogContext, Configuration>());

                using (var db = new BlogContext())
                {
                    db.Blogs.Add(new Blog { Name = "Another Blog " });
                    db.SaveChanges();

                    foreach (var blog in db.Blogs)
                    {
                        Console.WriteLine(blog.Name);
                    }
                }

                Console.WriteLine("Press any key to exit...");
                Console.ReadKey();
            }
        }
    }
```

现在，每当应用程序运行时，它首先会检查其目标数据库是否为最新，如果不是，则会应用各种挂起的迁移。
