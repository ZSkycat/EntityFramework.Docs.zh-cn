---
title: 设计器的表拆分的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 452f17c3-9f26-4de4-9894-8bc036e23b0f
ms.openlocfilehash: 8b0ca6778a06ed43b1365d2e5969ff15948f8004
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490682"
---
# <a name="designer-table-splitting"></a>设计器的表拆分
本演练演示如何通过修改模型与实体框架设计器 （EF 设计器） 将多个实体类型映射到单个表。

使用延迟加载以加载您的对象时，可能想要使用表拆分的一个原因延迟加载的某些属性。 您可以分隔可能会包含到另一个实体的数据量非常大并仅加载它时所需的属性。

下图显示了使用 EF 设计器时使用的主窗口。

![EF 设计器](~/ef6/media/efdesigner.png)

## <a name="prerequisites"></a>系统必备

若要完成此演练，您需要：

- 最新版本的 Visual Studio。
- [School 示例数据库](~/ef6/resources/school-database.md)。

## <a name="set-up-the-project"></a>设置项目

本演练使用 Visual Studio 2012。

-   打开 Visual Studio 2012。
-   在 **“文件”** 菜单上，指向 **“新建”**，然后单击 **“项目”**。
-   在左窗格中，单击 Visual C\#，然后选择控制台应用程序模板。
-   输入**TableSplittingSample**作为名称的项目并单击**确定**。

## <a name="create-a-model-based-on-the-school-database"></a>创建基于 School 数据库的模型

-   右键单击解决方案资源管理器中的项目名称，指向**外**，然后单击**新项**。
-   选择**数据**从左侧的菜单，然后选择**ADO.NET 实体数据模型**模板窗格中。
-   输入**TableSplittingModel.edmx**以及该文件的名称，然后单击**添加**。
-   在选择模型内容对话框中，选择**从数据库生成**，然后单击**下一步。**
-   单击新建连接。 在连接属性对话框中，输入服务器名称 (例如， **(localdb)\\mssqllocaldb**)，选择身份验证方法、 类型**学校**数据库名称，然后单击**确定**。
    选择数据连接对话框中使用数据库连接设置更新。
-   在选择数据库对象对话框中，展开**表**节点并检查**人员**表。 这将添加到指定的表**学校**模型。
-   单击 **“完成”**。

提供用于编辑模型的设计图面，实体设计器随即出现。 在您选择的所有对象**选择数据库对象**对话框的添加到模型。

## <a name="map-two-entities-to-a-single-table"></a>将两个实体映射到单个表

在本部分中，您将拆分**人员**为两个实体的实体并将其映射到单个表。

> [!NOTE]
> **人员**实体不包含任何属性可能包含大量的数据; 只是用作示例。

-   右键单击设计图面上的空白区域，指向**新添**，然后单击**实体**。
    **新实体**对话框随即出现。
-   类型**HireInfo**有关**实体名称**并**PersonID**有关**Key 属性**名称。
-   单击 **“确定”**。
-   新的实体类型创建完毕，并且显示在设计图面上。
-   选择**HireDate**的属性**人员**实体类型，然后按**Ctrl + X**密钥。
-   选择**HireInfo**实体，然后按**Ctrl + V**密钥。
-   创建之间的关联**Person**并**HireInfo**。 若要执行此操作，右键单击设计图面上的空白区域，指向**新添**，然后单击**关联**。
-   **添加关联**对话框随即出现。 **PersonHireInfo**默认情况下提供名称。
-   指定多重性**1(One)** 关系的两端。
-   按**确定**。

下一步需要**映射详细信息**窗口。 如果您无法查看此窗口中，右键单击设计图面并选择**映射详细信息**。

-   选择**HireInfo**实体类型，然后单击**&lt;添加表或视图&gt;** 中**映射详细信息**窗口。
-   选择**Person**从**&lt;添加表或视图&gt;** 字段下拉列表。 列表包含的表或视图到所选的实体可以映射。
    默认情况下，应映射相应的属性。

    ![映射](~/ef6/media/mapping.png)

-   选择**PersonHireInfo**设计图面上的关联。
-   右键单击设计图面，然后选择关联**属性**。
-   在中**属性**窗口中，选择**引用约束**属性，然后单击省略号按钮。
-   选择**Person**从**主体**下拉列表。
-   按**确定**。

 

## <a name="use-the-model"></a>使用模型

-   以下代码粘贴到 Main 方法中。

``` csharp
    using (var context = new SchoolEntities())
    {
        Person person = new Person()
        {
            FirstName = "Kimberly",
            LastName = "Morgan",
            Discriminator = "Instructor",
        };

        person.HireInfo = new HireInfo()
        {
            HireDate = DateTime.Now
        };

        // Add the new person to the context.
        context.People.Add(person);

        // Insert a row into the Person table.  
        context.SaveChanges();

        // Execute a query against the Person table.
        // The query returns columns that map to the Person entity.
        var existingPerson = context.People.FirstOrDefault();

        // Execute a query against the Person table.
        // The query returns columns that map to the Instructor entity.
        var hireInfo = existingPerson.HireInfo;

        Console.WriteLine("{0} was hired on {1}",
            existingPerson.LastName, hireInfo.HireDate);
    }
```
-   编译并运行该应用程序。

以下 T-SQL 语句执行针对**学校**由于运行此应用程序的数据库。 

-   以下**插入**因执行上下文执行。中的 savechanges （） 和合并数据**Person**并**HireInfo**实体

    ![Insert](~/ef6/media/insert.png)

-   以下**选择**因执行上下文执行。People.FirstOrDefault() 并选择只是列映射到**人员**

    ![选择 1](~/ef6/media/select1.png)

-   以下**选择**访问导航属性 existingPerson.Instructor 已执行与选择映射到列**HireInfo**

    ![选择 2](~/ef6/media/select2.png)
