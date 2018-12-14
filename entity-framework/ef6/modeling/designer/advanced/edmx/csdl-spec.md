---
title: CSDL 规范的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: c54255f4-253f-49eb-bec8-ad7927ac2fa3
ms.openlocfilehash: 438af83b8a1ad51ee8414341181412e950d0e117
ms.sourcegitcommit: 29f928a6116771fe78f306846e6f2d45cbe8d1f4
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/01/2018
ms.locfileid: "47460145"
---
# <a name="csdl-specification"></a>CSDL 规范
概念架构定义语言 (CSDL) 是一种基于 XML 的语言，它描述构成数据驱动应用程序的概念模型的实体、关系和函数。 实体框架或 WCF 数据服务可以使用此概念模型。 实体框架使用通过 CSDL 描述的元数据实体和关系数据源的概念模型中定义的映射。 有关详细信息，请参阅[SSDL 规范](~/ef6/modeling/designer/advanced/edmx/ssdl-spec.md)并[MSL 规范](~/ef6/modeling/designer/advanced/edmx/msl-spec.md)。

CSDL 是实体数据模型的实体框架的实现。

在实体框架应用程序，概念模型元数据是加载从.csdl 文件 （用 CSDL 编写） 到 System.Data.Metadata.Edm.EdmItemCollection 的实例，是可访问通过使用中的方法System.Data.Metadata.Edm.MetadataWorkspace 类。 实体框架使用概念模型元数据将针对数据源特定的命令对概念模型的查询。

在 EF 设计器将概念模型信息.edmx 文件中存储在设计时。 在生成时，EF 设计器使用.edmx 文件中的信息创建在运行时所需的实体框架的.csdl 文件。

CSDL 的版本按 XML 命名空间进行区分。

| CSDL 版本 | XML Namespace                                |
|:-------------|:---------------------------------------------|
| CSDL v1      | http://schemas.microsoft.com/ado/2006/04/edm |
| CSDL v2      | http://schemas.microsoft.com/ado/2008/09/edm |
| CSDL v3      | http://schemas.microsoft.com/ado/2009/11/edm |

 
## <a name="association-element-csdl"></a>Association 元素 (CSDL)

**关联**元素定义两个实体类型之间的关系。 关联必须指定关系中涉及的实体类型和关系的每一端可能的实体类型数量（也称为重数）。 关联端的多重性可以具有的一 (1)、 零个或一个值 (0.1) 或许多 (\*)。 在两个的子 End 元素中指定此信息。

通过导航属性或外键（如果在实体类型上公开了外键）可以访问关联一端的实体类型实例。

在应用程序中，关联的实例表示实体类型的实例之间的特定关联。 关联实例按逻辑分组在关联集中。

**关联**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   结束 （正好两个元素）
-   ReferentialConstraint （零个或一个元素）
-   批注元素 （零个或多个元素）

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**关联**元素。

| 特性名 | 是否必需 | “值”                        |
|:---------------|:------------|:-----------------------------|
| **名称**       | 是         | 关联的名称。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**关联**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示**关联**定义的元素**CustomerOrders**关联的外键不上公开时**客户**和**顺序**实体类型。 **多重性**值为每个**最终**关联的指示，许多**订单**可以关联**客户**，但只有一个**客户**可以关联**顺序**。 此外， **OnDelete**元素指示所有**订单**与特定相关的**客户**和已加载到 ObjectContext 将被删除如果**客户**被删除。

``` xml
 <Association Name="CustomerOrders">
   <End Type="ExampleModel.Customer" Role="Customer" Multiplicity="1" >
         <OnDelete Action="Cascade" />
   </End>
   <End Type="ExampleModel.Order" Role="Order" Multiplicity="*" />
 </Association>
```
 

下面的示例演示**关联**定义的元素**CustomerOrders**关联时已上公开外键**客户**并**顺序**实体类型。 通过公开的外键，实体之间的关系进行管理**ReferentialConstraint**元素。 相应的 AssociationSetMapping 元素不需要将此关联映射到数据源。

``` xml
 <Association Name="CustomerOrders">
   <End Type="ExampleModel.Customer" Role="Customer" Multiplicity="1" >
         <OnDelete Action="Cascade" />
   </End>
   <End Type="ExampleModel.Order" Role="Order" Multiplicity="*" />
   <ReferentialConstraint>
        <Principal Role="Customer">
            <PropertyRef Name="Id" />
        </Principal>
        <Dependent Role="Order">
             <PropertyRef Name="CustomerId" />
         </Dependent>
   </ReferentialConstraint>
 </Association>
```
 

 

## <a name="associationset-element-csdl"></a>AssociationSet 元素 (CSDL)

**AssociationSet**概念架构定义语言 (CSDL) 中的元素是相同类型的关联实例的逻辑容器。 关联集为对关联实例进行分组提供了定义，以便能够将它们映射到数据源。  

**AssociationSet**元素可以具有下列子元素 （按所列顺序）：

-   文档 （允许零个或一个元素）
-   结束 （所需的两个元素）
-   批注元素 （允许零个或多个元素）

**关联**属性指定关联集包含的关联的类型。 组成的一个关联集两端的实体集指定具有两个子级**最终**元素。

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**AssociationSet**元素。

| 特性名  | 是否必需 | “值”                                                                                                                                                             |
|:----------------|:------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **名称**        | 是         | 实体集的名称。 值**名称**属性不能为的值相同**关联**属性。                                 |
| **关联** | 是         | 关联集包含其实例的关联的完全限定名称。 关联必须与关联集位于同一个命名空间中。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**AssociationSet**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示**EntityContainer**元素具有两个**AssociationSet**元素：

``` xml
 <EntityContainer Name="BooksContainer" >
   <EntitySet Name="Books" EntityType="BooksModel.Book" />
   <EntitySet Name="Publishers" EntityType="BooksModel.Publisher" />
   <EntitySet Name="Authors" EntityType="BooksModel.Author" />
   <AssociationSet Name="PublishedBy" Association="BooksModel.PublishedBy">
     <End Role="Book" EntitySet="Books" />
     <End Role="Publisher" EntitySet="Publishers" />
   </AssociationSet>
   <AssociationSet Name="WrittenBy" Association="BooksModel.WrittenBy">
     <End Role="Book" EntitySet="Books" />
     <End Role="Author" EntitySet="Authors" />
   </AssociationSet>
 </EntityContainer>
```
 

 

## <a name="collectiontype-element-csdl"></a>CollectionType 元素 (CSDL)

**CollectionType**概念架构定义语言 (CSDL) 中的元素指定的函数参数或函数返回类型是一个集合。 **CollectionType**元素可以是参数元素或 ReturnType （函数） 元素的子级。 可以通过使用指定的集合类型**类型**特性或以下子元素之一：

-   **CollectionType**
-   ReferenceType
-   RowType
-   TypeRef

> [!NOTE]
> 如果两个指定集合的类型，不会验证模型**类型**属性和子元素。

 

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**CollectionType**元素。 请注意， **DefaultValue**， **MaxLength**， **FixedLength**，**精度**，**规模**， **Unicode**，并**排序规则**属性都只适用于集合的**Edmsimpletype**。

