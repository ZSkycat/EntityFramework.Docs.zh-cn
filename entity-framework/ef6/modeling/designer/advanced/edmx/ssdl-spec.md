---
title: SSDL 规范的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: a4af4b1a-40f4-48cc-b2e0-fa8f5d9d5419
ms.openlocfilehash: a8b1f844a34c44d283982a52cef3bf80afd7e679
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490345"
---
# <a name="ssdl-specification"></a>SSDL 规范
存储架构定义语言 (SSDL) 是一种基于 XML 的语言，用于描述实体框架应用程序的存储模型。

在实体框架应用程序，存储模型元数据是加载从.ssdl 文件 （用 SSDL 编写） 到 System.Data.Metadata.Edm.StoreItemCollection 的实例，是可访问通过使用中的方法System.Data.Metadata.Edm.MetadataWorkspace 类。 实体框架使用存储模型元数据将针对特定于应用商店的命令对概念模型的查询。

实体框架设计器 （EF 设计器） 在.edmx 文件中存储模型信息存储在设计时。 在生成时实体设计器使用.edmx 文件中的信息创建在运行时所需的实体框架的.ssdl 文件。

SSDL 的版本按 XML 命名空间进行区分。

| SSDL 版本 | XML Namespace                                     |
|:-------------|:--------------------------------------------------|
| SSDL v1      | http://schemas.microsoft.com/ado/2006/04/edm/ssdl |
| SSDL v2      | http://schemas.microsoft.com/ado/2009/02/edm/ssdl |
| SSDL v3      | http://schemas.microsoft.com/ado/2009/11/edm/ssdl |

## <a name="association-element-ssdl"></a>Association 元素 (SSDL)

**关联**存储架构定义语言 (SSDL) 中的元素指定参与的表列中基础数据库中的外键约束。 两个必需的子 End 元素指定位于关联各端的表和每一端的重数。 可选 ReferentialConstraint 元素指定的主体端和依赖端的关联，以及参与列。 如果没有**ReferentialConstraint**元素存在，必须使用 AssociationSetMapping 元素来指定列映射的关联。

**关联**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个）
-   结束 （正好两个）
-   ReferentialConstraint （零个或一个）
-   批注元素 （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**关联**元素。

| 特性名 | 是否必需 | “值”                                                                            |
|:---------------|:------------|:---------------------------------------------------------------------------------|
| **名称**       | 是         | 基础数据库中相应外键约束的名称。 |

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**关联**元素。 然而，自定义特性可能不属于为 SSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示**关联**使用的元素**ReferentialConstraint**元素指定参与的列**FK\_CustomerOrders**外键约束：

``` xml
 <Association Name="FK_CustomerOrders">
   <End Role="Customers"
        Type="ExampleModel.Store.Customers" Multiplicity="1">
     <OnDelete Action="Cascade" />
   </End>
   <End Role="Orders"
        Type="ExampleModel.Store.Orders" Multiplicity="*" />
   <ReferentialConstraint>
     <Principal Role="Customers">
       <PropertyRef Name="CustomerId" />
     </Principal>
     <Dependent Role="Orders">
       <PropertyRef Name="CustomerId" />
     </Dependent>
   </ReferentialConstraint>
 </Association>
```

## <a name="associationset-element-ssdl"></a>AssociationSet 元素 (SSDL)

**AssociationSet**存储架构定义语言 (SSDL) 中的元素表示基础数据库中的两个表之间的外键约束。 Association 元素中指定参与外键约束的表格列。 **关联**元素对应于给定**AssociationSet**元素中指定**关联**属性的**AssociationSet**元素。

SSDL 关联集的 AssociationSetMapping 元素映射到 CSDL 关联集。 但是，如果给定的 CSDL 关联 CSDL 关联集定义使用 ReferentialConstraint 元素，没有相应**AssociationSetMapping**元素是必要的。 在这种情况下，如果**AssociationSetMapping**元素存在，它定义的映射将被重写由**ReferentialConstraint**元素。

**AssociationSet**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个）
-   结束 （零个或两个）
-   批注元素 （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**AssociationSet**元素。

| 特性名  | 是否必需 | “值”                                                                                                |
|:----------------|:------------|:-----------------------------------------------------------------------------------------------------|
| **名称**        | 是         | 关联集表示的外键约束的名称。                          |
| **关联** | 是         | 定义参与外键约束的列的关联的名称。 |

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**AssociationSet**元素。 然而，自定义特性可能不属于为 SSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示**AssociationSet**元素，它表示`FK_CustomerOrders`基础数据库中的外键约束：

``` xml
 <AssociationSet Name="FK_CustomerOrders"
                 Association="ExampleModel.Store.FK_CustomerOrders">
   <End Role="Customers" EntitySet="Customers" />
   <End Role="Orders" EntitySet="Orders" />
 </AssociationSet>
```

## <a name="collectiontype-element-ssdl"></a>CollectionType 元素 (SSDL)

**CollectionType**存储架构定义语言 (SSDL) 中的元素指定函数的返回类型是集合。 **CollectionType**元素是 ReturnType 元素的子元素。 使用 RowType 子元素指定集合的类型：

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**CollectionType**元素。 然而，自定义特性可能不属于为 SSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示使用的函数**CollectionType**元素来指定该函数返回的行集合。

``` xml
   <Function Name="GetProducts" IsComposable="true" Schema="dbo">
     <ReturnType>
       <CollectionType>
         <RowType>
           <Property Name="ProductID" Type="int" Nullable="false" />
           <Property Name="CategoryID" Type="bigint" Nullable="false" />
           <Property Name="ProductName" Type="nvarchar" MaxLength="40" Nullable="false" />
           <Property Name="UnitPrice" Type="money" />
           <Property Name="Discontinued" Type="bit" />
         </RowType>
       </CollectionType>
     </ReturnType>
   </Function>
```

## <a name="commandtext-element-ssdl"></a>CommandText 元素 (SSDL)

