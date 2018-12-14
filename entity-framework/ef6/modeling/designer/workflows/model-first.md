---
title: 模型优先的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: e1b9c319-bb8a-4417-ac94-7890f257e7f6
ms.openlocfilehash: d429d5ea590b22c77f3f7f0bcfbd5dfc0a3e0049
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283870"
---
# <a name="model-first"></a>模型优先
此视频和分步演练会提供使用实体框架模型优先开发的简介。 模型首先允许你创建使用实体框架设计器的新模型，然后从模型生成数据库架构。 模型存储在 EDMX 文件 （扩展名为.edmx） 并可以查看和编辑实体框架设计器中。 从 EDMX 文件自动生成与交互应用程序中的类。

## <a name="watch-the-video"></a>观看视频
此视频和分步演练会提供使用实体框架模型优先开发的简介。 模型首先允许你创建使用实体框架设计器的新模型，然后从模型生成数据库架构。 模型存储在 EDMX 文件 （扩展名为.edmx） 并可以查看和编辑实体框架设计器中。 从 EDMX 文件自动生成与交互应用程序中的类。

**主讲人**：[Rowan Miller](http://romiller.com/)

**视频**: [WMV](https://download.microsoft.com/download/5/B/1/5B1C338C-AFA7-4F68-B304-48BB008146EF/HDI-ITPro-MSDN-winvideo-modelfirst.wmv) | [MP4](https://download.microsoft.com/download/5/B/1/5B1C338C-AFA7-4F68-B304-48BB008146EF/HDI-ITPro-MSDN-mp4video-modelfirst.m4v) | [WMV (ZIP)](https://download.microsoft.com/download/5/B/1/5B1C338C-AFA7-4F68-B304-48BB008146EF/HDI-ITPro-MSDN-winvideo-modelfirst.zip)

## <a name="pre-requisites"></a>系统必备组件

将需要安装 Visual Studio 2010 或 Visual Studio 2012 安装来完成本演练。

如果使用 Visual Studio 2010，您还需要具有[NuGet](https://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c)安装。

## <a name="1-create-the-application"></a>1.创建应用程序

为了简单起见，我们要生成使用模型优先执行数据访问的基本的控制台应用程序：

-   打开 Visual Studio
-   **文件-&gt;新增-&gt;项目...**
-   选择**Windows**左侧的菜单和**控制台应用程序**
-   输入**ModelFirstSample**作为名称
-   选择“确定”

## <a name="2-create-model"></a>2.创建模型

我们将使用实体框架设计器，它是作为 Visual Studio 的一部分，若要创建我们的模型。

-   **项目-&gt;添加新项...**
-   选择**数据**左侧的菜单，然后**ADO.NET 实体数据模型**
-   输入**BloggingModel**作为名称，然后单击**确定**，这将启动实体数据模型向导
-   选择**空模型**单击**完成**

    ![创建空模型](~/ef6/media/createemptymodel.png)

实体框架设计器将打开具有空模型。 现在我们可以开始向模型添加实体、 属性和关联。

-   右键单击设计图面，然后选择**属性**
-   在属性窗口中更改**实体容器名称**到**bloggingcontext**
    *这是将您上下文生成派生上下文的名称表示与数据库中，使我们能够查询和保存数据的会话*
-   右键单击设计图面，然后选择**添加新的&gt;实体...**
-   输入**博客**作为实体名称并**BlogId**作为键名称，然后单击**确定**

    ![添加博客实体](~/ef6/media/addblogentity.png)

-   右键单击设计图面，然后选择新的实体**添加新的&gt;标量属性**，输入**名称**作为属性的名称。
-   重复此过程以添加**Url**属性。
-   右键单击**Url**设计图面并选择属性**属性**，在属性窗口中更改**Nullable**设置为**True**
    *这使我们能够将一个博客保存到数据库而不将它分配一个 Url*
-   使用你只需了解到的技术将添加**Post**具有实体**PostId**键属性
-   添加**标题**并**内容**到标量属性**Post**实体

现在，我们有几个实体，就可以添加关联 （或关系） 它们之间。

-   右键单击设计图面，然后选择**添加新的&gt;关联...**
-   使一个关系一端的指向**博客**的重数**一个**和到其他终结点**Post**重数为**许多** 
    *这意味着博客中发布了许多文章和文章属于一个博客*
-   请确保**将外键属性添加到 Post 实体**框为选中状态，并单击**确定**

    ![添加关联 MF](~/ef6/media/addassociationmf.png)

现在，我们有一个我们可以生成从数据库和用于读取和写入数据的简单模型。

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

## <a name="3-generating-the-database"></a>3.生成数据库

鉴于我们的模型，实体框架可以计算将使我们能够存储和检索使用模型数据的数据库架构。

随 Visual Studio 一起安装的数据库服务器是已安装的 Visual Studio 的版本而异：

-   如果您使用的 Visual Studio 2010 系统将创建一个 SQL Express 数据库。
-   如果你正在使用 Visual Studio 2012，则您将创建[LocalDB](https://msdn.microsoft.com/library/hh510202(v=sql.110).aspx)数据库。

让我们继续并生成的数据库。

-   右键单击设计图面，然后选择**根据模型生成数据库...**
-   单击**新建连接...** 指定 LocalDB 或 SQL Express，具体取决于哪个版本的 Visual Studio 使用，并输入**ModelFirst.Blogging**作为数据库名称。

    ![LocalDB 连接 MF](~/ef6/media/localdbconnectionmf.png)

    ![Sql Express 连接 MF](~/ef6/media/sqlexpressconnectionmf.png)

-   选择**确定**并将你想要创建新数据库，请选择要求你**是**
-   选择**下一步**和实体框架设计器会计算一个脚本来创建数据库架构
-   该脚本显示后，单击**完成**和脚本将添加到你的项目，并打开
-   右键单击脚本，并选择**Execute**，系统会提示指定要连接到的数据库指定 LocalDB 或 SQL Server Express，具体取决于哪个版本的 Visual Studio 使用

## <a name="4-reading--writing-data"></a>4.读取和写入数据

现在，我们有一个模型就可以使用它来访问某些数据。 我们将这些类用于访问数据在自动生成的基于 EDMX 文件。

*此屏幕截图取自 Visual Studio 2012 中，如果您使用的 Visual Studio 2010 BloggingModel.tt 并 BloggingModel.Context.tt 文件将直接位于你的项目而不是嵌套在 EDMX 文件下。*

![生成的类](~/ef6/media/generatedclasses.png)

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

## <a name="5-dealing-with-model-changes"></a>5.处理模型更改

现在就应该对我们的模型，进行一些更改，当我们进行这些更改，我们还需要更新数据库架构。

我们将开始通过将新的用户实体添加到我们的模型。

-   添加一个新**用户**与实体名称**用户名**作为键名称并**字符串**作为键的属性类型

    ![添加用户实体](~/ef6/media/adduserentity.png)

-   右键单击**用户名**设计图面并选择属性**属性**，在属性窗口中更改**MaxLength**设置为**50** 
    *这会限制可存储在为 50 个字符的用户名中的数据*
-   添加**DisplayName**标量属性设置为**用户**实体

我们现在有更新的模型，我们也已准备好更新数据库以容纳我们新的用户实体类型。

-   右键单击设计图面，然后选择**根据模型生成数据库...**，实体框架将计算一个脚本来重新创建基于更新后的模型架构。
-   单击**完成**
-   可能会收到警告有关覆盖现有的 DDL 脚本和模型的映射和存储部分，单击**是**上述这两个警告
-   为您打开已更新的 SQL 脚本，以创建数据库  
    *生成脚本将删除所有现有的表，然后重新创建从零开始的架构。这可能适用于本地开发，但不是可用于将更改推送到已部署的数据库。如果你需要将更改发布到已部署的数据库，您需要编辑该脚本或使用架构比较工具来计算的迁移脚本。*
-   右键单击脚本，并选择**Execute**，系统会提示指定要连接到的数据库指定 LocalDB 或 SQL Server Express，具体取决于哪个版本的 Visual Studio 使用

## <a name="summary"></a>总结

在此演练中探讨了模型优先开发，这使我们能够在 EF 设计器中创建的模型，然后从该模型生成数据库。 然后，使用模型来读取和从数据库中写入一些数据。 最后，我们更新该模型，然后重新创建数据库架构以匹配模型。
