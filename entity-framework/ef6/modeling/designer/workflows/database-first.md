---
title: 数据库优先的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: cc6ffdb3-388d-4e79-a201-01ec2577c949
ms.openlocfilehash: c81025fe7c3ad6398f003f7be2a3f9f072eec327
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46284078"
---
# <a name="database-first"></a>数据库优先
此视频和分步演练会提供使用 Entity Framework 数据库优先开发的简介。 数据库第一次可以进行反向工程，从现有数据库的模型。 模型存储在 EDMX 文件 （扩展名为.edmx） 并可以查看和编辑实体框架设计器中。 从 EDMX 文件自动生成与交互应用程序中的类。

## <a name="watch-the-video"></a>观看视频
此视频提供了使用 Entity Framework 数据库优先开发的简介。 数据库第一次可以进行反向工程，从现有数据库的模型。 模型存储在 EDMX 文件 （扩展名为.edmx） 并可以查看和编辑实体框架设计器中。 从 EDMX 文件自动生成与交互应用程序中的类。

**主讲人**：[Rowan Miller](http://romiller.com/)

**视频**: [WMV](https://download.microsoft.com/download/8/F/0/8F0B5F63-4939-4DC8-A726-FF139B37F8D8/HDI-ITPro-MSDN-winvideo-databasefirst.wmv) | [MP4](https://download.microsoft.com/download/8/F/0/8F0B5F63-4939-4DC8-A726-FF139B37F8D8/HDI-ITPro-MSDN-mp4video-databasefirst.m4v) | [WMV (ZIP)](https://download.microsoft.com/download/8/F/0/8F0B5F63-4939-4DC8-A726-FF139B37F8D8/HDI-ITPro-MSDN-winvideo-databasefirst.zip)

## <a name="pre-requisites"></a>系统必备组件

将需要安装最低版本为 Visual Studio 2010 或 Visual Studio 2012 安装来完成本演练。

如果使用 Visual Studio 2010，您还需要具有[NuGet](https://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c)安装。

 

## <a name="1-create-an-existing-database"></a>1.创建现有数据库

通常当你面向的现有数据库，则将已创建，但我们需要在本演练中创建的数据库访问。

随 Visual Studio 一起安装的数据库服务器是已安装的 Visual Studio 的版本而异：

-   如果您使用的 Visual Studio 2010 系统将创建一个 SQL Express 数据库。
-   如果你正在使用 Visual Studio 2012，则您将创建[LocalDB](https://msdn.microsoft.com/library/hh510202(v=sql.110).aspx)数据库。

 

让我们继续并生成的数据库。

-   打开 Visual Studio
-   **视图-&gt;服务器资源管理器**
-   右键单击**数据连接-&gt;添加连接...**
-   如果你尚未连接到数据库服务器资源管理器之前将需要选择 Microsoft SQL Server 作为数据源

    ![选择数据源](~/ef6/media/selectdatasource.png)

-   连接到 LocalDB 或 SQL Express，具体取决于哪一个已安装，并输入**DatabaseFirst.Blogging**作为数据库名称

    ![Sql Express 连接 DF](~/ef6/media/sqlexpressconnectiondf.png)

    ![LocalDB 连接 DF](~/ef6/media/localdbconnectiondf.png)

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
```

## <a name="2-create-the-application"></a>2.创建应用程序

为了简单起见，我们要构建使用 Database First 来执行数据访问的基本的控制台应用程序：

-   打开 Visual Studio
-   **文件-&gt;新增-&gt;项目...**
-   选择**Windows**左侧的菜单和**控制台应用程序**
-   输入**DatabaseFirstSample**作为名称
-   选择“确定”

 

## <a name="3-reverse-engineer-model"></a>3.反向工程模型

我们将使用实体框架设计器，它是作为 Visual Studio 的一部分，若要创建我们的模型。

-   **项目-&gt;添加新项...**
-   选择**数据**左侧的菜单，然后**ADO.NET 实体数据模型**
-   输入**BloggingModel**作为名称，然后单击**确定**
-   这将启动**实体数据模型向导**
-   选择**从数据库生成**单击**下一步**

    ![向导步骤 1](~/ef6/media/wizardstep1.png)

-   选择连接到第一个部分中创建的数据库中，输入**bloggingcontext**作为名称的连接字符串和单击**下一步**

    ![向导步骤 2](~/ef6/media/wizardstep2.png)

-   单击表导入的所有表并单击完成旁边的复选框

    ![向导步骤 3](~/ef6/media/wizardstep3.png)

 

反向工程过程完成后的新模型添加到你的项目，并打开，以便在实体框架设计器中查看。 此外具有已 App.config 文件添加到你的项目数据库的连接详细信息。

![初始的模型](~/ef6/media/modelinitial.png)

### <a name="additional-steps-in-visual-studio-2010"></a>Visual Studio 2010 中的其他步骤

如果您正在一些其他步骤，您需要按照升级到 Entity Framework 的最新版本的 Visual Studio 2010 中。 升级非常重要，因为它可让你访问的改进了的 API 图面，为更易于使用，以及最新 bug 修补程序。

首先，我们需要从 NuGet 获取实体框架的最新版本。

-   **项目 –&gt;管理 NuGet 包...** 
    *如果你没有**管理 NuGet 包...** 选项应该安装[最新版本的 NuGet](https://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c)*
-   选择**Online**选项卡
-   选择**EntityFramework**包
-   单击**安装**

接下来，我们需要交换我们的模型来生成利用了 DbContext API，在实体框架的更高版本中引入的代码。

-   在您的模型在 EF 设计器中的某一空白点上右键单击并选择**添加代码生成项...**
-   选择**联机模板**从左侧的菜单并搜索**DbContext**
-   选择 EF **5.x DbContext 生成器 C\#**，输入**BloggingModel**作为名称，然后单击**添加**

    ![DbContext 模板](~/ef6/media/dbcontexttemplate.png)

 

## <a name="4-reading--writing-data"></a>4.读取和写入数据

现在，我们有一个模型就可以使用它来访问某些数据。 我们将这些类用于访问数据在自动生成的基于 EDMX 文件。

*此屏幕截图取自 Visual Studio 2012 中，如果您使用的 Visual Studio 2010 BloggingModel.tt 并 BloggingModel.Context.tt 文件将直接位于你的项目而不是嵌套在 EDMX 文件下。*

![生成的类 DF](~/ef6/media/generatedclassesdf.png)

 

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
 

## <a name="5-dealing-with-database-changes"></a>5.处理数据库更改

现在就应该对我们的数据库架构进行一些更改，当我们进行这些更改，我们还需要更新我们的模型以反映这些更改。

第一步是对数据库架构进行某些更改。 我们要将用户表添加到架构。

-   右键单击**DatabaseFirst.Blogging**数据库在服务器资源管理器并选择**新查询**
-   将下面的 SQL 复制到新的查询，然后右键单击查询并选择**Execute**

``` SQL
CREATE TABLE [dbo].[Users]
(
    [Username] NVARCHAR(50) NOT NULL PRIMARY KEY,  
    [DisplayName] NVARCHAR(MAX) NULL
)
```

现在，将架构更新，就可以使用这些更改更新模型。

-   右键单击某一空白点的 EF 设计器中您的模型并选择从数据库更新模型...，这将启动更新向导
-   在更新向导检查表旁边的框的添加选项卡上，这表示我们想要从架构中添加任何新表。
    *刷新选项卡显示在更新期间将检查其进行更改的模型中的任何现有的表。删除选项卡显示已从架构中删除，并且将还在从模型期间删除更新的任何表。这两个选项卡上的信息自动检测并提供仅供参考之用，无法更改任何设置。*

    ![刷新向导](~/ef6/media/refreshwizard.png)

-   单击完成上更新向导

 

模型现在更新以包括新的用户实体映射到我们添加到数据库的用户表。

![模型更新](~/ef6/media/modelupdated.png)

## <a name="summary"></a>总结

在本演练中，我们在第一个数据库开发，这使我们能够在 EF 设计器中基于现有数据库创建模型。 然后，使用该模型以读取和从数据库中写入一些数据。 最后，我们更新模型以反映我们对数据库架构所做的更改。
