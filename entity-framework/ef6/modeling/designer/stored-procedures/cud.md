---
title: 设计器 CUD 存储过程-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 1e773972-2da5-45e0-85a2-3cf3fbcfa5cf
ms.openlocfilehash: 35a00aa817c8643352c517c233977efd49e3baac
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489552"
---
# <a name="designer-cud-stored-procedures"></a>设计器 CUD 存储过程
此分步演练演示如何将映射创建\\插入、 更新和删除 (CUD) 操作的实体类型到存储过程使用实体框架设计器 （EF 设计器）。  默认情况下，实体框架会自动生成 SQL 语句的 CUD 操作，但还可以将存储的过程映射到这些操作。  

请注意，Code First 不支持映射到存储的过程或函数。 但是，可以通过使用 System.Data.Entity.DbSet.SqlQuery 方法中调用存储的过程或函数。 例如：
``` csharp
var query = context.Products.SqlQuery("EXECUTE [dbo].[GetAllProducts]");
```

## <a name="considerations-when-mapping-the-cud-operations-to-stored-procedures"></a>映射到 CUD 操作时的注意事项的存储过程

时 CUD 操作映射到存储过程，应该注意以下事项： 

-   如果 CUD 操作的其中一个映射到存储过程，请将所有这些映射。 如果未映射所有三个，映射的操作将失败，如果执行且**UpdateException**将引发。
-   必须将存储过程的每个参数映射到实体属性。
-   如果服务器生成的主键值为插入的行，必须将此值映射回实体的键属性。 在示例中， **InsertPerson**存储的过程作为存储的过程的结果集的一部分返回新创建的主键。 为主键映射到实体键 (**PersonID**) 使用**&lt;添加结果绑定&gt;** EF 设计器的功能。
-   存储的过程调用是使用概念模型中实体的映射的 1:1。 例如，如果您在概念模型，然后映射中实现继承层次结构 CUD 存储过程**父**（基） 和**子**（派生） 的实体，保存**子**更改只会调用**子**的存储过程，它不会触发**父**的存储过程调用。

## <a name="prerequisites"></a>系统必备

若要完成此演练，您需要：

- 最新版本的 Visual Studio。
- [School 示例数据库](~/ef6/resources/school-database.md)。

## <a name="set-up-the-project"></a>设置项目

-   打开 Visual Studio 2012。
-   选择**文件-&gt;新增-&gt;项目**
-   在左窗格中，单击**Visual C\#**，然后选择**控制台**模板。
-   输入**CUDSProcsSample**作为名称。
-   选择“确定”。

## <a name="create-a-model"></a>创建模型

-   右键单击解决方案资源管理器中的项目名称并选择**添加-&gt;新项**。
-   选择**数据**从左侧的菜单，然后选择**ADO.NET 实体数据模型**模板窗格中。
-   输入**CUDSProcs.edmx**以及该文件的名称，然后单击**添加**。
-   在选择模型内容对话框中，选择**从数据库生成**，然后单击**下一步**。
-   单击**新的连接**。 在连接属性对话框中，输入服务器名称 (例如， **(localdb)\\mssqllocaldb**)，选择身份验证方法、 类型**学校**数据库名称，然后单击**确定**。
    选择数据连接对话框中使用数据库连接设置更新。
-   在选择数据库对象对话框框中下,**表**节点中，选择**人员**表。
-   此外，请选择下的以下存储的过程**存储过程和函数**节点： **DeletePerson**， **InsertPerson**，和**UpdatePerson**. 
-   从 Visual Studio 2012 EF 设计器支持大容量导入的存储过程。 **导入选定的存储的过程和函数的实体模型到**默认选中的。 由于在此示例中我们拥有存储过程的插入、 更新和删除实体类型，因此我们不希望将其导入，并将取消选中此复选框。 

    ![导入 S 存储过程](~/ef6/media/importsprocs.jpg)

-   单击 **“完成”**。
    EF 设计器中，为编辑您的模型提供了设计图面上，会显示。

## <a name="map-the-person-entity-to-stored-procedures"></a>将 Person 实体映射到存储过程