**CommandText**存储架构定义语言 (SSDL) 中的元素是允许你定义在数据库中执行的 SQL 语句在 Function 元素的子级。 **CommandText**元素可用于添加类似于在数据库中，存储过程的功能，但不是定义**CommandText**存储模型中的元素。

**CommandText**元素不能具有子元素。 正文**CommandText**元素必须是有效的 SQL 语句的基础数据库。

任何属性都适用于**CommandText**元素。

### <a name="example"></a>示例

下面的示例演示**函数**具有一个子元素**CommandText**元素。 公开**UpdateProductInOrder**通过导入到概念模型函数作为 ObjectContext 的方法。  

``` xml
 <Function Name="UpdateProductInOrder" IsComposable="false">
   <CommandText>
     UPDATE Orders
     SET ProductId = @productId
     WHERE OrderId = @orderId;
   </CommandText>
   <Parameter Name="productId"
              Mode="In"
              Type="int"/>
   <Parameter Name="orderId"
              Mode="In"
              Type="int"/>
 </Function>
```

## <a name="definingquery-element-ssdl"></a>DefiningQuery 元素 (SSDL)

**DefiningQuery**存储架构定义语言 (SSDL) 中的元素，你可以在基础数据库中直接执行 SQL 语句。 **DefiningQuery**元素通常使用数据库视图类似，但该视图定义存储模型而不是数据库中。 中定义的视图**DefiningQuery**元素都可以映射到概念模型通过 EntitySetMapping 元素中的实体类型。 这些映射是只读的。  

下面的 SSDL 语法显示如何声明**EntitySet**跟**DefiningQuery**元素，其中包含用于检索视图的查询。

``` xml
 <Schema>
     <EntitySet Name="Tables" EntityType="Self.STable">
         <DefiningQuery>
           SELECT  TABLE_CATALOG,
                   'test' as TABLE_SCHEMA,
                   TABLE_NAME
           FROM    INFORMATION_SCHEMA.TABLES
         </DefiningQuery>
     </EntitySet>
 </Schema>
```

可以使用实体框架中的存储的过程对视图启用读写方案。 用于检索数据和更改处理由存储过程，可以为基本的表使用数据源视图或 Entity SQL 视图。

可以使用**DefiningQuery**元素用于目标 Microsoft SQL Server Compact 3.5。 尽管 SQL Server Compact 3.5 不支持存储的过程，您可以实现类似功能与**DefiningQuery**元素。 另一个有用之处在于，可以创建存储过程以克服编程语言中使用的数据类型与数据源的数据类型的不匹配。 你可以编写**DefiningQuery** ，采用特定的参数集，然后调用带有一组不同的参数，例如，删除数据的存储过程的存储的过程。

## <a name="dependent-element-ssdl"></a>Dependent 元素 (SSDL)

**依赖**存储架构定义语言 (SSDL) 中的元素是定义 （也称为引用约束） 的外键约束的依赖端 ReferentialConstraint 元素的子元素。 **依赖**元素引用的主键列 （或列） 的表中指定的列 （或列）。 **PropertyRef**元素指定所引用的列。 主体元素指定的列中指定的引用的主键列**依赖**元素。

**依赖**元素可以具有下列子元素 （按所列顺序）：

-   PropertyRef （一个或多个）
-   批注元素 （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**依赖**元素。

| 特性名 | 是否必需 | “值”                                                                                                                                                       |
|:---------------|:------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **角色**       | 是         | 相同的值**角色**相应的结束元素的特性 （若使用）; 否则为的表的名称包含的引用列。 |

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**依赖**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示使用 Association 元素**ReferentialConstraint**元素指定参与的列**FK\_CustomerOrders**外键约束。 **依赖**元素指定**CustomerId**的列**顺序**表约束的依赖端。

``` xml
 <Association Name="FK_CustomerOrders">
   <End Role="Customers"
        Type="ExampleModel.Store.Customers" Multiplicity="1">
     <OnDelete Action="Cascade" />
   </End>
   <End Role="Orders"
        Type="ExampleModel.Store.Orders" Multiplicity="*" />
   <ReferentialConstraint>
     <Principal Role="Customers">
       <PropertyRef Name="CustomerId" />
     </Principal>
     <Dependent Role="Orders">
       <PropertyRef Name="CustomerId" />
     </Dependent>
   </ReferentialConstraint>
 </Association>
```

## <a name="documentation-element-ssdl"></a>Documentation 元素 (SSDL)

**文档**存储架构定义语言 (SSDL) 中的元素可以用于提供有关的对象的父元素中定义的信息。

**文档**元素可以具有下列子元素 （按所列顺序）：

-   **摘要**： 父元素的简要说明。 （零个或一个元素）
-   **LongDescription**： 父元素的详细说明。 （零个或一个元素）

### <a name="applicable-attributes"></a>适用的特性

可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**文档**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示**文档**元素作为 EntityType 元素的子元素。

``` xml
 <EntityType Name="Customers">
   <Documentation>
     <Summary>Summary here.</Summary>
     <LongDescription>Long description here.</LongDescription>
   </Documentation>
   <Key>
     <PropertyRef Name="CustomerId" />
   </Key>
   <Property Name="CustomerId" Type="int" Nullable="false" />
   <Property Name="Name" Type="nvarchar(max)" Nullable="false" />
 </EntityType>
```

## <a name="end-element-ssdl"></a>End 元素 (SSDL)

**最终**存储架构定义语言 (SSDL) 中的元素与基础数据库中指定的表和外键约束一端的行数。 **最终**元素可以是 Association 元素或 AssociationSet 元素的子级。 在每种情况下，可能的子元素以及适用的特性都不同。

### <a name="end-element-as-a-child-of-the-association-element"></a>End 元素作为 Association 元素的子元素

**最终**元素 (作为子级**关联**元素) 指定的表和外键约束与结尾处的行数**类型**和**多重性**分别属性。 外键约束的两端定义为 SSDL 关联的一部分；SSDL 关联必须仅具有两端。

