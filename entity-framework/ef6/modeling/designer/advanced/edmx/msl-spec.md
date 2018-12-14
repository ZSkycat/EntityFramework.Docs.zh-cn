---
title: MSL 规范的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 13ae7bc1-74b4-4ee4-8d73-c337be841467
ms.openlocfilehash: 6bff1f5407bc0546e60b5bee1178be9aa4748bd8
ms.sourcegitcommit: 29f928a6116771fe78f306846e6f2d45cbe8d1f4
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/01/2018
ms.locfileid: "47460132"
---
# <a name="msl-specification"></a>MSL 规范
映射规范语言 (MSL) 是一种基于 XML 的语言描述概念模型和实体框架应用程序的存储模型之间的映射。

在实体框架应用程序，映射元数据从.msl 文件 （以 MSL 编写） 加载在生成时。 实体框架使用在运行时映射元数据将针对特定于应用商店的命令对概念模型的查询。

在设计时中，实体框架设计器 （EF 设计器） 将.edmx 文件中存储映射信息。 在生成时，实体设计器使用的信息在.edmx 文件创建在运行时所需的实体框架的.msl 文件

MSL 中引用的所有概念模型类型或存储模型类型的名称必须由其各自的命名空间名称限定。 有关概念模型命名空间名称的信息，请参阅[CSDL 规范](~/ef6/modeling/designer/advanced/edmx/csdl-spec.md)。 有关存储模型命名空间名称的信息，请参阅[SSDL 规范](~/ef6/modeling/designer/advanced/edmx/ssdl-spec.md)。

MSL 的版本按 XML 命名空间进行区分。

| MSL 版本 | XML Namespace                                        |
|:------------|:-----------------------------------------------------|
| MSL v1      | urn： 架构-microsoft-com:windows:storage:mapping:CS |
| MSL v2      | http://schemas.microsoft.com/ado/2008/09/mapping/cs  |
| MSL v3      | http://schemas.microsoft.com/ado/2009/11/mapping/cs  |

## <a name="alias-element-msl"></a>Alias 元素 (MSL)

**别名**以映射规范语言 （msl） 表示的元素是用于定义概念模型和存储模型命名空间的别名映射元素的子元素。 MSL 中引用的所有概念模型类型或存储模型类型的名称必须由其各自的命名空间名称限定。 有关概念模型命名空间名称的信息，请参阅架构元素 (CSDL)。 有关存储模型命名空间名称的信息，请参阅架构元素 (SSDL)。

**别名**元素不能具有子元素。

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**别名**元素。

| 特性名 | 是否必需 | “值”                                                                     |
|:---------------|:------------|:--------------------------------------------------------------------------|
| **Key**        | 是         | 指定的命名空间的别名**值**属性。 |
| **值**      | 是         | 为其命名空间的值**密钥**元素是一个别名。     |

### <a name="example"></a>示例

下面的示例演示**别名**元素，用于定义别名， `c`，概念模型中定义的类型。

``` xml
 <Mapping Space="C-S"
          xmlns="http://schemas.microsoft.com/ado/2009/11/mapping/cs">
   <Alias Key="c" Value="SchoolModel"/>
   <EntityContainerMapping StorageEntityContainer="SchoolModelStoreContainer"
                           CdmEntityContainer="SchoolModelEntities">
     <EntitySetMapping Name="Courses">
       <EntityTypeMapping TypeName="c.Course">
         <MappingFragment StoreEntitySet="Course">
           <ScalarProperty Name="CourseID" ColumnName="CourseID" />
           <ScalarProperty Name="Title" ColumnName="Title" />
           <ScalarProperty Name="Credits" ColumnName="Credits" />
           <ScalarProperty Name="DepartmentID" ColumnName="DepartmentID" />
         </MappingFragment>
       </EntityTypeMapping>
     </EntitySetMapping>
     <EntitySetMapping Name="Departments">
       <EntityTypeMapping TypeName="c.Department">
         <MappingFragment StoreEntitySet="Department">
           <ScalarProperty Name="DepartmentID" ColumnName="DepartmentID" />
           <ScalarProperty Name="Name" ColumnName="Name" />
           <ScalarProperty Name="Budget" ColumnName="Budget" />
           <ScalarProperty Name="StartDate" ColumnName="StartDate" />
           <ScalarProperty Name="Administrator" ColumnName="Administrator" />
         </MappingFragment>
       </EntityTypeMapping>
     </EntitySetMapping>
   </EntityContainerMapping>
 </Mapping>
```

## <a name="associationend-element-msl"></a>AssociationEnd 元素 (MSL)

**AssociationEnd**概念模型中的实体类型的修改函数映射到基础数据库中的存储过程时使用映射规范语言 (MSL) 中的元素。 如果存储的过程采用一个参数值保存在关联属性，修改**AssociationEnd**元素映射到的参数的属性值。 有关更多信息，请参见下面的示例。

有关实体类型的修改函数映射到存储过程的详细信息，请参阅 ModificationFunctionMapping 元素 (MSL) 和演练： 将一个实体映射到存储过程。

**AssociationEnd**元素可以具有以下子元素：

-   ScalarProperty

### <a name="applicable-attributes"></a>适用的特性

下表描述了适用于的特性**AssociationEnd**元素。

| 特性名     | 是否必需 | “值”                                                                                                                                                                             |
|:-------------------|:------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **AssociationSet** | 是         | 要映射的关联的名称。                                                                                                                                 |
| **From**           | 是         | 值**FromRole**对应于要映射的关联的导航属性的属性。 有关详细信息，请参阅 NavigationProperty 元素 (CSDL)。 |
| **若要**             | 是         | 值**ToRole**对应于要映射的关联的导航属性的属性。 有关详细信息，请参阅 NavigationProperty 元素 (CSDL)。   |

### <a name="example"></a>示例

请考虑使用以下概念模型实体类型：

``` xml
 <EntityType Name="Course">
   <Key>
     <PropertyRef Name="CourseID" />
   </Key>
   <Property Type="Int32" Name="CourseID" Nullable="false" />
   <Property Type="String" Name="Title" Nullable="false" MaxLength="100"
             FixedLength="false" Unicode="true" />
   <Property Type="Int32" Name="Credits" Nullable="false" />
   <NavigationProperty Name="Department"
                       Relationship="SchoolModel.FK_Course_Department"
                       FromRole="Course" ToRole="Department" />
 </EntityType>
```

另请考虑使用以下存储过程：

``` SQL
 CREATE PROCEDURE [dbo].[UpdateCourse]
                                @CourseID int,
                                @Title nvarchar(50),
                                @Credits int,
                                @DepartmentID int
                                AS
                                UPDATE Course SET Title=@Title,
                                                              Credits=@Credits,
                                                              DepartmentID=@DepartmentID
                                WHERE CourseID=@CourseID;
```

若要映射的更新函数`Course`实体与此存储过程，必须提供的值**DepartmentID**参数。 `DepartmentID` 的值与实体类型中的某个属性不对应；该值包含在独立的关联中，此关联的映射如下所示：

``` xml
 <AssociationSetMapping Name="FK_Course_Department"
                        TypeName="SchoolModel.FK_Course_Department"
                        StoreEntitySet="Course">
   <EndProperty Name="Course">
     <ScalarProperty Name="CourseID" ColumnName="CourseID" />
   </EndProperty>
   <EndProperty Name="Department">
     <ScalarProperty Name="DepartmentID" ColumnName="DepartmentID" />
   </EndProperty>
 </AssociationSetMapping>
```

下面的代码演示**AssociationEnd**元素用于映射**DepartmentID**属性**FK\_课程\_部门**关联要**UpdateCourse**存储过程 (所属的更新函数**课程**实体类型映射):

``` xml
 <EntitySetMapping Name="Courses">
   <EntityTypeMapping TypeName="SchoolModel.Course">
     <MappingFragment StoreEntitySet="Course">
       <ScalarProperty Name="Credits" ColumnName="Credits" />
       <ScalarProperty Name="Title" ColumnName="Title" />
       <ScalarProperty Name="CourseID" ColumnName="CourseID" />
     </MappingFragment>
   </EntityTypeMapping>
   <EntityTypeMapping TypeName="SchoolModel.Course">
     <ModificationFunctionMapping>
       <UpdateFunction FunctionName="SchoolModel.Store.UpdateCourse">
         <AssociationEnd AssociationSet="FK_Course_Department"
                         From="Course" To="Department">
           <ScalarProperty Name="DepartmentID"
                           ParameterName="DepartmentID"
                           Version="Current" />
         </AssociationEnd>
         <ScalarProperty Name="Credits" ParameterName="Credits"
                         Version="Current" />
         <ScalarProperty Name="Title" ParameterName="Title"
                         Version="Current" />
         <ScalarProperty Name="CourseID" ParameterName="CourseID"
                         Version="Current" />
       </UpdateFunction>
     </ModificationFunctionMapping>
   </EntityTypeMapping>
 </EntitySetMapping>
```

## <a name="associationsetmapping-element-msl"></a>AssociationSetMapping 元素 (MSL)

**AssociationSetMapping**以映射规范语言 （msl） 表示的元素定义概念模型和表基础数据库中的列中的关联之间的映射。

概念模型中的关联是一些属性表示基础数据库中的主键列和外键列的类型。 **AssociationSetMapping**元素使用两个 EndProperty 元素用于在数据库中定义关联类型属性与列之间的映射。 可以将条件放在条件元素具有这些映射。 将 insert、 update 和 delete 函数的关联映射到 ModificationFunctionMapping 元素与数据库中的存储过程。 QueryView 元素中使用实体 SQL 字符串来定义关联与表列之间的只读映射。

> [!NOTE]
> 如果概念模型中的某个关联定义引用约束，则关联不需要通过映射**AssociationSetMapping**元素。 如果**AssociationSetMapping**元素具有引用约束的关联存在，则中定义的映射**AssociationSetMapping**元素将被忽略。 有关详细信息，请参阅 ReferentialConstraint 元素 (CSDL)。

**AssociationSetMapping**元素可以具有以下子元素

-   QueryView （零个或一个）
-   EndProperty （零个或两个）
-   条件 （零个或多个）
-   ModificationFunctionMapping （零个或一个）

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**AssociationSetMapping**元素。

