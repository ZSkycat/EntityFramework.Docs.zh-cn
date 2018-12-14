---
title: 设计器 TPT 继承的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: efc78c31-b4ea-4ea3-a0cd-c69eb507020e
ms.openlocfilehash: 84330fba4807620aa242a70cd8ac76a60284416d
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489448"
---
# <a name="designer-tpt-inheritance"></a>设计器 TPT 继承
此分步演练演示如何使用实体框架设计器 （EF 设计器） 在模型中实现每种类型一个表 (TPT) 继承。 每种类型一个表继承使用数据库中单独的表为继承层次结构中的每种类型维护非继承属性和键属性的数据。

在本演练中我们将映射**课程**（基类型）， **OnlineCourse** （派生自课程），并且**OnsiteCourse** (派生自**课程**)对表具有相同名称的实体。 我们将从数据库创建模型，然后更改要实现 TPT 继承的模型。

可以还开始使用模型优先，然后从模型生成数据库。 在 EF 设计器默认情况下使用 TPT 策略并因此在模型中的任何继承将映射到单独表。

## <a name="other-inheritance-options"></a>其他继承选项

每个层次结构一张表 (TPH) 是继承的另一种类型的一个数据库中使用表来维护继承层次结构中的实体类型的所有数据。  有关如何将映射与实体设计器的每个层次结构一个表继承的信息，请参阅[EF 设计器 TPH 继承](~/ef6/modeling/designer/inheritance/tph.md)。 

请注意，每个具体表键入继承 (TPC) 和混合的继承模型由实体框架运行时支持，但不是受 EF 设计器。 如果你想要使用 TPC 或混合的继承，有两个选项： 使用 Code First，或手动编辑 EDMX 文件。 如果您选择使用 EDMX 文件，映射详细信息窗口将放入"安全模式"，您将不能够使用设计器来更改映射。

## <a name="prerequisites"></a>系统必备

若要完成此演练，您需要：

- 最新版本的 Visual Studio。
- [School 示例数据库](~/ef6/resources/school-database.md)。

## <a name="set-up-the-project"></a>设置项目

-   打开 Visual Studio 2012。
-   选择**文件-&gt;新增-&gt;项目**
-   在左窗格中，单击**Visual C\#**，然后选择**控制台**模板。
-   输入**TPTDBFirstSample**作为名称。
-   选择“确定”。

## <a name="create-a-model"></a>创建模型

-   右键单击解决方案资源管理器中的项目并选择**添加-&gt;新项**。
-   选择**数据**从左侧的菜单，然后选择**ADO.NET 实体数据模型**模板窗格中。
-   输入**TPTModel.edmx**以及该文件的名称，然后单击**添加**。
-   在选择模型内容对话框框中，选择 * * 生成从数据库 * *，然后依次**下一步**。
-   单击**新的连接**。
    在连接属性对话框中，输入服务器名称 (例如， **(localdb)\\mssqllocaldb**)，选择身份验证方法、 类型**学校**数据库名称，然后单击**确定**。
    选择数据连接对话框中使用数据库连接设置更新。
-   在选择数据库对象对话框中，节点下的表，选择**部门**，**课程、 OnlineCourse 和 OnsiteCourse**表。
-   单击 **“完成”**。

提供用于编辑模型的设计图面，实体设计器随即出现。 在选择数据库对象对话框中选择的所有对象都添加到模型中。

## <a name="implement-table-per-type-inheritance"></a>实现每种类型一个表继承

-   在设计图面上，右键单击**OnlineCourse**实体类型并选择**属性**。
-   在中**属性**窗口中，将 Base Type 属性设置为**课程**。
-   右键单击**OnsiteCourse**实体类型并选择**属性**。
-   在中**属性**窗口中，将 Base Type 属性设置为**课程**。
-   右键单击之间的关联 （行） **OnlineCourse**并**课程**实体类型。
    选择**从模型中删除**。
-   右键单击之间的关联**OnsiteCourse**并**课程**实体类型。
    选择**从模型中删除**。

我们现在将删除**CourseID**属性从**OnlineCourse**并**OnsiteCourse**这些类继承，因此**CourseID**从**课程**基类型。

-   右键单击**CourseID**的属性**OnlineCourse**实体类型，并选择**从模型中删除**。
-   右键单击**CourseID**的属性**OnsiteCourse**实体类型，并选择**从模型中删除**
-   至此已实现每种类型一个表继承。

![TPT](~/ef6/media/tpt.png)

## <a name="use-the-model"></a>使用模型

打开**Program.cs**文件，其中**Main**方法的定义。 粘贴下面的代码插入**Main**函数。 代码执行三个查询。 第一个查询返回所有**课程**与指定系建立联系。 第二个查询使用**OfType**方法返回**OnlineCourses**与指定系建立联系。 第三个查询将返回**OnsiteCourses**。

``` csharp
    using (var context = new SchoolEntities())
    {
        foreach (var department in context.Departments)
        {
            Console.WriteLine("The {0} department has the following courses:",
                               department.Name);

            Console.WriteLine("   All courses");
            foreach (var course in department.Courses )
            {
                Console.WriteLine("     {0}", course.Title);
            }

            foreach (var course in department.Courses.
                OfType<OnlineCourse>())
            {
                Console.WriteLine("   Online - {0}", course.Title);
            }

            foreach (var course in department.Courses.
                OfType<OnsiteCourse>())
            {
                Console.WriteLine("   Onsite - {0}", course.Title);
            }
        }
    }
```
