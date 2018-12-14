---
title: 空间的 EF 设计器-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 06baa6e1-d680-4a95-845b-81305c87a962
ms.openlocfilehash: 67cc6c007a4477b650e7c4875de8fac36a9ba2be
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283753"
---
# <a name="spatial---ef-designer"></a>空间的 EF 设计器
> [!NOTE]
> **EF5 及更高版本仅**的功能，Api，Entity Framework 5 中引入了此页所述的等。 如果使用的是早期版本，则部分或全部信息不适用。

视频和分步演练演示如何使用实体框架设计器的空间类型映射。 它还演示了如何使用 LINQ 查询来查找两个位置之间的距离。

本演练将使用第一个模型来创建新的数据库，但在 EF 设计器还可用于[Database First](~/ef6/modeling/designer/workflows/database-first.md)工作流，以便将映射到现有数据库。

在 Entity Framework 5 中引入的空间类型支持。 请注意，若要使用的空间类型、 枚举和表值函数等新功能，您必须面向.NET Framework 4.5。 默认情况下，visual Studio 2012 面向.NET 4.5。

若要使用的空间数据类型还必须使用具有空间支持实体框架提供程序。 请参阅[针对空间类型的提供程序支持](~/ef6/fundamentals/providers/spatial-support.md)有关详细信息。

有两种主要的空间数据类型： geography 和 geometry。 Geography 数据类型存储之类的椭球体数据 （例如 GPS 纬度和经度坐标表示）。 Geometry 数据类型表示欧几里得 （平面） 坐标系。

## <a name="watch-the-video"></a>观看视频
此视频介绍如何将映射与实体框架设计器的空间类型。 它还演示了如何使用 LINQ 查询来查找两个位置之间的距离。

**主讲人**： 以下 Julia Kornich

**视频**: [WMV](https://download.microsoft.com/download/E/C/9/EC9E6547-8983-4C1F-A919-D33210E4B213/HDI-ITPro-MSDN-winvideo-spatialwithdesigner.wmv) | [MP4](https://download.microsoft.com/download/E/C/9/EC9E6547-8983-4C1F-A919-D33210E4B213/HDI-ITPro-MSDN-mp4video-spatialwithdesigner.m4v) | [WMV (ZIP)](https://download.microsoft.com/download/E/C/9/EC9E6547-8983-4C1F-A919-D33210E4B213/HDI-ITPro-MSDN-winvideo-spatialwithdesigner.zip)

## <a name="pre-requisites"></a>系统必备组件

需要具有 Visual Studio 2012、 Ultimate、 Premium、 Professional 或 Web Express 版安装来完成本演练。

## <a name="set-up-the-project"></a>设置项目

1.  打开 Visual Studio 2012
2.  上**文件**菜单，依次指向**新建**，然后单击**项目**
3.  在左窗格中，单击**Visual C\#**，然后选择**控制台**模板
4.  输入**SpatialEFDesigner**作为名称的项目并单击**确定**

## <a name="create-a-new-model-using-the-ef-designer"></a>创建新的模型使用 EF 设计器

1.  右键单击解决方案资源管理器中的项目名称，指向**外**，然后单击**新项**
2.  选择**数据**从左侧的菜单，然后选择**ADO.NET 实体数据模型**中模板窗格
3.  输入**UniversityModel.edmx**以及该文件的名称，然后单击**添加**
4.  在实体数据模型向导页上，选择**空模型**中选择模型内容对话框
5.  单击**完成**

提供用于编辑模型的设计图面，实体设计器随即出现。

该向导执行下列操作：

-   生成定义概念模型、 存储模型以及两者之间的映射的 EnumTestModel.edmx 文件。 将.edmx 文件的元数据项目处理属性设置为嵌入在输出程序集，这样生成的元数据文件获取嵌入到程序集。
-   添加对以下程序集的引用： EntityFramework、 System.ComponentModel.DataAnnotations 和 system.data.entity 的引用。
-   创建 UniversityModel.tt 和 UniversityModel.Context.tt 文件并将它们添加.edmx 文件。 这些 T4 模板文件生成定义的 DbContext 派生类型和映射到.edmx 模型中实体的 POCO 类型的代码

## <a name="add-a-new-entity-type"></a>添加新的实体类型

1.  右键单击空白区域的设计图面上，选择**添加-&gt;实体**，出现新实体对话框
2.  指定**大学**的类型名称，并指定**UniversityID**对于键属性名称，将类型保留为**Int32**
3.  单击“确定” 
4.  右键单击该实体，然后选择**添加新的&gt;标量属性**
5.  重命名的新属性**名称**
6.  添加另一个标量属性和其重命名为**位置**打开属性窗口并更改到新属性的类型**Geography**
7.  保存模型，并生成项目
    > [!NOTE]
    > 在生成时，可能会在错误列表中出现有关未映射的实体和关联的警告。 你可以忽略这些警告，因为我们选择从模型生成数据库后，这些错误将会消失。

## <a name="generate-database-from-model"></a>从模型生成数据库

现在我们可以生成基于模型的数据库。

1.  右键单击实体设计器图面上，然后选择上的空白空间**根据模型生成数据库**
2.  生成数据库向导的在数据连接对话框为的选择显示单击**新的连接**按钮指定 **(localdb)\\mssqllocaldb**服务器名称和**大学**数据库并单击**确定**
3.  将弹出一个对话框，询问你是否想要创建新的数据库中，单击**是**。
4.  单击**下一步**并创建数据库向导生成数据定义语言 (DDL) 创建数据库生成的 DDL 将显示在摘要和设置对话框框下，DDL 不包含的定义映射到枚举类型的表
5.  单击**完成**单击完成不会执行 DDL 脚本。
6.  创建数据库向导将执行以下： 打开**UniversityModel.edmx.sql** T-SQL 编辑器生成存储架构和映射部分的 EDMX 的文件中添加连接字符串信息到 App.config 文件
7.  单击鼠标右键按钮在 T-SQL 编辑器中，然后选择**Execute**连接到服务器对话框出现，请输入在步骤 2 中的连接信息并单击**连接**
8.  若要查看生成的架构，右键单击 SQL Server 对象资源管理器中的数据库名称，然后选择**刷新**

## <a name="persist-and-retrieve-data"></a>保留和检索数据

打开 Program.cs 文件的 Main 方法定义的位置。 以下代码添加到 Main 函数。

该代码将两个新的大学对象添加到上下文。 使用 DbGeography.FromText 方法初始化空间属性。 表示为 WellKnownText 传递给该方法地理点。 然后，此代码将保存数据。 然后，LINQ 查询返回大学对象其位置是最接近指定的位置，在构建并执行。

``` csharp
using (var context = new UniversityModelContainer())
{
    context.Universities.Add(new University()
    {
        Name = "Graphic Design Institute",
        Location = DbGeography.FromText("POINT(-122.336106 47.605049)"),
    });

    context.Universities.Add(new University()
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

若要查看数据库中的数据，右键单击 SQL Server 对象资源管理器中的数据库名称，然后选择**刷新**。 然后，单击鼠标右键按钮表并选择**查看数据**。

## <a name="summary"></a>总结

在本演练中我们介绍了如何将使用实体框架设计器的空间类型映射以及如何在代码中使用的空间类型。 
