---
title: 到新的数据库的 EF6 代码优先
author: divega
ms.date: 10/23/2016
ms.assetid: 2df6cb0a-7d8b-4e28-9d05-e2b9a90125af
ms.openlocfilehash: a19db575b685cde98509fff4a0efaf26106b26bc
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46284117"
---
# <a name="code-first-to-a-new-database"></a>代码优先的新数据库
此视频和分步演练会提供针对新数据库 Code First 开发的简介。 此方案包括目标不存在的数据库和 Code First 将创建，或一个空数据库，Code First 将添加到新表。 代码首先允许你定义使用 C 模型\#或 VB.Net 类。 （可选） 可以使用特性在类和属性，或使用 fluent API 执行其他配置。

## <a name="watch-the-video"></a>观看视频
此视频提供了针对新数据库 Code First 开发的简介。 此方案包括目标不存在的数据库和 Code First 将创建，或一个空数据库，Code First 将添加到新表。 代码首先允许你定义使用 C# 或 VB.Net 类模型。 （可选） 可以使用特性在类和属性，或使用 fluent API 执行其他配置。

**主讲人**：[Rowan Miller](http://romiller.com/)

**视频**: [WMV](https://download.microsoft.com/download/B/A/5/BA57BADE-D558-4693-8F82-29E64E4084AB/HDI-ITPro-MSDN-winvideo-CodeFirstNewDatabase.wmv) | [MP4](https://download.microsoft.com/download/B/A/5/BA57BADE-D558-4693-8F82-29E64E4084AB/HDI-ITPro-MSDN-mp4Video-CodeFirstNewDatabase.m4v) | [WMV (ZIP)](https://download.microsoft.com/download/B/A/5/BA57BADE-D558-4693-8F82-29E64E4084AB/HDI-ITPro-MSDN-winvideo-CodeFirstNewDatabase.zip)

## <a name="pre-requisites"></a>系统必备组件

将需要安装最低版本为 Visual Studio 2010 或 Visual Studio 2012 安装来完成本演练。

如果使用 Visual Studio 2010，您还需要具有[NuGet](https://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c)安装。

## <a name="1-create-the-application"></a>1.创建应用程序

为了简单起见，我们要构建使用 Code First 来执行数据访问的基本的控制台应用程序。

-   打开 Visual Studio
-   **文件-&gt;新增-&gt;项目...**
-   选择**Windows**左侧的菜单和**控制台应用程序**
-   输入**CodeFirstNewDatabaseSample**作为名称
-   选择“确定”

## <a name="2-create-the-model"></a>2.创建模型

让我们定义一个非常简单的模型使用的类。 我们只将其定义中的 Program.cs 文件，但在实际应用程序会将出将类拆分为单独的文件和可能一个单独的项目中。

在 Program.cs 中到 Program 类定义的下面添加以下两个类。

``` csharp
public class Blog
{
    public int BlogId { get; set; }
    public string Name { get; set; }

    public virtual List<Post> Posts { get; set; }
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    public int BlogId { get; set; }
    public virtual Blog Blog { get; set; }
}
```

您会注意到，我们正在实现两个导航属性 （Blog.Posts 和 Post.Blog） 虚拟。 这使实体框架的延迟加载功能。 延迟加载意味着，这些属性的内容将自动从数据库加载时尝试对其进行访问。

## <a name="3-create-a-context"></a>3.创建上下文

现在就来定义派生的上下文，它表示与数据库中，使我们能够查询和保存数据的会话。 我们定义的上下文，它派生自 System.Data.Entity.DbContext 并公开的类型化的 DbSet&lt;TEntity&gt;我们的模型中的一个类。

我们现在已经开始使用实体框架中的类型，因此我们需要添加 EntityFramework NuGet 包。

-   **项目 –&gt;管理 NuGet 包...**
    注意： 如果你没有**管理 NuGet 包...** 应安装的选项[最新版本的 NuGet](https://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c)
-   选择**Online**选项卡
-   选择**EntityFramework**包
-   单击**安装**

添加 using 语句 System.Data.Entity Program.cs 的顶部。

``` csharp
using System.Data.Entity;
```

在 Program.cs 中的 Post 类下面添加以下派生的上下文。

``` csharp
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
}
```

以下是 Program.cs 现在应如何包含的完整列表。

``` csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Data.Entity;

namespace CodeFirstNewDatabaseSample
{
    class Program
    {
        static void Main(string[] args)
        {
        }
    }

    public class Blog
    {
        public int BlogId { get; set; }
        public string Name { get; set; }

        public virtual List<Post> Posts { get; set; }
    }

    public class Post
    {
        public int PostId { get; set; }
        public string Title { get; set; }
        public string Content { get; set; }

        public int BlogId { get; set; }
        public virtual Blog Blog { get; set; }
    }

    public class BloggingContext : DbContext
    {
        public DbSet<Blog> Blogs { get; set; }
        public DbSet<Post> Posts { get; set; }
    }
}
```

这是我们需要开始存储和检索数据的所有代码。 很显然没有在后台发生的大量，我们来看看，但第一个稍后我们来看它的操作。

## <a name="4-reading--writing-data"></a>4.读取和写入数据

在 Program.cs 中实现的 Main 方法，如下所示。 此代码将创建我们的上下文的新实例，并使用它来插入新博客。 然后它使用 LINQ 查询以从按标题按字母顺序排序的数据库中检索所有博客。

``` csharp
class Program
{
    static void Main(string[] args)
    {
        using (var db = new BloggingContext())
        {
            // Create and save a new Blog
            Console.Write("Enter a name for a new Blog: ");
            var name = Console.ReadLine();

            var blog = new Blog { Name = name };
            db.Blogs.Add(blog);
            db.SaveChanges();

            // Display all Blogs from the database
            var query = from b in db.Blogs
                        orderby b.Name
                        select b;

            Console.WriteLine("All blogs in the database:");
            foreach (var item in query)
            {
                Console.WriteLine(item.Name);
            }

            Console.WriteLine("Press any key to exit...");
            Console.ReadKey();
        }
    }
}
```

你现在可以运行该应用程序和它进行测试。

```
Enter a name for a new Blog: ADO.NET Blog
All blogs in the database:
ADO.NET Blog
Press any key to exit...
```
### <a name="wheres-my-data"></a>我的数据位于何处？

按照约定 DbContext 已为您创建数据库。

-   如果本地 SQL Express 实例不可用 （默认情况下使用 Visual Studio 2010 安装） 然后代码优先已创建了数据库在该实例上
-   如果 SQL Express 不可用，则 Code First 将尝试并使用[LocalDB](https://msdn.microsoft.com/library/hh510202(v=sql.110).aspx) （默认情况下使用 Visual Studio 2012 安装）
-   派生上下文中，在本例中是完全限定名称命名数据库**CodeFirstNewDatabaseSample.BloggingContext**

这些是仅默认约定，通过多种方式，将使用 Code First 的数据库中提供了详细信息**DbContext 的模型和数据库连接的发现**主题。
可以连接到此数据库在 Visual Studio 中使用服务器资源管理器

-   **视图-&gt;服务器资源管理器**
-   右键单击**数据连接**，然后选择**添加连接...**
-   如果你尚未连接到数据库服务器资源管理器之前将需要选择 Microsoft SQL Server 作为数据源

    ![选择数据源](~/ef6/media/selectdatasource.png)

-   连接到 LocalDB 或 SQL Express，具体取决于哪一个已安装

我们现在可以检查 Code First 创建的架构。

![架构初始](~/ef6/media/schemainitial.png)

DbContext 计算出来以查看我们定义的 DbSet 属性包含在模型中哪些类。 然后使用默认组 Code First 约定来确定表和列的名称，确定数据类型，查找主键，等等。 在本演练后面将介绍如何重写这些约定。

## <a name="5-dealing-with-model-changes"></a>5.处理模型更改

现在就应该对我们的模型，进行一些更改，当我们进行这些更改，我们还需要更新数据库架构。 若要执行此操作，我们将要使用简称称为 Code First 迁移或迁移的功能。

迁移使我们可以有序的集的步骤，描述了如何进行升级 （和降级） 我们数据库架构。 每个步骤，称为迁移时，包含一些描述了要应用的更改的代码。 

第一步是为我们 bloggingcontext 启用 Code First 迁移。

-   **工具-&gt;库程序包管理器-&gt;包管理器控制台**
-   在包管理器控制台中运行 Enable-Migrations 命令
-   一个新的 Migrations 文件夹已添加到我们包含两个项目的项目：
    -   **Configuration.cs** – 此文件包含迁移将用于迁移的设置 bloggingcontext。 我们无需更改任何内容对于本演练，但下面是你可以在其中指定种子数据，对于其他数据库，注册提供程序更改命名空间等中生成的迁移。
    -   **&lt;时间戳&gt;\_InitialCreate.cs** – 这是你第一次迁移，它表示已应用到数据库，以使其成为空数据库为包括博客和文章表的更改. 尽管我们让代码优先会自动创建对我们而言，这些表，现在，我们已选择加入它们已转换为迁移的迁移。 首先我们已应用此迁移的本地数据库中也有记录代码。 文件名上的时间戳用于排序目的。

    现在让我们与我们的模型进行更改时，将 Url 属性添加到博客类：

``` csharp
public class Blog
{
    public int BlogId { get; set; }
    public string Name { get; set; }
    public string Url { get; set; }

    public virtual List<Post> Posts { get; set; }
}
```

-   运行**Add-migration AddUrl**程序包管理器控制台命令。
    Add-migration 命令检查自上次迁移以来的更改和搭建基架以新的迁移与所发现的任何更改。 我们可以为迁移指定的名称;在这种情况下，我们将调用迁移 AddUrl。
    基架的代码说我们需要添加 Url 列时，可以保存字符串数据，为 dbo。Blogs 表。 必要时，我们无法编辑已搭建基架的代码，但在这种情况下不需要的。

``` csharp
namespace CodeFirstNewDatabaseSample.Migrations
{
    using System;
    using System.Data.Entity.Migrations;

    public partial class AddUrl : DbMigration
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

-   运行**Update-database**程序包管理器控制台命令。 此命令将任何待定迁移应用到数据库。 因此迁移将只应用我们的新 AddUrl 迁移已应用我们 InitialCreate 迁移。
    提示： 可以使用 **– Verbose**切换调用要查看正在执行对数据库的 SQL 数据库更新时。

新的 Url 列现在添加到数据库中的 Blogs 表：

![使用 Url 的架构](~/ef6/media/schemawithurl.png)

## <a name="6-data-annotations"></a>6.数据注释

到目前为止我们只需让 EF 发现在模型中使用其默认约定，但存在将为我们的类不遵循的约定，并需要能够执行进一步的配置。 有两个选项;我们将介绍本部分中的数据注释，然后选择下一节中的 fluent API。

-   让我们将用户类添加到我们的模型

``` csharp
public class User
{
    public string Username { get; set; }
    public string DisplayName { get; set; }
}
```

-   我们还需要将一组添加到我们派生上下文

``` csharp
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
    public DbSet<User> Users { get; set; }
}
```

-   如果我们尝试添加迁移，则会得到错误消息，指出"*EntityType 'User' 已定义任何键。为该 EntityType 定义键。"* 因为 EF 有没有办法知道用户名应为用户的主键。
-   我们现在使用数据注释，因此我们需要添加一个 using 语句在 Program.cs 的顶部

```
using System.ComponentModel.DataAnnotations;
```

-   现在添加批注以确定它是主键的 Username 属性

``` csharp
public class User
{
    [Key]
    public string Username { get; set; }
    public string DisplayName { get; set; }
}
```

-   使用**Add-migration AddUser**命令搭建基架，迁移，以应用这些更改到数据库
-   运行**Update-database**命令，将新的迁移应用到数据库

现在，新表已添加到数据库：

![与用户的架构](~/ef6/media/schemawithusers.png)

是由 EF 支持注释的完整列表：

-   [KeyAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.keyattribute)
-   [StringLengthAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.stringlengthattribute)
-   [MaxLengthAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.maxlengthattribute)
-   [ConcurrencyCheckAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.concurrencycheckattribute)
-   [有 RequiredAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.requiredattribute)
-   [TimestampAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.timestampattribute)
-   [ComplexTypeAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.complextypeattribute)
-   [ColumnAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.columnattribute)
-   [TableAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.tableattribute)
-   [InversePropertyAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.inversepropertyattribute)
-   [ForeignKeyAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.foreignkeyattribute)
-   [DatabaseGeneratedAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.databasegeneratedattribute)
-   [NotMappedAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.notmappedattribute)

## <a name="7-fluent-api"></a>7.Fluent API

在上一节中介绍了使用数据批注以补充或替代所检测到的约定。 通过 Code First fluent API 配置模型的其他方法。

模型配置最多可以使用简单的数据批注。 Fluent API 是指定数据批注可以做另外一些更高级的配置不可能的数据注释的一切内容的模型配置的更高级的方法。 数据注释和 fluent API 可以一起使用。

若要访问 fluent API 重写中 DbContext 的 OnModelCreating 方法。 假设我们想要 User.DisplayName 存储中，若要显示此列重命名\_名称。

-   使用重写 OnModelCreating 方法上 bloggingcontext 下面的代码

``` csharp
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
    public DbSet<User> Users { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        modelBuilder.Entity<User>()
            .Property(u => u.DisplayName)
            .HasColumnName("display_name");
    }
}
```

-   使用**Add-migration ChangeDisplayName**命令搭建基架，迁移，以应用这些更改到数据库。
-   运行**Update-database**命令，将新的迁移应用到数据库。

DisplayName 列现在重命名以显示\_名称：

![重命名的显示名称的架构](~/ef6/media/schemawithdisplaynamerenamed.png)

## <a name="summary"></a>总结

在本演练中介绍了使用新数据库 Code First 开发。 我们定义使用类的模型，然后使用该模型来创建数据库和存储和检索数据。 创建数据库后我们将使用 Code First 迁移演变成我们的模型中更改架构。 我们还了解到如何将模型配置为使用数据注释和 Fluent API。