| 特性名     | 是否必需 | “值”                                                                                       |
|:-------------------|:------------|:--------------------------------------------------------------------------------------------|
| **名称**           | 是         | 要映射的概念模型关联集的名称。                      |
| **类型名称**       | 否          | 要映射的概念模型关联类型的命名空间限定的名称。 |
| **StoreEntitySet** | 否          | 要映射的表的名称。                                                 |

### <a name="example"></a>示例

下面的示例演示**AssociationSetMapping**中的元素**FK\_课程\_部门**中概念模型关联集映射到**课程**数据库表中的。 关联类型属性与表列之间的映射指定子级**EndProperty**元素。

``` xml
 <AssociationSetMapping Name="FK_Course_Department"
                        TypeName="SchoolModel.FK_Course_Department"
                        StoreEntitySet="Course">
   <EndProperty Name="Department">
     <ScalarProperty Name="DepartmentID" ColumnName="DepartmentID" />
   </EndProperty>
   <EndProperty Name="Course">
     <ScalarProperty Name="CourseID" ColumnName="CourseID" />
   </EndProperty>
 </AssociationSetMapping>
```

## <a name="complexproperty-element-msl"></a>ComplexProperty 元素 (MSL)

一个**ComplexProperty**以映射规范语言 （msl） 表示的元素定义概念模型实体类型的表列基础数据库中的复杂类型属性之间的映射。 在子 ScalarProperty 元素中指定属性-列映射。

**ComplexType**属性元素可以具有以下子元素：

-   ScalarProperty （零个或多个）
-   **ComplexProperty** （零个或多个）
-   ComplextTypeMapping （零个或多个）
-   条件 （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

下表描述了适用于的特性**ComplexProperty**元素：

| 特性名 | 是否必需 | “值”                                                                                            |
|:---------------|:------------|:-------------------------------------------------------------------------------------------------|
| **名称**       | 是         | 概念模型中要映射的实体类型的复杂属性的名称。 |
| **类型名称**   | 否          | 概念模型属性类型的命名空间限定名称。                              |

### <a name="example"></a>示例

下面的示例基于 School 模型。 下面的复杂类型已添加到概念模型中：

``` xml
 <ComplexType Name="FullName">
   <Property Type="String" Name="LastName"
             Nullable="false" MaxLength="50"
             FixedLength="false" Unicode="true" />
   <Property Type="String" Name="FirstName"
             Nullable="false" MaxLength="50"
             FixedLength="false" Unicode="true" />
 </ComplexType>
```

**LastName**并**FirstName**的属性**人员**实体类型已被替换为一个复杂属性**名称**:

``` xml
 <EntityType Name="Person">
   <Key>
     <PropertyRef Name="PersonID" />
   </Key>
   <Property Name="PersonID" Type="Int32" Nullable="false"
             annotation:StoreGeneratedPattern="Identity" />
   <Property Name="HireDate" Type="DateTime" />
   <Property Name="EnrollmentDate" Type="DateTime" />
   <Property Name="Name" Type="SchoolModel.FullName" Nullable="false" />
 </EntityType>
```

下面的 MSL 演示**ComplexProperty**元素用于映射**名称**到基础数据库中的列的属性：

``` xml
 <EntitySetMapping Name="People">
   <EntityTypeMapping TypeName="SchoolModel.Person">
     <MappingFragment StoreEntitySet="Person">
       <ScalarProperty Name="PersonID" ColumnName="PersonID" />
       <ScalarProperty Name="HireDate" ColumnName="HireDate" />
       <ScalarProperty Name="EnrollmentDate" ColumnName="EnrollmentDate" />
       <ComplexProperty Name="Name" TypeName="SchoolModel.FullName">
         <ScalarProperty Name="FirstName" ColumnName="FirstName" />
         <ScalarProperty Name="LastName" ColumnName="LastName" />  
       </ComplexProperty>
     </MappingFragment>
   </EntityTypeMapping>
 </EntitySetMapping>
```

## <a name="complextypemapping-element-msl"></a>ComplexTypeMapping 元素 (MSL)

**ComplexTypeMapping**以映射规范语言 （msl） 表示的元素是 ResultMapping 元素的子元素，定义中的基础概念模型中的函数导入和存储的过程之间的映射当满足以下条件的数据库：

-   函数导入返回一个概念复杂类型。
-   存储过程返回的列的名称与复杂类型的属性名称不完全匹配。

默认情况下，存储过程返回的列与复杂类型返回的列之间的映射基于列名称和属性名称。 如果列名称与属性名称不完全匹配，则必须使用**ComplexTypeMapping**元素定义的映射。 有关默认映射的示例，请参阅 FunctionImportMapping 元素 (MSL)。

**ComplexTypeMapping**元素可以具有以下子元素：

-   ScalarProperty （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

下表描述了适用于的特性**ComplexTypeMapping**元素。

| 特性名 | 是否必需 | “值”                                                                  |
|:---------------|:------------|:-----------------------------------------------------------------------|
| **类型名称**   | 是         | 要映射的复杂类型的命名空间限定的名称。 |

### <a name="example"></a>示例

请考虑使用以下存储过程：

``` SQL
 CREATE PROCEDURE [dbo].[GetGrades]
             @student_Id int
             AS
             SELECT     EnrollmentID as enroll_id,
                                                                             Grade as grade,
                                                                             CourseID as course_id,
                                                                             StudentID as student_id
                                               FROM dbo.StudentGrade
             WHERE StudentID = @student_Id
```

另请考虑使用以下概念模型复杂类型：

``` xml
 <ComplexType Name="GradeInfo">
   <Property Type="Int32" Name="EnrollmentID" Nullable="false" />
   <Property Type="Decimal" Name="Grade" Nullable="true"
             Precision="3" Scale="2" />
   <Property Type="Int32" Name="CourseID" Nullable="false" />
   <Property Type="Int32" Name="StudentID" Nullable="false" />
 </ComplexType>
```

若要创建返回前一个复杂类型的实例的函数导入，各列之间的映射存储过程返回的和必须中定义的实体类型**ComplexTypeMapping**元素：

``` xml
 <FunctionImportMapping FunctionImportName="GetGrades"
                        FunctionName="SchoolModel.Store.GetGrades" >
   <ResultMapping>
     <ComplexTypeMapping TypeName="SchoolModel.GradeInfo">
       <ScalarProperty Name="EnrollmentID" ColumnName="enroll_id"/>
       <ScalarProperty Name="CourseID" ColumnName="course_id"/>
       <ScalarProperty Name="StudentID" ColumnName="student_id"/>
       <ScalarProperty Name="Grade" ColumnName="grade"/>
     </ComplexTypeMapping>
   </ResultMapping>
 </FunctionImportMapping>
```

## <a name="condition-element-msl"></a>Condition 元素 (MSL)

**条件**以映射规范语言 （msl） 表示的元素的条件置于概念模型与基础数据库之间的映射。 XML 节点内定义的映射是有效，如果所有条件，即在指定的子**条件**满足元素。 否则，该映射无效。 例如，如果 MappingFragment 元素包含一个或多个**条件**子元素内定义的映射**MappingFragment**节点将仅在如果所有有效的子条件**条件**满足元素。

每个条件均可应用于**名称**(由指定的概念模型实体属性的名称**名称**属性)，或**ColumnName** （中的列的名称指定的数据库**ColumnName**属性)。 当**名称**设置属性，根据实体属性值检查条件。 当**ColumnName**属性设置，条件检查针对某列的值。 只有一个**名称**或**ColumnName**中，可以指定属性**条件**元素。

> [!NOTE]
> 当**条件**元素内 FunctionImportMapping 元素，仅使用**名称**特性不适用。

**条件**元素可以是以下元素的子元素：

-   AssociationSetMapping
-   ComplexProperty
-   EntitySetMapping
-   MappingFragment
-   EntityTypeMapping

**条件**元素可以具有任何子元素。

### <a name="applicable-attributes"></a>适用的特性

下表描述了适用于的特性**条件**元素：

| 特性名 | 是否必需 | “值”                                                                                                                                                                                                                                                                                         |
|:---------------|:------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **列名称** | 否          | 表列的名称，其值用于计算条件。                                                                                                                                                                                                                   |
| **IsNull**     | 否          | **True**或**False**。 如果值为 **，则返回 True**和列的值是**null**，或如果值为**False**和列的值不是**null**，条件为 true. 否则，条件为 False。 <br/> **IsNull**并**值**属性不能在同一时间使用。 |
| **值**      | 否          | 要与列值进行比较的值。 如果值不同，则该条件为 True。 否则，条件为 False。 <br/> **IsNull**并**值**属性不能在同一时间使用。                                                                       |
| **名称**       | 否          | 概念模型实体属性的名称，其值用于计算条件。 <br/> 此属性不适用如果**条件**FunctionImportMapping 元素中使用元素。                                                                           |

### <a name="example"></a>示例

下面的示例演示**条件**元素的子级作为**MappingFragment**元素。 当**HireDate**不为 null 和**EnrollmentDate**是为 null，将数据映射之间**SchoolModel.Instructor**类型和**PersonID**并**HireDate**的列**人员**表。 当**EnrollmentDate**不为 null 和**HireDate**是为 null，将数据映射之间**SchoolModel.Student**类型和**PersonID**并**注册**的列**人员**表。

``` xml
 <EntitySetMapping Name="People">
   <EntityTypeMapping TypeName="IsTypeOf(SchoolModel.Person)">
     <MappingFragment StoreEntitySet="Person">
       <ScalarProperty Name="PersonID" ColumnName="PersonID" />
       <ScalarProperty Name="FirstName" ColumnName="FirstName" />
       <ScalarProperty Name="LastName" ColumnName="LastName" />
     </MappingFragment>
   </EntityTypeMapping>
   <EntityTypeMapping TypeName="IsTypeOf(SchoolModel.Instructor)">
     <MappingFragment StoreEntitySet="Person">
       <ScalarProperty Name="PersonID" ColumnName="PersonID" />
       <ScalarProperty Name="HireDate" ColumnName="HireDate" />
       <Condition ColumnName="HireDate" IsNull="false" />
       <Condition ColumnName="EnrollmentDate" IsNull="true" />
     </MappingFragment>
   </EntityTypeMapping>
   <EntityTypeMapping TypeName="IsTypeOf(SchoolModel.Student)">
     <MappingFragment StoreEntitySet="Person">
       <ScalarProperty Name="PersonID" ColumnName="PersonID" />
       <ScalarProperty Name="EnrollmentDate"
                       ColumnName="EnrollmentDate" />
       <Condition ColumnName="EnrollmentDate" IsNull="false" />
       <Condition ColumnName="HireDate" IsNull="true" />
     </MappingFragment>
   </EntityTypeMapping>
 </EntitySetMapping>
```

