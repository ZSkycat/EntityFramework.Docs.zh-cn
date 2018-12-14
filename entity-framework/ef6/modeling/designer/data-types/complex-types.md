---
title: 复杂类型的 EF 设计器-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 9a8228ef-acfd-4575-860d-769d2c0e18a1
ms.openlocfilehash: a3c5578acee23688c04772d2dd0a2221779af562
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489903"
---
# <a name="complex-types---ef-designer"></a>复杂类型的 EF 设计器
本主题演示如何将映射与实体框架设计器 （EF 设计器） 的复杂类型以及如何查询实体包含复杂类型的属性。

下图显示了使用 EF 设计器时使用的主窗口。

![EF 设计器](~/ef6/media/efdesigner.png)

> [!NOTE]
> 生成概念模型时，可能会在错误列表中出现有关未映射的实体和关联的警告。 可以忽略这些警告，因为您选择从模型生成数据库后，这些错误将会消失。

## <a name="what-is-a-complex-type"></a>复杂类型是什么

复杂类型是实体类型的非标量属性，实体类型允许在实体内组织标量属性。 与实体相似，复杂类型由标量属性或者其他复杂类型属性组成。

如果使用表示复杂类型的对象时，应注意以下：

-   复杂类型没有键，因此不能独立存在。 复杂类型只能作为实体类型或其他复杂类型的属性而存在。
-   复杂类型不能参与关联，并且不能包含导航属性。
-   不能为复杂类型属性**null**。 * * InvalidOperationException * * 发生时**DbContext.SaveChanges**调用并且遇到 null 复杂对象。 复杂对象的标量属性可以是**null**。
-   复杂类型不能从其他复杂类型继承。
-   必须定义为复杂类型**类**。 
-   EF 检测对复杂类型对象的成员的更改时**DbContext.DetectChanges**调用。 实体框架将调用**DetectChanges**自动以下成员调用时： **DbSet.Find**， **DbSet.Local**， **DbSet.Remove**， **DbSet.Add**， **DbSet.Attach**， **DbContext.SaveChanges**， **DbContext.GetValidationErrors**，**DbContext.Entry**， **DbChangeTracker.Entries**。

## <a name="refactor-an-entitys-properties-into-new-complex-type"></a>将实体属性重构为新的复杂类型

如果已在概念模型中有一个实体可能想要的某些属性重构为复杂类型属性。

设计器图面中，选择一个或多个属性 （不包括导航属性） 的实体，然后右键单击并选择**重构-&gt;移到新的复杂类型**。

![重构](~/ef6/media/refactor.png)

具有所选属性的新复杂类型添加到**模型浏览器**。 此复杂类型会被赋予一个默认名称。

新创建类型的复杂属性将替换选定属性。 所有属性映射都将保留。

![重构 2](~/ef6/media/refactor2.png)

## <a name="create-a-new-complex-type"></a>创建新的复杂类型

此外可以创建新的复杂类型不包含现有实体的属性。

右键单击**复杂类型**文件夹在模型浏览器，指向**AddNew 复杂类型...**. 或者，可以选择**复杂类型**文件夹，然后按**插入**键盘上键。

![添加新复杂类型](~/ef6/media/addnewcomplextype.png)

新复杂类型将添加到具有默认名称的文件夹。 现在可以为类型添加属性。

## <a name="add-properties-to-a-complex-type"></a>将属性添加到复杂类型

复杂类型的属性可以是标量类型或现有的复杂类型。 但是，复杂类型属性无法具有循环引用。 例如，一种复杂类型**OnsiteCourseDetails**不能具有复杂类型的属性**OnsiteCourseDetails**。

可以通过下面列出的任何方式向复杂类型添加属性。

-   右键单击模型浏览器中的复杂类型，指向**外**，然后指向**标量属性**或**复杂属性**，然后选择所需的属性类型。 或者，您可以选择一种复杂类型，然后按**插入**键盘上键。  

    ![将属性添加到复杂类型](~/ef6/media/addpropertiestocomplextype.png)

    新属性将添加到具有默认名称的复杂类型。

- OR-

-   右键单击实体属性**EF 设计器**面，然后选择**副本**，然后右键单击中的复杂类型**模型浏览器**，然后选择**粘贴**。

## <a name="rename-a-complex-type"></a>重命名复杂类型

重命名复杂类型时，将通过项目更新对类型的所有引用。