| 特性名                                                          | 是否必需 | “值”                                                                                                                                                                                                                            |
|:------------------------------------------------------------------------|:------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Type**                                                                | 否          | 集合的类型。                                                                                                                                                                                                      |
| **可以为 null**                                                            | 否          | **True** （默认值） 或**False**具体取决于属性是否可以具有 null 值。 <br/> [!NOTE]                                                                                                                 |
| > 在 CSDL v1 中，复杂类型属性必须具有`Nullable="False"`。 |             |                                                                                                                                                                                                                                  |
| **DefaultValue**                                                        | 否          | 属性的默认值。                                                                                                                                                                                               |
| **MaxLength**                                                           | 否          | 属性值的最大长度。                                                                                                                                                                                        |
| **FixedLength**                                                         | 否          | **True**或**False**具体取决于属性值是否将作为固定的长度字符串存储。                                                                                                                           |
| **精度**                                                           | 否          | 属性值的精度。                                                                                                                                                                                             |
| 缩放                                                               | 否          | 属性值的刻度。                                                                                                                                                                                                 |
| **SRID**                                                                | 否          | 系统的空间引用标识符。 仅对空间类型的属性有效。   有关详细信息，请参阅[SRID](http://en.wikipedia.org/wiki/SRID)和[SRID (SQL Server)](https://msdn.microsoft.com/library/bb964707.aspx) |
| **Unicode**                                                             | 否          | **True**或**False**具体取决于是否在属性值将存储为 Unicode 字符串。                                                                                                                                |
| **排序规则**                                                           | 否          | 指定要在数据源中使用的排序序列的字符串。                                                                                                                                                    |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**CollectionType**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示使用的模型定义函数**CollectionType**元素来指定该函数返回一系列**人员**实体类型 (指定与**ElementType**属性)。

``` xml
 <Function Name="LastNamesAfter">
        <Parameter Name="someString" Type="Edm.String"/>
        <ReturnType>
             <CollectionType  ElementType="SchoolModel.Person"/>
        </ReturnType>
        <DefiningExpression>
             SELECT VALUE p
             FROM SchoolEntities.People AS p
             WHERE p.LastName >= someString
        </DefiningExpression>
 </Function>
```
 

下面的示例演示使用的模型定义函数**CollectionType**元素来指定该函数返回的行集合 (中指定的那样**RowType**元素)。

``` xml
 <Function Name="LastNamesAfter">
   <Parameter Name="someString" Type="Edm.String" />
   <ReturnType>
    <CollectionType>
      <RowType>
        <Property Name="FirstName" Type="Edm.String" Nullable="false" />
        <Property Name="LastName" Type="Edm.String" Nullable="false" />
      </RowType>
    </CollectionType>
   </ReturnType>
   <DefiningExpression>
             SELECT VALUE ROW(p.FirstName, p.LastName)
             FROM SchoolEntities.People AS p
             WHERE p.LastName &gt;= somestring
   </DefiningExpression>
 </Function>
```
 

下面的示例演示使用的模型定义函数**CollectionType**元素来指定该函数接受作为参数的集合**部门**实体类型。

``` xml
 <Function Name="GetAvgBudget">
      <Parameter Name="Departments">
          <CollectionType>
             <TypeRef Type="SchoolModel.Department"/>
          </CollectionType>
           </Parameter>
       <ReturnType Type="Collection(Edm.Decimal)"/>
       <DefiningExpression>
             SELECT VALUE AVG(d.Budget) FROM Departments AS d
       </DefiningExpression>
 </Function>
```
 

 

## <a name="complextype-element-csdl"></a>ComplexType 元素 (CSDL)

一个**ComplexType**元素定义组成的数据结构**EdmSimpleType**属性或其他复杂类型。  复杂类型可以是实体类型的属性或其他复杂类型的属性。 复杂类型类似于复杂类型定义数据中的实体类型。 但是，在复杂类型与实体类型之间仍存在着一些重要区别：

-   复杂类型没有标识（或键），因此不能独立存在。 复杂类型只能作为实体类型或其他复杂类型的属性而存在。
-   复杂类型不能参与关联。 既不关联端可以是复杂类型，并因此不能为复杂类型定义导航属性。
-   复杂类型属性不能具有 null 值，但可以将复杂类型的每个标量属性设置为 null。

一个**ComplexType**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   属性 （零个或多个元素）
-   批注元素 （零个或多个元素）

下表描述了可应用于的特性**ComplexType**元素。

| 特性名                                                                                                 | 是否必需 | “值”                                                                                                                                                                               |
|:---------------------------------------------------------------------------------------------------------------|:------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name                                                                                                           | 是         | 复杂类型的名称。 复杂类型的名称不能与模型作用域中的其他复杂类型、实体类型或关联的名称相同。 |
| BaseType                                                                                                       | 否          | 作为所定义的复杂类型的基类型的另一个复杂类型的名称。 <br/> [!NOTE]                                                                     |
| > 此属性不是适用于 CSDL v1。 在该版本中不支持复杂类型的继承。 |             |                                                                                                                                                                                     |
| 抽象                                                                                                       | 否          | **True**或**False** （默认值） 具体取决于复杂类型是否为抽象类型。 <br/> [!NOTE]                                                                  |
| > 此属性不是适用于 CSDL v1。 该版本中的复杂类型不能是抽象类型。         |             |                                                                                                                                                                                     |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**ComplexType**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示一个复杂类型，**地址**，使用**EdmSimpleType**属性**StreetAddress**，**城市**， **StateOrProvince**，**国家/地区**，和**PostalCode**。

``` xml
 <ComplexType Name="Address" >
   <Property Type="String" Name="StreetAddress" Nullable="false" />
   <Property Type="String" Name="City" Nullable="false" />
   <Property Type="String" Name="StateOrProvince" Nullable="false" />
   <Property Type="String" Name="Country" Nullable="false" />
   <Property Type="String" Name="PostalCode" Nullable="false" />
 </ComplexType>
```
 

若要将复杂类型定义**地址**（如上所示） 作为实体类型的属性必须声明实体类型定义中的属性类型。 下面的示例演示**地址**实体类型上的复杂类型属性 (**发布服务器**):

``` xml
 <EntityType Name="Publisher">
       <Key>
         <PropertyRef Name="Id" />
       </Key>
       <Property Type="Int32" Name="Id" Nullable="false" />
       <Property Type="String" Name="Name" Nullable="false" />
       <Property Type="BooksModel.Address" Name="Address" Nullable="false" />
       <NavigationProperty Name="Books" Relationship="BooksModel.PublishedBy"
                           FromRole="Publisher" ToRole="Book" />
     </EntityType>
```
 

 

## <a name="definingexpression-element-csdl"></a>DefiningExpression 元素 (CSDL)

**DefiningExpression**概念架构定义语言 (CSDL) 中的元素包含在概念模型中定义了一个函数的 Entity SQL 表达式。  

> [!NOTE]
> 出于验证目的**DefiningExpression**元素可以包含任意内容。 但是，实体框架将引发异常在运行时**DefiningExpression**元素不包含有效的实体 SQL。

 

### <a name="applicable-attributes"></a>适用的特性

可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**DefiningExpression**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例使用**DefiningExpression**元素来定义返回的年数，因为已发表一本书的函数。 内容**DefiningExpression**元素编写实体 SQL 中。

``` xml
 <Function Name="GetYearsInPrint" ReturnType="Edm.Int32" >
       <Parameter Name="book" Type="BooksModel.Book" />
       <DefiningExpression>
         Year(CurrentDateTime()) - Year(cast(book.PublishedDate as DateTime))
       </DefiningExpression>
     </Function>
```
 

 

## <a name="dependent-element-csdl"></a>Dependent 元素 (CSDL)

**依赖**概念架构定义语言 (CSDL) 中的元素是 ReferentialConstraint 元素的子元素，定义引用约束的依赖端。 一个**ReferentialConstraint**元素定义的是关系数据库中的引用完整性约束类似的功能。 与数据库表中的一个（或多个）列可以引用另一个表的主键相同，实体类型的一个（或多个）属性可以引用另一个实体类型的实体键。 引用的实体类型称为*主体端*的约束。 引用主体端的实体类型称为*依赖端*的约束。 **PropertyRef**元素用于指定哪些键引用主体端。

**依赖**元素可以具有下列子元素 （按所列顺序）：

-   PropertyRef （一个或多个元素）
-   批注元素 （零个或多个元素）

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**依赖**元素。

| 特性名 | 是否必需 | “值”                                                                |
|:---------------|:------------|:---------------------------------------------------------------------|
| **角色**       | 是         | 关联的依赖端的实体类型的名称。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**依赖**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示**ReferentialConstraint**元素的定义的一部分用作**PublishedBy**关联。 **PublisherId**的属性**书籍**实体类型构成引用约束的依赖端。

``` xml
 <Association Name="PublishedBy">
   <End Type="BooksModel.Book" Role="Book" Multiplicity="*" >
   </End>
   <End Type="BooksModel.Publisher" Role="Publisher" Multiplicity="1" />
   <ReferentialConstraint>
     <Principal Role="Publisher">
       <PropertyRef Name="Id" />
     </Principal>
     <Dependent Role="Book">
       <PropertyRef Name="PublisherId" />
     </Dependent>
   </ReferentialConstraint>
 </Association>
```
 

 

## <a name="documentation-element-csdl"></a>Documentation 元素 (CSDL)

**文档**概念架构定义语言 (CSDL) 中的元素可以用于提供有关的对象的父元素中定义的信息。 在.edmx 文件中，当**文档**元素是显示为 （如实体、 关联或属性），在 EF 设计器的设计图面上的对象的内容的元素的子级**文档**元素将显示在 Visual Studio**属性**窗口对象。

**文档**元素可以具有下列子元素 （按所列顺序）：

-   **摘要**： 父元素的简要说明。 （零个或一个元素）
-   **LongDescription**： 父元素的详细说明。 （零个或一个元素）
-   批注元素。 （零个或多个元素）

### <a name="applicable-attributes"></a>适用的特性

可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**文档**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示**文档**元素作为 EntityType 元素的子元素。 如果下面的代码段是 CSDL 中内容的.edmx 文件的内容**摘要**并**LongDescription**元素会显示在 Visual Studio**属性**窗口上单击时`Customer`实体类型。

``` xml
 <EntityType Name="Customer">
    <Documentation>
      <Summary>Summary here.</Summary>
      <LongDescription>Long description here.</LongDescription>
    </Documentation>
    <Key>
      <PropertyRef Name="CustomerId" />
    </Key>
    <Property Type="Int32" Name="CustomerId" Nullable="false" />
    <Property Type="String" Name="Name" Nullable="false" />
 </EntityType>
```
 

 

## <a name="end-element-csdl"></a>End 元素 (CSDL)

**最终**概念架构定义语言 (CSDL) 中的元素可以是 Association 元素或 AssociationSet 元素的子级。 每种情况下的角色**最终**元素是不同的适用的特性也不同。

### <a name="end-element-as-a-child-of-the-association-element"></a>End 元素作为 Association 元素的子元素

**最终**元素 (作为子级**关联**元素) 标识关联的一端可以存在的实体类型实例数，关联的另一端的实体类型。 关联端定义为关联的一部分；关联必须正好有两个关联端。 通过导航属性或外键（如果在实体类型上公开了外键）可以访问关联一端的实体类型实例。  

**最终**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   OnDelete （零个或一个元素）
-   批注元素 （零个或多个元素）

#### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**最终**时的子元素**关联**元素。

| 特性名   | 是否必需 | “值”                                                                                                                                                                                                                                                                                                                                                                                                              |
|:-----------------|:------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Type**         | 是         | 关联一端的实体类型的名称。                                                                                                                                                                                                                                                                                                                                                         |
| **角色**         | 否          | 关联端的名称。 如果不提供名称，将使用关联端的实体类型的名称。                                                                                                                                                                                                                                                                                           |
| **重数** | 是         | **1**， **0.1**，或**\*** 具体取决于实体类型实例的关联端处可以存在的数量。 <br/> **1**指示在关联端存在的一个实体类型实例。 <br/> **0.1**表明在关联端存在零个或一个实体类型实例。 <br/> **\*** 指示在关联端存在零行、 一行或多个实体类型实例。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**最终**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

#### <a name="example"></a>示例

下面的示例演示**关联**定义的元素**CustomerOrders**关联。 **多重性**值为每个**最终**关联的指示，许多**订单**可以关联**客户**，但只有一个**客户**可以关联**顺序**。 此外， **OnDelete**元素指示所有**订单**与特定相关的**客户**和已加载到 ObjectContext 将如果已删除**客户**被删除。

``` xml
 <Association Name="CustomerOrders">
   <End Type="ExampleModel.Customer" Role="Customer" Multiplicity="1" />
   <End Type="ExampleModel.Order" Role="Order" Multiplicity="*">
         <OnDelete Action="Cascade" />
   </End>
 </Association>
```
 

### <a name="end-element-as-a-child-of-the-associationset-element"></a>End 元素作为 AssociationSet 元素的子元素

**最终**元素指定关联集的一端。 **AssociationSet**元素必须包含两个**最终**元素。 中包含的信息**最终**元素用于映射到数据源设置的关联。

**最终**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   批注元素 （零个或多个元素）

> [!NOTE]
> Annotation 元素必须出现在所有其他子元素之后。 批注元素只能在 CSDL v2 和更高版本。

 

#### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**最终**时的子元素**AssociationSet**元素。

| 特性名 | 是否必需 | “值”                                                                                                                                                                                                                 |
|:---------------|:------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **实体集**  | 是         | 名称**EntitySet**元素，用于定义父级的一端**AssociationSet**元素。 **EntitySet**必须为父级的同一个实体容器定义元素**AssociationSet**元素。 |
| **角色**       | 否          | 关联集端的名称。 如果**角色**未使用属性、 关联集端的名称将为实体集的名称。                                                                   |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**最终**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

#### <a name="example"></a>示例

下面的示例演示**EntityContainer**元素具有两个**AssociationSet**元素，每个都有两个**最终**元素：

``` xml
 <EntityContainer Name="BooksContainer" >
   <EntitySet Name="Books" EntityType="BooksModel.Book" />
   <EntitySet Name="Publishers" EntityType="BooksModel.Publisher" />
   <EntitySet Name="Authors" EntityType="BooksModel.Author" />
   <AssociationSet Name="PublishedBy" Association="BooksModel.PublishedBy">
     <End Role="Book" EntitySet="Books" />
     <End Role="Publisher" EntitySet="Publishers" />
   </AssociationSet>
   <AssociationSet Name="WrittenBy" Association="BooksModel.WrittenBy">
     <End Role="Book" EntitySet="Books" />
     <End Role="Author" EntitySet="Authors" />
   </AssociationSet>
 </EntityContainer>
```
 

 

## <a name="entitycontainer-element-csdl"></a>EntityContainer 元素 (CSDL)

**EntityContainer**概念架构定义语言 (CSDL) 中的元素是实体集、 关联集和函数导入的逻辑容器。 概念模型实体容器映射到存储模型实体容器通过 EntityContainerMapping 元素。 存储模型实体容器描述数据库的结构：实体集描述表、关联集描述外键约束、函数导入描述数据库中的存储过程。

**EntityContainer**元素可以具有零个或一个文档元素。 如果**文档**元素存在，则它必须位于所有之前**EntitySet**， **AssociationSet**，以及**FunctionImport**元素。

**EntityContainer**元素可以具有零个或多个下列子元素 （按所列顺序）：

-   EntitySet
-   AssociationSet
-   FunctionImport
-   批注元素

您可以扩展**EntityContainer**元素包含的另一个内容**EntityContainer**中相同的命名空间。 包含的另一个内容**EntityContainer**，在引用**EntityContainer**元素中，设置的值**扩展**的名称的属性**EntityContainer**想要包括的元素。 所包含的所有子元素**EntityContainer**元素将被视为引用的子元素**EntityContainer**元素。

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**Using**元素。

| 特性名 | 是否必需 | “值”                                                           |
|:---------------|:------------|:----------------------------------------------------------------|
| **名称**       | 是         | 实体容器的名称。                               |
| **扩展**    | 否          | 同一命名空间中另一实体容器的名称。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**EntityContainer**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示**EntityContainer**定义三个实体集和两个关联集的元素。

``` xml
 <EntityContainer Name="BooksContainer" >
   <EntitySet Name="Books" EntityType="BooksModel.Book" />
   <EntitySet Name="Publishers" EntityType="BooksModel.Publisher" />
   <EntitySet Name="Authors" EntityType="BooksModel.Author" />
   <AssociationSet Name="PublishedBy" Association="BooksModel.PublishedBy">
     <End Role="Book" EntitySet="Books" />
     <End Role="Publisher" EntitySet="Publishers" />
   </AssociationSet>
   <AssociationSet Name="WrittenBy" Association="BooksModel.WrittenBy">
     <End Role="Book" EntitySet="Books" />
     <End Role="Author" EntitySet="Authors" />
   </AssociationSet>
 </EntityContainer>
```
 

 

## <a name="entityset-element-csdl"></a>EntitySet 元素 (CSDL)

**EntitySet**以概念架构定义语言元素是实体类型的实例和派生自该实体类型的任何类型的实例的逻辑容器。 实体类型与实体集之间的关系类似于关系数据库中行与表之间的关系。 实体类型与行类似，它定义一组相关数据；而实体集与表类似，它包含该定义的实例。 实体集为对实体类型实例分组提供了一个构造，以便能够将它们映射到数据源中的相关数据结构。  

可以为特定实体类型定义多个实体集。

> [!NOTE]
> 在 EF 设计器不支持包含每种类型的多个实体集的概念模型。

 

**EntitySet**元素可以具有下列子元素 （按所列顺序）：

-   Documentation 元素 （允许零个或一个元素）
-   批注元素 （允许零个或多个元素）

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**EntitySet**元素。

| 特性名 | 是否必需 | “值”                                                                                    |
|:---------------|:------------|:-----------------------------------------------------------------------------------------|
| **名称**       | 是         | 实体集的名称。                                                              |
| **EntityType** | 是         | 实体集包含其实例的实体类型的完全限定名称。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**EntitySet**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示**EntityContainer**具有三个元素**EntitySet**元素：

``` xml
 <EntityContainer Name="BooksContainer" >
   <EntitySet Name="Books" EntityType="BooksModel.Book" />
   <EntitySet Name="Publishers" EntityType="BooksModel.Publisher" />
   <EntitySet Name="Authors" EntityType="BooksModel.Author" />
   <AssociationSet Name="PublishedBy" Association="BooksModel.PublishedBy">
     <End Role="Book" EntitySet="Books" />
     <End Role="Publisher" EntitySet="Publishers" />
   </AssociationSet>
   <AssociationSet Name="WrittenBy" Association="BooksModel.WrittenBy">
     <End Role="Book" EntitySet="Books" />
     <End Role="Author" EntitySet="Authors" />
   </AssociationSet>
 </EntityContainer>
```
 

每个类型可以定义多个实体集 (MEST)。 下面的示例定义了具有两个实体集的一个实体容器**通讯簿**实体类型：

``` xml
 <EntityContainer Name="BooksContainer" >
   <EntitySet Name="Books" EntityType="BooksModel.Book" />
   <EntitySet Name="FictionBooks" EntityType="BooksModel.Book" />
   <EntitySet Name="Publishers" EntityType="BooksModel.Publisher" />
   <EntitySet Name="Authors" EntityType="BooksModel.Author" />
   <AssociationSet Name="PublishedBy" Association="BooksModel.PublishedBy">
     <End Role="Book" EntitySet="Books" />
     <End Role="Publisher" EntitySet="Publishers" />
   </AssociationSet>
   <AssociationSet Name="BookAuthor" Association="BooksModel.BookAuthor">
     <End Role="Book" EntitySet="Books" />
     <End Role="Author" EntitySet="Authors" />
   </AssociationSet>
 </EntityContainer>
```
 

 

## <a name="entitytype-element-csdl"></a>EntityType 元素 (CSDL)

**EntityType**元素表示顶级概念，如客户或订单，概念模型中的结构。 实体类型是应用程序中实体类型实例的模板。 每个模板都包含以下信息：

-   唯一名称。 （必选。）
-   由一个或多个属性定义的实体键。 （必选。）
-   用于包含数据的属性。 （可选）。
-   导航属性，用于从关联的一端导航至另一端。 （可选）。

在应用程序中，实体类型的实例表示一个特定对象（例如特定客户或订单）。 实体类型的每个实例在实体集中都必须具有唯一的实体键。

只有两个实体类型实例的类型相同且其实体键的值也相同时，才认为它们是相等的。

**EntityType**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   密钥 （零个或一个元素）
-   属性 （零个或多个元素）
-   NavigationProperty （零个或多个元素）
-   批注元素 （零个或多个元素）

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**EntityType**元素。

| 特性名                                                                                                                                  | 是否必需 | “值”                                                                                            |
|:------------------------------------------------------------------------------------------------------------------------------------------------|:------------|:-------------------------------------------------------------------------------------------------|
| **名称**                                                                                                                                        | 是         | 实体类型的名称。                                                                     |
| **BaseType**                                                                                                                                    | 否          | 实体类型的名称，该实体类型是所定义的另一个实体类型的基类型。  |
| **抽象**                                                                                                                                    | 否          | **True**或**False**，取决于实体类型是否为抽象类型。                 |
| **OpenType**                                                                                                                                    | 否          | **True**或**False**具体取决于实体类型是否为开放实体类型。 <br/> [!NOTE] |
| > **OpenType**属性特性仅适用于与 ADO.NET Data Services 一起使用的概念模型中定义的实体类型。 |             |                                                                                                  |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**EntityType**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示**EntityType**具有三个元素**属性**元素和两个**NavigationProperty**元素：

``` xml
 <EntityType Name="Book">
   <Key>
     <PropertyRef Name="ISBN" />
   </Key>
   <Property Type="String" Name="ISBN" Nullable="false" />
   <Property Type="String" Name="Title" Nullable="false" />
   <Property Type="Decimal" Name="Revision" Nullable="false" Precision="29" Scale="29" />
   <NavigationProperty Name="Publisher" Relationship="BooksModel.PublishedBy"
                       FromRole="Book" ToRole="Publisher" />
   <NavigationProperty Name="Authors" Relationship="BooksModel.WrittenBy"
                       FromRole="Book" ToRole="Author" />
 </EntityType>
```
 

 

## <a name="enumtype-element-csdl"></a>EnumType 元素 (CSDL)

**EnumType**元素表示一个枚举的类型。

**EnumType**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   成员 （零个或多个元素）
-   批注元素 （零个或多个元素）

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**EnumType**元素。

| 特性名     | 是否必需 | “值”                                                                                                                                                                                         |
|:-------------------|:------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **名称**           | 是         | 实体类型的名称。                                                                                                                                                                  |
| **IsFlags**        | 否          | **True**或**False**，取决于是否枚举类型可以用作一组标志。 默认值是**False。**。                                                                     |
| **UnderlyingType** | 否          | **Edm.Byte**， **Edm.Int16**， **Edm.Int32**， **Edm.Int64**或**Edm.SByte**定义类型的值的范围。   默认基础枚举元素的类型是**Edm.Int32。**。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**EnumType**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示**EnumType**具有三个元素**成员**元素：

``` xml
 <EnumType Name="Color" IsFlags=”false” UnderlyingTyp=”Edm.Byte”>
   <Member Name="Red" />
   <Member Name="Green" />
   <Member Name="Blue" />
 </EntityType>
```
 

 

## <a name="function-element-csdl"></a>Function 元素 (CSDL)

**函数**概念架构定义语言 (CSDL) 中的元素用于定义或声明函数概念模型中的。 通过使用 DefiningExpression 元素定义一个函数。  

一个**函数**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   参数 （零个或多个元素）
-   DefiningExpression （零个或一个元素）
-   ReturnType （函数） （零个或一个元素）
-   批注元素 （零个或多个元素）

返回类型的函数必须指定与**ReturnType** （函数） 元素或**ReturnType**属性 （见下文），但不可同时使用两者。 可能的返回类型为任何 EdmSimpleType、实体类型、复杂类型、行类型或引用类型（或这些类型之一的集合）。

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**函数**元素。

| 特性名 | 是否必需 | “值”                              |
|:---------------|:------------|:-----------------------------------|
| **名称**       | 是         | 函数名。          |
| **ReturnType** | 否          | 函数返回的类型。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**函数**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例使用**函数**元素来定义已聘用一名讲师返回年数的函数。

``` xml
 <Function Name="YearsSince" ReturnType="Edm.Int32">
   <Parameter Name="date" Type="Edm.DateTime" />
   <DefiningExpression>
     Year(CurrentDateTime()) - Year(date)
   </DefiningExpression>
 </Function>
```
 

 

## <a name="functionimport-element-csdl"></a>FunctionImport 元素 (CSDL)

**FunctionImport**概念架构定义语言 (CSDL) 中的元素表示一个函数，则数据源中定义，但通过概念模型的对象可用的。 例如，在存储模型中的函数元素可以用于表示数据库中的存储的过程。 一个**FunctionImport**概念模型中的元素表示的实体框架应用程序中的相应函数，通过使用 FunctionImportMapping 元素映射到存储模型函数。 在应用程序中调用函数时，会在数据库中执行相应的存储过程。

**FunctionImport**元素可以具有下列子元素 （按所列顺序）：

-   文档 （允许零个或一个元素）
-   参数 （允许零个或多个元素）
-   批注元素 （允许零个或多个元素）
-   ReturnType (FunctionImport) （允许零个或多个元素）

一个**参数**元素应为该函数接受每个参数定义。

返回类型的函数必须指定与**ReturnType** (FunctionImport) 元素或**ReturnType**属性 （见下文），但不可同时使用两者。 返回类型值必须为 EdmSimpleType、 EntityType 或 ComplexType 的集合。

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**FunctionImport**元素。

| 特性名   | 是否必需 | “值”                                                                                                                                                                                                 |
|:-----------------|:------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **名称**         | 是         | 导入的函数的名称。                                                                                                                                                                    |
| **ReturnType**   | 否          | 函数返回的类型。 如果函数不返回值，不要使用此特性。 否则，值必须是 ComplexType、 EntityType 或 EDMSimpleType 的集合。        |
| **实体集**    | 否          | 如果该函数返回的实体集合类型，值**EntitySet**必须将实体设置为该集合所属。 否则为**EntitySet**属性不能使用。 |
| **IsComposable** | 否          | 如果值设置为 true，该函数是可组合 （表值函数） 并可以在 LINQ 查询中。  默认值为 false。                                                           |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**FunctionImport**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示**FunctionImport**元素接受一个参数并返回实体类型的集合：

``` xml
 <FunctionImport Name="GetStudentGrades"
                 EntitySet="StudentGrade"
                 ReturnType="Collection(SchoolModel.StudentGrade)">
        <Parameter Name="StudentID" Mode="In" Type="Int32" />
 </FunctionImport>
```
 

 

## <a name="key-element-csdl"></a>Key 元素 (CSDL)

**键**元素是 EntityType 元素的子元素，并定义*实体键*（属性或实体类型的属性，用于确定标识一组）。 构成实体键的属性是在设计时选择的。 实体键属性的值必须在运行时唯一标识实体集中的实体类型实例。 在选择构成实体键的属性时应确保实例在实体集中的唯一性。 **密钥**元素定义实体键通过引用一个或多个实体类型的属性。

**密钥**元素可以具有以下子元素：

-   PropertyRef （一个或多个元素）
-   批注元素 （零个或多个元素）

### <a name="applicable-attributes"></a>适用的特性

可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**密钥**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例定义名为的实体类型**通讯簿**。 通过引用来定义实体键**ISBN**实体类型的属性。

``` xml
 <EntityType Name="Book">
   <Key>
     <PropertyRef Name="ISBN" />
   </Key>
   <Property Type="String" Name="ISBN" Nullable="false" />
   <Property Type="String" Name="Title" Nullable="false" />
   <Property Type="Decimal" Name="Revision" Nullable="false" Precision="29" Scale="29" />
   <NavigationProperty Name="Publisher" Relationship="BooksModel.PublishedBy"
                       FromRole="Book" ToRole="Publisher" />
   <NavigationProperty Name="Authors" Relationship="BooksModel.WrittenBy"
                       FromRole="Book" ToRole="Author" />
 </EntityType>
```
 

**ISBN**属性是实体键的一个好选择，因为国际标准书号 (ISBN) 唯一标识一本书。

下面的示例演示一个实体类型 (**作者**) 具有两个属性组成的实体键**名称**并**地址**。

``` xml
 <EntityType Name="Author">
   <Key>
     <PropertyRef Name="Name" />
     <PropertyRef Name="Address" />
   </Key>
   <Property Type="String" Name="Name" Nullable="false" />
   <Property Type="String" Name="Address" Nullable="false" />
   <NavigationProperty Name="Books" Relationship="BooksModel.WrittenBy"
                       FromRole="Author" ToRole="Book" />
 </EntityType>
```
 

使用**名称**并**地址**的实体键是合理的选择，，因为两个具有相同名称的作者不太可能 live 在相同的地址。 但是，针对实体键的这种选择并不能绝对确保实体键在实体集中的唯一性。 添加一个属性，如**作者 Id**，可用来唯一地标识将在这种情况下建议的作者。

 

## <a name="member-element-csdl"></a>Member 元素 (CSDL)

**成员**元素是 EnumType 元素的子元素，定义枚举类型的成员。

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**FunctionImport**元素。

| 特性名 | 是否必需 | “值”                                                                                                                                                                                    |
|:---------------|:------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **名称**       | 是         | 成员名。                                                                                                                                                                  |
| **值**      | 否          | 成员的值。 默认情况下，第一个成员具有值 0，并且每个连续的枚举数的值加 1。 可能存在多个具有相同的值成员。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**FunctionImport**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示**EnumType**具有三个元素**成员**元素：

``` xml
 <EnumType Name="Color">
   <Member Name="Red" Value=”1”/>
   <Member Name="Green" Value=”3” />
   <Member Name="Blue" Value=”5”/>
 </EntityType>
```
 

 

## <a name="navigationproperty-element-csdl"></a>NavigationProperty 元素 (CSDL)

一个**NavigationProperty**元素定义导航属性，用于提供对关联另一端的引用。 与属性元素具有定义的属性，不同的形状和特征的数据定义导航属性。 它们提供了一种在两个实体类型之间导航关联的方法。

注意，对于关联两端的两种实体类型，导航属性都是可选的。 如果您对于位于关联一端的实体类型定义一个导航属性，则不需要对于位于关联另一端的实体类型定义导航属性。

导航属性返回的数据类型是由其远程关联端的重数确定的。 例如，假设导航属性， **OrdersNavProp**，位于**客户**实体类型和定位之间的一对多关联**客户**和**顺序**。 因为导航属性的远程关联端的重数很多 (\*)，其数据类型是一个集合 (的**顺序**)。 同样，如果导航属性**CustomerNavProp**，位于**顺序**其数据类型应为实体类型**客户**由于远程端的多重性是一 （1)。