## <a name="deletefunction-element-msl"></a>DeleteFunction 元素 (MSL)

**DeleteFunction**以映射规范语言 （msl） 表示的元素将概念模型中的实体类型或关联的删除函数映射到基础数据库中的存储过程。 将修改函数映射到其中的存储过程必须在存储模型中声明。 有关详细信息，请参阅函数元素 (SSDL)。

> [!NOTE]
> 如果不将所有这三个插入、 更新或删除一个实体类型到存储过程的操作、 映射的操作将失败，则在运行时执行和 UpdateException 引发。

### <a name="deletefunction-applied-to-entitytypemapping"></a>应用于 EntityTypeMapping 的 DeleteFunction

当应用于 EntityTypeMapping 元素**DeleteFunction**元素将概念模型中的实体类型的删除函数映射到存储过程。

**DeleteFunction**元素可以具有以下子元素时应用于**EntityTypeMapping**元素：

-   AssociationEnd （零个或多个）
-   ComplexProperty （零个或多个）
-   ScarlarProperty （零个或多个）

#### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**DeleteFunction**时应用于元素**EntityTypeMapping**元素。

| 特性名            | 是否必需 | “值”                                                                                                                                                    |
|:--------------------------|:------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------|
| **FunctionName**          | 是         | 删除函数要映射到的存储过程的命名空间限定名称。 存储过程必须在存储模型中声明。 |
| **RowsAffectedParameter** | 否          | 返回受影响行数的输出参数的名称。                                                                               |

#### <a name="example"></a>示例

以下示例基于 School 模型，并显示**DeleteFunction**元素的删除函数映射**人员**实体类型设置为**DeletePerson**存储的过程。 **DeletePerson**在存储模型中声明的存储的过程。

``` xml
 <EntitySetMapping Name="People">
   <EntityTypeMapping TypeName="SchoolModel.Person">
     <MappingFragment StoreEntitySet="Person">
       <ScalarProperty Name="PersonID" ColumnName="PersonID" />
       <ScalarProperty Name="LastName" ColumnName="LastName" />
       <ScalarProperty Name="FirstName" ColumnName="FirstName" />
       <ScalarProperty Name="HireDate" ColumnName="HireDate" />
       <ScalarProperty Name="EnrollmentDate"
                       ColumnName="EnrollmentDate" />
     </MappingFragment>
 </EntityTypeMapping>
   <EntityTypeMapping TypeName="SchoolModel.Person">
     <ModificationFunctionMapping>
       <InsertFunction FunctionName="SchoolModel.Store.InsertPerson">
         <ScalarProperty Name="EnrollmentDate"
                         ParameterName="EnrollmentDate" />
         <ScalarProperty Name="HireDate" ParameterName="HireDate" />
         <ScalarProperty Name="FirstName" ParameterName="FirstName" />
         <ScalarProperty Name="LastName" ParameterName="LastName" />
         <ResultBinding Name="PersonID" ColumnName="NewPersonID" />
       </InsertFunction>
       <UpdateFunction FunctionName="SchoolModel.Store.UpdatePerson">
         <ScalarProperty Name="EnrollmentDate"
                         ParameterName="EnrollmentDate"
                         Version="Current" />
         <ScalarProperty Name="HireDate" ParameterName="HireDate"
                         Version="Current" />
         <ScalarProperty Name="FirstName" ParameterName="FirstName"
                         Version="Current" />
         <ScalarProperty Name="LastName" ParameterName="LastName"
                         Version="Current" />
         <ScalarProperty Name="PersonID" ParameterName="PersonID"
                         Version="Current" />
       </UpdateFunction>
       <DeleteFunction FunctionName="SchoolModel.Store.DeletePerson">
         <ScalarProperty Name="PersonID" ParameterName="PersonID" />
       </DeleteFunction>
     </ModificationFunctionMapping>
   </EntityTypeMapping>
 </EntitySetMapping>
```

### <a name="deletefunction-applied-to-associationsetmapping"></a>应用于 AssociationSetMapping 的 DeleteFunction

当应用于 AssociationSetMapping 元素**DeleteFunction**元素将概念模型中的关联的删除函数映射到存储过程。

**DeleteFunction**元素可以具有以下子元素时应用于**AssociationSetMapping**元素：

-   EndProperty

#### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**DeleteFunction**时应用于元素**AssociationSetMapping**元素。

| 特性名            | 是否必需 | “值”                                                                                                                                                    |
|:--------------------------|:------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------|
| **FunctionName**          | 是         | 删除函数要映射到的存储过程的命名空间限定名称。 存储过程必须在存储模型中声明。 |
| **RowsAffectedParameter** | 否          | 返回受影响行数的输出参数的名称。                                                                               |

#### <a name="example"></a>示例

以下示例基于 School 模型，并显示**DeleteFunction**用于的删除函数映射元素**CourseInstructor**关联到**DeleteCourseInstructor**存储过程。 **DeleteCourseInstructor**在存储模型中声明的存储的过程。

``` xml
 <AssociationSetMapping Name="CourseInstructor"
                        TypeName="SchoolModel.CourseInstructor"
                        StoreEntitySet="CourseInstructor">
   <EndProperty Name="Person">
     <ScalarProperty Name="PersonID" ColumnName="PersonID" />
   </EndProperty>
   <EndProperty Name="Course">
     <ScalarProperty Name="CourseID" ColumnName="CourseID" />
   </EndProperty>
   <ModificationFunctionMapping>
     <InsertFunction FunctionName="SchoolModel.Store.InsertCourseInstructor" >   
       <EndProperty Name="Course">
         <ScalarProperty Name="CourseID" ParameterName="courseId"/>
       </EndProperty>
       <EndProperty Name="Person">
         <ScalarProperty Name="PersonID" ParameterName="instructorId"/>
       </EndProperty>
     </InsertFunction>
     <DeleteFunction FunctionName="SchoolModel.Store.DeleteCourseInstructor">
       <EndProperty Name="Course">
         <ScalarProperty Name="CourseID" ParameterName="courseId"/>
       </EndProperty>
       <EndProperty Name="Person">
         <ScalarProperty Name="PersonID" ParameterName="instructorId"/>
       </EndProperty>
     </DeleteFunction>
   </ModificationFunctionMapping>
 </AssociationSetMapping>
```

## <a name="endproperty-element-msl"></a>EndProperty 元素 (MSL)

**EndProperty**以映射规范语言 （msl） 表示的元素定义结束或修改函数的概念模型关联和基础数据库之间的映射。 子 ScalarProperty 元素中指定的属性列映射。

当**EndProperty**元素用于定义概念模型关联端的映射，它是 AssociationSetMapping 元素的子元素。 当**EndProperty**元素用于定义概念模型关联的修改函数映射，它是 InsertFunction 元素或 DeleteFunction 元素的子级。

**EndProperty**元素可以具有以下子元素：

-   ScalarProperty （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

下表描述了适用于的特性**EndProperty**元素：

| 特性名 | 是否必需 | “值”                                                 |
|:---------------|:------------|:------------------------------------------------------|
| name           | 是         | 要映射的关联端的名称。 |

### <a name="example"></a>示例

下面的示例演示**AssociationSetMapping**中的元素**FK\_课程\_部门**概念模型中的关联映射到**课程**数据库表中的。 关联类型属性与表列之间的映射指定子级**EndProperty**元素。

``` xml
 <AssociationSetMapping Name="FK_Course_Department"
                        TypeName="SchoolModel.FK_Course_Department"
                        StoreEntitySet="Course">
   <EndProperty Name="Department">
     <ScalarProperty Name="DepartmentID" ColumnName="DepartmentID" />
   </EndProperty>
   <EndProperty Name="Course">
     <ScalarProperty Name="CourseID" ColumnName="CourseID" />
   </EndProperty>
 </AssociationSetMapping>
```

### <a name="example"></a>示例

下面的示例演示**EndProperty**元素关联的插入和删除函数映射 (**CourseInstructor**) 到基础数据库中的存储过程。 映射到的函数在存储模型中声明。

``` xml
 <AssociationSetMapping Name="CourseInstructor"
                        TypeName="SchoolModel.CourseInstructor"
                        StoreEntitySet="CourseInstructor">
   <EndProperty Name="Person">
     <ScalarProperty Name="PersonID" ColumnName="PersonID" />
   </EndProperty>
   <EndProperty Name="Course">
     <ScalarProperty Name="CourseID" ColumnName="CourseID" />
   </EndProperty>
   <ModificationFunctionMapping>
     <InsertFunction FunctionName="SchoolModel.Store.InsertCourseInstructor" >   
       <EndProperty Name="Course">
         <ScalarProperty Name="CourseID" ParameterName="courseId"/>
       </EndProperty>
       <EndProperty Name="Person">
         <ScalarProperty Name="PersonID" ParameterName="instructorId"/>
       </EndProperty>
     </InsertFunction>
     <DeleteFunction FunctionName="SchoolModel.Store.DeleteCourseInstructor">
       <EndProperty Name="Course">
         <ScalarProperty Name="CourseID" ParameterName="courseId"/>
       </EndProperty>
       <EndProperty Name="Person">
         <ScalarProperty Name="PersonID" ParameterName="instructorId"/>
       </EndProperty>
     </DeleteFunction>
   </ModificationFunctionMapping>
 </AssociationSetMapping>
```

## <a name="entitycontainermapping-element-msl"></a>EntityContainerMapping 元素 (MSL)

**EntityContainerMapping**以映射规范语言 （msl） 表示的元素将概念模型中的实体容器映射到存储模型中的实体容器。 **EntityContainerMapping**元素是映射元素的子元素。