**最终**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   OnDelete （零个或一个元素）
-   批注元素 （零个或多个元素）

#### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**最终**时的子元素**关联**元素。

| 特性名   | 是否必需 | “值”                                                                                                                                                                                                                                                                                                                                                                                      |
|:-----------------|:------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Type**         | 是         | 在外键约束的末尾处的 SSDL 实体集的完全限定的名称。                                                                                                                                                                                                                                                                                          |
| **角色**         | 否          | 值**角色**属性中的主体或依赖于元素的相应 ReferentialConstraint 元素 （如果使用）。                                                                                                                                                                                                                                             |
| **重数** | 是         | **1**， **0..1**，或**\*** 具体取决于外键约束一端可以存在的行数。 <br/> **1**指示外键约束端处存在恰好一个行。 <br/> **0..1**指示外键约束端处存在零个或一个行。 <br/> **\*** 指示外键约束端处存在零、 一个或多个行。 |

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**最终**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

#### <a name="example"></a>示例

下面的示例演示**关联**定义的元素**FK\_CustomerOrders**外键约束。 **多重性**上每个指定值**最终**元素指示中的多行**订单**表可以与中的行相关联**客户**表中，但中的仅有一行**客户**表可以与中的行相关联**订单**表。 此外， **OnDelete**元素指示的所有行中**订单**引用中的特定行的表**客户**如果将删除的表中的行**客户**删除该表。

``` xml
 <Association Name="FK_CustomerOrders">
   <End Role="Customers"
        Type="ExampleModel.Store.Customers" Multiplicity="1">
     <OnDelete Action="Cascade" />
   </End>
   <End Role="Orders"
        Type="ExampleModel.Store.Orders" Multiplicity="*" />
   <ReferentialConstraint>
     <Principal Role="Customers">
       <PropertyRef Name="CustomerId" />
     </Principal>
     <Dependent Role="Orders">
       <PropertyRef Name="CustomerId" />
     </Dependent>
   </ReferentialConstraint>
 </Association>
```

### <a name="end-element-as-a-child-of-the-associationset-element"></a>End 元素作为 AssociationSet 元素的子元素

**最终**元素 (作为子级**AssociationSet**元素) 与基础数据库中指定的外键约束一端的表。

**最终**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个）
-   批注元素 （零个或多个）

#### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**最终**时的子元素**AssociationSet**元素。

| 特性名 | 是否必需 | “值”                                                                                                                  |
|:---------------|:------------|:-----------------------------------------------------------------------------------------------------------------------|
| **实体集**  | 是         | 在外键约束的末尾处的 SSDL 实体集的名称。                                      |
| **角色**       | 否          | 之一的值**角色**属性指定一个**最终**相应关联元素的元素。 |

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**最终**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

#### <a name="example"></a>示例

下面的示例演示**EntityContainer**具有元素**AssociationSet**元素具有两个**最终**元素：

``` xml
 <EntityContainer Name="ExampleModelStoreContainer">
   <EntitySet Name="Customers"
              EntityType="ExampleModel.Store.Customers"
              Schema="dbo" />
   <EntitySet Name="Orders"
              EntityType="ExampleModel.Store.Orders"
              Schema="dbo" />
   <AssociationSet Name="FK_CustomerOrders"
                   Association="ExampleModel.Store.FK_CustomerOrders">
     <End Role="Customers" EntitySet="Customers" />
     <End Role="Orders" EntitySet="Orders" />
   </AssociationSet>
 </EntityContainer>
```

## <a name="entitycontainer-element-ssdl"></a>EntityContainer 元素 (SSDL)

**EntityContainer**存储架构定义语言 (SSDL) 中的元素描述中的实体框架应用程序的基础数据源的结构： SSDL 实体集 （在 EntitySet 元素中定义） 表示中的表数据库中，SSDL 实体类型 （EntityType 元素中定义） 表示表中的行和关联集 （在 AssociationSet 元素中定义） 表示在数据库中的外键约束。 存储模型实体容器映射到概念模型实体容器通过 EntityContainerMapping 元素。

**EntityContainer**元素可以具有零个或一个文档元素。 如果**文档**元素存在，则它必须位于所有其他子元素之前。

**EntityContainer**元素可以具有零个或多个下列子元素 （按所列顺序）：

-   EntitySet
-   AssociationSet
-   批注元素

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**EntityContainer**元素。

| 特性名 | 是否必需 | “值”                                                                   |
|:---------------|:------------|:------------------------------------------------------------------------|
| **名称**       | 是         | 实体容器的名称。 此名称不能包含句点 (.)。 |

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**EntityContainer**元素。 然而，自定义特性可能不属于为 SSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示**EntityContainer**定义两个实体集和一个关联集的元素。 注意，实体类型和关联类型名称由概念模型命名空间名称限定。

``` xml
 <EntityContainer Name="ExampleModelStoreContainer">
   <EntitySet Name="Customers"
              EntityType="ExampleModel.Store.Customers"
              Schema="dbo" />
   <EntitySet Name="Orders"
              EntityType="ExampleModel.Store.Orders"
              Schema="dbo" />
   <AssociationSet Name="FK_CustomerOrders"
                   Association="ExampleModel.Store.FK_CustomerOrders">
     <End Role="Customers" EntitySet="Customers" />
     <End Role="Orders" EntitySet="Orders" />
   </AssociationSet>
 </EntityContainer>
```

## <a name="entityset-element-ssdl"></a>EntitySet 元素 (SSDL)

**EntitySet**存储架构定义语言 (SSDL) 中的元素表示表或视图基础数据库中的。 SSDL 中的 EntityType 元素表示表或视图中的行。 **EntityType**的属性**EntitySet**元素指定表示 SSDL 实体集中的行的特定 SSDL 实体类型。 EntitySetMapping 元素中指定的 CSDL 实体集和 SSDL 实体集之间的映射。

**EntitySet**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   DefiningQuery （零个或一个元素）
-   批注元素

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**EntitySet**元素。

