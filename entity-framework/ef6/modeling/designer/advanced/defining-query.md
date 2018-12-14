---
title: 定义查询的 EF 设计器-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: e52a297e-85aa-42f6-a922-ba960f8a4b22
ms.openlocfilehash: b1589dc12ccb50754c2e950932a2d82bc4869f6b
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489467"
---
# <a name="defining-query---ef-designer"></a>定义查询的 EF 设计器
本演练演示如何添加定义查询和相应的实体类型到使用 EF 设计器的模型。 定义查询通常用于提供类似于数据库视图，由提供的功能，但该视图定义在模型中，不是数据库。 定义查询，你可以执行 SQL 语句中指定**DefiningQuery** .edmx 文件的元素。 有关详细信息，请参阅**DefiningQuery**中[SSDL 规范](~/ef6/modeling/designer/advanced/edmx/ssdl-spec.md)。

当使用定义查询时，您还必须在模型中定义的实体类型。 实体类型用于定义查询公开的数据。 请注意，显示此实体类型的数据是只读的。

无法将参数化查询作为定义查询执行。 但是，可以通过将显示数据的实体类型的插入、更新和删除函数映射到存储过程来更新数据。 有关详细信息，请参阅[插入、 更新和删除的存储过程](~/ef6/modeling/designer/stored-procedures/cud.md)。

本主题演示如何执行以下任务。

-   添加定义查询
-   向模型添加实体类型
-   将定义查询映射到实体类型

## <a name="prerequisites"></a>系统必备

若要完成此演练，您需要：

- 最新版本的 Visual Studio。
- [School 示例数据库](~/ef6/resources/school-database.md)。

## <a name="set-up-the-project"></a>设置项目

本演练使用 Visual Studio 2012 或更高版本。

-   打开 Visual Studio。
-   在 **“文件”** 菜单上，指向 **“新建”**，然后单击 **“项目”**。
-   在左窗格中，单击**Visual C\#**，然后选择**控制台应用程序**模板。
-   输入**DefiningQuerySample**作为名称的项目并单击**确定**。

 

## <a name="create-a-model-based-on-the-school-database"></a>创建基于 School 数据库的模型

-   右键单击解决方案资源管理器中的项目名称，指向**外**，然后单击**新项**。
-   选择**数据**从左侧的菜单，然后选择**ADO.NET 实体数据模型**模板窗格中。
-   输入**DefiningQueryModel.edmx**以及该文件的名称，然后单击**添加**。
-   在选择模型内容对话框中，选择**从数据库生成**，然后单击**下一步**。
-   单击新建连接。 在连接属性对话框中，输入服务器名称 (例如， **(localdb)\\mssqllocaldb**)，选择身份验证方法、 类型**学校**数据库名称，然后单击**确定**。
    选择数据连接对话框中使用数据库连接设置更新。
-   在选择数据库对象对话框中，检查**表**节点。 这将添加到的所有表**学校**模型。
-   单击 **“完成”**。
-   在解决方案资源管理器中右键单击**DefiningQueryModel.edmx**文件，然后选择**打开方式...**.
-   选择**XML （文本） 编辑器**。

    ![XML 编辑器](~/ef6/media/xmleditor.png)

-   单击**是**如果系统提示并显示以下消息：

    ![警告 2](~/ef6/media/warning2.png)

 

## <a name="add-a-defining-query"></a>添加定义查询

在此步骤中，我们将使用 XML 编辑器来添加定义查询和的实体类型到.edmx 文件的 SSDL 部分。 

-   添加**EntitySet**向.edmx 文件 （通过 13 第 5 行） 的 SSDL 部分的元素。 指定下列各项：
    -   仅**名称**并**EntityType**的属性**EntitySet**指定元素。
    -   中使用的实体类型的完全限定名称**EntityType**属性。
    -   要执行的 SQL 语句中指定**DefiningQuery**元素。

``` xml
    <!-- SSDL content -->
    <edmx:StorageModels>
      <Schema Namespace="SchoolModel.Store" Alias="Self" Provider="System.Data.SqlClient" ProviderManifestToken="2008" xmlns:store="http://schemas.microsoft.com/ado/2007/12/edm/EntityStoreSchemaGenerator" xmlns="http://schemas.microsoft.com/ado/2009/11/edm/ssdl">
        <EntityContainer Name="SchoolModelStoreContainer">
           <EntitySet Name="GradeReport" EntityType="SchoolModel.Store.GradeReport">
              <DefiningQuery>
                SELECT CourseID, Grade, FirstName, LastName
                FROM StudentGrade
                JOIN
                (SELECT * FROM Person WHERE EnrollmentDate IS NOT NULL) AS p
                ON StudentID = p.PersonID
              </DefiningQuery>
          </EntitySet>
          <EntitySet Name="Course" EntityType="SchoolModel.Store.Course" store:Type="Tables" Schema="dbo" />
```