**EntityContainerMapping**元素可以具有下列子元素 （按所列顺序）：

-   EntitySetMapping （零个或多个）
-   AssociationSetMapping （零个或多个）
-   FunctionImportMapping （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**EntityContainerMapping**元素。

| 特性名            | 是否必需 | “值”                                                                                                                                                                                                                                                    |
|:--------------------------|:------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **StorageModelContainer** | 是         | 要映射到的存储模型实体容器的名称。                                                                                                                                                                                     |
| **CdmEntityContainer**    | 是         | 要映射的概念模型实体容器的名称。                                                                                                                                                                                  |
| **GenerateUpdateViews**   | 否          | **True**或**False**。 如果**False**，会生成任何更新视图。 此属性应设置为**False**后将会无效，因为数据可能会往返不成功的只读映射。 <br/> 默认值是 **，则返回 True**。 |

### <a name="example"></a>示例

下面的示例演示**EntityContainerMapping**映射元素**SchoolModelEntities**容器 （概念模型实体容器） 到**SchoolModelStoreContainer**容器 （存储模型实体容器）：

``` xml
 <EntityContainerMapping StorageEntityContainer="SchoolModelStoreContainer"
                         CdmEntityContainer="SchoolModelEntities">
   <EntitySetMapping Name="Courses">
     <EntityTypeMapping TypeName="c.Course">
       <MappingFragment StoreEntitySet="Course">
         <ScalarProperty Name="CourseID" ColumnName="CourseID" />
         <ScalarProperty Name="Title" ColumnName="Title" />
         <ScalarProperty Name="Credits" ColumnName="Credits" />
         <ScalarProperty Name="DepartmentID" ColumnName="DepartmentID" />
       </MappingFragment>
     </EntityTypeMapping>
   </EntitySetMapping>
   <EntitySetMapping Name="Departments">
     <EntityTypeMapping TypeName="c.Department">
       <MappingFragment StoreEntitySet="Department">
         <ScalarProperty Name="DepartmentID" ColumnName="DepartmentID" />
         <ScalarProperty Name="Name" ColumnName="Name" />
         <ScalarProperty Name="Budget" ColumnName="Budget" />
         <ScalarProperty Name="StartDate" ColumnName="StartDate" />
         <ScalarProperty Name="Administrator" ColumnName="Administrator" />
       </MappingFragment>
     </EntityTypeMapping>
   </EntitySetMapping>
 </EntityContainerMapping>
```

## <a name="entitysetmapping-element-msl"></a>EntitySetMapping 元素 (MSL)

**EntitySetMapping**的存储模型中的元素以映射规范语言 (MSL) 映射的概念模型实体中的所有类型都设置为实体集。 概念模型中的实体集是为的逻辑容器相同的类型 （和派生的类型） 的实体的实例。 存储模型中的实体集表示表或视图基础数据库中。 概念模型实体集指定的值**名称**的属性**EntitySetMapping**元素。 指定映射到的表或视图指定**StoreEntitySet**属性在每个子 MappingFragment 元素中或在**EntitySetMapping**元素本身。

**EntitySetMapping**元素可以具有以下子元素：

-   EntityTypeMapping （零个或多个）
-   QueryView （零个或一个）
-   MappingFragment （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**EntitySetMapping**元素。

| 特性名           | 是否必需 | “值”                                                                                                                                                                                                                         |
|:-------------------------|:------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **名称**                 | 是         | 要映射的概念模型实体集的名称。                                                                                                                                                             |
| **TypeName** **1**       | 否          | 要映射的概念模型实体类型的名称。                                                                                                                                                            |
| **StoreEntitySet** **1** | 否          | 要映射到的存储模型实体集的名称。                                                                                                                                                             |
| **MakeColumnsDistinct**  | 否          | **True**或**False**具体取决于是否返回仅非重复行。 <br/> 如果此属性设置为 **，则返回 True**，则**GenerateUpdateViews** EntityContainerMapping 元素的属性必须设置为**False**。 |

 

**1** **TypeName**并**StoreEntitySet**属性可用于代替 EntityTypeMapping 和 MappingFragment 子元素将单个实体类型映射到单个表。

### <a name="example"></a>示例

下面的示例演示**EntitySetMapping**将三种类型 （基类型和两个派生的类型） 映射中的元素**课程**的三个不同表中对概念模型实体集基础数据库。 通过指定表**StoreEntitySet**中每个属性**MappingFragment**元素。

``` xml
 <EntitySetMapping Name="Courses">
   <EntityTypeMapping TypeName="IsTypeOf(SchoolModel1.Course)">
     <MappingFragment StoreEntitySet="Course">
       <ScalarProperty Name="CourseID" ColumnName="CourseID" />
       <ScalarProperty Name="DepartmentID" ColumnName="DepartmentID" />
       <ScalarProperty Name="Credits" ColumnName="Credits" />
       <ScalarProperty Name="Title" ColumnName="Title" />
     </MappingFragment>
   </EntityTypeMapping>
   <EntityTypeMapping TypeName="IsTypeOf(SchoolModel1.OnlineCourse)">
     <MappingFragment StoreEntitySet="OnlineCourse">
       <ScalarProperty Name="CourseID" ColumnName="CourseID" />
       <ScalarProperty Name="URL" ColumnName="URL" />
     </MappingFragment>
   </EntityTypeMapping>
   <EntityTypeMapping TypeName="IsTypeOf(SchoolModel1.OnsiteCourse)">
     <MappingFragment StoreEntitySet="OnsiteCourse">
       <ScalarProperty Name="CourseID" ColumnName="CourseID" />
       <ScalarProperty Name="Time" ColumnName="Time" />
       <ScalarProperty Name="Days" ColumnName="Days" />
       <ScalarProperty Name="Location" ColumnName="Location" />
     </MappingFragment>
   </EntityTypeMapping>
 </EntitySetMapping>
```

## <a name="entitytypemapping-element-msl"></a>EntityTypeMapping 元素 (MSL)

**EntityTypeMapping**元素以映射规范语言 （msl） 表示基础数据库中定义的概念模型和表中的实体类型或视图之间的映射。 有关概念模型实体类型和基础数据库表或视图的信息，请参阅 EntityType 元素 (CSDL) 和 EntitySet 元素 (SSDL)。 指定要映射的概念模型实体类型**TypeName**的属性**EntityTypeMapping**元素。 指定的表或视图要映射**StoreEntitySet**子 MappingFragment 元素的属性。

ModificationFunctionMapping 子元素可用于映射插入、 更新或删除实体类型到数据库中的存储过程的函数。

**EntityTypeMapping**元素可以具有以下子元素：

-   MappingFragment （零个或多个）
-   ModificationFunctionMapping （零个或一个）
-   ScalarProperty
-   条件

> [!NOTE]
> **MappingFragment**并**ModificationFunctionMapping**元素的子元素不能**EntityTypeMapping**在同一时间元素。


> [!NOTE]
> **ScalarProperty**并**条件**元素只能是子元素的**EntityTypeMapping** FunctionImportMapping 元素内使用时的元素。

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**EntityTypeMapping**元素。

| 特性名 | 是否必需 | “值”                                                                                                                                                                                                |
|:---------------|:------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **类型名称**   | 是         | 要映射的概念模型实体类型的命名空间限定的名称。 <br/> 如果类型为抽象类型或派生类型，则值必须为 `IsOfType(Namespace-qualified_type_name)`。 |

### <a name="example"></a>示例

下面的示例演示具有两个子的 EntitySetMapping 元素**EntityTypeMapping**元素。 在第一个**EntityTypeMapping**元素， **SchoolModel.Person**实体类型映射到**人员**表。 在第二个**EntityTypeMapping**元素中的更新功能**SchoolModel.Person**类型映射到存储过程**UpdatePerson**，在数据库中.

``` xml
 <EntitySetMapping Name="People">
   <EntityTypeMapping TypeName="SchoolModel.Person">
     <MappingFragment StoreEntitySet="Person">
       <ScalarProperty Name="PersonID" ColumnName="PersonID" />
       <ScalarProperty Name="LastName" ColumnName="LastName" />
       <ScalarProperty Name="FirstName" ColumnName="FirstName" />
       <ScalarProperty Name="HireDate" ColumnName="HireDate" />
       <ScalarProperty Name="EnrollmentDate" ColumnName="EnrollmentDate" />
     </MappingFragment>
   </EntityTypeMapping>
   <EntityTypeMapping TypeName="SchoolModel.Person">
     <ModificationFunctionMapping>
       <UpdateFunction FunctionName="SchoolModel.Store.UpdatePerson">
         <ScalarProperty Name="EnrollmentDate" ParameterName="EnrollmentDate"
                         Version="Current" />
         <ScalarProperty Name="HireDate" ParameterName="HireDate"
                         Version="Current" />
         <ScalarProperty Name="FirstName" ParameterName="FirstName"
                         Version="Current" />
         <ScalarProperty Name="LastName" ParameterName="LastName"
                         Version="Current" />
         <ScalarProperty Name="PersonID" ParameterName="PersonID"
                         Version="Current" />
       </UpdateFunction>
     </ModificationFunctionMapping>
   </EntityTypeMapping>
 </EntitySetMapping>
```

### <a name="example"></a>示例

下一示例演示其中根类型为抽象类型的类型层次结构的映射。 请注意，使用`IsOfType`语法**TypeName**属性。

``` xml
 <EntitySetMapping Name="People">
   <EntityTypeMapping TypeName="IsTypeOf(SchoolModel.Person)">
     <MappingFragment StoreEntitySet="Person">
       <ScalarProperty Name="PersonID" ColumnName="PersonID" />
       <ScalarProperty Name="FirstName" ColumnName="FirstName" />
       <ScalarProperty Name="LastName" ColumnName="LastName" />
     </MappingFragment>
   </EntityTypeMapping>
   <EntityTypeMapping TypeName="IsTypeOf(SchoolModel.Instructor)">
     <MappingFragment StoreEntitySet="Person">
       <ScalarProperty Name="PersonID" ColumnName="PersonID" />
       <ScalarProperty Name="HireDate" ColumnName="HireDate" />
       <Condition ColumnName="HireDate" IsNull="false" />
       <Condition ColumnName="EnrollmentDate" IsNull="true" />
     </MappingFragment>
   </EntityTypeMapping>
   <EntityTypeMapping TypeName="IsTypeOf(SchoolModel.Student)">
     <MappingFragment StoreEntitySet="Person">
       <ScalarProperty Name="PersonID" ColumnName="PersonID" />
       <ScalarProperty Name="EnrollmentDate"
                       ColumnName="EnrollmentDate" />
       <Condition ColumnName="EnrollmentDate" IsNull="false" />
       <Condition ColumnName="HireDate" IsNull="true" />
     </MappingFragment>
   </EntityTypeMapping>
 </EntitySetMapping>
```

