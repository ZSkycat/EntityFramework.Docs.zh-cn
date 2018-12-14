---
title: 自动 Code First 迁移的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 0eb86787-2161-4cb4-9cb8-67c5d6e95650
ms.openlocfilehash: 2713afaf09707b7696e90464aac9945c2d82d274
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283909"
---
# <a name="automatic-code-first-migrations"></a>自动 Code First 迁移
自动迁移，可使用 Code First 迁移，而无需每个所做的更改在项目中的代码文件。 并非所有更改都将自动都应用-例如列重命名要求使用基于代码的迁移。

> [!NOTE]
> 本文假定你知道如何在基本方案中使用 Code First 迁移。 如果不这样做，则你将需要读取[Code First 迁移](~/ef6/modeling/code-first/migrations/index.md)然后再继续。

## <a name="recommendation-for-team-environments"></a>对于团队环境的建议

可以自动和基于代码的迁移单引号分隔文件，但这不建议在团队开发方案。 如果您属于一组使用源控件的开发人员应使用完全自动迁移或纯粹基于代码的迁移。 给定的自动迁移的限制，我们建议在团队环境中使用基于代码的迁移。

## <a name="building-an-initial-model--database"></a>生成初始模型和数据库

开始使用迁移之前，需要会用到项目和 Code First 模型。 对于此演练，我们将使用规范的“博客”和“帖子”模型。

-   创建一个新**MigrationsAutomaticDemo**控制台应用程序
-   将最新版本的 EntityFramework NuGet 包添加到项目中
    -   “工具”–&gt;“库包管理器”–&gt;“包管理器控制台”
    -   运行 Install-Package EntityFramework 命令
-   添加 Model.cs 文件，其代码如下所示。 此代码定义了构成域模型的单个“博客”类和 EF Code First 上下文 BlogContext 类

  ``` csharp
      using System.Data.Entity;
      using System.Collections.Generic;
      using System.ComponentModel.DataAnnotations;
      using System.Data.Entity.Infrastructure;

      namespace MigrationsAutomaticDemo
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

      namespace MigrationsAutomaticDemo
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

-   运行应用程序并将会看到**MigrationsAutomaticCodeDemo.BlogContext**为您创建数据库。

    ![数据库 LocalDB](~/ef6/media/databaselocaldb.png)

## <a name="enabling-migrations"></a>启用迁移

现可对模型进行更多更改。

-   将 Url 属性引入“博客”类。

``` csharp
    public string Url { get; set; }
```

如果要再次运行应用程序，则会收到一个 InvalidOperationException，指出“创建数据库后，支持‘BlogContext’上下文的模型已发生变化。请考虑使用 Code First 迁移更新数据库 ([http://go.microsoft.com/fwlink/?LinkId=238269](https://go.microsoft.com/fwlink/?LinkId=238269))”*。

如异常情况所述，可开始使用 Code First 迁移。 因为我们想要使用自动迁移，我们要指定 **– EnableAutomaticMigrations**切换。

-   运行**Enable-migrations – EnableAutomaticMigrations**添加了包管理器控制台此命令中的命令**迁移**到我们的项目的文件夹。 此新文件夹包含一个文件：

-   **配置类。** 此类允许配置迁移对上下文的行为方式。 对于此演练，将只使用默认配置。
    由于项目中只有一个 Code First 上下文，因此 Enable-Migrations 已自动填充此配置适用的上下文类型。

 

## <a name="your-first-automatic-migration"></a>第一个自动迁移

Code First 迁移具有两个需要用户了解的主要命令。

-   Add-Migration 将基于自上次迁移创建以来对模型所做的更改来构建下一次迁移
-   Update-Database 将对数据库应用任意挂起的迁移

我们要避免使用添加迁移 （除非我们真的有必要），专注于让 Code First 迁移自动计算并应用更改。 让我们使用**Update-database**若要获取代码优先迁移来将更改推送到我们的模型 (新**Blog.Ur**l 属性) 到数据库。

-   运行**Update-database**程序包管理器控制台命令。

**MigrationsAutomaticDemo.BlogContext**数据库现已更新以包括**Url**中的列**博客**表。

 

## <a name="your-second-automatic-migration"></a>第二个自动迁移

让我们进行一次更改并让代码优先迁移自动推送到数据库为我们所做的更改。

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

现在，使用**Update-database**使数据库保持最新状态。 这次指定 –Verbose 标志，以便可以看到 Code First 迁移正在运行的 SQL。

-   在包管理器控制台中运行 Update-Database –Verbose 命令。

## <a name="adding-a-code-based-migration"></a>将代码添加基于迁移

现在让我们看看我们可能想要使用的基于代码的迁移的内容。

-   让我们添加**评级**属性设置为**博客**类

``` csharp
    public int Rating { get; set; }
```

我们可能只需运行**Update-database**这些更改推送到数据库。 但是，我们将添加不可为 null **Blogs.Rating**列中，如果表中的任何现有数据将获取其分配新的列的数据类型的 CLR 默认值 (分级是整数，那么这是**0**). 但我们要将默认值指定为 3，以便“博客”表中的现有行以合适的分级开始。
让我们使用 Add-migration 命令对基于代码的迁移编写出此更改，以便我们可以对其进行编辑。 **Add-migration**命令使我们能够为这些迁移提供一个名称，让我们只需调用我们的愿景**AddBlogRating**。

-   运行**Add-migration AddBlogRating**程序包管理器控制台命令。
-   在中**迁移**文件夹现在我们有了一个新**AddBlogRating**迁移。 预先使用时间戳修复迁移文件名以帮助进行排序。 让我们编辑生成的代码以指定默认值为 3 的 Blog.Rating (在下面的代码行 10)

*迁移还具有捕获一些元数据的代码隐藏文件。此元数据将允许代码优先迁移来复制此代码基于在迁移之前，我们通过自动迁移。如果其他开发人员想要运行我们的迁移或部署我们的应用程序时间时，这非常重要。*

``` csharp
    namespace MigrationsAutomaticDemo.Migrations
    {
        using System;
        using System.Data.Entity.Migrations;

        public partial class AddBlogRating : DbMigration
        {
            public override void Up()
            {
                AddColumn("Blogs", "Rating", c => c.Int(nullable: false, defaultValue: 3));
            }

            public override void Down()
            {
                DropColumn("Blogs", "Rating");
            }
        }
    }
```

已编辑的迁移一切正常，所以我们可使用 Update-Database 来更新数据库。

-   运行**Update-database**程序包管理器控制台命令。

## <a name="back-to-automatic-migrations"></a>返回到自动迁移

我们现在可以随意切换回自动迁移的我们更简单的更改。 Code First 迁移将负责在正确的顺序基于每个基于代码的迁移的代码隐藏文件中存储的元数据中执行自动和基于代码的迁移。

-   让我们将 Post.Abstract 属性添加到我们的模型

``` csharp
    public string Abstract { get; set; }
```

现在我们可以使用**Update-database**获取代码优先迁移来将此更改推送到使用自动迁移的数据库。

-   运行**Update-database**程序包管理器控制台命令。

## <a name="summary"></a>总结

在本演练中了解到如何使用自动迁移推送模型对数据库进行更改。 您还了解到如何搭建基架，并在需要更多控制时运行之间自动迁移基于代码的迁移。