> [!NOTE]
> （此处未列出） 的某些属性可能会用限定**存储**别名。 模型更新向导更新模型时使用这些属性。

| 特性名 | 是否必需 | “值”                                                                                    |
|:---------------|:------------|:-----------------------------------------------------------------------------------------|
| **名称**       | 是         | 实体集的名称。                                                              |
| **EntityType** | 是         | 实体集包含其实例的实体类型的完全限定名称。 |
| **架构**     | 否          | 数据库架构。                                                                     |
| **Table**      | 否          | 数据库表。                                                                      |

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**EntitySet**元素。 然而，自定义特性可能不属于为 SSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示**EntityContainer**元素具有两个**EntitySet**元素和一个**AssociationSet**元素：

``` xml
 <EntityContainer Name="ExampleModelStoreContainer">
   <EntitySet Name="Customers"
              EntityType="ExampleModel.Store.Customers"
              Schema="dbo" />
   <EntitySet Name="Orders"
              EntityType="ExampleModel.Store.Orders"
              Schema="dbo" />
   <AssociationSet Name="FK_CustomerOrders"
                   Association="ExampleModel.Store.FK_CustomerOrders">
     <End Role="Customers" EntitySet="Customers" />
     <End Role="Orders" EntitySet="Orders" />
   </AssociationSet>
 </EntityContainer>
```

## <a name="entitytype-element-ssdl"></a>EntityType 元素 (SSDL)

**EntityType**存储架构定义语言 (SSDL) 中的元素表示表或视图的基础数据库中的行。 SSDL 中的 EntitySet 元素表示表或视图，其中出现行。 **EntityType**的属性**EntitySet**元素指定表示 SSDL 实体集中的行的特定 SSDL 实体类型。 EntityTypeMapping 元素中指定的 SSDL 实体类型和 CSDL 实体类型之间的映射。

**EntityType**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个元素）
-   密钥 （零个或一个元素）
-   批注元素

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**EntityType**元素。

| 特性名 | 是否必需 | “值”                                                                                                                                                                  |
|:---------------|:------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **名称**       | 是         | 实体类型的名称。 此值通常与以实体类型表示行的表的名称相同。 此值可以不包含句点 (.)。 |

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**EntityType**元素。 然而，自定义特性可能不属于为 SSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示**EntityType**元素具有两个属性：

``` xml
 <EntityType Name="Customers">
   <Documentation>
     <Summary>Summary here.</Summary>
     <LongDescription>Long description here.</LongDescription>
   </Documentation>
   <Key>
     <PropertyRef Name="CustomerId" />
   </Key>
   <Property Name="CustomerId" Type="int" Nullable="false" />
   <Property Name="Name" Type="nvarchar(max)" Nullable="false" />
 </EntityType>
```

## <a name="function-element-ssdl"></a>Function 元素 (SSDL)

**函数**存储架构定义语言 (SSDL) 中的元素指定基础数据库中存在的存储的过程。

**函数**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个）
-   参数 （零个或多个）
-   CommandText （零个或一个）
-   ReturnType （零个或多个）
-   批注元素 （零个或多个）

返回类型的函数必须指定与**ReturnType**元素或**ReturnType**属性 （见下文），但不可同时使用两者。

可以将在存储模型中指定的存储过程导入应用程序的概念模型。 有关详细信息，请参阅[存储过程使用查询](~/ef6/modeling/designer/stored-procedures/query.md)。 **函数**元素还可用于在存储模型中定义自定义函数。  

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**函数**元素。

> [!NOTE]
> （此处未列出） 的某些属性可能会用限定**存储**别名。 模型更新向导更新模型时使用这些属性。

| 特性名             | 是否必需 | “值”                                                                                                                                                                                                              |
|:---------------------------|:------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **名称**                   | 是         | 存储过程的名称。                                                                                                                                                                                  |
| **ReturnType**             | 否          | 存储过程的返回类型。                                                                                                                                                                           |
| **Aggregate**              | 否          | **True**如果存储的过程返回的聚合值; 否则为**False**。                                                                                                                                  |
| **内置**                | 否          | **True**如果函数为内置<sup>1</sup>函数; 否则为**False**。                                                                                                                                  |
| **StoreFunctionName**      | 否          | 存储过程的名称。                                                                                                                                                                                  |
| **NiladicFunction**        | 否          | **True**如果函数为 niladic<sup>2</sup>函数;**False**否则为。                                                                                                                                   |
| **IsComposable**           | 否          | **True**如果该函数是可组合<sup>3</sup>函数;**False**否则为。                                                                                                                                |
| **ParameterTypeSemantics** | 否          | 定义用于解析函数重载的类型语义的枚举。 该枚举是在提供程序清单中根据函数定义来定义的。 默认值是**AllowImplicitConversion**。 |
| **架构**                 | 否          | 在其中定义存储过程的架构的名称。                                                                                                                                                   |

<sup>1</sup>内置函数是在数据库中定义的函数。 有关存储模型中定义的函数的信息，请参阅 CommandText 元素 (SSDL)。

<sup>2</sup> niladic 函数是不接受任何参数，并调用时，不需要使用括号的函数。

<sup>3</sup>两个函数是可组合，如果一个函数的输出可以是其他函数的输入。

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**函数**元素。 然而，自定义特性可能不属于为 SSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示**函数**对应的元素**UpdateOrderQuantity**存储过程。 该存储过程接受两个参数，且不返回值。

``` xml
 <Function Name="UpdateOrderQuantity"
           Aggregate="false"
           BuiltIn="false"
           NiladicFunction="false"
           IsComposable="false"
           ParameterTypeSemantics="AllowImplicitConversion"
           Schema="dbo">
   <Parameter Name="orderId" Type="int" Mode="In" />
   <Parameter Name="newQuantity" Type="int" Mode="In" />
 </Function>
```

## <a name="key-element-ssdl"></a>Key 元素 (SSDL)

**密钥**存储架构定义语言 (SSDL) 中的元素表示基础数据库中的表的主键。 **密钥**是 EntityType 元素，它表示表中的行的子元素。 在中定义的主键**键**通过引用一个或多个属性元素上定义的元素**EntityType**元素。

