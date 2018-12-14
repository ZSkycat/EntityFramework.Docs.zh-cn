---
title: 代码优先的现有数据库-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: a7e60b74-973d-4480-868f-500a3899932e
ms.openlocfilehash: f05420beb3dff2d632151fcbf48986b0d9cd18ff
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490605"
---
# <a name="code-first-to-an-existing-database"></a>代码优先的现有数据库
此视频和分步演练会提供面向现有数据库的代码优先开发的简介。 代码首先允许你定义使用 C 模型\#或 VB.Net 类。 可使用特性在类和属性，或使用 fluent API 执行可选的其他配置。

## <a name="watch-the-video"></a>观看视频
此视频[第 9 频道上现已推出](http://channel9.msdn.com/blogs/ef/code-first-to-existing-database-ef6-1-onwards-)。

## <a name="pre-requisites"></a>系统必备组件

需要能够**Visual Studio 2012**或**Visual Studio 2013**安装来完成本演练。

您还需要版本**6.1** （或更高版本） 的**Entity Framework Tools for Visual Studio**安装。 请参阅[获取实体框架](~/ef6/fundamentals/install.md)安装实体框架工具的最新版本的信息。

## <a name="1-create-an-existing-database"></a>1.创建现有数据库

通常当你面向的现有数据库，则将已创建，但我们需要在本演练中创建的数据库访问。

让我们继续并生成的数据库。

-   打开 Visual Studio
-   **视图-&gt;服务器资源管理器**
-   右键单击**数据连接-&gt;添加连接...**
-   如果尚未连接到将数据库从**服务器资源管理器**将需要选择之前**Microsoft SQL Server**作为数据源

    ![选择数据源](~/ef6/media/selectdatasource.png)

-   连接到 LocalDB 实例，并输入**博客**作为数据库名称

    ![LocalDB 连接](~/ef6/media/localdbconnection.png)

-   选择**确定**并将你想要创建新数据库，请选择要求你**是**

    ![创建数据库对话框](~/ef6/media/createdatabasedialog.png)

-   新数据库现在将出现在服务器资源管理器，右键单击它并选择**新查询**
-   将下面的 SQL 复制到新的查询，然后右键单击查询并选择**Execute**

``` SQL
CREATE TABLE [dbo].[Blogs] (
    [BlogId] INT IDENTITY (1, 1) NOT NULL,
    [Name] NVARCHAR (200) NULL,
    [Url]  NVARCHAR (200) NULL,
    CONSTRAINT [PK_dbo.Blogs] PRIMARY KEY CLUSTERED ([BlogId] ASC)
);

CREATE TABLE [dbo].[Posts] (
    [PostId] INT IDENTITY (1, 1) NOT NULL,
    [Title] NVARCHAR (200) NULL,
    [Content] NTEXT NULL,
    [BlogId] INT NOT NULL,
    CONSTRAINT [PK_dbo.Posts] PRIMARY KEY CLUSTERED ([PostId] ASC),
    CONSTRAINT [FK_dbo.Posts_dbo.Blogs_BlogId] FOREIGN KEY ([BlogId]) REFERENCES [dbo].[Blogs] ([BlogId]) ON DELETE CASCADE
);

INSERT INTO [dbo].[Blogs] ([Name],[Url])
VALUES ('The Visual Studio Blog', 'http://blogs.msdn.com/visualstudio/')

INSERT INTO [dbo].[Blogs] ([Name],[Url])
VALUES ('.NET Framework Blog', 'http://blogs.msdn.com/dotnet/')
```

## <a name="2-create-the-application"></a>2.创建应用程序

为了简单起见，我们要构建使用 Code First 来执行数据访问的基本的控制台应用程序：

-   打开 Visual Studio
-   **文件-&gt;新增-&gt;项目...**
-   选择**Windows**左侧的菜单和**控制台应用程序**
-   输入**CodeFirstExistingDatabaseSample**作为名称
-   选择“确定”

 

## <a name="3-reverse-engineer-model"></a>3.反向工程模型

我们将充分利用适用于 Visual Studio 的实体框架工具来帮助我们获得一些初始代码来映射到数据库。 这些工具只生成代码，您也可以键入手动您的喜好而定。

-   **项目-&gt;添加新项...**
-   选择**数据**左侧的菜单，然后**ADO.NET 实体数据模型**
-   输入**bloggingcontext**作为名称，然后单击**确定**
-   这将启动**实体数据模型向导**
-   选择**从数据库 Code First**单击**下一步**

    ![向导一个 CFE](~/ef6/media/wizardonecfe.png)

-   选择第一个部分中创建的数据库的连接，然后单击**下一步**

    ![向导两个 CFE](~/ef6/media/wizardtwocfe.png)

-   单击复选框旁边**表**导入的所有表并单击**完成**

    ![向导三个 CFE](~/ef6/media/wizardthreecfe.png)

反向工程过程完成的项目数后将已添加到项目中，让我们看看添加的内容。

### <a name="configuration-file"></a>配置文件

App.config 文件已添加到项目中，此文件包含到现有数据库的连接字符串。

``` xml
<connectionStrings>
  <add  
    name="BloggingContext"  
    connectionString="data source=(localdb)\mssqllocaldb;initial catalog=Blogging;integrated security=True;MultipleActiveResultSets=True;App=EntityFramework"  
    providerName="System.Data.SqlClient" />
</connectionStrings>
```

*也可以看到使用配置文件中的某些其他设置，这些是默认 EF 设置，以告诉 Code First 创建数据库的位置。由于我们要在我们的应用程序中映射到现有数据库将忽略这些设置。*

### <a name="derived-context"></a>派生的上下文

一个**bloggingcontext**类已添加到项目。 上下文表示与数据库中，使我们能够查询和保存数据的会话。
上下文公开**DbSet&lt;TEntity&gt;** 我们的模型中每个类型。 您还会发现默认构造函数调用基构造函数使用**名称 =** 语法。 这将告知 Code First，应从配置文件加载要使用此上下文的连接字符串。

``` csharp
public partial class BloggingContext : DbContext
    {
        public BloggingContext()
            : base("name=BloggingContext")
        {
        }

        public virtual DbSet<Blog> Blogs { get; set; }
        public virtual DbSet<Post> Posts { get; set; }

        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
        }
    }
```

*应始终使用**名称 =** 语法时配置文件中使用的连接字符串。这可确保，如果连接字符串不存在则实体框架将引发而不是通过约定创建新数据库。*

### <a name="model-classes"></a>模型类

最后，**博客**并**Post**类也已添加到项目。 这些是构成了我们的模型的域类。 你将看到数据注释应用于的类来指定配置其中的代码优先约定将与现有数据库的结构不一致。 例如，你将看到**StringLength**上的批注**Blog.Name**并**Blog.Url**因为它们具有的最大长度**200**中数据库 (Code First 的默认设置是使用数据库提供程序-支持的最大长度**nvarchar （max)** SQL Server 中)。

