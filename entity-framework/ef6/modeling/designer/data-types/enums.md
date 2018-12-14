---
title: 枚举支持的 EF 设计器-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: c6ae6d8f-1ace-47db-ad47-b1718f1ba082
ms.openlocfilehash: 331182c4311565c94cf072eb9b9ad372ac76180a
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283935"
---
# <a name="enum-support---ef-designer"></a>枚举支持的 EF 设计器
> [!NOTE]
> **EF5 及更高版本仅**的功能，Api，Entity Framework 5 中引入了此页所述的等。 如果使用的是早期版本，则部分或全部信息不适用。

此视频和分步演练演示如何使用实体框架设计器的枚举类型。 它还演示了如何在 LINQ 查询中使用的枚举。

本演练将使用第一个模型来创建新的数据库，但在 EF 设计器还可用于[Database First](~/ef6/modeling/designer/workflows/database-first.md)工作流，以便将映射到现有数据库。

Entity Framework 5 中引入了枚举支持。 若要使用新功能，如枚举、 空间数据类型和表值函数，必须面向.NET Framework 4.5。 默认情况下，visual Studio 2012 面向.NET 4.5。

在实体框架中，枚举可以具有以下基础类型：**字节**， **Int16**， **Int32**， **Int64** ，或**SByte**。

## <a name="watch-the-video"></a>观看视频
此视频介绍如何使用实体框架设计器的枚举类型。 它还演示了如何在 LINQ 查询中使用的枚举。

**主讲人**： 以下 Julia Kornich