一个**NavigationProperty**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   批注元素 （零个或多个元素）

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**NavigationProperty**元素。

| 特性名   | 是否必需 | “值”                                                                                                                                                                                                                                            |
|:-----------------|:------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **名称**         | 是         | 导航属性的名称。                                                                                                                                                                                                             |
| **关系** | 是         | 处于模型的作用域中的关联的名称。                                                                                                                                                                                |
| **ToRole**       | 是         | 导航在此结束的关联端。 值**ToRole**属性必须为之一的值与相同**角色**上一个 （AssociationEnd 元素中定义） 的关联端定义的属性。       |
| **FromRole**     | 是         | 导航从其开始的关联端。 值**FromRole**属性必须为之一的值与相同**角色**上一个 （AssociationEnd 元素中定义） 的关联端定义的属性。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**NavigationProperty**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例定义一个实体类型 (**书籍**) 具有两个导航属性 (**PublishedBy**并**WrittenBy**):

``` xml
 <EntityType Name="Book">
   <Key>
     <PropertyRef Name="ISBN" />
   </Key>
   <Property Type="String" Name="ISBN" Nullable="false" />
   <Property Type="String" Name="Title" Nullable="false" />
   <Property Type="Decimal" Name="Revision" Nullable="false" Precision="29" Scale="29" />
   <NavigationProperty Name="Publisher" Relationship="BooksModel.PublishedBy"
                       FromRole="Book" ToRole="Publisher" />
   <NavigationProperty Name="Authors" Relationship="BooksModel.WrittenBy"
                       FromRole="Book" ToRole="Author" />
 </EntityType>
```
 

 

