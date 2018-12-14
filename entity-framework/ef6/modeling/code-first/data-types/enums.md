---
title: 枚举支持的代码优先的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 77a42501-27c9-4f4b-96df-26c128021467
ms.openlocfilehash: 1cecbf7065367deb3d202977fe39187bd907d824
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283715"
---
# <a name="enum-support---code-first"></a>枚举支持的代码优先
> [!NOTE]
> **EF5 及更高版本仅**的功能，Api，Entity Framework 5 中引入了此页所述的等。 如果使用的是早期版本，则部分或全部信息不适用。

此视频和分步演练演示如何使用 Entity Framework Code First 枚举类型。 它还演示了如何在 LINQ 查询中使用的枚举。

本演练将使用 Code First 来创建新的数据库，但也可以使用[Code First 将映射到现有数据库](~/ef6/modeling/code-first/workflows/existing-database.md)。

Entity Framework 5 中引入了枚举支持。 若要使用新功能，如枚举、 空间数据类型和表值函数，必须面向.NET Framework 4.5。 默认情况下，visual Studio 2012 面向.NET 4.5。

在实体框架中，枚举可以具有以下基础类型：**字节**， **Int16**， **Int32**， **Int64** ，或**SByte**。

## <a name="watch-the-video"></a>观看视频
此视频介绍如何将使用与 Entity Framework Code First 枚举类型。 它还演示了如何在 LINQ 查询中使用的枚举。

**主讲人**： 以下 Julia Kornich

**视频**: [WMV](https://download.microsoft.com/download/A/5/8/A583DEE8-FD5C-47EE-A4E1-966DDF39D1DA/HDI-ITPro-MSDN-winvideo-enumwithcodefirst.wmv) | [MP4](https://download.microsoft.com/download/A/5/8/A583DEE8-FD5C-47EE-A4E1-966DDF39D1DA/HDI-ITPro-MSDN-mp4video-enumwithcodefirst.m4v) | [WMV (ZIP)](https://download.microsoft.com/download/A/5/8/A583DEE8-FD5C-47EE-A4E1-966DDF39D1DA/HDI-ITPro-MSDN-winvideo-enumwithcodefirst.zip)

## <a name="pre-requisites"></a>系统必备组件

需要具有 Visual Studio 2012、 Ultimate、 Premium、 Professional 或 Web Express 版安装来完成本演练。

 

## <a name="set-up-the-project"></a>设置项目

1.  打开 Visual Studio 2012
2.  上**文件**菜单，依次指向**新建**，然后单击**项目**
3.  在左窗格中，单击**Visual C\#**，然后选择**控制台**模板
4.  输入**EnumCodeFirst**作为名称的项目并单击**确定**

## <a name="define-a-new-model-using-code-first"></a>定义使用 Code First 的新模型

使用 Code First 开发时通常首先编写.NET Framework 类定义概念 （域） 模型。 下面的代码定义部门类。

该代码还定义 DepartmentNames 枚举。 默认情况下，枚举属于**int**类型。 部门类上的名称属性是 DepartmentNames 类型。

打开 Program.cs 文件并粘贴以下类定义。

``` csharp
public enum DepartmentNames
{
    English,
    Math,
    Economics
}     

public partial class Department
{
    public int DepartmentID { get; set; }
    public DepartmentNames Name { get; set; }
    public decimal Budget { get; set; }
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

请注意，除了 EntityFramework 程序集，System.ComponentModel.DataAnnotations 和 system.data.entity 的引用程序集的引用将添加。

在 Program.cs 文件的顶部，添加以下 using 语句：

``` csharp
using System.Data.Entity;
```

在 Program.cs 中添加上下文定义。 

``` csharp
public partial class EnumTestContext : DbContext
{
    public DbSet<Department> Departments { get; set; }
}
```
 

## <a name="persist-and-retrieve-data"></a>保留和检索数据

打开 Program.cs 文件的 Main 方法定义的位置。 以下代码添加到 Main 函数。 该代码将新的部门对象添加到上下文。 然后，它将保存数据。 代码也会执行 LINQ 查询返回其中的名称是 DepartmentNames.English 的部门。

``` csharp
using (var context = new EnumTestContext())
{
    context.Departments.Add(new Department { Name = DepartmentNames.English });

    context.SaveChanges();

    var department = (from d in context.Departments
                        where d.Name == DepartmentNames.English
                        select d).FirstOrDefault();

    Console.WriteLine(
        "DepartmentID: {0} Name: {1}",
        department.DepartmentID,  
        department.Name);
}
```

编译并运行该应用程序。 该程序生成以下输出：

``` csharp
DepartmentID: 1 Name: English
```
 

## <a name="view-the-generated-database"></a>查看生成的数据库

当您运行应用程序第一次时，Entity Framework 创建数据库。 由于我们具有安装 Visual Studio 2012，则数据库将创建 LocalDB 实例上。 默认情况下，实体框架的数据库的名称派生上下文的完全限定名称 (对于本示例中为**EnumCodeFirst.EnumTestContext**)。 将使用现有数据库的后续时间。  

请注意，是否创建数据库后，可以对模型进行任何更改，则应使用 Code First 迁移更新数据库架构。 请参阅[到新数据库 Code First](~/ef6/modeling/code-first/workflows/new-database.md)有关使用迁移的示例。

若要查看的数据库和数据，请执行以下操作：

1.  在 Visual Studio 2012 主菜单中，选择**视图** - &gt; **SQL Server 对象资源管理器**。
2.  如果 LocalDB 不在的服务器列表中，单击鼠标右键**SQL Server** ，然后选择**添加 SQL Server**使用默认**Windows 身份验证**连接到LocalDB 实例
3.  展开 LocalDB 节点
4.  Unfold**数据库**文件夹，以查看新数据库并浏览到**部门**表注意第一个代码不会创建一个表映射到枚举类型
5.  若要查看数据，右键单击表并选择**查看数据**

## <a name="summary"></a>总结

在本演练中介绍了如何将使用与 Entity Framework Code First 枚举类型。 