## <a name="functionimportmapping-element-msl"></a>FunctionImportMapping 元素 (MSL)

**FunctionImportMapping**元素以映射规范语言 （msl） 表示基础数据库中定义的函数导入在概念模型和存储的过程或函数之间的映射。 函数导入必须在概念模型中进行声明，存储的过程必须在存储模型中进行声明。 有关详细信息，请参阅 FunctionImport 元素 (CSDL) 和函数元素 (SSDL)。

> [!NOTE]
> 默认情况下，如果函数导入返回概念模型实体类型或复杂类型，则基础存储过程返回的列名称必须与概念模型类型中的属性名称完全匹配。 如果列名称与属性名称不完全匹配，则必须 ResultMapping 元素中定义映射。

**FunctionImportMapping**元素可以具有以下子元素：

-   ResultMapping （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

下表描述了适用于的特性**FunctionImportMapping**元素：

| 特性名         | 是否必需 | “值”                                                                                   |
|:-----------------------|:------------|:----------------------------------------------------------------------------------------|
| **FunctionImportName** | 是         | 概念模型中要映射的函数导入的名称。           |
| **FunctionName**       | 是         | 存储模型中要映射的函数的命名空间限定名称。 |

### <a name="example"></a>示例

下面的示例基于 School 模型。 请考虑在存储模型中使用以下函数：

``` xml
 <Function Name="GetStudentGrades" Aggregate="false"
           BuiltIn="false" NiladicFunction="false"
           IsComposable="false" ParameterTypeSemantics="AllowImplicitConversion"
           Schema="dbo">
   <Parameter Name="StudentID" Type="int" Mode="In" />
 </Function>
```

另请考虑在概念模型中使用此函数导入：

``` xml
 <FunctionImport Name="GetStudentGrades" EntitySet="StudentGrades"
                 ReturnType="Collection(SchoolModel.StudentGrade)">
   <Parameter Name="StudentID" Mode="In" Type="Int32" />
 </FunctionImport>
```

下面的示例演示**FunctionImportMapping**元素用于映射函数和函数导入上面彼此：

``` xml
 <FunctionImportMapping FunctionImportName="GetStudentGrades"
                        FunctionName="SchoolModel.Store.GetStudentGrades" />
```
 
## <a name="insertfunction-element-msl"></a>InsertFunction 元素 (MSL)

**InsertFunction**以映射规范语言 （msl） 表示的元素将概念模型中的实体类型或关联的插入函数映射到基础数据库中的存储过程。 将修改函数映射到其中的存储过程必须在存储模型中声明。 有关详细信息，请参阅函数元素 (SSDL)。

> [!NOTE]
> 如果不将所有这三个插入、 更新或删除一个实体类型到存储过程的操作、 映射的操作将失败，则在运行时执行和 UpdateException 引发。

**InsertFunction**元素可以是 ModificationFunctionMapping 元素的子元素，并应用于 EntityTypeMapping 元素或 AssociationSetMapping 元素。

### <a name="insertfunction-applied-to-entitytypemapping"></a>应用于 EntityTypeMapping 的 InsertFunction

当应用于 EntityTypeMapping 元素**InsertFunction**元素将概念模型中的实体类型的插入函数映射到存储过程。

**InsertFunction**元素可以具有以下子元素时应用于**EntityTypeMapping**元素：

-   AssociationEnd （零个或多个）
-   ComplexProperty （零个或多个）
-   ResultBinding （零个或一个）
-   ScarlarProperty （零个或多个）

#### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**InsertFunction**元素时应用于**EntityTypeMapping**元素。

| 特性名            | 是否必需 | “值”                                                                                                                                                    |
|:--------------------------|:------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------|
| **FunctionName**          | 是         | 插入函数要映射到的存储过程的命名空间限定名称。 存储过程必须在存储模型中声明。 |
| **RowsAffectedParameter** | 否          | 返回受影响行数的输出参数的名称。                                                                               |

#### <a name="example"></a>示例

以下示例基于 School 模型，并显示**InsertFunction**元素用于将 Person 实体类型的插入函数映射**InsertPerson**存储过程。 **InsertPerson**在存储模型中声明的存储的过程。

``` xml
 <EntityTypeMapping TypeName="SchoolModel.Person">
   <ModificationFunctionMapping>
     <InsertFunction FunctionName="SchoolModel.Store.InsertPerson">
       <ScalarProperty Name="EnrollmentDate"
                       ParameterName="EnrollmentDate" />
       <ScalarProperty Name="HireDate" ParameterName="HireDate" />
       <ScalarProperty Name="FirstName" ParameterName="FirstName" />
       <ScalarProperty Name="LastName" ParameterName="LastName" />
       <ResultBinding Name="PersonID" ColumnName="NewPersonID" />
     </InsertFunction>
     <UpdateFunction FunctionName="SchoolModel.Store.UpdatePerson">
       <ScalarProperty Name="EnrollmentDate"
                       ParameterName="EnrollmentDate"
                       Version="Current" />
       <ScalarProperty Name="HireDate" ParameterName="HireDate"
                       Version="Current" />
       <ScalarProperty Name="FirstName" ParameterName="FirstName"
                       Version="Current" />
       <ScalarProperty Name="LastName" ParameterName="LastName"
                       Version="Current" />
       <ScalarProperty Name="PersonID" ParameterName="PersonID"
                       Version="Current" />
     </UpdateFunction>
     <DeleteFunction FunctionName="SchoolModel.Store.DeletePerson">
       <ScalarProperty Name="PersonID" ParameterName="PersonID" />
     </DeleteFunction>
   </ModificationFunctionMapping>
 </EntityTypeMapping>
```
### <a name="insertfunction-applied-to-associationsetmapping"></a>应用于 AssociationSetMapping 的 InsertFunction

当应用于 AssociationSetMapping 元素**InsertFunction**元素将概念模型中的关联的插入函数映射到存储过程。

**InsertFunction**元素可以具有以下子元素时应用于**AssociationSetMapping**元素：

-   EndProperty

#### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**InsertFunction**时应用于元素**AssociationSetMapping**元素。

| 特性名            | 是否必需 | “值”                                                                                                                                                    |
|:--------------------------|:------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------|
| **FunctionName**          | 是         | 插入函数要映射到的存储过程的命名空间限定名称。 存储过程必须在存储模型中声明。 |
| **RowsAffectedParameter** | 否          | 返回受影响行数的输出参数的名称。                                                                               |

#### <a name="example"></a>示例

以下示例基于 School 模型，并显示**InsertFunction**元素使用的插入函数映射**CourseInstructor**关联到**InsertCourseInstructor**存储过程。 **InsertCourseInstructor**在存储模型中声明的存储的过程。

``` xml
 <AssociationSetMapping Name="CourseInstructor"
                        TypeName="SchoolModel.CourseInstructor"
                        StoreEntitySet="CourseInstructor">
   <EndProperty Name="Person">
     <ScalarProperty Name="PersonID" ColumnName="PersonID" />
   </EndProperty>
   <EndProperty Name="Course">
     <ScalarProperty Name="CourseID" ColumnName="CourseID" />
   </EndProperty>
   <ModificationFunctionMapping>
     <InsertFunction FunctionName="SchoolModel.Store.InsertCourseInstructor" >   
       <EndProperty Name="Course">
         <ScalarProperty Name="CourseID" ParameterName="courseId"/>
       </EndProperty>
       <EndProperty Name="Person">
         <ScalarProperty Name="PersonID" ParameterName="instructorId"/>
       </EndProperty>
     </InsertFunction>
     <DeleteFunction FunctionName="SchoolModel.Store.DeleteCourseInstructor">
       <EndProperty Name="Course">
         <ScalarProperty Name="CourseID" ParameterName="courseId"/>
       </EndProperty>
       <EndProperty Name="Person">
         <ScalarProperty Name="PersonID" ParameterName="instructorId"/>
       </EndProperty>
     </DeleteFunction>
   </ModificationFunctionMapping>
 </AssociationSetMapping>
```

## <a name="mapping-element-msl"></a>Mapping 元素 (MSL)

**映射**以映射规范语言 （msl） 表示的元素包含对象 （如存储模型中所述） 到数据库的概念模型中定义的映射的信息。 有关详细信息，请参阅 CSDL 规范和 SSDL 规范。

**映射**元素是映射规范的根元素。 映射规范的 XML 命名空间是 http://schemas.microsoft.com/ado/2009/11/mapping/cs 。

映射元素可以具有以下子元素（按所列顺序）：

-   别名 （零个或多个）
-   EntityContainerMapping （恰好一个）

MSL 中引用的概念模型类型和存储模型类型的名称必须由其相应的命名空间名称限定。 有关概念模型命名空间名称的信息，请参阅架构元素 (CSDL)。 有关存储模型命名空间名称的信息，请参阅架构元素 (SSDL)。 可以使用别名元素定义 MSL 中使用的命名空间的别名。

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**映射**元素。

| 特性名 | 是否必需 | “值”                                                 |
|:---------------|:------------|:------------------------------------------------------|
| 空格键      | 是         | **C S**。 这是固定值，因此不能更改。 |

### <a name="example"></a>示例

下面的示例演示**映射**基于 School 模型的一部分的元素。 有关 School 模型的详细信息，请参阅快速入门 （实体框架）：