-   添加**EntityType**向.edmx 文件的 SSDL 部分的元素。 如图所示以下文件。 请注意以下事项：
    -   值**名称**属性相对应的值**EntityType**属性中**EntitySet**元素更高版本，尽管的完全限定名称在使用实体类型**EntityType**属性。
    -   属性名称对应于中的 SQL 语句返回的列名**DefiningQuery**元素 （上述）。
    -   在此示例中，实体键由三个属性组成以确保唯一键值。

``` xml
    <EntityType Name="GradeReport">
      <Key>
        <PropertyRef Name="CourseID" />
        <PropertyRef Name="FirstName" />
        <PropertyRef Name="LastName" />
      </Key>
      <Property Name="CourseID"
                Type="int"
                Nullable="false" />
      <Property Name="Grade"
                Type="decimal"
                Precision="3"
                Scale="2" />
      <Property Name="FirstName"
                Type="nvarchar"
                Nullable="false"
                MaxLength="50" />
      <Property Name="LastName"
                Type="nvarchar"
                Nullable="false"
                MaxLength="50" />
    </EntityType>
```

>[!NOTE]
> 如果您运行更高版本**模型更新向导**对话框中，对存储模型，包括定义查询，所做的任何更改将被覆盖。

 

## <a name="add-an-entity-type-to-the-model"></a>向模型添加实体类型

在此步骤中我们将添加到在概念模型中使用 EF 设计器的实体类型。  请注意以下事项：

-   **名称**实体的值相对应的**EntityType**属性中**EntitySet**上述元素。
-   属性名称对应于中的 SQL 语句返回的列名**DefiningQuery**上述元素。
-   在此示例中，实体键由三个属性组成以确保唯一键值。

在 EF 设计器中打开模型。

-   双击 DefiningQueryModel.edmx。
-   说**是**于以下消息：

    ![警告 2](~/ef6/media/warning2.png)

 

提供用于编辑模型的设计图面，实体设计器随即出现。

-   右键单击设计器图面上，然后选择**新添**-&gt;**实体...**.
-   指定**GradeReport**的实体名称并**CourseID**有关**键属性**。
-   右键单击**GradeReport**实体，然后选择**添加新**- &gt; **标量属性**。
-   更改到的属性的默认名称**FirstName**。
-   添加另一个标量属性，并指定**LastName**的名称。
-   添加另一个标量属性，并指定**级**的名称。
-   在中**属性**窗口中，更改**级**的**类型**属性设置为**十进制**。
-   选择**FirstName**并**LastName**属性。
-   在中**属性**窗口中，更改**EntityKey**属性值设置为**True**。

因此，以下元素已添加到**CSDL** .edmx 文件的部分。

``` xml
    <EntitySet Name="GradeReport" EntityType="SchoolModel.GradeReport" />

    <EntityType Name="GradeReport">
    . . .
    </EntityType>
```

 

## <a name="map-the-defining-query-to-the-entity-type"></a>将定义查询映射到实体类型

在此步骤中，我们将使用映射详细信息窗口中要映射的概念和存储实体类型。

-   右键单击**GradeReport**实体上设计图面并选择**表映射**。  
    **映射详细信息**显示窗口。
-   选择**GradeReport**从**&lt;添加表或视图&gt;** 下拉列表中 (位于**表**s)。  
    默认的概念之间的映射和存储**GradeReport**实体类型出现。  
    ![映射 Details3](~/ef6/media/mappingdetails.png)

因此， **EntitySetMapping**元素添加到.edmx 文件的映射部分。 

``` xml
    <EntitySetMapping Name="GradeReports">
      <EntityTypeMapping TypeName="IsTypeOf(SchoolModel.GradeReport)">
        <MappingFragment StoreEntitySet="GradeReport">
          <ScalarProperty Name="LastName" ColumnName="LastName" />
          <ScalarProperty Name="FirstName" ColumnName="FirstName" />
          <ScalarProperty Name="Grade" ColumnName="Grade" />
          <ScalarProperty Name="CourseID" ColumnName="CourseID" />
        </MappingFragment>
      </EntityTypeMapping>
    </EntitySetMapping>
```

-   编译该应用程序。

 

## <a name="call-the-defining-query-in-your-code"></a>在代码中调用定义的查询

现在可以通过使用执行定义查询**GradeReport**实体类型。 

``` csharp
    using (var context = new SchoolEntities())
    {
        var report = context.GradeReports.FirstOrDefault();
        Console.WriteLine("{0} {1} got {2}",
            report.FirstName, report.LastName, report.Grade);
    }
```