-   右键单击**Person**实体类型并选择**存储过程映射**。
-   存储的过程映射将显示在**映射详细信息**窗口。
-   单击**&lt;选择插入函数&gt;**。
    该字段变成一个下拉列表，该列表中包含存储模型中可以映射到概念模型中的实体类型的存储过程。
    选择**InsertPerson**从下拉列表。
-   此时将显示存储过程参数和实体属性之间的默认映射。 请注意，箭头指示映射方向：向存储过程参数提供属性值。
-   单击**&lt;添加结果绑定&gt;**。
-   类型**NewPersonID**，返回的参数的名称**InsertPerson**存储过程。 请确保不键入前导或尾部空格。
-   按 **Enter**。
-   默认情况下**NewPersonID**映射到实体键**PersonID**。 请注意，箭头指示映射的方向：向属性提供结果列的值。

    ![映射的详细信息](~/ef6/media/mappingdetails.png)

-   单击**&lt;选择更新函数&gt;** ，然后选择**UpdatePerson**从产生的下拉列表。
-   此时将显示存储过程参数和实体属性之间的默认映射。
-   单击**&lt;选择删除函数&gt;** ，然后选择**DeletePerson**从产生的下拉列表。
-   此时将显示存储过程参数和实体属性之间的默认映射。

插入、 更新和删除操作**人员**实体类型现在映射到存储过程。

如果你想要启用并发检查更新或删除的实体的存储过程时，，使用以下选项之一：

-   使用**输出**参数来返回数受影响的行从存储的过程并检查**受影响的行数参数**参数名称旁边的复选框。 如果返回的值为零，当调用该操作时， [ **OptimisticConcurrencyException** ](https://msdn.microsoft.com/library/system.data.optimisticconcurrencyexception.aspx)将引发。
-   检查**使用原始值**你想要使用的并发检查的属性旁边的复选框。 当试图更新时，写入数据发回数据库时将使用最初从数据库中读取的属性的值。 如果值与在数据库中，值不匹配**OptimisticConcurrencyException**将引发。

## <a name="use-the-model"></a>使用模型

打开**Program.cs**文件，其中**Main**方法的定义。 以下代码添加到 Main 函数。

代码创建一个新**人员**对象，然后更新对象，并最终删除对象。         

``` csharp
    using (var context = new SchoolEntities())
    {
        var newInstructor = new Person
        {
            FirstName = "Robyn",
            LastName = "Martin",
            HireDate = DateTime.Now,
            Discriminator = "Instructor"
        }

        // Add the new object to the context.
        context.People.Add(newInstructor);

        Console.WriteLine("Added {0} {1} to the context.",
            newInstructor.FirstName, newInstructor.LastName);

        Console.WriteLine("Before SaveChanges, the PersonID is: {0}",
            newInstructor.PersonID);

        // SaveChanges will call the InsertPerson sproc.  
        // The PersonID property will be assigned the value
        // returned by the sproc.
        context.SaveChanges();

        Console.WriteLine("After SaveChanges, the PersonID is: {0}",
            newInstructor.PersonID);

        // Modify the object and call SaveChanges.
        // This time, the UpdatePerson will be called.
        newInstructor.FirstName = "Rachel";
        context.SaveChanges();

        // Remove the object from the context and call SaveChanges.
        // The DeletePerson sproc will be called.
        context.People.Remove(newInstructor);
        context.SaveChanges();

        Person deletedInstructor = context.People.
            Where(p => p.PersonID == newInstructor.PersonID).
            FirstOrDefault();

        if (deletedInstructor == null)
            Console.WriteLine("A person with PersonID {0} was deleted.",
                newInstructor.PersonID);
    }
```

-   编译并运行该应用程序。 该程序生成以下输出 *
    >[!NOTE]
> PersonID 是自动生成服务器，因此很可能会看到不同数 *

```
Added Robyn Martin to the context.
Before SaveChanges, the PersonID is: 0
After SaveChanges, the PersonID is: 51
A person with PersonID 51 was deleted.
```

如果您正在使用 Visual Studio Ultimate 版本，可以使用调试器使用 Intellitrace，若要查看执行的 SQL 语句。

![Intellitrace](~/ef6/media/intellitrace.png)