-   缓慢双击复杂类型中的**模型浏览器**。
    名称将选定并处于编辑模式。

- OR-

-   右击复杂类型中的**模型浏览器**，然后选择**重命名**。

- OR-

-   在模型浏览器中选择复杂类型，按 F2 键。

- OR-

-   右击复杂类型中的**模型浏览器**，然后选择**属性**。 在编辑该名称**属性**窗口。

## <a name="add-an-existing-complex-type-to-an-entity-and-map-its-properties-to-table-columns"></a>将现有复杂类型添加到实体并将其属性映射到表中的列

1.  右键单击实体，指向**新添**，然后选择**复杂属性**。
    具有默认名称的复杂类型属性将添加到该实体。 默认类型（从现有复杂类型中选择）将指派给该属性。
2.  将所需的类型分配给中的属性**属性**窗口。
    在将复杂类型属性添加到实体后，必须将其属性映射到表列。
3.  右键单击设计图面上或在实体类型**模型浏览器**，然后选择**表映射**。
    中将显示表映射**映射详细信息**窗口。
4.  展开**映射到&lt;表名&gt;** 节点。
    一个**列映射**节点出现。
5.  展开**列映射**节点。
    出现一个包含表中所有列的列表。 默认属性 （如果有） 的列映射到下列出**值/属性**标题。
6.  选择你想要映射的列，然后右键单击相应**值/属性**字段。
    出现一个包含所有标量属性的下拉列表。
7.  选择适当的属性。

    ![将复杂类型映射](~/ef6/media/mapcomplextype.png)

8.  对每一个表列重复步骤 6 和 7。

>[!NOTE]
> 若要删除的列映射，请选择你想要将映射，然后单击列**值/属性**字段。 然后，选择**删除**从下拉列表。

## <a name="map-a-function-import-to-a-complex-type"></a>将函数导入映射到复杂类型

函数导入基于存储过程。 若要将函数导入映射到复杂类型，相应存储过程返回的列必须在数量上与复杂类型的属性匹配并且必须具有与属性类型兼容的存储类型。

-   双击你想要将映射到复杂类型导入的函数。

    ![函数导入](~/ef6/media/functionimports.png)

-   填入新函数导入的设置，如下所示：
    -   指定要为其创建中的函数导入该存储的过程**存储过程名称**字段。 此字段是一个下拉列表，其中显示存储模型中的所有存储过程。
    -   指定的名称中的函数导入**函数导入名称**字段。
    -   选择**复杂**作为返回类型，然后通过从下拉列表中选择适当的类型指定特定的复杂返回类型。

        ![编辑函数导入](~/ef6/media/editfunctionimport.png)

-   单击 **“确定”**。
    此时将在概念模型中创建函数导入项。

### <a name="customize-column-mapping-for-function-import"></a>自定义的列映射的函数导入

-   函数导入模型浏览器中的右键单击并选择**函数导入映射**。
    **映射的详细信息**窗口将出现并显示函数导入的默认映射。 箭头指示列值和属性值之间的映射。 默认情况，假设列名称与复杂类型的属性名称相同。 默认的列名称以灰色文本显示。
-   如有必要，请更改列名称以匹配由对应于函数导入的存储过程返回的列名称。

## <a name="delete-a-complex-type"></a>删除复杂类型

删除复杂类型时，从概念模型删除类型，并且将删除该类型所有实例的映射。 但是，不更新对类型的引用。 例如，如果实体具有类型 ComplexType1 的复杂类型属性中, 删除 ComplexType1**模型浏览器**，未更新相应的实体属性。 因为它包含引用已删除复杂类型的实体，将不验证模型。 可以使用实体设计器更新或删除对已删除复杂类型的引用。

-   右键单击模型浏览器中的复杂类型并选择**删除**。

- OR-

-   在模型浏览器中选择复杂类型，然后按键盘上的“Delete”键。

## <a name="query-for-entities-containing-properties-of-complex-type"></a>查询的实体包含复杂类型的属性

下面的代码演示如何执行返回实体集合包含复杂类型属性的类型对象的查询。

``` csharp
    using (SchoolEntities context = new SchoolEntities())
    {
        var courses =
            from c in context.OnsiteCourses
            order by c.Details.Time
            select c;

        foreach (var c in courses)
        {
            Console.WriteLine("Time: " + c.Details.Time);
            Console.WriteLine("Days: " + c.Details.Days);
            Console.WriteLine("Location: " + c.Details.Location);
        }
    }
```