``` csharp
public partial class Blog
{
    public Blog()
    {
        Posts = new HashSet<Post>();
    }

    public int BlogId { get; set; }

    [StringLength(200)]
    public string Name { get; set; }

    [StringLength(200)]
    public string Url { get; set; }

    public virtual ICollection<Post> Posts { get; set; }
}
```

## <a name="4-reading--writing-data"></a>4.读取和写入数据

现在，我们有一个模型就可以使用它来访问某些数据。 实现**Main**中的方法**Program.cs** ，如下所示。 此代码创建我们的上下文的新实例，并使用它来插入一个新**博客**。 然后，使用 LINQ 查询将检索所有**博客**从数据库按字母顺序排序**标题**。

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
.NET Framework Blog
ADO.NET Blog
The Visual Studio Blog
Press any key to exit...
```
 
## <a name="what-if-my-database-changes"></a>如果我的数据库更改？

Code First 数据库向导旨在生成的类，则可以调整和修改一起始点组。 如果数据库架构的更改可以手动编辑类也可以执行另一个反向工程，以覆盖类。

## <a name="using-code-first-migrations-to-an-existing-database"></a>使用 Code First 迁移对现有数据库

如果你想要使用现有的数据库使用 Code First 迁移，请参阅[Code First 迁移对现有数据库](~/ef6/modeling/code-first/migrations/existing-database.md)。

## <a name="summary"></a>总结

在本演练中介绍了使用现有数据库的 Code First 开发。 我们使用 Entity Framework Tools for Visual Studio 进行反向工程，一组类，它映射到数据库并且无法用于存储和检索数据。