**密钥**元素可以具有下列子元素 （按所列顺序）：

-   PropertyRef （一个或多个）
-   批注元素

任何属性都适用于**密钥**元素。

### <a name="example"></a>示例

下面的示例演示**EntityType**具有引用一个属性的键的元素：

``` xml
 <EntityType Name="Customers">
   <Documentation>
     <Summary>Summary here.</Summary>
     <LongDescription>Long description here.</LongDescription>
   </Documentation>
   <Key>
     <PropertyRef Name="CustomerId" />
   </Key>
   <Property Name="CustomerId" Type="int" Nullable="false" />
   <Property Name="Name" Type="nvarchar(max)" Nullable="false" />
 </EntityType>
```

## <a name="ondelete-element-ssdl"></a>OnDelete 元素 (SSDL)

**OnDelete**删除参与外键约束的行时，存储架构定义语言 (SSDL) 中的元素反映数据库的行为。 如果将操作设置为**Cascade**，则还将删除引用要删除的行的行。 如果将操作设置为**None**，然后引用要删除的行的行也不会删除。 **OnDelete**元素是子元素的结束元素。

**OnDelete**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个）
-   批注元素 （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**OnDelete**元素。

| 特性名 | 是否必需 | “值”                                                                                               |
|:---------------|:------------|:----------------------------------------------------------------------------------------------------|
| **操作**     | 是         | **级联**或**None**。 (该值**Restricted**有效，但具有相同的行为**None**。) |

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**OnDelete**元素。 然而，自定义特性可能不属于为 SSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示**关联**定义的元素**FK\_CustomerOrders**外键约束。 **OnDelete**元素指示的所有行中**订单**引用中的特定行的表**客户**将删除的表，如果中行**客户**删除该表。

``` xml
 <Association Name="FK_CustomerOrders">
   <End Role="Customers"
        Type="ExampleModel.Store.Customers" Multiplicity="1">
     <OnDelete Action="Cascade" />
   </End>
   <End Role="Orders"
        Type="ExampleModel.Store.Orders" Multiplicity="*" />
   <ReferentialConstraint>
     <Principal Role="Customers">
       <PropertyRef Name="CustomerId" />
     </Principal>
     <Dependent Role="Orders">
       <PropertyRef Name="CustomerId" />
     </Dependent>
   </ReferentialConstraint>
 </Association>
```

## <a name="parameter-element-ssdl"></a>Parameter 元素 (SSDL)

**参数**存储架构定义语言 (SSDL) 中的元素是指定数据库中的存储过程参数的函数元素的子级。

**参数**元素可以具有下列子元素 （按所列顺序）：

-   文档 （零个或一个）
-   批注元素 （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**参数**元素。

