---
title: 设计器的查询存储过程-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 9554ed25-c5c1-43be-acad-5da37739697f
ms.openlocfilehash: 04478ea1c8cd43a7ba4ee788e464992af3de7f64
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283896"
---
# <a name="designer-query-stored-procedures"></a>设计器的查询存储过程
此分步演练演示如何使用实体框架设计器 （EF 设计器） 导入存储的过程转换为模型，然后调用导入的存储的过程来检索结果。 

请注意，Code First 不支持映射到存储的过程或函数。 但是，可以通过使用 System.Data.Entity.DbSet.SqlQuery 方法中调用存储的过程或函数。 例如：
``` csharp
var query = context.Products.SqlQuery("EXECUTE [dbo].[GetAllProducts]")`;
```

## <a name="prerequisites"></a>系统必备

若要完成此演练，您需要：

- 最新版本的 Visual Studio。
- [School 示例数据库](~/ef6/resources/school-database.md)。

## <a name="set-up-the-project"></a>设置项目

-   打开 Visual Studio 2012。
-   选择**文件-&gt;新增-&gt;项目**
-   在左窗格中，单击**Visual C\#**，然后选择**控制台**模板。
-   输入**EFwithSProcsSample**作为名称。
-   选择“确定”。

## <a name="create-a-model"></a>创建模型

-   右键单击解决方案资源管理器中的项目并选择**添加-&gt;新项**。
-   选择**数据**从左侧的菜单，然后选择**ADO.NET 实体数据模型**模板窗格中。
-   输入**EFwithSProcsModel.edmx**以及该文件的名称，然后单击**添加**。
-   在选择模型内容对话框中，选择**从数据库生成**，然后单击**下一步**。
-   单击**新的连接**。  
    在连接属性对话框中，输入服务器名称 (例如， **(localdb)\\mssqllocaldb**)，选择身份验证方法、 类型**学校**数据库名称，然后单击**确定**。  
    选择数据连接对话框中使用数据库连接设置更新。
-   在选择数据库对象对话框中，检查**表**复选框以选择所有表。  
    此外，请选择下的以下存储的过程**存储过程和函数**节点： **GetStudentGrades**并**GetDepartmentName**。 

    ![导入](~/ef6/media/import.jpg)

    *从 Visual Studio 2012 EF 设计器支持大容量导入的存储过程。**导入选定的存储的过程和函数到实体模型**默认选中的。*
-   单击 **“完成”**。

默认情况下，每个导入存储的过程或函数，可返回多个列的结果形状将自动成为新的复杂类型。 在此示例中我们想要的结果映射**GetStudentGrades**函数来**StudentGrade**实体和结果**GetDepartmentName**到**none** (**无**是默认值)。

函数导入返回实体类型，相应的存储过程返回的列必须与返回的实体类型的标量属性完全匹配。 函数导入还可以返回简单类型、 复杂类型或没有值的集合。

-   右键单击设计图面，然后选择**模型浏览器**。
-   在中**模型浏览器**，选择**函数导入**，然后双击**GetStudentGrades**函数。
-   在编辑函数导入对话框中，选择**实体**，然后选择**StudentGrade**。  
    ***是可组合的函数导入**顶部的复选框**函数导入**对话框将允许您将映射到撰写的函数。如果不要选中此框，仅可组合的函数 （表值函数） 会出现在**存储过程 / 函数名称**下拉列表。如果不选中此框，仅非撰写的函数将在列表中所示。*

## <a name="use-the-model"></a>使用模型

打开**Program.cs**文件，其中**Main**方法的定义。 以下代码添加到 Main 函数。

该代码调用两个存储的过程： **GetStudentGrades** (返回**StudentGrades**指定*StudentId*) 和**GetDepartmentName**（输出参数中返回的部门名称）。  

``` csharp
    using (SchoolEntities context = new SchoolEntities())
    {
        // Specify the Student ID.
        int studentId = 2;

        // Call GetStudentGrades and iterate through the returned collection.
        foreach (StudentGrade grade in context.GetStudentGrades(studentId))
        {
            Console.WriteLine("StudentID: {0}\tSubject={1}", studentId, grade.Subject);
            Console.WriteLine("Student grade: " + grade.Grade);
        }

        // Call GetDepartmentName.
        // Declare the name variable that will contain the value returned by the output parameter.
        ObjectParameter name = new ObjectParameter("Name", typeof(String));
        context.GetDepartmentName(1, name);
        Console.WriteLine("The department name is {0}", name.Value);

    }
```

编译并运行该应用程序。 该程序生成以下输出：

```
StudentID: 2
Student grade: 4.00
StudentID: 2
Student grade: 3.50
The department name is Engineering
```

<a name="output-parameters"></a>输出参数
-----------------

如果使用输出参数，其值将不可用，直到已完全读取结果。 这是由于 DbDataReader 的基础行为，请参阅[使用 DataReader 检索数据](https://go.microsoft.com/fwlink/?LinkID=398589)的更多详细信息。
