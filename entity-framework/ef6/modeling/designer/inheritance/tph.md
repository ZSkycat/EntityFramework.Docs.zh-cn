---
title: 设计器 TPH 继承-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 72d26a8e-20ab-4500-bd13-394a08e73394
ms.openlocfilehash: 43ba34a98c3960a7a3052a00e2ed2751c2f2b121
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490092"
---
# <a name="designer-tph-inheritance"></a>设计器 TPH 继承
此分步演练演示如何使用实体框架设计器 （EF 设计器） 在概念模型中实现每个层次结构一个表 (TPH) 继承。 TPH 继承使用数据库表维护继承层次结构中的实体类型的所有数据。

在本演练中我们会将 Person 表映射到三个实体类型： Person （基类型）、 （派生自人员） 的学生和教师 （派生自人员）。 我们将从数据库 (Database First) 创建概念模型，然后更改模型来实现 TPH 继承使用 EF 设计器。

可以将映射到使用模型优先 TPH 继承，但您必须编写您自己的数据库生成工作流即复杂。 然后会将分配到此工作流**数据库生成工作流**EF 设计器中的属性。 更轻松的替代方法是使用 Code First。

## <a name="other-inheritance-options"></a>其他继承选项

每种类型一个表 (TPT) 是继承的另一种类型在其数据库中的单独表映射到参与继承的实体。  有关如何将映射与 EF 设计器的每个类型一个表继承的信息，请参阅[EF 设计器 TPT 继承](~/ef6/modeling/designer/inheritance/tpt.md)。

表每个具体类型继承 (TPC) 和混合的继承模型由实体框架运行时支持，但不是支持通过 EF 设计器。 如果你想要使用 TPC 或混合的继承，有两个选项： 使用 Code First，或手动编辑 EDMX 文件。 如果您选择使用 EDMX 文件，映射详细信息窗口将放入"安全模式"，您将不能够使用设计器来更改映射。

## <a name="prerequisites"></a>系统必备

若要完成此演练，您需要：

- 最新版本的 Visual Studio。
- [School 示例数据库](~/ef6/resources/school-database.md)。

## <a name="set-up-the-project"></a>设置项目

-   打开 Visual Studio 2012。
-   选择**文件-&gt;新增-&gt;项目**
-   在左窗格中，单击**Visual C\#**，然后选择**控制台**模板。
-   输入**TPHDBFirstSample**作为名称。
-   选择“确定”。

## <a name="create-a-model"></a>创建模型

-   右键单击解决方案资源管理器中的项目名称并选择**添加-&gt;新项**。
-   选择**数据**从左侧的菜单，然后选择**ADO.NET 实体数据模型**模板窗格中。
-   输入**TPHModel.edmx**以及该文件的名称，然后单击**添加**。
-   在选择模型内容对话框中，选择**从数据库生成**，然后单击**下一步**。
-   单击**新的连接**。
    在连接属性对话框中，输入服务器名称 (例如， **(localdb)\\mssqllocaldb**)，选择身份验证方法、 类型**学校**数据库名称，然后单击**确定**。
    选择数据连接对话框中使用数据库连接设置更新。
-   在选择数据库对象对话框中，节点下的表，选择**人员**表。
-   单击 **“完成”**。

提供用于编辑模型的设计图面，实体设计器随即出现。 在选择数据库对象对话框中选择的所有对象都添加到模型中。

它是如何**人员**表查找数据库中。

![Person 表](~/ef6/media/persontable.png) 

## <a name="implement-table-per-hierarchy-inheritance"></a>实现每个层次结构一个表继承

**Person**表中有**鉴别器**列，该列可以具有两个值之一:"学生"和"讲师"。 根据值**Person**表将映射到**学生**实体或**讲师**实体。 **Person**表还包含两个列**HireDate**和**EnrollmentDate**，它必须是**可以为 null**因为用户不能为学生和同时在一名讲师，（至少不在本演练）。