| 特性名 | 是否必需 | “值”                                                                                                                                                                                                                           |
|:---------------|:------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **名称**       | 是         | 参数的名称。                                                                                                                                                                                                      |
| **Type**       | 是         | 参数类型。                                                                                                                                                                                                             |
| **模式**       | 否          | **在中**，**出**，或**InOut**具体取决于参数是输入、 输出或输入/输出参数。                                                                                                                |
| **MaxLength**  | 否          | 参数的最大长度。                                                                                                                                                                                            |
| **精度**  | 否          | 参数的精度。                                                                                                                                                                                                 |
| 缩放      | 否          | 参数的小数位数。                                                                                                                                                                                                     |
| **SRID**       | 否          | 系统的空间引用标识符。 仅对空间类型的参数有效。 有关详细信息，请参阅[SRID](http://en.wikipedia.org/wiki/SRID)并[SRID (SQL Server)](https://msdn.microsoft.com/library/bb964707.aspx)。 |

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**参数**元素。 然而，自定义特性可能不属于为 SSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示**函数**元素具有两个**参数**指定输入的参数的元素：

``` xml
 <Function Name="UpdateOrderQuantity"
           Aggregate="false"
           BuiltIn="false"
           NiladicFunction="false"
           IsComposable="false"
           ParameterTypeSemantics="AllowImplicitConversion"
           Schema="dbo">
   <Parameter Name="orderId" Type="int" Mode="In" />
   <Parameter Name="newQuantity" Type="int" Mode="In" />
 </Function>
```

## <a name="principal-element-ssdl"></a>Principal 元素 (SSDL)

**主体**存储架构定义语言 (SSDL) 中的元素是定义 （也称为引用约束） 的外键约束的主体端 ReferentialConstraint 元素的子元素。 **主体**元素引用的另一列 （或列） 的表中指定的主键列 （或列）。 **PropertyRef**元素指定所引用的列。 Dependent 元素指定引用中指定的主键列的列**主体**元素。

**主体**元素可以具有下列子元素 （按所列顺序）：

-   PropertyRef （一个或多个）
-   批注元素 （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**主体**元素。

| 特性名 | 是否必需 | “值”                                                                                                                                                      |
|:---------------|:------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| **角色**       | 是         | 相同的值**角色**相应的结束元素的特性 （若使用）; 否则为的表的名称包含的被引用的列。 |

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**主体**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示使用 Association 元素**ReferentialConstraint**元素指定参与的列**FK\_CustomerOrders**外键约束。 **主体**元素指定**CustomerId**的列**客户**表约束的主体端。

``` xml
 <Association Name="FK_CustomerOrders">
   <End Role="Customers"
        Type="ExampleModel.Store.Customers" Multiplicity="1">
     <OnDelete Action="Cascade" />
   </End>
   <End Role="Orders"
        Type="ExampleModel.Store.Orders" Multiplicity="*" />
   <ReferentialConstraint>
     <Principal Role="Customers">
       <PropertyRef Name="CustomerId" />
     </Principal>
     <Dependent Role="Orders">
       <PropertyRef Name="CustomerId" />
     </Dependent>
   </ReferentialConstraint>
 </Association>
```

## <a name="property-element-ssdl"></a>Property 元素 (SSDL)

**属性**存储架构定义语言 (SSDL) 中的元素表示基础数据库中的表中的列。 **属性**元素是 EntityType 元素，它表示表中的行的子级。 每个**属性**元素中定义**EntityType**元素表示的列。

一个**属性**元素不能具有任何子元素。

### <a name="applicable-attributes"></a>适用的特性

下表介绍可应用于的特性**属性**元素。

| 特性名            | 是否必需 | “值”                                                                                                                                                                                                                           |
|:--------------------------|:------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **名称**                  | 是         | 对应列的名称。                                                                                                                                                                                           |
| **Type**                  | 是         | 对应列的类型。                                                                                                                                                                                           |
| **可以为 null**              | 否          | **True** （默认值） 或**False**具体取决于相应的列是否可以具有 null 值。                                                                                                                  |
| **DefaultValue**          | 否          | 对应列的默认值。                                                                                                                                                                                  |
| **MaxLength**             | 否          | 对应列的最大长度。                                                                                                                                                                                 |
| **FixedLength**           | 否          | **True**或**False**具体取决于对应列值是否将作为固定的长度字符串存储。                                                                                                              |
| **精度**             | 否          | 对应列的精度。                                                                                                                                                                                      |
| 缩放                 | 否          | 对应列的小数位数。                                                                                                                                                                                          |
| **Unicode**               | 否          | **True**或**False**具体取决于对应列值是否将存储为 Unicode 字符串。                                                                                                                   |
| **排序规则**             | 否          | 指定要在数据源中使用的排序序列的字符串。                                                                                                                                                   |
| **SRID**                  | 否          | 系统的空间引用标识符。 仅对空间类型的属性有效。 有关详细信息，请参阅[SRID](http://en.wikipedia.org/wiki/SRID)并[SRID (SQL Server)](https://msdn.microsoft.com/library/bb964707.aspx)。 |
| **StoreGeneratedPattern** | 否          | **无**，**标识**（如果对应列值为数据库中生成的标识），或**计算**（如果相应列计算的值是在数据库中）。 不是有效行类型属性。 |

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**属性**元素。 然而，自定义特性可能不属于为 SSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示**EntityType**具有两个子元素**属性**元素：

``` xml
 <EntityType Name="Customers">
   <Documentation>
     <Summary>Summary here.</Summary>
     <LongDescription>Long description here.</LongDescription>
   </Documentation>
   <Key>
     <PropertyRef Name="CustomerId" />
   </Key>
   <Property Name="CustomerId" Type="int" Nullable="false" />
   <Property Name="Name" Type="nvarchar(max)" Nullable="false" />
 </EntityType>
```

## <a name="propertyref-element-ssdl"></a>PropertyRef 元素 (SSDL)

**PropertyRef**存储架构定义语言 (SSDL) 中的元素引用以指示该属性将承担以下角色之一的 EntityType 元素上定义的属性：

-   是表的主键的一部分的**EntityType**表示。 一个或多个**PropertyRef**元素可用于定义主键。 有关详细信息，请参阅密钥元素。
-   是引用约束的依赖端或主体端。 有关详细信息，请参阅 ReferentialConstraint 元素。

**PropertyRef**元素只能具有以下子元素：

-   文档 （零个或一个）
-   批注元素

### <a name="applicable-attributes"></a>适用的特性

下表描述了可应用于的特性**PropertyRef**元素。

| 特性名 | 是否必需 | “值”                                |
|:---------------|:------------|:-------------------------------------|
| **名称**       | 是         | 所引用属性的名称。 |

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**PropertyRef**元素。 然而，自定义特性可能不属于为 CSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示**PropertyRef**用来通过引用定义的属性定义为主键的元素**EntityType**元素。

``` xml
 <EntityType Name="Customers">
   <Documentation>
     <Summary>Summary here.</Summary>
     <LongDescription>Long description here.</LongDescription>
   </Documentation>
   <Key>
     <PropertyRef Name="CustomerId" />
   </Key>
   <Property Name="CustomerId" Type="int" Nullable="false" />
   <Property Name="Name" Type="nvarchar(max)" Nullable="false" />
 </EntityType>
```

## <a name="referentialconstraint-element-ssdl"></a>ReferentialConstraint 元素 (SSDL)

**ReferentialConstraint**存储架构定义语言 (SSDL) 中的元素表示基础数据库中的外键约束 （也称为引用完整性约束）。 分别由主体和依赖于子元素，指定的主体端和依赖端约束。 PropertyRef 元素引用参与主体端和依赖端的列。

**ReferentialConstraint**元素是 Association 元素的可选子元素。 如果**ReferentialConstraint**元素不用于映射中指定的外键约束**关联**元素，AssociationSetMapping 元素必须用来执行此操作。

**ReferentialConstraint**元素可以具有以下子元素：

-   文档 （零个或一个）
-   主体 （恰好一个）
-   依赖于 （恰好一个）
-   批注元素 （零个或多个）

### <a name="applicable-attributes"></a>适用的特性

可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**ReferentialConstraint**元素。 然而，自定义特性可能不属于为 SSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例演示**关联**使用的元素**ReferentialConstraint**元素指定参与的列**FK\_CustomerOrders**外键约束：

``` xml
 <Association Name="FK_CustomerOrders">
   <End Role="Customers"
        Type="ExampleModel.Store.Customers" Multiplicity="1">
     <OnDelete Action="Cascade" />
   </End>
   <End Role="Orders"
        Type="ExampleModel.Store.Orders" Multiplicity="*" />
   <ReferentialConstraint>
     <Principal Role="Customers">
       <PropertyRef Name="CustomerId" />
     </Principal>
     <Dependent Role="Orders">
       <PropertyRef Name="CustomerId" />
     </Dependent>
   </ReferentialConstraint>
 </Association>
```

## <a name="returntype-element-ssdl"></a>ReturnType 元素 (SSDL)

**ReturnType**元素中存储架构定义语言 (SSDL) 指定中定义的函数的返回类型**函数**元素。 此外可以使用指定返回类型的函数**ReturnType**属性。

使用指定函数的返回类型**类型**属性或**ReturnType**元素。

**ReturnType**元素可以具有以下子元素：

- CollectionType （一个）  

> [!NOTE]
> 可以将任意数量的 （自定义 XML 特性） 的批注特性应用于**ReturnType**元素。 然而，自定义特性可能不属于为 SSDL 保留的任何 XML 命名空间。 任何两个自定义特性的完全限定名称都不能相同。

### <a name="example"></a>示例

下面的示例使用**函数**返回的行集合。

``` xml
   <Function Name="GetProducts" IsComposable="true" Schema="dbo">
     <ReturnType>
       <CollectionType>
         <RowType>
           <Property Name="ProductID" Type="int" Nullable="false" />
           <Property Name="CategoryID" Type="bigint" Nullable="false" />
           <Property Name="ProductName" Type="nvarchar" MaxLength="40" Nullable="false" />
           <Property Name="UnitPrice" Type="money" />
           <Property Name="Discontinued" Type="bit" />
         </RowType>
       </CollectionType>
     </ReturnType>
   </Function>
```


## <a name="rowtype-element-ssdl"></a>RowType 元素 (SSDL)

一个**RowType**元素中的以存储架构定义语言 (SSDL) 定义为返回一个未命名的结构在存储中定义的函数的类型。

一个**RowType**元素是子元素**CollectionType**元素：

一个**RowType**元素可以具有以下子元素：

- 属性 （一个或多个）  

### <a name="example"></a>示例

下面的示例演示使用的存储函数**CollectionType**元素来指定该函数返回的行集合 (中指定的那样**RowType**元素)。


``` xml
   <Function Name="GetProducts" IsComposable="true" Schema="dbo">
     <ReturnType>
       <CollectionType>
         <RowType>
           <Property Name="ProductID" Type="int" Nullable="false" />
           <Property Name="CategoryID" Type="bigint" Nullable="false" />
           <Property Name="ProductName" Type="nvarchar" MaxLength="40" Nullable="false" />
           <Property Name="UnitPrice" Type="money" />
           <Property Name="Discontinued" Type="bit" />
         </RowType>
       </CollectionType>
     </ReturnType>
   </Function>
```

## <a name="schema-element-ssdl"></a>Schema 元素 (SSDL)

**架构**存储架构定义语言 (SSDL) 中的元素是存储模型定义的根元素。 它包括构成存储模型的对象、函数和容器的定义。

**架构**元素可能包含零个或多个下列子元素：

-   关联
-   EntityType
-   EntityContainer
-   函数

**架构**元素使用**Namespace**属性来定义存储模型中的实体类型和关联对象的命名空间。 在命名空间内，任何两个对象都不能同名。

存储模型命名空间是不同的 XML 命名空间**架构**元素。 存储模型命名空间 (由定义**Namespace**属性) 是实体类型和关联类型的逻辑容器。 XML 命名空间 (由**xmlns**属性) 的**架构**元素是子元素和属性的默认命名空间**架构**元素。 窗体的 XML 命名空间 http://schemas.microsoft.com/ado/YYYY/MM/edm/ssdl （其中 YYYY 和 MM 表示年和月分别） 是为 SSDL 保留。 自定义元素和特性不能位于具有此格式的命名空间中。

### <a name="applicable-attributes"></a>适用的特性

下表描述了特性可应用于**架构**元素。

| 特性名            | 是否必需 | “值”                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|:--------------------------|:------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **命名空间**             | 是         | 存储模型的命名空间。 值**Namespace**特性用于形成一种类型的完全限定的名称。 例如，如果**EntityType**名为*客户*位于 ExampleModel.Store 命名空间，则完全限定的名称**EntityType**是ExampleModel.Store.Customer。 <br/> 以下字符串不能使用的值作为**Namespace**属性：**系统**，**暂时性**，或者**Edm**。 值**Namespace**属性不能与的值相同**Namespace** CSDL 架构元素中的属性。 |
| **Alias**                 | 否          | 用于取代命名空间名称的标识符。 例如，如果**EntityType**名为*客户*位于 ExampleModel.Store 命名空间和值**别名**特性是*StorageModel*，则您可以将 StorageModel.Customer 用作完全限定的名称**EntityType。**                                                                                                                                                                                                                                                                                    |
| **提供程序**              | 是         | 数据提供程序。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| **ProviderManifestToken** | 是         | 一个标记，该标记指示提供程序清单返回到的提供程序。 没有为该标记定义格式。 标记的值由提供程序定义。 有关 SQL Server 提供程序清单标记信息，请参阅实体框架的 SqlClient。                                                                                                                                                                                                                                                                                                                        |

### <a name="example"></a>示例

下面的示例演示**架构**元素，其中包含**EntityContainer**元素、 两个**EntityType**元素，其中一个**关联**元素。

``` xml
 <Schema Namespace="ExampleModel.Store"
       Alias="Self" Provider="System.Data.SqlClient"
       ProviderManifestToken="2008"
       xmlns="http://schemas.microsoft.com/ado/2009/11/edm/ssdl">
   <EntityContainer Name="ExampleModelStoreContainer">
     <EntitySet Name="Customers"
                EntityType="ExampleModel.Store.Customers"
                Schema="dbo" />
     <EntitySet Name="Orders"
                EntityType="ExampleModel.Store.Orders"
                Schema="dbo" />
     <AssociationSet Name="FK_CustomerOrders"
                     Association="ExampleModel.Store.FK_CustomerOrders">
       <End Role="Customers" EntitySet="Customers" />
       <End Role="Orders" EntitySet="Orders" />
     </AssociationSet>
   </EntityContainer>
   <EntityType Name="Customers">
     <Documentation>
       <Summary>Summary here.</Summary>
       <LongDescription>Long description here.</LongDescription>
     </Documentation>
     <Key>
       <PropertyRef Name="CustomerId" />
     </Key>
     <Property Name="CustomerId" Type="int" Nullable="false" />
     <Property Name="Name" Type="nvarchar(max)" Nullable="false" />
   </EntityType>
   <EntityType Name="Orders" xmlns:c="http://CustomNamespace">
     <Key>
       <PropertyRef Name="OrderId" />
     </Key>
     <Property Name="OrderId" Type="int" Nullable="false"
               c:CustomAttribute="someValue"/>
     <Property Name="ProductId" Type="int" Nullable="false" />
     <Property Name="Quantity" Type="int" Nullable="false" />
     <Property Name="CustomerId" Type="int" Nullable="false" />
     <c:CustomElement>
       Custom data here.
     </c:CustomElement>
   </EntityType>
   <Association Name="FK_CustomerOrders">
     <End Role="Customers"
          Type="ExampleModel.Store.Customers" Multiplicity="1">
       <OnDelete Action="Cascade" />
     </End>
     <End Role="Orders"
          Type="ExampleModel.Store.Orders" Multiplicity="*" />
     <ReferentialConstraint>
       <Principal Role="Customers">
         <PropertyRef Name="CustomerId" />
       </Principal>
       <Dependent Role="Orders">
         <PropertyRef Name="CustomerId" />
       </Dependent>
     </ReferentialConstraint>
   </Association>
   <Function Name="UpdateOrderQuantity"
             Aggregate="false"
             BuiltIn="false"
             NiladicFunction="false"
             IsComposable="false"
             ParameterTypeSemantics="AllowImplicitConversion"
             Schema="dbo">
     <Parameter Name="orderId" Type="int" Mode="In" />
     <Parameter Name="newQuantity" Type="int" Mode="In" />
   </Function>
   <Function Name="UpdateProductInOrder" IsComposable="false">
     <CommandText>
       UPDATE Orders
       SET ProductId = @productId
       WHERE OrderId = @orderId;
     </CommandText>
     <Parameter Name="productId"
                Mode="In"
                Type="int"/>
     <Parameter Name="orderId"
                Mode="In"
                Type="int"/>
   </Function>
 </Schema>
```

## <a name="annotation-attributes"></a>Annotation 特性

以存储架构定义语言 (SSDL) 表示的批注特性在存储模型中是自定义 XML 特性，这些特性提供有关存储模型中元素的额外元数据。 除了具有有效的 XML 结构之外，以下约束也适用于批注特性：

-   批注特性不能位于为 SSDL 保留的任何 XML 命名空间中。
-   任何两个批注特性的完全限定名称都不能相同。

可以将多个批注特性应用于一个给定的 SSDL 元素。 可以使用 System.Data.Metadata.Edm 命名空间中的类，在运行时访问批注元素中包含的元数据。

### <a name="example"></a>示例

下面的示例演示一个具有将批注特性应用到的 EntityType 元素**OrderId**属性。 本示例还演示批注元素添加到**EntityType**元素。

``` xml
 <EntityType Name="Orders" xmlns:c="http://CustomNamespace">
   <Key>
     <PropertyRef Name="OrderId" />
   </Key>
   <Property Name="OrderId" Type="int" Nullable="false"
             c:CustomAttribute="someValue"/>
   <Property Name="ProductId" Type="int" Nullable="false" />
   <Property Name="Quantity" Type="int" Nullable="false" />
   <Property Name="CustomerId" Type="int" Nullable="false" />
   <c:CustomElement>
     Custom data here.
   </c:CustomElement>
 </EntityType>
```

## <a name="annotation-elements-ssdl"></a>批注元素 (SSDL)

以存储架构定义语言 (SSDL) 表示的批注元素是存储模型中的自定义 XML 元素，可提供有关存储模型的额外元数据。 除了具有有效的 XML 结构之外，批注元素还应满足以下约束：

-   批注元素不能位于为 SSDL 保留的任何 XML 命名空间中。
-   任何两个批注元素的完全限定名称都不能相同。
-   批注元素必须出现在给定 SSDL 元素的所有其他子元素之后。

多个批注元素可能是某个给定 SSDL 元素的子元素。 从.NET Framework 版本 4 开始，访问元数据批注元素中包含可以在运行时使用 System.Data.Metadata.Edm 命名空间中的类。

### <a name="example"></a>示例

下面的示例演示一个具有一个批注元素的 EntityType 元素 (**CustomElement**)。 该示例还显示应用于的批注特性**OrderId**属性。

``` xml
 <EntityType Name="Orders" xmlns:c="http://CustomNamespace">
   <Key>
     <PropertyRef Name="OrderId" />
   </Key>
   <Property Name="OrderId" Type="int" Nullable="false"
             c:CustomAttribute="someValue"/>
   <Property Name="ProductId" Type="int" Nullable="false" />
   <Property Name="Quantity" Type="int" Nullable="false" />
   <Property Name="CustomerId" Type="int" Nullable="false" />
   <c:CustomElement>
     Custom data here.
   </c:CustomElement>
 </EntityType>
```

## <a name="facets-ssdl"></a>方面 (SSDL)

方面以存储架构定义语言 (SSDL) 表示的属性元素中指定的列类型的约束。 方面作为 XML 特性实现上**属性**元素。

下表描述了 SSDL 中支持的方面：

| 方面           | 描述                                                                                                                                                                                                                                                 |
|:----------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **排序规则**   | 指定在对属性值执行比较和排序操作时要使用的排序序列。                                                                                                             |
| **FixedLength** | 指定列值的长度是否可变。                                                                                                                                                                                                  |
| **MaxLength**   | 指定列值的最大长度。                                                                                                                                                                                                           |
| **精度**   | 类型的属性**十进制**，指定属性值可以具有的位数。 类型的属性**时间**， **DateTime**，并**DateTimeOffset**，指定列值的秒的小数部分的位数。 |
| 缩放       | 指定列值小数点右侧的位数。                                                                                                                                                                      |
| **Unicode**     | 指示是否将列值存储为 Unicode。                                                                                                                                                                                                    |