``` xml
 <Mapping Space="C-S"
          xmlns="http://schemas.microsoft.com/ado/2009/11/mapping/cs">
   <Alias Key="c" Value="SchoolModel"/>
   <EntityContainerMapping StorageEntityContainer="SchoolModelStoreContainer"
                           CdmEntityContainer="SchoolModelEntities">
     <EntitySetMapping Name="Courses">
       <EntityTypeMapping TypeName="c.Course">
         <MappingFragment StoreEntitySet="Course">
           <ScalarProperty Name="CourseID" ColumnName="CourseID" />
           <ScalarProperty Name="Title" ColumnName="Title" />
           <ScalarProperty Name="Credits" ColumnName="Credits" />
           <ScalarProperty Name="DepartmentID" ColumnName="DepartmentID" />
         </MappingFragment>
       </EntityTypeMapping>
     </EntitySetMapping>
     <EntitySetMapping Name="Departments">
       <EntityTypeMapping TypeName="c.Department">
         <MappingFragment StoreEntitySet="Department">
           <ScalarProperty Name="DepartmentID" ColumnName="DepartmentID" />
           <ScalarProperty Name="Name" ColumnName="Name" />
           <ScalarProperty Name="Budget" ColumnName="Budget" />
           <ScalarProperty Name="StartDate" ColumnName="StartDate" />
           <ScalarProperty Name="Administrator" ColumnName="Administrator" />
         </MappingFragment>
       </EntityTypeMapping>
     </EntitySetMapping>
   </EntityContainerMapping>
 </Mapping>
```

## <a name="mappingfragment-element-msl"></a>MappingFragment 元素 (MSL)

**MappingFragment**以映射规范语言 （msl） 表示的元素定义概念模型实体类型的表或视图的数据库中的属性之间的映射。 有关概念模型实体类型和基础数据库表或视图的信息，请参阅 EntityType 元素 (CSDL) 和 EntitySet 元素 (SSDL)。 **MappingFragment**可以是 EntityTypeMapping 元素或 EntitySetMapping 元素的子元素。

**MappingFragment**元素可以具有以下子元素：

-   复杂类型 （零个或多个）
-   ScalarProperty （零个或多个）
-   条件 （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**MappingFragment**元素。

| 特性名          | 是否必需 | “值”                                                                                                                                                                                                                         |
|:------------------------|:------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **StoreEntitySet**      | 是         | 要映射的表或视图的名称。                                                                                                                                                                           |
| **MakeColumnsDistinct** | 否          | **True**或**False**具体取决于是否返回仅非重复行。 <br/> 如果此属性设置为 **，则返回 True**，则**GenerateUpdateViews** EntityContainerMapping 元素的属性必须设置为**False**。 |

### <a name="example"></a>示例

下面的示例演示**MappingFragment**元素的子级作为**EntityTypeMapping**元素。 在此示例中，属性**课程**概念模型中的类型映射到的列**课程**数据库表中的。

``` xml
 <EntitySetMapping Name="Courses">
   <EntityTypeMapping TypeName="SchoolModel.Course">
     <MappingFragment StoreEntitySet="Course">
       <ScalarProperty Name="CourseID" ColumnName="CourseID" />
       <ScalarProperty Name="Title" ColumnName="Title" />
       <ScalarProperty Name="Credits" ColumnName="Credits" />
       <ScalarProperty Name="DepartmentID" ColumnName="DepartmentID" />
     </MappingFragment>
   </EntityTypeMapping>
 </EntitySetMapping>
```

### <a name="example"></a>示例

下面的示例演示**MappingFragment**元素的子级作为**EntitySetMapping**元素。 如上面的属性的示例中所示**课程**概念模型中的类型映射到的列**课程**数据库表中的。

``` xml
 <EntitySetMapping Name="Courses" TypeName="SchoolModel.Course">
     <MappingFragment StoreEntitySet="Course">
       <ScalarProperty Name="CourseID" ColumnName="CourseID" />
       <ScalarProperty Name="Title" ColumnName="Title" />
       <ScalarProperty Name="Credits" ColumnName="Credits" />
       <ScalarProperty Name="DepartmentID" ColumnName="DepartmentID" />
     </MappingFragment>
 </EntitySetMapping>
```

## <a name="modificationfunctionmapping-element-msl"></a>ModificationFunctionMapping 元素 (MSL)

**ModificationFunctionMapping**以映射规范语言 （msl） 表示的元素映射插入、 更新和删除函数的概念模型实体类型到基础数据库中的存储过程。 **ModificationFunctionMapping**元素还可以将映射插入和删除到基础数据库中的存储过程概念模型中的多对多关联的函数。 将修改函数映射到其中的存储过程必须在存储模型中声明。 有关详细信息，请参阅函数元素 (SSDL)。

> [!NOTE]
> 如果不将所有这三个插入、 更新或删除一个实体类型到存储过程的操作、 映射的操作将失败，则在运行时执行和 UpdateException 引发。


> [!NOTE]
> 如果将继承层次结构中的一个实体的修改函数映射到存储过程，则必须将该层次结构中所有类型的修改函数都映射到存储过程。

**ModificationFunctionMapping**元素可以是 EntityTypeMapping 元素或 AssociationSetMapping 元素的子级。

**ModificationFunctionMapping**元素可以具有以下子元素：

-   DeleteFunction （零个或一个）
-   InsertFunction （零个或一个）
-   UpdateFunction （零个或一个）

任何属性都适用于**ModificationFunctionMapping**元素。

### <a name="example"></a>示例

下面的示例演示实体集映射**人**School 模型中实体集。 除了的列映射**Person**实体类型的映射的插入、 更新和删除函数**人员**显示类型。 映射到的函数在存储模型中声明。

``` xml
 <EntitySetMapping Name="People">
   <EntityTypeMapping TypeName="SchoolModel.Person">
     <MappingFragment StoreEntitySet="Person">
       <ScalarProperty Name="PersonID" ColumnName="PersonID" />
       <ScalarProperty Name="LastName" ColumnName="LastName" />
       <ScalarProperty Name="FirstName" ColumnName="FirstName" />
       <ScalarProperty Name="HireDate" ColumnName="HireDate" />
       <ScalarProperty Name="EnrollmentDate"
                       ColumnName="EnrollmentDate" />
     </MappingFragment>
 </EntityTypeMapping>
   <EntityTypeMapping TypeName="SchoolModel.Person">
     <ModificationFunctionMapping>
       <InsertFunction FunctionName="SchoolModel.Store.InsertPerson">
         <ScalarProperty Name="EnrollmentDate"
                         ParameterName="EnrollmentDate" />
         <ScalarProperty Name="HireDate" ParameterName="HireDate" />
         <ScalarProperty Name="FirstName" ParameterName="FirstName" />
         <ScalarProperty Name="LastName" ParameterName="LastName" />
         <ResultBinding Name="PersonID" ColumnName="NewPersonID" />
       </InsertFunction>
       <UpdateFunction FunctionName="SchoolModel.Store.UpdatePerson">
         <ScalarProperty Name="EnrollmentDate"
                         ParameterName="EnrollmentDate"
                         Version="Current" />
         <ScalarProperty Name="HireDate" ParameterName="HireDate"
                         Version="Current" />
         <ScalarProperty Name="FirstName" ParameterName="FirstName"
                         Version="Current" />
         <ScalarProperty Name="LastName" ParameterName="LastName"
                         Version="Current" />
         <ScalarProperty Name="PersonID" ParameterName="PersonID"
                         Version="Current" />
       </UpdateFunction>
       <DeleteFunction FunctionName="SchoolModel.Store.DeletePerson">
         <ScalarProperty Name="PersonID" ParameterName="PersonID" />
       </DeleteFunction>
     </ModificationFunctionMapping>
   </EntityTypeMapping>
 </EntitySetMapping>
```

### <a name="example"></a>示例

下面的示例演示关联集映射**CourseInstructor** School 模型中关联集。 除了的列映射**CourseInstructor**关联的插入和删除函数映射**CourseInstructor**关联显示。 映射到的函数在存储模型中声明。

``` xml
 <AssociationSetMapping Name="CourseInstructor"
                        TypeName="SchoolModel.CourseInstructor"
                        StoreEntitySet="CourseInstructor">
   <EndProperty Name="Person">
     <ScalarProperty Name="PersonID" ColumnName="PersonID" />
   </EndProperty>
   <EndProperty Name="Course">
     <ScalarProperty Name="CourseID" ColumnName="CourseID" />
   </EndProperty>
   <ModificationFunctionMapping>
     <InsertFunction FunctionName="SchoolModel.Store.InsertCourseInstructor" >   
       <EndProperty Name="Course">
         <ScalarProperty Name="CourseID" ParameterName="courseId"/>
       </EndProperty>
       <EndProperty Name="Person">
         <ScalarProperty Name="PersonID" ParameterName="instructorId"/>
       </EndProperty>
     </InsertFunction>
     <DeleteFunction FunctionName="SchoolModel.Store.DeleteCourseInstructor">
       <EndProperty Name="Course">
         <ScalarProperty Name="CourseID" ParameterName="courseId"/>
       </EndProperty>
       <EndProperty Name="Person">
         <ScalarProperty Name="PersonID" ParameterName="instructorId"/>
       </EndProperty>
     </DeleteFunction>
   </ModificationFunctionMapping>
 </AssociationSetMapping>
```
 

 

## <a name="queryview-element-msl"></a>QueryView 元素 (MSL)

**QueryView**以映射规范语言 （msl） 表示的元素定义概念模型和基础数据库中的表中之间对实体类型或关联的只读映射。 使用针对存储模型进行评估的 Entity SQL 查询定义映射，将结果集的实体或关联在概念模型中的表示。 因为查询视图是只读的，所以不能使用标准更新命令来更新查询视图所定义的类型。 可以使用修改函数来更新这些类型。 有关详细信息，请参阅如何： 将修改函数映射到存储过程。

> [!NOTE]
> 在中**QueryView**元素中包含的 Entity SQL 表达式**GroupBy**，不支持组聚合或导航属性。

 

**QueryView**元素可以是 EntitySetMapping 元素或 AssociationSetMapping 元素的子级。 在前一种情况中，查询视图定义概念模型中实体的只读映射。 在后一种情况中，查询视图定义概念模型中关联的只读映射。

> [!NOTE]
> 如果**AssociationSetMapping**元素不用于与引用约束的关联**AssociationSetMapping**元素将被忽略。 有关详细信息，请参阅 ReferentialConstraint 元素 (CSDL)。