### <a name="add-new-entities"></a>添加新实体

-   添加新实体。
    若要执行此操作，右键单击实体框架设计器设计图面的空白区域，然后选择**Add-&gt;实体**。
-   类型**讲师**有关**实体名称**，然后选择**人员**从下拉列表中为**基类型**。
-   单击 **“确定”**。
-   添加另一个新实体。 类型**学生**有关**实体名称**，然后选择**人员**从下拉列表中为**基类型**。

两个新的实体类型已添加到设计图面。 从新的实体类型有一个箭头**Person**实体类型; 这指示**人员**是新的实体类型的基类型。

-   右键单击**HireDate**的属性**人员**实体。 选择**剪切**（或使用 Ctrl-X 键）。
-   右键单击**讲师**实体，然后选择**粘贴**（或使用 Ctrl + V 键）。
-   右键单击**HireDate**属性，然后选择**属性**。
-   在中**属性**窗口中，将**Nullable**属性设置为**false**。
-   右键单击**EnrollmentDate**的属性**人员**实体。 选择**剪切**（或使用 Ctrl-X 键）。
-   右键单击**学生**实体，然后选择**粘贴 （或使用 Ctrl + V 键）。**
-   选择**EnrollmentDate**属性并设置**Nullable**属性设置为**false**。
-   选择**人员**实体类型。 在中**属性**窗口中，将其**抽象**属性设置为**true**。
-   删除**鉴别器**属性从**人员**。 应删除的原因是下一节中所述。

### <a name="map-the-entities"></a>将实体映射

-   右键单击**讲师**，然后选择**表映射。**
    Instructor 实体映射详细信息窗口中选择。
-   单击**&lt;添加表或视图&gt;** 中**映射详细信息**窗口。
    **&lt;添加表或视图&gt;** 字段将变成一个下拉列表的表或视图到所选的实体可以映射。
-   选择**人员**从下拉列表。
-   **映射详细信息**窗口将会更新默认的列映射和添加条件的选项。
-   单击**&lt;添加条件&gt;**。
    **&lt;添加条件&gt;** 字段变为一个下拉列表可以为其设置条件的列。
-   选择**鉴别器**从下拉列表。
-   在中**运算符**的列**映射详细信息**窗口中，选择 = 从下拉列表。
-   在中**值/属性**列中，键入**讲师**。 最终结果应如下所示：

    ![映射的详细信息](~/ef6/media/mappingdetails2.png)

-   重复这些步骤**学生**实体类型，但请等于条件**学生**值。  
    *我们想要删除的原因**鉴别器**属性，是因为您不能超过一次将映射的表列。将为条件映射时，使用此列，因此不能为属性映射还使用它。它可用于这种，如果条件使用的唯一方式**Is Null**或**Is Not Null**比较。*

“每个层次结构一个表”继承实现完毕。

![最终 TPH](~/ef6/media/finaltph.png)

## <a name="use-the-model"></a>使用模型

打开**Program.cs**文件，其中**Main**方法的定义。 粘贴下面的代码插入**Main**函数。 代码执行三个查询。 第一个查询返回所有**人员**对象。 第二个查询使用**OfType**方法返回**讲师**对象。 第三个查询使用**OfType**方法返回**学生**对象。

``` csharp
    using (var context = new SchoolEntities())
    {
        Console.WriteLine("All people:");
        foreach (var person in context.People)
        {
            Console.WriteLine("    {0} {1}", person.FirstName, person.LastName);
        }

        Console.WriteLine("Instructors only: ");
        foreach (var person in context.People.OfType<Instructor>())
        {
            Console.WriteLine("    {0} {1}", person.FirstName, person.LastName);
        }

        Console.WriteLine("Students only: ");
        foreach (var person in context.People.OfType<Student>())
        {
            Console.WriteLine("    {0} {1}", person.FirstName, person.LastName);
        }
    }
```