## <a name="ondelete-element-csdl"></a>OnDelete 元素 (CSDL)

**OnDelete**概念架构定义语言 (CSDL) 中的元素定义与关联相关的行为。 如果**操作**属性设置为**Cascade** ，另一端的关联相关的删除第一个一端的实体类型时，会删除关联另一端的实体类型。 如果两个实体类型之间的关联是主键的键-主键键关系中，则加载的依赖对象删除而不考虑关联另一端上的主体对象时，删除**OnDelete**规范。  

> [!NOTE]
> **OnDelete**元素仅会影响应用程序的运行时行为; 它不会影响数据源中的行为。 在数据源中定义的行为应与在应用程序中定义的行为相同。

 

**OnDelete**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   批注元素 （零个或多个元素）

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**OnDelete**元素。

| 特性名 | 是否必需 | “值”                                                                                                                                                                                                                         |
|:---------------|:------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **操作**     | 是         | **级联**或**None**。 如果**Cascade**，当删除主体实体类型时，将删除从属实体类型。 如果**None**，当删除主体实体类型时，将不会删除从属实体类型。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**关联**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示**关联**定义的元素**CustomerOrders**关联。 **OnDelete**元素指示所有**订单**与特定相关的**客户**和已加载到 ObjectContext 时，将删除**客户**被删除。

``` xml
 <Association Name="CustomerOrders">
   <End Type="ExampleModel.Customer" Role="Customer" Multiplicity="1">
         <OnDelete Action="Cascade" />
   </End>
   <End Type="ExampleModel.Order" Role="Order" Multiplicity="*" />
 </Association>
```
 

 

## <a name="parameter-element-csdl"></a>Parameter 元素 (CSDL)

**参数**概念架构定义语言 (CSDL) 中的元素可以是 FunctionImport 元素或 Function 元素的子级。

### <a name="functionimport-element-application"></a>FunctionImport 元素应用程序

一个**参数**元素 (作为子级**FunctionImport**元素) 用于在 CSDL 中定义的函数导入声明输入和输出参数。

**参数**元素可以具有下列子元素 （按所列顺序）：

-   文档 （允许零个或一个元素）
-   批注元素 （允许零个或多个元素）

#### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**参数**元素。