**QueryView**元素不能具有任何子元素。

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**QueryView**元素。

| 特性名 | 是否必需 | “值”                                                                         |
|:---------------|:------------|:------------------------------------------------------------------------------|
| **类型名称**   | 否          | 要由查询视图映射的概念模型类型的名称。 |

### <a name="example"></a>示例

下面的示例演示**QueryView**的子元素**EntitySetMapping**元素，并定义查询视图映射**部门**中的实体类型School 模型。

``` xml
 <EntitySetMapping Name="Departments" >
   <QueryView>
     SELECT VALUE SchoolModel.Department(d.DepartmentID,
                                         d.Name,
                                         d.Budget,
                                         d.StartDate)
     FROM SchoolModelStoreContainer.Department AS d
     WHERE d.Budget > 150000
   </QueryView>
 </EntitySetMapping>
```

因为该查询仅返回的成员的子集**部门**在存储模型中，类型**部门**School 模型中的类型已被修改，基于此映射，如下所示：

``` xml
 <EntityType Name="Department">
   <Key>
     <PropertyRef Name="DepartmentID" />
   </Key>
   <Property Type="Int32" Name="DepartmentID" Nullable="false" />
   <Property Type="String" Name="Name" Nullable="false"
             MaxLength="50" FixedLength="false" Unicode="true" />
   <Property Type="Decimal" Name="Budget" Nullable="false"
             Precision="19" Scale="4" />
   <Property Type="DateTime" Name="StartDate" Nullable="false" />
   <NavigationProperty Name="Courses"
                       Relationship="SchoolModel.FK_Course_Department"
                       FromRole="Department" ToRole="Course" />
 </EntityType>
```

### <a name="example"></a>示例

下面的示例说明**QueryView**元素的子级作为**AssociationSetMapping**元素和定义的只读映射`FK_Course_Department`School 模型中的关联。

``` xml
 <EntityContainerMapping StorageEntityContainer="SchoolModelStoreContainer"
                         CdmEntityContainer="SchoolEntities">
   <EntitySetMapping Name="Courses" >
     <QueryView>
       SELECT VALUE SchoolModel.Course(c.CourseID,
                                       c.Title,
                                       c.Credits)
       FROM SchoolModelStoreContainer.Course AS c
     </QueryView>
   </EntitySetMapping>
   <EntitySetMapping Name="Departments" >
     <QueryView>
       SELECT VALUE SchoolModel.Department(d.DepartmentID,
                                           d.Name,
                                           d.Budget,
                                           d.StartDate)
       FROM SchoolModelStoreContainer.Department AS d
       WHERE d.Budget > 150000
     </QueryView>
   </EntitySetMapping>
   <AssociationSetMapping Name="FK_Course_Department" >
     <QueryView>
       SELECT VALUE SchoolModel.FK_Course_Department(
         CREATEREF(SchoolEntities.Departments, row(c.DepartmentID), SchoolModel.Department),
         CREATEREF(SchoolEntities.Courses, row(c.CourseID)) )
       FROM SchoolModelStoreContainer.Course AS c
     </QueryView>
   </AssociationSetMapping>
 </EntityContainerMapping>
```
 
### <a name="comments"></a>注释

可以定义查询视图来实现以下方案：

-   在概念模型中定义一个实体，该实体不包含存储模型中的实体的所有属性。 这包括不具有默认值并且不支持的属性**null**值。
-   将存储模型中计算的列映射到概念模型中实体类型的属性。
-   定义一个映射，其中用于对概念模型中的实体进行分区的条件不基于相等性。 当指定条件映射使用**条件**元素，提供的条件必须等于指定的值。 有关详细信息，请参阅条件元素 (MSL)。
-   将存储模型中的同一列映射到概念模型中的多个类型。
-   将多个类型映射到同一个表。
-   在概念模型中定义不基于关系架构中的外键的关联。
-   使用自定义业务逻辑设置概念模型中的属性值。 例如，你可以将字符串值"T"的值的数据源中的映射 **，则返回 true**，一个布尔值，在概念模型中的。
-   为查询结果定义条件筛选器。
-   强制对概念模型中的数据施加比存储模型中更少的限制。 例如，您可以制作属性在概念模型中可以为 null 即使映射到的列不支持**null**值。

为实体定义查询视图时需要考虑以下注意事项：

-   查询视图是只读的。 只能使用修改函数来更新实体。
-   通过查询视图定义实体类型时，必须也通过查询视图来定义所有相关实体。
-   当您映射到表示关系架构中的链接表的存储模型中实体的多对多关联时，必须定义**QueryView**中的元素**AssociationSetMapping**此链接表的的元素。
-   必须为类型层次结构中的所有类型定义查询视图。 可以使用下列方式来实现：
-   -   使用单个**QueryView**元素，它指定返回层次结构中的所有实体类型的联合的单一实体 SQL 查询。
    -   使用单个**QueryView**指定单个实体 SQL 查询使用 CASE 运算符可在层次结构中返回的特定实体类型的元素根据特定条件。
    -   带有附加**QueryView**层次结构中的特定类型的元素。 在这种情况下，使用**TypeName**的属性**QueryView**元素来指定每个视图的实体类型。
-   当定义查询视图时，不能指定**StorageSetName**特性，可以在**EntitySetMapping**元素。
-   当定义查询视图时， **EntitySetMapping**元素不能同时包含**属性**映射。

## <a name="resultbinding-element-msl"></a>ResultBinding 元素 (MSL)

**ResultBinding**以映射规范语言 （msl） 表示的元素映射列的值由存储过程到概念模型中的实体属性时返回实体类型修改函数映射到存储基础数据库中的过程。 例如，当标识列的值返回插入的存储过程中， **ResultBinding**元素映射到概念模型中实体类型属性返回的值。

**ResultBinding**元素可以是 InsertFunction 元素或 UpdateFunction 元素的子级。

**ResultBinding**元素不能具有任何子元素。

### <a name="applicable-attributes"></a>适用的特性

下表描述了适用于的特性**ResultBinding**元素：

| 特性名 | 是否必需 | “值”                                                                         |
|:---------------|:------------|:------------------------------------------------------------------------------|
| **名称**       | 是         | 概念模型中要映射的实体属性的名称。 |
| **列名称** | 是         | 要映射的列的名称。                                          |

### <a name="example"></a>示例

以下示例基于 School 模型，并显示**InsertFunction**元素用于映射的插入函数**人员**实体类型设置为**InsertPerson**存储的过程。 ( **InsertPerson**存储的过程如下所示，在存储模型中声明。)一个**ResultBinding**元素用于映射存储过程返回的列值 (**NewPersonID**) 到实体类型属性 (**PersonID**)。

``` xml
 <EntityTypeMapping TypeName="SchoolModel.Person">
   <ModificationFunctionMapping>
     <InsertFunction FunctionName="SchoolModel.Store.InsertPerson">
       <ScalarProperty Name="EnrollmentDate"
                       ParameterName="EnrollmentDate" />
       <ScalarProperty Name="HireDate" ParameterName="HireDate" />
       <ScalarProperty Name="FirstName" ParameterName="FirstName" />
       <ScalarProperty Name="LastName" ParameterName="LastName" />
       <ResultBinding Name="PersonID" ColumnName="NewPersonID" />
     </InsertFunction>
     <UpdateFunction FunctionName="SchoolModel.Store.UpdatePerson">
       <ScalarProperty Name="EnrollmentDate"
                       ParameterName="EnrollmentDate"
                       Version="Current" />
       <ScalarProperty Name="HireDate" ParameterName="HireDate"
                       Version="Current" />
       <ScalarProperty Name="FirstName" ParameterName="FirstName"
                       Version="Current" />
       <ScalarProperty Name="LastName" ParameterName="LastName"
                       Version="Current" />
       <ScalarProperty Name="PersonID" ParameterName="PersonID"
                       Version="Current" />
     </UpdateFunction>
     <DeleteFunction FunctionName="SchoolModel.Store.DeletePerson">
       <ScalarProperty Name="PersonID" ParameterName="PersonID" />
     </DeleteFunction>
   </ModificationFunctionMapping>
 </EntityTypeMapping>
```

以下 TRANSACT-SQL 描述**InsertPerson**存储过程：

``` SQL
 CREATE PROCEDURE [dbo].[InsertPerson]
                                @LastName nvarchar(50),
                                @FirstName nvarchar(50),
                                @HireDate datetime,
                                @EnrollmentDate datetime
                                AS
                                INSERT INTO dbo.Person (LastName,
                                                                             FirstName,
                                                                             HireDate,
                                                                             EnrollmentDate)
                                VALUES (@LastName,
                                               @FirstName,
                                               @HireDate,
                                               @EnrollmentDate);
                                SELECT SCOPE_IDENTITY() as NewPersonID;
```

## <a name="resultmapping-element-msl"></a>ResultMapping 元素 (MSL)

**ResultMapping**元素以映射规范语言 （msl） 表示基础数据库中定义概念模型中的函数导入和存储的过程之间的映射，当满足以下条件：

-   函数导入返回一个概念模型实体类型或复杂类型。
-   存储过程返回的列的名称与实体类型或复杂类型中的属性名称不完全匹配。

默认情况下，存储过程返回的列与实体类型或复杂类型中的属性之间的映射基于列名称和属性名称。 如果列名称与属性名称不完全匹配，则必须使用**ResultMapping**元素定义的映射。 有关默认映射的示例，请参阅 FunctionImportMapping 元素 (MSL)。

**ResultMapping**元素是 FunctionImportMapping 元素的子元素。

**ResultMapping**元素可以具有以下子元素：

-   EntityTypeMapping （零个或多个）
-   ComplexTypeMapping

任何属性都适用于**ResultMapping**元素。

### <a name="example"></a>示例

请考虑使用以下存储过程：

``` SQL
 CREATE PROCEDURE [dbo].[GetGrades]
             @student_Id int
             AS
             SELECT     EnrollmentID as enroll_id,
                                                                             Grade as grade,
                                                                             CourseID as course_id,
                                                                             StudentID as student_id
                                               FROM dbo.StudentGrade
             WHERE StudentID = @student_Id
```

另请考虑使用以下概念模型实体类型：

