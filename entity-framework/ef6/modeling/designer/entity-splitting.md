---
title: 设计器的实体拆分的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: aa2dd48a-1f0e-49dd-863d-d6b4f5834832
ms.openlocfilehash: ba1895ae491cec909ff88a8784eea82f1876f595
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490840"
---
# <a name="designer-entity-splitting"></a>设计器的实体拆分
本演练演示如何通过修改具有实体框架设计器 （EF 设计器） 的模型的实体类型映射到两个表。 如果多个表共享同一个键，则可以将一个实体映射到这些表。 适用于将一个实体类型映射到两个表的概念，可以轻松扩展为将一个实体类型映射到两个以上的表。

下图显示了使用 EF 设计器时使用的主窗口。

![EF 设计器](~/ef6/media/efdesigner.png)

## <a name="prerequisites"></a>系统必备

Visual Studio 2012 或 Visual Studio 2010、 Ultimate、 Premium、 Professional 或 Web Express edition。

## <a name="create-the-database"></a>创建数据库

随 Visual Studio 一起安装的数据库服务器是已安装的 Visual Studio 的版本而异：

-   如果使用 Visual Studio 2012，则您将创建 LocalDB 数据库。
-   如果您使用的 Visual Studio 2010 系统将创建一个 SQL Express 数据库。

首先我们将使用我们要合并到单个实体的两个表创建一个数据库。

-   打开 Visual Studio
-   **视图-&gt;服务器资源管理器**
-   右键单击**数据连接-&gt;添加连接...**
-   如果你尚未连接到数据库服务器资源管理器之前将需要选择**Microsoft SQL Server**作为数据源
-   连接到 LocalDB 或 SQL Express，具体取决于哪一个已安装
-   输入**EntitySplitting**作为数据库名称
-   选择**确定**并将你想要创建新数据库，请选择要求你**是**
-   新的数据库现在将出现在服务器资源管理器
-   如果你正在使用 Visual Studio 2012
    -   右键单击服务器资源管理器中的数据库，然后选择**新查询**
    -   将下面的 SQL 复制到新的查询，然后右键单击查询并选择**Execute**
-   如果您使用的 Visual Studio 2010
    -   选择**数据-&gt; Transact SQL 编辑器-&gt;新建查询连接...**
    -   输入 **。\\SQLEXPRESS**作为服务器名称，然后单击**确定**
    -   选择**EntitySplitting**数据库从下拉列表顶部的查询编辑器
    -   将下面的 SQL 复制到新的查询，然后右键单击查询并选择**执行 SQL**

``` SQL
CREATE TABLE [dbo].[Person] (
[PersonId] INT IDENTITY (1, 1) NOT NULL,
[FirstName] NVARCHAR (200) NULL,
[LastName] NVARCHAR (200) NULL,
CONSTRAINT [PK_Person] PRIMARY KEY CLUSTERED ([PersonId] ASC)
);

CREATE TABLE [dbo].[PersonInfo] (
[PersonId] INT NOT NULL,
[Email] NVARCHAR (200) NULL,
[Phone] NVARCHAR (50) NULL,
CONSTRAINT [PK_PersonInfo] PRIMARY KEY CLUSTERED ([PersonId] ASC),
CONSTRAINT [FK_Person_PersonInfo] FOREIGN KEY ([PersonId]) REFERENCES [dbo].[Person] ([PersonId]) ON DELETE CASCADE
);
```

## <a name="create-the-project"></a>创建项目

-   在 **“文件”** 菜单上，指向 **“新建”**，然后单击 **“项目”**。
-   在左窗格中，单击**Visual C\#**，然后选择**控制台应用程序**模板。
-   输入**MapEntityToTablesSample**作为名称的项目并单击**确定**。
-   单击**否**如果提示您保存第一个部分中创建的 SQL 查询。

## <a name="create-a-model-based-on-the-database"></a>创建基于数据库的模型

-   右键单击解决方案资源管理器中的项目名称，指向**外**，然后单击**新项**。
-   选择**数据**从左侧的菜单，然后选择**ADO.NET 实体数据模型**模板窗格中。
-   输入**MapEntityToTablesModel.edmx**以及该文件的名称，然后单击**添加**。
-   在选择模型内容对话框中，选择**从数据库生成**，然后单击**下一步。**
-   选择**EntitySplitting**从删除的连接，并单击**下一步**。
-   在选择数据库对象对话框中，旁边的复选框**表**节点。
    这将添加来自所有表**EntitySplitting**到模型数据库。
-   单击 **“完成”**。

提供用于编辑模型的设计图面，实体设计器随即出现。

## <a name="map-an-entity-to-two-tables"></a>将一个实体映射到两个表

在此步骤中我们将更新**人员**实体类型将数据从合并**人员**并**PersonInfo**表。

-   选择**电子邮件**并**电话**的属性 * * PersonInfo * * 实体，然后按**Ctrl + X**密钥。
-   选择 * * 人员 * * 实体，然后按**Ctrl + V**密钥。
-   在设计图面上，选择**PersonInfo**实体，然后按**删除**键盘上的按钮。
-   单击**否**当系统询问是否要删除**PersonInfo**表从模型中，我们正准备将其映射到**人员**实体。

    ![删除表](~/ef6/media/deletetables.png)

下一步骤要求**映射详细信息**窗口。 如果您无法查看此窗口中，右键单击设计图面并选择**映射详细信息**。

-   选择**Person**实体类型，然后单击**&lt;添加表或视图&gt;** 中**映射详细信息**窗口。
-   选择 * * PersonInfo * * 从下拉列表。
    **映射详细信息**窗口将会更新默认列映射，这些是适用于我们的方案。

**Person**实体类型现在所映射到**人员**并**PersonInfo**表。

![映射 2](~/ef6/media/mapping2.png)

## <a name="use-the-model"></a>使用模型

-   以下代码粘贴到 Main 方法中。

``` csharp
    using (var context = new EntitySplittingEntities())
    {
        var person = new Person
        {
            FirstName = "John",
            LastName = "Doe",
            Email = "john@example.com",
            Phone = "555-555-5555"
        };

        context.People.Add(person);
        context.SaveChanges();

        foreach (var item in context.People)
        {
            Console.WriteLine(item.FirstName);
        }
    }
```

-   编译并运行该应用程序。

针对因运行此应用程序而数据库执行以下 T-SQL 语句。 

-   以下两个**插入**语句执行因执行上下文。Savechanges （)。 它们采用的数据**Person**实体并将其之间拆分**人员**并**PersonInfo**表。

    ![插入 1](~/ef6/media/insert1.png)

    ![插入 2](~/ef6/media/insert2.png)
-   以下**选择**由于枚举在数据库中的人员执行。 它将从数据相结合**Person**并**PersonInfo**表。

    ![选择](~/ef6/media/select.png)
