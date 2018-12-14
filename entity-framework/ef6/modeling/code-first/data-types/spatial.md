---
title: 空间的 EF6 Code First –
author: divega
ms.date: 10/23/2016
ms.assetid: d617aed1-15f2-48a9-b187-186991c666e3
ms.openlocfilehash: b8f2485a7002dcb4079f4045432f3224123fdb2f
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283597"
---
# <a name="spatial---code-first"></a>空间的代码优先
> [!NOTE]
> **EF5 及更高版本仅**的功能，Api，Entity Framework 5 中引入了此页所述的等。 如果使用的是早期版本，则部分或全部信息不适用。

视频和分步演练演示如何使用 Entity Framework Code First 的空间类型映射。 它还演示了如何使用 LINQ 查询来查找两个位置之间的距离。

本演练将使用 Code First 来创建新的数据库，但也可以使用[Code First 到现有数据库](~/ef6/modeling/code-first/workflows/existing-database.md)。

在 Entity Framework 5 中引入的空间类型支持。 请注意，若要使用的空间类型、 枚举和表值函数等新功能，您必须面向.NET Framework 4.5。 默认情况下，visual Studio 2012 面向.NET 4.5。

若要使用的空间数据类型还必须使用具有空间支持实体框架提供程序。 请参阅[针对空间类型的提供程序支持](~/ef6/fundamentals/providers/spatial-support.md)有关详细信息。

有两种主要的空间数据类型： geography 和 geometry。 Geography 数据类型存储之类的椭球体数据 （例如 GPS 纬度和经度坐标表示）。 Geometry 数据类型表示欧几里得 （平面） 坐标系。

## <a name="watch-the-video"></a>观看视频
此视频介绍如何将映射与 Entity Framework Code First 的空间类型。 它还演示了如何使用 LINQ 查询来查找两个位置之间的距离。

**主讲人**： 以下 Julia Kornich

