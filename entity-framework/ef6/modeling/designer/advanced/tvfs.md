---
title: 表值函数 (Tvf)-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: f019c97b-87b0-4e93-98f4-2c539f77b2dc
ms.openlocfilehash: 34aebd8f5f2c3b43c80e21c1a17a386597596c05
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283948"
---
# <a name="table-valued-functions-tvfs"></a>表值函数 (Tvf)
> [!NOTE]
> **EF5 及更高版本仅**的功能，Api，Entity Framework 5 中引入了此页所述的等。 如果使用的是早期版本，则部分或全部信息不适用。

视频和分步演练演示如何将映射表值函数 (Tvf 使用实体框架设计器)。 它还演示了如何从 LINQ 查询中调用 TVF。

Tvf 目前仅支持数据库优先工作流中。

实体框架版本 5 中引入了 TVF 的支持。 请注意，若要使用新功能，如表值函数、 枚举和空间类型必须面向.NET Framework 4.5。 默认情况下，visual Studio 2012 面向.NET 4.5。

Tvf 是非常类似于存储过程有一个重大差异： TVF 的结果是可组合。 这意味着可以在 LINQ 查询中使用 TVF 的结果，而不能存储过程的结果。

## <a name="watch-the-video"></a>观看视频

**主讲人**： 以下 Julia Kornich

[WMV](https://download.microsoft.com/download/6/0/A/60A6E474-5EF3-4E1E-B9EA-F51D2DDB446A/HDI-ITPro-MSDN-winvideo-tvf.wmv) | [MP4](https://download.microsoft.com/download/6/0/A/60A6E474-5EF3-4E1E-B9EA-F51D2DDB446A/HDI-ITPro-MSDN-mp4video-tvf.m4v) | [WMV (ZIP)](https://download.microsoft.com/download/6/0/A/60A6E474-5EF3-4E1E-B9EA-F51D2DDB446A/HDI-ITPro-MSDN-winvideo-tvf.zip)

## <a name="pre-requisites"></a>系统必备组件

若要完成本演练，需要：

- 安装[School 数据库](~/ef6/resources/school-database.md)。

- 具有最新版本的 Visual Studio

## <a name="set-up-the-project"></a>设置项目

1.  打开 Visual Studio
2.  上**文件**菜单，依次指向**新建**，然后单击**项目**
3.  在左窗格中，单击**Visual C\#**，然后选择**控制台**模板
4.  输入**TVF**作为名称的项目并单击**确定**

## <a name="add-a-tvf-to-the-database"></a>向数据库添加 TVF

-   选择**视图-&gt; SQL Server 对象资源管理器**
-   如果 LocalDB 中的服务器列表不是： 右键单击**SQL Server** ，然后选择**添加 SQL Server**使用默认**Windows 身份验证**连接到 LocalDB 服务器
-   展开 LocalDB 节点
-   在数据库节点下右键单击 School 数据库节点，然后选择**新建查询...**
-   在 T-SQL 编辑器中，粘贴以下 TVF 定义

``` SQL
CREATE FUNCTION [dbo].[GetStudentGradesForCourse]

(@CourseID INT)

RETURNS TABLE

RETURN
    SELECT [EnrollmentID],
           [CourseID],
           [StudentID],
           [Grade]
    FROM   [dbo].[StudentGrade]
    WHERE  CourseID = @CourseID
```

-   单击鼠标右键在 T-SQL 编辑器，然后选择**Execute**
-   GetStudentGradesForCourse 函数添加到 School 数据库

 

## <a name="create-a-model"></a>创建模型

1.  右键单击解决方案资源管理器中的项目名称，指向**外**，然后单击**新项**
2.  选择**数据**从左侧的菜单，然后选择**ADO.NET 实体数据模型**中**模板**窗格
3.  输入**TVFModel.edmx**以及该文件的名称，然后单击**添加**
4.  在选择模型内容对话框中，选择**从数据库生成**，然后单击**下一步**
5.  单击**新的连接**Enter **(localdb)\\mssqllocaldb**在服务器名称文本框中输入**学校**数据库命名为单击**确定**
6.  在选择数据库对象对话框框中下,**表**节点中，选择**人员**， **StudentGrade**，以及**课程**表
7.  选择**GetStudentGradesForCourse**函数位于**存储过程和函数**节点请注意，开始 Visual Studio 2012 中，实体设计器允许你批量导入到存储的过程和函数
8.  单击**完成**
9.  提供用于编辑模型的设计图面，实体设计器随即出现。 在您选择的所有对象**选择数据库对象**对话框的添加到模型。
10. 默认情况下，每个导入存储的过程或函数的结果形状将自动变为实体模型中的新复杂类型。 但我们想要将 GetStudentGradesForCourse 函数的结果映射到 StudentGrade 实体： 右键单击设计图面，然后选择**模型浏览器**在模型浏览器中，选择**函数导入**，然后双击**GetStudentGradesForCourse**函数中编辑函数导入对话框中，选择**实体**，然后选择**StudentGrade**

## <a name="persist-and-retrieve-data"></a>保留和检索数据

打开其中定义 Main 方法的文件。 以下代码添加到 Main 函数。

下面的代码演示如何构建使用表值函数的查询。 该查询将结果投影到匿名类型包含的相关的课程标题和相关的学生的等级大于或等于 3.5。

``` csharp
using (var context = new SchoolEntities())
{
    var CourseID = 4022;
    var Grade = 3.5M;

    // Return all the best students in the Microeconomics class.
    var students = from s in context.GetStudentGradesForCourse(CourseID)
                            where s.Grade >= Grade
                            select new
                            {
                                s.Person,
                                s.Course.Title
                            };

    foreach (var result in students)
    {
        Console.WriteLine(
            "Couse: {0}, Student: {1} {2}",
            result.Title,  
            result.Person.FirstName,  
            result.Person.LastName);
    }
}
```

编译并运行该应用程序。 该程序生成以下输出：

```
Couse: Microeconomics, Student: Arturo Anand
Couse: Microeconomics, Student: Carson Bryant
```

## <a name="summary"></a>总结

在本演练中介绍了如何将映射表值函数 (Tvf 使用实体框架设计器)。 它还演示了如何从 LINQ 查询中调用 TVF。