**视频**: [WMV](https://download.microsoft.com/download/0/7/A/07ADECC9-7893-415D-9F20-8B97D46A37EC/HDI-ITPro-MSDN-winvideo-enumwithdesiger.wmv) | [MP4](https://download.microsoft.com/download/0/7/A/07ADECC9-7893-415D-9F20-8B97D46A37EC/HDI-ITPro-MSDN-mp4video-enumwithdesiger.m4v) | [WMV (ZIP)](https://download.microsoft.com/download/0/7/A/07ADECC9-7893-415D-9F20-8B97D46A37EC/HDI-ITPro-MSDN-winvideo-enumwithdesiger.zip)

## <a name="pre-requisites"></a>系统必备组件

需要具有 Visual Studio 2012、 Ultimate、 Premium、 Professional 或 Web Express 版安装来完成本演练。

## <a name="set-up-the-project"></a>设置项目

1.  打开 Visual Studio 2012
2.  上**文件**菜单，依次指向**新建**，然后单击**项目**
3.  在左窗格中，单击**Visual C\#**，然后选择**控制台**模板
4.  输入**EnumEFDesigner**作为名称的项目并单击**确定**

## <a name="create-a-new-model-using-the-ef-designer"></a>创建新的模型使用 EF 设计器

1.  右键单击解决方案资源管理器中的项目名称，指向**外**，然后单击**新项**
2.  选择**数据**从左侧的菜单，然后选择**ADO.NET 实体数据模型**中模板窗格
3.  输入**EnumTestModel.edmx**以及该文件的名称，然后单击**添加**
4.  在实体数据模型向导页上，选择**空模型**中选择模型内容对话框
5.  单击**完成**

提供用于编辑模型的设计图面，实体设计器随即出现。

该向导执行下列操作：

-   生成定义概念模型、 存储模型以及两者之间的映射的 EnumTestModel.edmx 文件。 将.edmx 文件的元数据项目处理属性设置为嵌入在输出程序集，这样生成的元数据文件获取嵌入到程序集。
-   添加对以下程序集的引用： EntityFramework、 System.ComponentModel.DataAnnotations 和 system.data.entity 的引用。
-   创建 EnumTestModel.tt 和 EnumTestModel.Context.tt 文件并将它们添加.edmx 文件。 这些 T4 模板文件生成定义的 DbContext 派生类型和映射到.edmx 模型中实体的 POCO 类型的代码。

## <a name="add-a-new-entity-type"></a>添加新的实体类型

1.  右键单击空白区域的设计图面上，选择**添加-&gt;实体**，出现新实体对话框
2.  指定**部门**的类型名称，并指定**DepartmentID**对于键属性名称，将类型保留为**Int32**
3.  单击“确定” 
4.  右键单击该实体，然后选择**添加新的&gt;标量属性**
5.  重命名的新属性**名称**
6.  更改到的新属性的类型**Int32** （默认情况下，新属性为字符串类型） 若要更改的类型，打开属性窗口并更改类型属性等于**Int32**
7.  添加另一个标量属性和其重命名为**预算**，将类型更改为**十进制**

## <a name="add-an-enum-type"></a>添加枚举类型

1.  在实体框架设计器中，右键单击名称属性中，选择**将转换为枚举**

    ![将转换为枚举](~/ef6/media/converttoenum.png)

2.  在**添加枚举**对话框框中，键入**DepartmentNames**对于枚举类型名称，将基础类型更改为**Int32**，然后将以下成员添加到该类型： 适用于英语，数学和经济性

    ![添加枚举类型](~/ef6/media/addenumtype.png)

3.  按**确定**
4.  保存模型，并生成项目
    > [!NOTE]
    > 在生成时，可能会在错误列表中出现有关未映射的实体和关联的警告。 你可以忽略这些警告，因为我们选择从模型生成数据库后，这些错误将会消失。

如果您看一下属性窗口，您会注意到 Name 属性的类型已更改为**DepartmentNames**和新添加的枚举类型已添加到类型的列表。

如果切换到模型浏览器窗口时，将看到该类型也添加到枚举类型的节点。

![模型浏览器](~/ef6/media/modelbrowser.png)

>[!NOTE]
> 您还可以添加新的枚举类型在此窗口中通过单击鼠标右键并选择**添加枚举类型**。 创建类型后则会在类型的列表中，并能够与属性相关联

## <a name="generate-database-from-model"></a>从模型生成数据库

现在我们可以生成基于模型的数据库。

1.  右键单击实体设计器图面上，然后选择上的空白空间**根据模型生成数据库**
2.  生成数据库向导的在数据连接对话框为的选择显示单击**新的连接**按钮指定 **(localdb)\\mssqllocaldb**服务器名称和**EnumTest**数据库并单击**确定**
3.  将弹出一个对话框，询问你是否想要创建新的数据库中，单击**是**。
4.  单击**下一步**并创建数据库向导生成数据定义语言 (DDL) 创建数据库生成的 DDL 将显示在摘要和设置对话框框下，DDL 不包含的定义映射到枚举类型的表
5.  单击**完成**单击完成不会执行 DDL 脚本。
6.  创建数据库向导将执行以下： 打开**EnumTest.edmx.sql** T-SQL 编辑器生成存储架构和映射部分的 EDMX 的文件中添加连接字符串信息到 App.config 文件
7.  单击鼠标右键按钮在 T-SQL 编辑器中，然后选择**Execute**连接到服务器对话框出现，请输入在步骤 2 中的连接信息并单击**连接**
8.  若要查看生成的架构，右键单击 SQL Server 对象资源管理器中的数据库名称，然后选择**刷新**

## <a name="persist-and-retrieve-data"></a>保留和检索数据

打开 Program.cs 文件的 Main 方法定义的位置。 以下代码添加到 Main 函数。 该代码将新的部门对象添加到上下文。 然后，它将保存数据。 代码也会执行 LINQ 查询返回其中的名称是 DepartmentNames.English 的部门。

``` csharp
using (var context = new EnumTestModelContainer())
{
    context.Departments.Add(new Department{ Name = DepartmentNames.English });

    context.SaveChanges();

    var department = (from d in context.Departments
                        where d.Name == DepartmentNames.English
                        select d).FirstOrDefault();

    Console.WriteLine(
        "DepartmentID: {0} and Name: {1}",
        department.DepartmentID,  
        department.Name);
}
```

编译并运行该应用程序。 该程序生成以下输出：

```
DepartmentID: 1 Name: English
```

若要查看数据库中的数据，右键单击 SQL Server 对象资源管理器中的数据库名称，然后选择**刷新**。 然后，单击鼠标右键按钮表并选择**查看数据**。

## <a name="summary"></a>总结

在本演练中我们介绍了如何将使用实体框架设计器的枚举类型映射以及如何在代码中使用的枚举。 