**视频**: [WMV](https://download.microsoft.com/download/9/1/3/913EA17E-6F97-41D8-A4FE-805A0D83D26A/HDI-ITPro-MSDN-winvideo-spatialwithcodefirst.wmv) | [MP4](https://download.microsoft.com/download/9/1/3/913EA17E-6F97-41D8-A4FE-805A0D83D26A/HDI-ITPro-MSDN-mp4video-spatialwithcodefirst.m4v) | [WMV (ZIP)](https://download.microsoft.com/download/9/1/3/913EA17E-6F97-41D8-A4FE-805A0D83D26A/HDI-ITPro-MSDN-winvideo-spatialwithcodefirst.zip)

## <a name="pre-requisites"></a>系统必备组件

需要具有 Visual Studio 2012、 Ultimate、 Premium、 Professional 或 Web Express 版安装来完成本演练。

## <a name="set-up-the-project"></a>设置项目

1.  打开 Visual Studio 2012
2.  上**文件**菜单，依次指向**新建**，然后单击**项目**
3.  在左窗格中，单击**Visual C\#**，然后选择**控制台**模板
4.  输入**SpatialCodeFirst**作为名称的项目并单击**确定**

## <a name="define-a-new-model-using-code-first"></a>定义使用 Code First 的新模型

使用 Code First 开发时通常首先编写.NET Framework 类定义概念 （域） 模型。 下面的代码定义大学类。

该大学有 DbGeography 类型的位置属性。 若要使用 DbGeography 类型时，必须添加对 system.data.entity 的引用程序集的引用，并还将添加 using 语句 System.Data.Spatial。

打开 Program.cs 文件并粘贴以下语句使用的文件的顶部：

``` csharp
using System.Data.Spatial;
```

将以下大学类定义添加到 Program.cs 文件。

``` csharp
public class University  
{
    public int UniversityID { get; set; }
    public string Name { get; set; }
    public DbGeography Location { get; set; }
}
```

## <a name="define-the-dbcontext-derived-type"></a>定义 DbContext 派生类型

除了定义实体，还需要定义一个类，派生自 DbContext，并公开 DbSet&lt;TEntity&gt;属性。 DbSet&lt;TEntity&gt;属性使知道你想要在模型中包含哪些的类型的上下文。

在运行时，其中包括填充数据库中的数据的对象期间将 DbContext 派生类型的实例管理的实体对象，到数据库更改跟踪，并保留数据。

EntityFramework 程序集中定义的 DbContext 和 DbSet 类型。 我们将使用 EntityFramework NuGet 包添加到此 DLL 的引用。

1.  在解决方案资源管理器，右键单击项目名称。
2.  选择**管理 NuGet 包...**
3.  在管理 NuGet 包对话框中，选择**联机**选项卡，选择**EntityFramework**包。
4.  单击**安装**

请注意，EntityFramework 程序集，除了 System.ComponentModel.DataAnnotations 程序集的引用也添加。

在 Program.cs 文件的顶部，添加以下 using 语句：

``` csharp
using System.Data.Entity;
```

在 Program.cs 中添加上下文定义。 

``` csharp
public partial class UniversityContext : DbContext
{
    public DbSet<University> Universities { get; set; }
}
```

## <a name="persist-and-retrieve-data"></a>保留和检索数据

打开 Program.cs 文件的 Main 方法定义的位置。 以下代码添加到 Main 函数。

该代码将两个新的大学对象添加到上下文。 使用 DbGeography.FromText 方法初始化空间属性。 表示为 WellKnownText 传递给该方法地理点。 然后，此代码将保存数据。 然后，LINQ 查询返回大学对象其位置是最接近指定的位置，在构建并执行。

``` csharp
using (var context = new UniversityContext ())
{
    context.Universities.Add(new University()
        {
            Name = "Graphic Design Institute",
            Location = DbGeography.FromText("POINT(-122.336106 47.605049)"),
        });

    context. Universities.Add(new University()
        {
            Name = "School of Fine Art",
            Location = DbGeography.FromText("POINT(-122.335197 47.646711)"),
        });

    context.SaveChanges();

    var myLocation = DbGeography.FromText("POINT(-122.296623 47.640405)");

    var university = (from u in context.Universities
                        orderby u.Location.Distance(myLocation)
                        select u).FirstOrDefault();

    Console.WriteLine(
        "The closest University to you is: {0}.",
        university.Name);
}
```

编译并运行该应用程序。 该程序生成以下输出：

```
The closest University to you is: School of Fine Art.
```

## <a name="view-the-generated-database"></a>查看生成的数据库

当您运行应用程序第一次时，Entity Framework 创建数据库。 由于我们具有安装 Visual Studio 2012，则数据库将创建 LocalDB 实例上。 默认情况下，实体框架的数据库的名称派生上下文的完全限定名称 (在本示例中为**SpatialCodeFirst.UniversityContext**)。 将使用现有数据库的后续时间。  

请注意，是否创建数据库后，可以对模型进行任何更改，则应使用 Code First 迁移更新数据库架构。 请参阅[到新数据库 Code First](~/ef6/modeling/code-first/workflows/new-database.md)有关使用迁移的示例。

若要查看的数据库和数据，请执行以下操作：

1.  在 Visual Studio 2012 主菜单中，选择**视图** - &gt; **SQL Server 对象资源管理器**。
2.  如果 LocalDB 不在的服务器列表中，单击鼠标右键**SQL Server** ，然后选择**添加 SQL Server**使用默认**Windows 身份验证**连接到LocalDB 实例
3.  展开 LocalDB 节点
4.  Unfold**数据库**文件夹，以查看新数据库并浏览到**大学**表
5.  若要查看数据，右键单击表并选择**查看数据**

## <a name="summary"></a>总结

在本演练中介绍了如何将使用与 Entity Framework Code First 的空间类型。 