| 特性名 | 是否必需 | “值”                                                                                                                                                                                                                           |
|:---------------|:------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **名称**       | 是         | 参数的名称。                                                                                                                                                                                                      |
| **Type**       | 是         | 参数类型。 该值必须是**EDMSimpleType**或模型的作用域内的复杂类型。                                                                                                             |
| **模式**       | 否          | **在中**，**出**，或**InOut**具体取决于参数是输入、 输出或输入/输出参数。                                                                                                                |
| **MaxLength**  | 否          | 参数的最大允许长度。                                                                                                                                                                                    |
| **精度**  | 否          | 参数的精度。                                                                                                                                                                                                 |
| 缩放      | 否          | 参数的小数位数。                                                                                                                                                                                                     |
| **SRID**       | 否          | 系统的空间引用标识符。 仅对空间类型的参数有效。 有关详细信息，请参阅[SRID](http://en.wikipedia.org/wiki/SRID)并[SRID (SQL Server)](https://msdn.microsoft.com/library/bb964707.aspx)。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**参数**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

#### <a name="example"></a>示例

下面的示例演示**FunctionImport**具有一个**参数**子元素。 函数接受一个输入参数并返回实体类型的集合。

``` xml
 <FunctionImport Name="GetStudentGrades"
                 EntitySet="StudentGrade"
                 ReturnType="Collection(SchoolModel.StudentGrade)">
        <Parameter Name="StudentID" Mode="In" Type="Int32" />
 </FunctionImport>
```
 

### <a name="function-element-application"></a>Function 元素应用程序

一个**参数**元素 (作为子级**函数**元素) 定义概念模型中的函数的定义或声明的参数。

**参数**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   CollectionType （零个或一个元素）
-   ReferenceType （零个或一个元素）
-   RowType （零个或一个元素）

> [!NOTE]
> 只有一个**CollectionType**， **ReferenceType**，或**RowType**元素的子元素可以**属性**元素。

 

-   批注元素 （允许零个或多个元素）

> [!NOTE]
> Annotation 元素必须出现在所有其他子元素之后。 批注元素只能在 CSDL v2 和更高版本。

 

#### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**参数**元素。

| 特性名   | 是否必需 | “值”                                                                                                                                                                                                                           |
|:-----------------|:------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **名称**         | 是         | 参数的名称。                                                                                                                                                                                                      |
| **Type**         | 否          | 参数类型。 参数可以是以下任意类型（或这些类型的集合）： <br/> **EdmSimpleType** <br/> Entity Type — 实体类型 <br/> Complex Type — 复杂类型 <br/> 行类型 <br/> Reference Type — 引用类型                             |
| **可以为 null**     | 否          | **True** （默认值） 或**False**具体取决于属性是否可以具有**null**值。                                                                                                                          |
| **DefaultValue** | 否          | 属性的默认值。                                                                                                                                                                                              |
| **MaxLength**    | 否          | 属性值的最大长度。                                                                                                                                                                                       |
| **FixedLength**  | 否          | **True**或**False**具体取决于属性值是否将作为固定的长度字符串存储。                                                                                                                          |
| **精度**    | 否          | 属性值的精度。                                                                                                                                                                                            |
| 缩放        | 否          | 属性值的刻度。                                                                                                                                                                                                |
| **SRID**         | 否          | 系统的空间引用标识符。 仅对空间类型的属性有效。 有关详细信息，请参阅[SRID](http://en.wikipedia.org/wiki/SRID)并[SRID (SQL Server)](https://msdn.microsoft.com/library/bb964707.aspx)。 |
| **Unicode**      | 否          | **True**或**False**具体取决于是否在属性值将存储为 Unicode 字符串。                                                                                                                               |
| **排序规则**    | 否          | 指定要在数据源中使用的排序序列的字符串。                                                                                                                                                   |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**参数**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

#### <a name="example"></a>示例

下面的示例演示**函数**元素使用一个**参数**子元素来定义函数参数。

``` xml
 <Function Name="GetYearsEmployed" ReturnType="Edm.Int32">
 <Parameter Name="Instructor" Type="SchoolModel.Person" />
   <DefiningExpression>
   Year(CurrentDateTime()) - Year(cast(Instructor.HireDate as DateTime))
   </DefiningExpression>
 </Function>
```

 

## <a name="principal-element-csdl"></a>Principal 元素 (CSDL)

**主体**概念架构定义语言 (CSDL) 中的元素是定义引用约束的主体端 ReferentialConstraint 元素的子元素。 一个**ReferentialConstraint**元素定义的是关系数据库中的引用完整性约束类似的功能。 与数据库表中的一个（或多个）列可以引用另一个表的主键相同，实体类型的一个（或多个）属性可以引用另一个实体类型的实体键。 引用的实体类型称为*主体端*的约束。 引用主体端的实体类型称为*依赖端*的约束。 **PropertyRef**元素用来指定哪些键引用的依赖端。

**主体**元素可以具有下列子元素 （按所列顺序）：

-   PropertyRef （一个或多个元素）
-   批注元素 （零个或多个元素）

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**主体**元素。

| 特性名 | 是否必需 | “值”                                                                |
|:---------------|:------------|:---------------------------------------------------------------------|
| **角色**       | 是         | 关联的主体端的实体类型的名称。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**主体**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示**ReferentialConstraint**的定义一部分的元素**PublishedBy**关联。 **Id**的属性**发布服务器**实体类型构成引用约束的主体端。

``` xml
 <Association Name="PublishedBy">
   <End Type="BooksModel.Book" Role="Book" Multiplicity="*" >
   </End>
   <End Type="BooksModel.Publisher" Role="Publisher" Multiplicity="1" />
   <ReferentialConstraint>
     <Principal Role="Publisher">
       <PropertyRef Name="Id" />
     </Principal>
     <Dependent Role="Book">
       <PropertyRef Name="PublisherId" />
     </Dependent>
   </ReferentialConstraint>
 </Association>
```
 

 

## <a name="property-element-csdl"></a>Property 元素 (CSDL)

**属性**概念架构定义语言 (CSDL) 中的元素可以是 EntityType 元素、 ComplexType 元素或 RowType 元素的子级。

### <a name="entitytype-and-complextype-element-applications"></a>EntityType 和 ComplexType 元素应用程序

**属性**元素 (作为子级**EntityType**或**ComplexType**元素) 定义的形状和实体类型实例或复杂类型实例要包含的数据的特征. 概念模型中的属性类似于为类定义的属性。 正如类的属性定义类的形状和携带有关对象的信息一样，概念模型中的属性也定义实体类型的形状和携带有关实体类型实例的信息。

**属性**元素可以具有下列子元素 （按所列顺序）：

-   Documentation 元素 （允许零个或一个元素）
-   批注元素 （允许零个或多个元素）

以下方面可以应用于**属性**元素： **Nullable**， **DefaultValue**， **MaxLength**， **FixedLength**，**精度**，**规模**， **Unicode**，**排序规则**， **ConcurrencyMode**。 方面是一些 XML 特性，它们提供有关如何在数据存储区中存储属性值的信息。

> [!NOTE]
> 方面只能应用于类型的属性**EDMSimpleType**。

 

#### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**属性**元素。

| 特性名                                                         | 是否必需 | “值”                                                                                                                                                                                                                           |
|:-----------------------------------------------------------------------|:------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **名称**                                                               | 是         | 属性的名称。                                                                                                                                                                                                       |
| **Type**                                                               | 是         | 属性值的类型。 属性值类型必须是**EDMSimpleType**或复杂类型 （由完全限定名称指示） 范围内的模型。                                                 |
| **可以为 null**                                                           | 否          | **True** （默认值） 或<strong>False</strong>具体取决于属性是否可以具有 null 值。 <br/> [!NOTE]                                                                                                   |
| > CSDL v1 中的复杂类型属性必须具有`Nullable="False"`。 |             |                                                                                                                                                                                                                                 |
| **DefaultValue**                                                       | 否          | 属性的默认值。                                                                                                                                                                                              |
| **MaxLength**                                                          | 否          | 属性值的最大长度。                                                                                                                                                                                       |
| **FixedLength**                                                        | 否          | **True**或**False**具体取决于属性值是否将作为固定的长度字符串存储。                                                                                                                          |
| **精度**                                                          | 否          | 属性值的精度。                                                                                                                                                                                            |
| 缩放                                                              | 否          | 属性值的刻度。                                                                                                                                                                                                |
| **SRID**                                                               | 否          | 系统的空间引用标识符。 仅对空间类型的属性有效。 有关详细信息，请参阅[SRID](http://en.wikipedia.org/wiki/SRID)并[SRID (SQL Server)](https://msdn.microsoft.com/library/bb964707.aspx)。 |
| **Unicode**                                                            | 否          | **True**或**False**具体取决于是否在属性值将存储为 Unicode 字符串。                                                                                                                               |
| **排序规则**                                                          | 否          | 指定要在数据源中使用的排序序列的字符串。                                                                                                                                                   |
| **ConcurrencyMode**                                                    | 否          | **无**（默认值） 或**Fixed**。 如果值设置为**Fixed**，将开放式并发检查中使用的属性值。                                                                                  |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**属性**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

#### <a name="example"></a>示例

下面的示例演示**EntityType**具有三个元素**属性**元素：

``` xml
 <EntityType Name="Book">
   <Key>
     <PropertyRef Name="ISBN" />
   </Key>
   <Property Type="String" Name="ISBN" Nullable="false" />
   <Property Type="String" Name="Title" Nullable="false" />
   <Property Type="Decimal" Name="Revision" Nullable="false" Precision="29" Scale="29" />
   <NavigationProperty Name="Publisher" Relationship="BooksModel.PublishedBy"
                       FromRole="Book" ToRole="Publisher" />
   <NavigationProperty Name="Authors" Relationship="BooksModel.WrittenBy"
                       FromRole="Book" ToRole="Author" />
 </EntityType>
```
 

下面的示例演示**ComplexType**具有五个元素**属性**元素：

``` xml
 <ComplexType Name="Address" >
   <Property Type="String" Name="StreetAddress" Nullable="false" />
   <Property Type="String" Name="City" Nullable="false" />
   <Property Type="String" Name="StateOrProvince" Nullable="false" />
   <Property Type="String" Name="Country" Nullable="false" />
   <Property Type="String" Name="PostalCode" Nullable="false" />
 </ComplexType>
```
 

### <a name="rowtype-element-application"></a>RowType 元素应用

**属性**元素 (作为的子级**RowType**元素) 定义的形状和特征可以传递到或从模型定义函数返回的数据。  

**属性**元素可以具有下列子元素之一：

-   CollectionType
-   ReferenceType
-   RowType

**属性**元素可以具有任何数字子批注元素的数目。

> [!NOTE]
> 批注元素只能在 CSDL v2 和更高版本。

 

#### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**属性**元素。

| 特性名                                                     | 是否必需 | “值”                                                                                                                                                                                                                           |
|:-------------------------------------------------------------------|:------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **名称**                                                           | 是         | 属性的名称。                                                                                                                                                                                                       |
| **Type**                                                           | 是         | 属性值的类型。                                                                                                                                                                                                 |
| **可以为 null**                                                       | 否          | **True** （默认值） 或**False**具体取决于属性是否可以具有 null 值。 <br/> [!NOTE]                                                                                                                |
| > CSDL v1 中的复杂类型属性必须具有`Nullable="False"`。 |             |                                                                                                                                                                                                                                 |
| **DefaultValue**                                                   | 否          | 属性的默认值。                                                                                                                                                                                              |
| **MaxLength**                                                      | 否          | 属性值的最大长度。                                                                                                                                                                                       |
| **FixedLength**                                                    | 否          | **True**或**False**具体取决于属性值是否将作为固定的长度字符串存储。                                                                                                                          |
| **精度**                                                      | 否          | 属性值的精度。                                                                                                                                                                                            |
| 缩放                                                          | 否          | 属性值的刻度。                                                                                                                                                                                                |
| **SRID**                                                           | 否          | 系统的空间引用标识符。 仅对空间类型的属性有效。 有关详细信息，请参阅[SRID](http://en.wikipedia.org/wiki/SRID)并[SRID (SQL Server)](https://msdn.microsoft.com/library/bb964707.aspx)。 |
| **Unicode**                                                        | 否          | **True**或**False**具体取决于是否在属性值将存储为 Unicode 字符串。                                                                                                                               |
| **排序规则**                                                      | 否          | 指定要在数据源中使用的排序序列的字符串。                                                                                                                                                   |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**属性**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

#### <a name="example"></a>示例

下面的示例演示**属性**使用以定义模型定义函数的返回类型的形状的元素。

``` xml
 <Function Name="LastNamesAfter">
   <Parameter Name="someString" Type="Edm.String" />
   <ReturnType>
    <CollectionType>
      <RowType>
        <Property Name="FirstName" Type="Edm.String" Nullable="false" />
        <Property Name="LastName" Type="Edm.String" Nullable="false" />
      </RowType>
    </CollectionType>
   </ReturnType>
   <DefiningExpression>
             SELECT VALUE ROW(p.FirstName, p.LastName)
             FROM SchoolEntities.People AS p
             WHERE p.LastName &gt;= somestring
   </DefiningExpression>
 </Function>
```
 

 

## <a name="propertyref-element-csdl"></a>PropertyRef 元素 (CSDL)

**PropertyRef**概念架构定义语言 (CSDL) 中的元素引用的实体类型，以指示该属性将承担以下角色之一的属性：

-   实体键的一部分（实体类型的用于确定标识的一个或一组属性）。 一个或多个**PropertyRef**元素可用于定义实体键。
-   引用约束的依赖端或主体端。

**PropertyRef**元素只能具有批注元素 （零个或多个） 作为子元素。

> [!NOTE]
> 批注元素只能在 CSDL v2 和更高版本。

 

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**PropertyRef**元素。

| 特性名 | 是否必需 | “值”                                |
|:---------------|:------------|:-------------------------------------|
| **名称**       | 是         | 所引用属性的名称。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**PropertyRef**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例定义一个实体类型 (**通讯簿**)。 通过引用来定义实体键**ISBN**实体类型的属性。

``` xml
 <EntityType Name="Book">
   <Key>
     <PropertyRef Name="ISBN" />
   </Key>
   <Property Type="String" Name="ISBN" Nullable="false" />
   <Property Type="String" Name="Title" Nullable="false" />
   <Property Type="Decimal" Name="Revision" Nullable="false" Precision="29" Scale="29" />
   <NavigationProperty Name="Publisher" Relationship="BooksModel.PublishedBy"
                       FromRole="Book" ToRole="Publisher" />
   <NavigationProperty Name="Authors" Relationship="BooksModel.WrittenBy"
                       FromRole="Book" ToRole="Author" />
 </EntityType>
```
 

在下一步的示例中，两个**PropertyRef**元素用于指示两个属性 (**Id**并**PublisherId**) 是主体端和依赖端引用约束。

``` xml
 <Association Name="PublishedBy">
   <End Type="BooksModel.Book" Role="Book" Multiplicity="*" >
   </End>
   <End Type="BooksModel.Publisher" Role="Publisher" Multiplicity="1" />
   <ReferentialConstraint>
     <Principal Role="Publisher">
       <PropertyRef Name="Id" />
     </Principal>
     <Dependent Role="Book">
       <PropertyRef Name="PublisherId" />
     </Dependent>
   </ReferentialConstraint>
 </Association>
```
 

 

## <a name="referencetype-element-csdl"></a>ReferenceType 元素 (CSDL)

**ReferenceType**概念架构定义语言 (CSDL) 中的元素指定对实体类型的引用。 **ReferenceType**元素可以是以下元素的子元素：

-   ReturnType （函数）
-   参数
-   CollectionType

**ReferenceType**元素定义的函数的参数或返回类型时使用。

一个**ReferenceType**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   批注元素 （零个或多个元素）

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**ReferenceType**元素。

| 特性名 | 是否必需 | “值”                                         |
|:---------------|:------------|:----------------------------------------------|
| **Type**       | 是         | 所引用的实体类型的名称。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**ReferenceType**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示**ReferenceType**元素的小孩在用**参数**接受对引用的模型定义函数中的元素**人员**实体类型：

``` xml
 <Function Name="GetYearsEmployed" ReturnType="Edm.Int32">
   <Parameter Name="instructor">
     <ReferenceType Type="SchoolModel.Person" />
   </Parameter>
   <DefiningExpression>
   Year(CurrentDateTime()) - Year(cast(instructor.HireDate as DateTime))
   </DefiningExpression>
 </Function>
```
 

下面的示例演示**ReferenceType**元素的小孩在用**ReturnType**中返回的引用的模型定义函数 （函数） 元素**人员**实体类型：

``` xml
 <Function Name="GetPersonReference">
     <Parameter Name="p" Type="SchoolModel.Person" />
     <ReturnType>
         <ReferenceType Type="SchoolModel.Person" />
     </ReturnType>
     <DefiningExpression>
           REF(p)
     </DefiningExpression>
 </Function>
```
 

 

## <a name="referentialconstraint-element-csdl"></a>ReferentialConstraint 元素 (CSDL)

一个**ReferentialConstraint**概念架构定义语言 (CSDL) 中的元素定义的是关系数据库中的引用完整性约束类似的功能。 与数据库表中的一个（或多个）列可以引用另一个表的主键相同，实体类型的一个（或多个）属性可以引用另一个实体类型的实体键。 引用的实体类型称为*主体端*的约束。 引用主体端的实体类型称为*依赖端*的约束。

如果一个实体类型公开的外键引用另一个实体类型上的属性**ReferentialConstraint**元素定义两个实体类型之间的关联。 因为**ReferentialConstraint**元素提供了两个实体类型的信息相关的没有相应的 AssociationSetMapping 元素是有必要在映射规范语言 (MSL)。 没有公开外键的两个实体类型之间的关联必须具有相应**AssociationSetMapping**元素中，以便将关联信息映射到数据源。

如果一个实体类型，不会公开的外键**ReferentialConstraint**元素只能定义实体类型与另一个实体类型之间的主要键到主键键约束。

一个**ReferentialConstraint**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   主体 （恰好一个元素）
-   依赖于 （恰好一个元素）
-   批注元素 （零个或多个元素）

### <a name="applicable-attributes"></a>适用的特性

**ReferentialConstraint**元素可以具有任意数量的批注特性 （自定义 XML 特性）。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示**ReferentialConstraint**元素的定义的一部分用作**PublishedBy**关联。

``` xml
 <Association Name="PublishedBy">
   <End Type="BooksModel.Book" Role="Book" Multiplicity="*" >
   </End>
   <End Type="BooksModel.Publisher" Role="Publisher" Multiplicity="1" />
   <ReferentialConstraint>
     <Principal Role="Publisher">
       <PropertyRef Name="Id" />
     </Principal>
     <Dependent Role="Book">
       <PropertyRef Name="PublisherId" />
     </Dependent>
   </ReferentialConstraint>
 </Association>
```
 

 

## <a name="returntype-function-element-csdl"></a>ReturnType （函数） 元素 (CSDL)

**ReturnType** （函数） 在概念性架构定义语言 (CSDL) 中的元素指定函数元素中定义的函数的返回类型。 此外可以使用指定返回类型的函数**ReturnType**属性。

返回类型可以是任意**EdmSimpleType**，实体类型、 复杂类型、 行类型、 ref 类型或其中一种类型的集合。

可以通过指定函数的返回类型**类型**的属性**ReturnType** （函数） 元素，或使用以下子元素之一：

-   CollectionType
-   ReferenceType
-   RowType

> [!NOTE]
> 如果您指定的函数返回类型同时具有无法验证模型**类型**的属性**ReturnType** （函数） 元素和子元素之一。

 

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**ReturnType** （函数） 元素。

| 特性名 | 是否必需 | “值”                              |
|:---------------|:------------|:-----------------------------------|
| **ReturnType** | 否          | 函数返回的类型。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**ReturnType** （函数） 元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例使用**函数**元素来定义返回某书籍已在打印中的年数的函数。 请注意，通过指定的返回类型**类型**的属性**ReturnType** （函数） 元素。

``` xml
 <Function Name="GetYearsInPrint">
   <ReturnType Type=="Edm.Int32">
   <Parameter Name="book" Type="BooksModel.Book" />
   <DefiningExpression>
    Year(CurrentDateTime()) - Year(cast(book.PublishedDate as DateTime))
   </DefiningExpression>
 </Function>
```
 

 

## <a name="returntype-functionimport-element-csdl"></a>ReturnType (FunctionImport) 元素 (CSDL)

**ReturnType** (FunctionImport) 元素在概念架构定义语言 (CSDL) 中的指定的 FunctionImport 元素中定义的函数的返回类型。 此外可以使用指定返回类型的函数**ReturnType**属性。

返回类型可以是实体类型、 复杂类型的任何集合或**EdmSimpleType**，

使用指定函数的返回类型**类型**的属性**ReturnType** (FunctionImport) 元素。

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**ReturnType** (FunctionImport) 元素。

| 特性名 | 是否必需 | “值”                                                                                                                                                                                                 |
|:---------------|:------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Type**       | 否          | 函数返回的类型。 值必须是 ComplexType、 EntityType 或 EDMSimpleType 的集合。                                                                                      |
| **实体集**  | 否          | 如果该函数返回的实体集合类型，值**EntitySet**必须将实体设置为该集合所属。 否则为**EntitySet**属性不能使用。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**ReturnType** (FunctionImport) 元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例使用**FunctionImport**返回书籍和发布服务器。 请注意，该函数返回两个结果集，因此两个**ReturnType**指定了 (FunctionImport) 元素。

``` xml
 <FunctionImport Name="GetBooksAndPublishers">
   <ReturnType Type=="Collection(BooksModel.Book )" EntitySet=”Books”>
   <ReturnType Type=="Collection(BooksModel.Publisher)" EntitySet=”Publishers”>
 </FunctionImport>
```
 

 

## <a name="rowtype-element-csdl"></a>RowType 元素 (CSDL)

一个**RowType**概念架构定义语言 (CSDL) 中的元素定义未命名的结构作为参数或返回类型的概念模型中定义的函数。

一个**RowType**元素可以是以下元素的子级：

-   CollectionType
-   参数
-   ReturnType （函数）

一个**RowType**元素可以具有下列子元素 （按所列顺序）：

-   属性 （一个或多个）
-   批注元素 （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**RowType**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示使用的模型定义函数**CollectionType**元素来指定该函数返回的行集合 (中指定的那样**RowType**元素)。

``` xml
 <Function Name="LastNamesAfter">
   <Parameter Name="someString" Type="Edm.String" />
   <ReturnType>
    <CollectionType>
      <RowType>
        <Property Name="FirstName" Type="Edm.String" Nullable="false" />
        <Property Name="LastName" Type="Edm.String" Nullable="false" />
      </RowType>
    </CollectionType>
   </ReturnType>
   <DefiningExpression>
             SELECT VALUE ROW(p.FirstName, p.LastName)
             FROM SchoolEntities.People AS p
             WHERE p.LastName &gt;= somestring
   </DefiningExpression>
 </Function>
```

## <a name="schema-element-csdl"></a>Schema 元素 (CSDL)

**架构**元素是概念模型定义的根元素。 它包括构成概念模型的对象、函数和容器的定义。

**架构**元素可能包含零个或多个下列子元素：

-   Using
-   EntityContainer
-   EntityType
-   EnumType
-   关联
-   ComplexType
-   函数

一个**架构**元素可能包含零个或一个批注元素。

> [!NOTE]
> **函数**元素和批注元素只能在 CSDL v2 和更高版本。

 

**架构**元素使用**Namespace**属性来定义概念模型中的实体类型、 复杂类型和关联对象的命名空间。 在命名空间内，任何两个对象都不能同名。 命名空间可以跨多个**架构**元素和多个.csdl 文件。

概念模型命名空间是不同的 XML 命名空间**架构**元素。 概念模型命名空间 (由定义**Namespace**属性) 是实体类型、 复杂类型和关联类型的逻辑容器。 XML 命名空间 (由**xmlns**属性) 的**架构**元素是子元素和属性的默认命名空间**架构**元素。 窗体的 XML 命名空间 http://schemas.microsoft.com/ado/YYYY/MM/edm （其中 YYYY 和 MM 表示年和月分别） 是为 CSDL 保留。 自定义元素和特性不能位于具有此格式的命名空间中。

### <a name="applicable-attributes"></a>适用的特性

下表描述了特性可应用于**架构**元素。

| 特性名 | 是否必需 | “值”                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|:---------------|:------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **命名空间**  | 是         | 概念模型的命名空间。 值**Namespace**特性用于形成一种类型的完全限定的名称。 例如，如果**EntityType**名为*客户*位于 Simple.Example.Model 命名空间，则完全限定的名称**EntityType**是SimpleExampleModel.Customer。 <br/> 以下字符串不能使用的值作为**Namespace**属性：**系统**，**暂时性**，或者**Edm**。 值**Namespace**属性不能与的值相同**Namespace** SSDL 架构元素中的属性。 |
| **Alias**      | 否          | 用于取代命名空间名称的标识符。 例如，如果**EntityType**名为*客户*位于 Simple.Example.Model 命名空间和值**别名**特性是*模型*，则您可以将 Model.Customer 用作完全限定的名称**EntityType。**                                                                                                                                                                                                                                                                                                     |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**架构**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示**架构**元素，其中包含**EntityContainer**元素、 两个**EntityType**元素，其中一个**关联**元素。

``` xml
 <Schema xmlns="http://schemas.microsoft.com/ado/2009/11/edm"
      xmlns:cg="http://schemas.microsoft.com/ado/2009/11/codegeneration"
      xmlns:store="http://schemas.microsoft.com/ado/2009/11/edm/EntityStoreSchemaGenerator"
       Namespace="ExampleModel" Alias="Self">
         <EntityContainer Name="ExampleModelContainer">
           <EntitySet Name="Customers"
                      EntityType="ExampleModel.Customer" />
           <EntitySet Name="Orders" EntityType="ExampleModel.Order" />
           <AssociationSet
                       Name="CustomerOrder"
                       Association="ExampleModel.CustomerOrders">
             <End Role="Customer" EntitySet="Customers" />
             <End Role="Order" EntitySet="Orders" />
           </AssociationSet>
         </EntityContainer>
         <EntityType Name="Customer">
           <Key>
             <PropertyRef Name="CustomerId" />
           </Key>
           <Property Type="Int32" Name="CustomerId" Nullable="false" />
           <Property Type="String" Name="Name" Nullable="false" />
           <NavigationProperty
                    Name="Orders"
                    Relationship="ExampleModel.CustomerOrders"
                    FromRole="Customer" ToRole="Order" />
         </EntityType>
         <EntityType Name="Order">
           <Key>
             <PropertyRef Name="OrderId" />
           </Key>
           <Property Type="Int32" Name="OrderId" Nullable="false" />
           <Property Type="Int32" Name="ProductId" Nullable="false" />
           <Property Type="Int32" Name="Quantity" Nullable="false" />
           <NavigationProperty
                    Name="Customer"
                    Relationship="ExampleModel.CustomerOrders"
                    FromRole="Order" ToRole="Customer" />
           <Property Type="Int32" Name="CustomerId" Nullable="false" />
         </EntityType>
         <Association Name="CustomerOrders">
           <End Type="ExampleModel.Customer"
                Role="Customer" Multiplicity="1" />
           <End Type="ExampleModel.Order"
                Role="Order" Multiplicity="*" />
           <ReferentialConstraint>
             <Principal Role="Customer">
               <PropertyRef Name="CustomerId" />
             </Principal>
             <Dependent Role="Order">
               <PropertyRef Name="CustomerId" />
             </Dependent>
           </ReferentialConstraint>
         </Association>
       </Schema>
```
 

 

## <a name="typeref-element-csdl"></a>TypeRef 元素 (CSDL)

**TypeRef**概念架构定义语言 (CSDL) 中的元素提供对现有的已命名类型的引用。 **TypeRef**元素可以是 CollectionType 元素用于指定函数具有集合作为参数或返回类型的子元素。

一个**TypeRef**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   批注元素 （零个或多个元素）

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**TypeRef**元素。 请注意， **DefaultValue**， **MaxLength**， **FixedLength**，**精度**，**规模**， **Unicode**，并**排序规则**属性都只适用于**Edmsimpletype**。

| 特性名                                                     | 是否必需 | “值”                                                                                                                                                                                                                           |
|:-------------------------------------------------------------------|:------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Type**                                                           | 否          | 所引用的类型的名称。                                                                                                                                                                                          |
| **可以为 null**                                                       | 否          | **True** （默认值） 或**False**具体取决于属性是否可以具有 null 值。 <br/> [!NOTE]                                                                                                                |
| > CSDL v1 中的复杂类型属性必须具有`Nullable="False"`。 |             |                                                                                                                                                                                                                                 |
| **DefaultValue**                                                   | 否          | 属性的默认值。                                                                                                                                                                                              |
| **MaxLength**                                                      | 否          | 属性值的最大长度。                                                                                                                                                                                       |
| **FixedLength**                                                    | 否          | **True**或**False**具体取决于属性值是否将作为固定的长度字符串存储。                                                                                                                          |
| **精度**                                                      | 否          | 属性值的精度。                                                                                                                                                                                            |
| 缩放                                                          | 否          | 属性值的刻度。                                                                                                                                                                                                |
| **SRID**                                                           | 否          | 系统的空间引用标识符。 仅对空间类型的属性有效。 有关详细信息，请参阅[SRID](http://en.wikipedia.org/wiki/SRID)并[SRID (SQL Server)](https://msdn.microsoft.com/library/bb964707.aspx)。 |
| **Unicode**                                                        | 否          | **True**或**False**具体取决于是否在属性值将存储为 Unicode 字符串。                                                                                                                               |
| **排序规则**                                                      | 否          | 指定要在数据源中使用的排序序列的字符串。                                                                                                                                                   |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**CollectionType**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示使用的模型定义函数**TypeRef**元素 (作为子级**CollectionType**元素) 来指定该函数接受一系列**部门**实体类型。

``` xml
 <Function Name="GetAvgBudget">
      <Parameter Name="Departments">
          <CollectionType>
             <TypeRef Type="SchoolModel.Department"/>
          </CollectionType>
           </Parameter>
       <ReturnType Type="Collection(Edm.Decimal)"/>
       <DefiningExpression>
             SELECT VALUE AVG(d.Budget) FROM Departments AS d
       </DefiningExpression>
 </Function>
```
 

 

## <a name="using-element-csdl"></a>Using 元素 (CSDL)

**Using**元素在概念架构定义语言 (CSDL) 中的导入不同的命名空间中存在的概念模型的内容。 通过设置的值**Namespace**属性，您可以引用实体类型、 复杂类型和另一个概念模型中定义的关联类型。 多个**Using**元素可以是子元素的**架构**元素。

> [!NOTE]
> **Using** CSDL 中的元素不能完全相同**使用**编程语言中的语句。 通过导入的命名空间**使用**语句在编程语言中，您不会影响原始命名空间中的对象。 在 CSDL 中，导入的命名空间可以包含从原始命名空间中的实体类型导出的实体类型。 这可能影响在原始命名空间中声明的实体集。

 

**Using**元素可以具有以下子元素：

-   文档 （允许零个或一个元素）
-   批注元素 （允许零个或多个元素）

### <a name="applicable-attributes"></a>适用的特性

下表描述了特性可应用于**Using**元素。

| 特性名 | 是否必需 | “值”                                                                                                                                                                              |
|:---------------|:------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **命名空间**  | 是         | 导入的命名空间的名称。                                                                                                                                                |
| **Alias**      | 是         | 用于取代命名空间名称的标识符。 尽管此特性是必需的，但并不要求使用它来取代命名空间以限定对象名称。 |

 

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**Using**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

 

### <a name="example"></a>示例

下面的示例演示**Using**用于导入的命名空间的元素定义的其他位置。 请注意，为命名空间**架构**所示的元素是`BooksModel`。 `Address`上的属性`Publisher` **EntityType**是中定义的复杂类型`ExtendedBooksModel`命名空间 (使用导入**Using**元素)。

``` xml
 <Schema xmlns="http://schemas.microsoft.com/ado/2009/11/edm"
           xmlns:cg="http://schemas.microsoft.com/ado/2009/11/codegeneration"
           xmlns:store="http://schemas.microsoft.com/ado/2009/11/edm/EntityStoreSchemaGenerator"
           Namespace="BooksModel" Alias="Self">

     <Using Namespace="BooksModel.Extended" Alias="BMExt" />

 <EntityContainer Name="BooksContainer" >
       <EntitySet Name="Publishers" EntityType="BooksModel.Publisher" />
     </EntityContainer>

 <EntityType Name="Publisher">
       <Key>
         <PropertyRef Name="Id" />
       </Key>
       <Property Type="Int32" Name="Id" Nullable="false" />
       <Property Type="String" Name="Name" Nullable="false" />
       <Property Type="BMExt.Address" Name="Address" Nullable="false" />
     </EntityType>

 </Schema>
```
 

 

## <a name="annotation-attributes-csdl"></a>Annotation 特性 (CSDL)

以概念架构定义语言 (CSDL) 表示的批注特性是概念模型中的自定义 XML 特性。 除了具有有效的 XML 结构之外，还必须满足批注特性的以下各项条件：

-   批注特性不能位于为 CSDL 保留的任何 XML 命名空间中。
-   可以将多个批注特性应用于给定的 CSDL 元素。
-   任何两个批注特性的完全限定名称都不能相同。

可以使用批注特性提供有关概念模型中元素的额外元数据。 可以使用 System.Data.Metadata.Edm 命名空间中的类，在运行时访问批注元素中包含的元数据。

### <a name="example"></a>示例

下面的示例演示**EntityType**元素具有一个批注特性 (**CustomAttribute**)。 本示例还演示了一个应用于实体类型元素的批注元素。

``` xml
 <Schema Namespace="SchoolModel" Alias="Self"
         xmlns:annotation="http://schemas.microsoft.com/ado/2009/02/edm/annotation"
         xmlns="http://schemas.microsoft.com/ado/2009/11/edm">
   <EntityContainer Name="SchoolEntities" annotation:LazyLoadingEnabled="true">
     <EntitySet Name="People" EntityType="SchoolModel.Person" />
   </EntityContainer>
   <EntityType Name="Person" xmlns:p="http://CustomNamespace.com"
               p:CustomAttribute="Data here.">
     <Key>
       <PropertyRef Name="PersonID" />
     </Key>
     <Property Name="PersonID" Type="Int32" Nullable="false"
               annotation:StoreGeneratedPattern="Identity" />
     <Property Name="LastName" Type="String" Nullable="false"
               MaxLength="50" Unicode="true" FixedLength="false" />
     <Property Name="FirstName" Type="String" Nullable="false"
               MaxLength="50" Unicode="true" FixedLength="false" />
     <Property Name="HireDate" Type="DateTime" />
     <Property Name="EnrollmentDate" Type="DateTime" />
     <p:CustomElement>
       Custom metadata.
     </p:CustomElement>
   </EntityType>
 </Schema>
```
 

下面的代码检索批注特性中的元数据并将其写入控制台：

``` xml
 EdmItemCollection collection = new EdmItemCollection("School.csdl");
 MetadataWorkspace workspace = new MetadataWorkspace();
 workspace.RegisterItemCollection(collection);
 EdmType contentType;
 workspace.TryGetType("Person", "SchoolModel", DataSpace.CSpace, out contentType);
 if (contentType.MetadataProperties.Contains("http://CustomNamespace.com:CustomAttribute"))
 {
     MetadataProperty annotationProperty =
         contentType.MetadataProperties["http://CustomNamespace.com:CustomAttribute"];
     object annotationValue = annotationProperty.Value;
     Console.WriteLine(annotationValue.ToString());
 }
```
 

上面的代码假定 `School.csdl` 文件位于项目的输出目录中，且您已将下面的 `Imports` 和 `Using` 语句添加到项目中：

``` csharp
 using System.Data.Metadata.Edm;
```
 

 

## <a name="annotation-elements-csdl"></a>Annotation 元素 (CSDL)

以概念架构定义语言 (CSDL) 表示的批注元素是概念模型中的自定义 XML 元素。 除了具有有效的 XML 结构之外，还必须满足批注元素的以下各项条件：

-   批注元素不能位于为 CSDL 保留的任何 XML 命名空间中。
-   多个批注元素可能是某个给定 CSDL 元素的子元素。
-   任何两个批注元素的完全限定名称都不能相同。
-   批注元素必须出现在给定 CSDL 元素的所有其他子元素之后。

可以使用批注元素提供有关概念模型中元素的额外元数据。 从.NET Framework 版本 4 开始，访问元数据批注元素中包含可以在运行时使用 System.Data.Metadata.Edm 命名空间中的类。

### <a name="example"></a>示例

下面的示例演示**EntityType**具有一个批注元素的元素 (**CustomElement**)。 本示例还演示了一个应用于实体类型元素的批注特性。

``` xml
 <Schema Namespace="SchoolModel" Alias="Self"
         xmlns:annotation="http://schemas.microsoft.com/ado/2009/02/edm/annotation"
         xmlns="http://schemas.microsoft.com/ado/2009/11/edm">
   <EntityContainer Name="SchoolEntities" annotation:LazyLoadingEnabled="true">
     <EntitySet Name="People" EntityType="SchoolModel.Person" />
   </EntityContainer>
   <EntityType Name="Person" xmlns:p="http://CustomNamespace.com"
               p:CustomAttribute="Data here.">
     <Key>
       <PropertyRef Name="PersonID" />
     </Key>
     <Property Name="PersonID" Type="Int32" Nullable="false"
               annotation:StoreGeneratedPattern="Identity" />
     <Property Name="LastName" Type="String" Nullable="false"
               MaxLength="50" Unicode="true" FixedLength="false" />
     <Property Name="FirstName" Type="String" Nullable="false"
               MaxLength="50" Unicode="true" FixedLength="false" />
     <Property Name="HireDate" Type="DateTime" />
     <Property Name="EnrollmentDate" Type="DateTime" />
     <p:CustomElement>
       Custom metadata.
     </p:CustomElement>
   </EntityType>
 </Schema>
```
 

下面的代码检索批注元素中的元数据并将其写入控制台：

``` csharp
 EdmItemCollection collection = new EdmItemCollection("School.csdl");
 MetadataWorkspace workspace = new MetadataWorkspace();
 workspace.RegisterItemCollection(collection);
 EdmType contentType;
 workspace.TryGetType("Person", "SchoolModel", DataSpace.CSpace, out contentType);
 if (contentType.MetadataProperties.Contains("http://CustomNamespace.com:CustomElement"))
 {
     MetadataProperty annotationProperty =
         contentType.MetadataProperties["http://CustomNamespace.com:CustomElement"];
     object annotationValue = annotationProperty.Value;
     Console.WriteLine(annotationValue.ToString());
 }
```
 

上面的代码假定 School.csdl 文件位于项目的输出目录中并且您已将下面的 `Imports` 和 `Using` 语句添加到项目中：

``` csharp
 using System.Data.Metadata.Edm;
```
 

 

## <a name="conceptual-model-types-csdl"></a>概念模型类型 (CSDL)

概念架构定义语言 (CSDL) 支持一种抽象的基元数据类型，称为**Edmsimpletype**，用于定义概念模型中的属性。 **Edmsimpletype**是存储或承载环境中支持的基元数据类型的代理。

下表列出了 CSDL 支持的基元数据类型。 该表还列出了可以应用于每个方面**EDMSimpleType**。

| EDMSimpleType                    | 描述                                                | 适用的方面                                                        |
|:---------------------------------|:-----------------------------------------------------------|:-------------------------------------------------------------------------|
| **Edm.Binary**                   | 包含二进制数据。                                      | MaxLength、FixedLength、Nullable、Default                                |
| **Edm.Boolean**                  | 包含值 **，则返回 true**或**false**。                  | Nullable、Default                                                        |
| **Edm.Byte**                     | 包含一个无符号的 8 位整数值。                  | Precision、Nullable、Default                                             |
| **Edm.DateTime**                 | 表示日期和时间。                                | Precision、Nullable、Default                                             |
| **Edm.DateTimeOffset**           | 包含以相对于 GMT 的偏移量（以分钟为单位）表示的日期和时间。 | Precision、Nullable、Default                                             |
| **Edm.Decimal**                  | 包含一个具有固定精度和小数位数的数值。   | Precision、Nullable、Default                                             |
| **Edm.Double**                   | 包含具有 15 位精度浮点数   | Precision、Nullable、Default                                             |
| **Edm.Float**                    | 包含一个具有 7 位精度的浮点数。   | Precision、Nullable、Default                                             |
| **Edm.Guid**                     | 包含一个 16 字节的唯一标识符。                      | Precision、Nullable、Default                                             |
| **Edm.Int16**                    | 包含一个带符号的 16 位整数值。                    | Precision、Nullable、Default                                             |
| **Edm.Int32**                    | 包含一个带符号的 32 位整数值。                    | Precision、Nullable、Default                                             |
| **Edm.Int64**                    | 包含一个带符号的 64 位整数值。                    | Precision、Nullable、Default                                             |
| **Edm.SByte**                    | 包含一个带符号的 8 位整数值。                     | Precision、Nullable、Default                                             |
| **Edm.String**                   | 包含字符数据。                                   | Unicode、FixedLength、MaxLength、Collation、Precision、Nullable、Default |
| **Edm.Time**                     | 包含当天的时间。                                    | Precision、Nullable、Default                                             |
| **Edm.Geography**                |                                                            | 可以为 null，默认的 SRID                                                  |
| **Edm.GeographyPoint**           |                                                            | 可以为 null，默认的 SRID                                                  |
| **Edm.GeographyLineString**      |                                                            | 可以为 null，默认的 SRID                                                  |
| **Edm.GeographyPolygon**         |                                                            | 可以为 null，默认的 SRID                                                  |
| **Edm.GeographyMultiPoint**      |                                                            | 可以为 null，默认的 SRID                                                  |
| **Edm.GeographyMultiLineString** |                                                            | 可以为 null，默认的 SRID                                                  |
| **Edm.GeographyMultiPolygon**    |                                                            | 可以为 null，默认的 SRID                                                  |
| **Edm.GeographyCollection**      |                                                            | 可以为 null，默认的 SRID                                                  |
| **Edm.Geometry**                 |                                                            | 可以为 null，默认的 SRID                                                  |
| **Edm.GeometryPoint**            |                                                            | 可以为 null，默认的 SRID                                                  |
| **Edm.GeometryLineString**       |                                                            | 可以为 null，默认的 SRID                                                  |
| **Edm.GeometryPolygon**          |                                                            | 可以为 null，默认的 SRID                                                  |
| **Edm.GeometryMultiPoint**       |                                                            | 可以为 null，默认的 SRID                                                  |
| **Edm.GeometryMultiLineString**  |                                                            | 可以为 null，默认的 SRID                                                  |
| **Edm.GeometryMultiPolygon**     |                                                            | 可以为 null，默认的 SRID                                                  |
| **Edm.GeometryCollection**       |                                                            | 可以为 null，默认的 SRID                                                  |

## <a name="facets-csdl"></a>方面 (CSDL)

以概念架构定义语言 (CSDL) 表示的方面表示对于实体类型和复杂类型的属性的约束。 方面作为 XML 特性出现在以下 CSDL 元素上：

-   属性
-   TypeRef
-   参数

下表描述了 CSDL 中支持的方面。 所有方面都是可选的。 从概念模型生成数据库时，将使用实体框架的下面列出一些方面。

> [!NOTE]
> 有关概念模型中的数据类型的信息，请参阅概念模型类型 (CSDL)。

| 方面               | 描述                                                                                                                                                                                                                                                   | 适用对象                                                                                                                                                                                                                                                                                                                                                                           | 用于生成数据库 | 由运行时使用 |
|:--------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------|:--------------------|
| **排序规则**       | 指定在对属性值执行比较和排序操作时要使用的排序序列。                                                                                                               | **Edm.String**                                                                                                                                                                                                                                                                                                                                                                       | 是                              | 否                  |
| **ConcurrencyMode** | 表示应使用属性的值来进行开放式并发检查。                                                                                                                                                                    | 所有**EDMSimpleType**属性                                                                                                                                                                                                                                                                                                                                                     | 否                               | 是                 |
| **默认**         | 如果在安装时未提供值，则指定属性的默认值。                                                                                                                                                                       | 所有**EDMSimpleType**属性                                                                                                                                                                                                                                                                                                                                                     | 是                              | 是                 |
| **FixedLength**     | 指定属性值的长度是否可变。                                                                                                                                                                                                  | **Edm.Binary**， **Edm.String**                                                                                                                                                                                                                                                                                                                                                       | 是                              | 否                  |
| **MaxLength**       | 指定属性值的最大长度。                                                                                                                                                                                                           | **Edm.Binary**， **Edm.String**                                                                                                                                                                                                                                                                                                                                                       | 是                              | 否                  |
| **可以为 null**        | 指定属性是否可以具有**null**值。                                                                                                                                                                                                     | 所有**EDMSimpleType**属性                                                                                                                                                                                                                                                                                                                                                     | 是                              | 是                 |
| **精度**       | 类型的属性**十进制**，指定属性值可以具有的位数。 类型的属性**时间**， **DateTime**，并**DateTimeOffset**，指定属性值的秒小数部分的位数。 | **Edm.DateTime**， **Edm.DateTimeOffset**， **Edm.Decimal**， **Edm.Time**                                                                                                                                                                                                                                                                                                              | 是                              | 否                  |
| 缩放           | 指定属性值小数点右侧的位数。                                                                                                                                                                      | **Edm.Decimal**                                                                                                                                                                                                                                                                                                                                                                      | 是                              | 否                  |
| **SRID**            | 指定空间系统引用系统 id。 有关详细信息，请参阅[SRID](http://en.wikipedia.org/wiki/SRID)并[SRID (SQL Server)](https://msdn.microsoft.com/library/bb964707.aspx)。                                                              | **Edm.Geography、 Edm.GeographyPoint、 Edm.GeographyLineString、 Edm.GeographyPolygon、 Edm.GeographyMultiPoint、 Edm.GeographyMultiLineString、 Edm.GeographyMultiPolygon、 Edm.GeographyCollection、 Edm.Geometry、 Edm.GeometryPoint，Edm.GeometryLineString、 Edm.GeometryPolygon、 Edm.GeometryMultiPoint、 Edm.GeometryMultiLineString、 Edm.GeometryMultiPolygon、 Edm.GeometryCollection** | 否                               | 是                 |
| **Unicode**         | 指示是否将属性值存储为 Unicode。                                                                                                                                                                                                    | **Edm.String**                                                                                                                                                                                                                                                                                                                                                                       | 是                              | 是                 |

>[!NOTE]
> 当从概念模型生成数据库，生成数据库向导将识别的值**StoreGeneratedPattern**特性，可以在**属性**元素是否在下面的示例命名空间： http://schemas.microsoft.com/ado/2009/02/edm/annotation 。 支持的属性值为**标识**并**计算**。 值为**标识**将生成具有在数据库中生成的标识值的数据库列。 值为**计算**将生成具有在数据库中计算的值的列。

### <a name="example"></a>示例

下面的示例演示了应用于实体类型的属性的方面：

``` xml
 <EntityType Name="Product">
   <Key>
     <PropertyRef Name="ProductId" />
   </Key>
   <Property Type="Int32"
             Name="ProductId" Nullable="false"
             a:StoreGeneratedPattern="Identity"
    xmlns:a="http://schemas.microsoft.com/ado/2009/02/edm/annotation" />
   <Property Type="String"
             Name="ProductName"
             Nullable="false"
             MaxLength="50" />
   <Property Type="String"
             Name="Location"
             Nullable="true"
             MaxLength="25" />
 </EntityType>
```