``` xml
 <EntityType Name="StudentGrade">
   <Key>
     <PropertyRef Name="EnrollmentID" />
   </Key>
   <Property Name="EnrollmentID" Type="Int32" Nullable="false"
             annotation:StoreGeneratedPattern="Identity" />
   <Property Name="CourseID" Type="Int32" Nullable="false" />
   <Property Name="StudentID" Type="Int32" Nullable="false" />
   <Property Name="Grade" Type="Decimal" Precision="3" Scale="2" />
 </EntityType>
```

若要创建函数导入返回上一个实体类型的实例，列之间的映射存储过程返回的和必须中定义的实体类型**ResultMapping**元素：

``` xml
 <FunctionImportMapping FunctionImportName="GetGrades"
                        FunctionName="SchoolModel.Store.GetGrades" >
   <ResultMapping>
     <EntityTypeMapping TypeName="SchoolModel.StudentGrade">
       <ScalarProperty Name="EnrollmentID" ColumnName="enroll_id"/>
       <ScalarProperty Name="CourseID" ColumnName="course_id"/>
       <ScalarProperty Name="StudentID" ColumnName="student_id"/>
       <ScalarProperty Name="Grade" ColumnName="grade"/>
     </EntityTypeMapping>
   </ResultMapping>
 </FunctionImportMapping>
```

## <a name="scalarproperty-element-msl"></a>ScalarProperty 元素 (MSL)

**ScalarProperty**以映射规范语言 （msl） 表示的元素将概念模型实体类型、 复杂类型或关联的属性映射到表列或基础数据库中的存储的过程参数。

> [!NOTE]
> 将修改函数映射到其中的存储过程必须在存储模型中声明。 有关详细信息，请参阅函数元素 (SSDL)。

**ScalarProperty**元素可以是以下元素的子元素：

-   MappingFragment
-   InsertFunction
-   UpdateFunction
-   DeleteFunction
-   EndProperty
-   ComplexProperty
-   ResultMapping

为的子**MappingFragment**， **ComplexProperty**，或**EndProperty**元素**ScalarProperty**元素的属性映射在数据库中的列对概念模型。 为的子**InsertFunction**， **UpdateFunction**，或**DeleteFunction**元素**ScalarProperty**元素的属性映射在存储的过程参数对概念模型。

**ScalarProperty**元素不能具有任何子元素。

### <a name="applicable-attributes"></a>适用的特性

将应用于的特性**ScalarProperty**元素而异的元素的角色。

下表描述了所适用的属性**ScalarProperty**元素用于将概念模型属性映射到数据库中的列：

| 特性名 | 是否必需 | “值”                                                           |
|:---------------|:------------|:----------------------------------------------------------------|
| **名称**       | 是         | 要映射的概念模型属性的名称。 |
| **列名称** | 是         | 要映射的表列的名称。              |

下表描述了适用于的特性**ScalarProperty**元素用于将概念模型属性映射到存储的过程参数时：

| 特性名    | 是否必需 | “值”                                                                                                                                           |
|:------------------|:------------|:------------------------------------------------------------------------------------------------------------------------------------------------|
| **名称**          | 是         | 要映射的概念模型属性的名称。                                                                                 |
| **参数名** | 是         | 正在映射的参数的名称。                                                                                                 |
| **Version**       | 否          | **当前**或**原始**具体取决于是否当前值或属性的原始值应使用为了进行并发检查。 |

### <a name="example"></a>示例

下面的示例演示**ScalarProperty**元素使用以下两种方式：

-   若要映射的属性**Person**实体类型的列**人员**表。
-   若要映射的属性**Person**实体类型的参数**UpdatePerson**存储过程。 存储过程在存储模型中声明。

``` xml
 <EntitySetMapping Name="People">
   <EntityTypeMapping TypeName="SchoolModel.Person">
     <MappingFragment StoreEntitySet="Person">
       <ScalarProperty Name="PersonID" ColumnName="PersonID" />
       <ScalarProperty Name="LastName" ColumnName="LastName" />
       <ScalarProperty Name="FirstName" ColumnName="FirstName" />
       <ScalarProperty Name="HireDate" ColumnName="HireDate" />
       <ScalarProperty Name="EnrollmentDate"
                       ColumnName="EnrollmentDate" />
     </MappingFragment>
 </EntityTypeMapping>
   <EntityTypeMapping TypeName="SchoolModel.Person">
     <ModificationFunctionMapping>
       <InsertFunction FunctionName="SchoolModel.Store.InsertPerson">
         <ScalarProperty Name="EnrollmentDate"
                         ParameterName="EnrollmentDate" />
         <ScalarProperty Name="HireDate" ParameterName="HireDate" />
         <ScalarProperty Name="FirstName" ParameterName="FirstName" />
         <ScalarProperty Name="LastName" ParameterName="LastName" />
         <ResultBinding Name="PersonID" ColumnName="NewPersonID" />
       </InsertFunction>
       <UpdateFunction FunctionName="SchoolModel.Store.UpdatePerson">
         <ScalarProperty Name="EnrollmentDate"
                         ParameterName="EnrollmentDate"
                         Version="Current" />
         <ScalarProperty Name="HireDate" ParameterName="HireDate"
                         Version="Current" />
         <ScalarProperty Name="FirstName" ParameterName="FirstName"
                         Version="Current" />
         <ScalarProperty Name="LastName" ParameterName="LastName"
                         Version="Current" />
         <ScalarProperty Name="PersonID" ParameterName="PersonID"
                         Version="Current" />
       </UpdateFunction>
       <DeleteFunction FunctionName="SchoolModel.Store.DeletePerson">
         <ScalarProperty Name="PersonID" ParameterName="PersonID" />
       </DeleteFunction>
     </ModificationFunctionMapping>
   </EntityTypeMapping>
 </EntitySetMapping>
```

### <a name="example"></a>示例

下面的示例说明**ScalarProperty**元素用于映射插入和删除函数的概念模型关联到数据库中的存储过程。 存储过程在存储模型中声明。

``` xml
 <AssociationSetMapping Name="CourseInstructor"
                        TypeName="SchoolModel.CourseInstructor"
                        StoreEntitySet="CourseInstructor">
   <EndProperty Name="Person">
     <ScalarProperty Name="PersonID" ColumnName="PersonID" />
   </EndProperty>
   <EndProperty Name="Course">
     <ScalarProperty Name="CourseID" ColumnName="CourseID" />
   </EndProperty>
   <ModificationFunctionMapping>
     <InsertFunction FunctionName="SchoolModel.Store.InsertCourseInstructor" >   
       <EndProperty Name="Course">
         <ScalarProperty Name="CourseID" ParameterName="courseId"/>
       </EndProperty>
       <EndProperty Name="Person">
         <ScalarProperty Name="PersonID" ParameterName="instructorId"/>
       </EndProperty>
     </InsertFunction>
     <DeleteFunction FunctionName="SchoolModel.Store.DeleteCourseInstructor">
       <EndProperty Name="Course">
         <ScalarProperty Name="CourseID" ParameterName="courseId"/>
       </EndProperty>
       <EndProperty Name="Person">
         <ScalarProperty Name="PersonID" ParameterName="instructorId"/>
       </EndProperty>
     </DeleteFunction>
   </ModificationFunctionMapping>
 </AssociationSetMapping>
```

## <a name="updatefunction-element-msl"></a>UpdateFunction 元素 (MSL)

**UpdateFunction**以映射规范语言 （msl） 表示的元素将概念模型中的实体类型的更新函数映射到基础数据库中的存储过程。 将修改函数映射到其中的存储过程必须在存储模型中声明。 有关详细信息，请参阅函数元素 (SSDL)。

> [!NOTE]
>  如果不将所有这三个插入、 更新或删除一个实体类型到存储过程的操作、 映射的操作将失败，则在运行时执行和 UpdateException 引发。

**UpdateFunction**元素可以是 ModificationFunctionMapping 元素的子元素，并应用于 EntityTypeMapping 元素。

**UpdateFunction**元素可以具有以下子元素：

-   AssociationEnd （零个或多个）
-   ComplexProperty （零个或多个）
-   ResultBinding （零个或一个）
-   ScarlarProperty （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**UpdateFunction**元素。

| 特性名            | 是否必需 | “值”                                                                                                                                                    |
|:--------------------------|:------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------|
| **FunctionName**          | 是         | 更新函数要映射到的存储过程的命名空间限定名称。 存储过程必须在存储模型中声明。 |
| **RowsAffectedParameter** | 否          | 返回受影响行数的输出参数的名称。                                                                               |

### <a name="example"></a>示例

以下示例基于 School 模型，并显示**UpdateFunction**元素使用的更新函数映射**人员**实体类型设置为**UpdatePerson**存储的过程。 **UpdatePerson**在存储模型中声明的存储的过程。

``` xml
 <EntityTypeMapping TypeName="SchoolModel.Person">
   <ModificationFunctionMapping>
     <InsertFunction FunctionName="SchoolModel.Store.InsertPerson">
       <ScalarProperty Name="EnrollmentDate"
                       ParameterName="EnrollmentDate" />
       <ScalarProperty Name="HireDate" ParameterName="HireDate" />
       <ScalarProperty Name="FirstName" ParameterName="FirstName" />
       <ScalarProperty Name="LastName" ParameterName="LastName" />
       <ResultBinding Name="PersonID" ColumnName="NewPersonID" />
     </InsertFunction>
     <UpdateFunction FunctionName="SchoolModel.Store.UpdatePerson">
       <ScalarProperty Name="EnrollmentDate"
                       ParameterName="EnrollmentDate"
                       Version="Current" />
       <ScalarProperty Name="HireDate" ParameterName="HireDate"
                       Version="Current" />
       <ScalarProperty Name="FirstName" ParameterName="FirstName"
                       Version="Current" />
       <ScalarProperty Name="LastName" ParameterName="LastName"
                       Version="Current" />
       <ScalarProperty Name="PersonID" ParameterName="PersonID"
                       Version="Current" />
     </UpdateFunction>
     <DeleteFunction FunctionName="SchoolModel.Store.DeletePerson">
       <ScalarProperty Name="PersonID" ParameterName="PersonID" />
     </DeleteFunction>
   </ModificationFunctionMapping>
 </EntityTypeMapping>
```
