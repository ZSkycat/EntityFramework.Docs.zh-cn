---
title: 代码优先约定-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: bc644573-c2b2-4ed7-8745-3c37c41058ad
ms.openlocfilehash: 4d03a32db5d84eb37c22617a95005b272172a65d
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490988"
---
# <a name="code-first-conventions"></a>代码优先约定
Code First 允许使用 C# 或 Visual Basic.NET 类的描述模型。 通过使用约定来检测到该模型的基本形状。 约定是用于自动配置基于类定义时使用 Code First 的概念模型的规则集。 约定 System.Data.Entity.ModelConfiguration.Conventions 命名空间中定义。  

通过使用数据注释或 fluent API，可以进一步配置您的模型。 优先级通过 fluent API 跟数据注释，然后约定的配置。 有关详细信息请参阅[数据批注](~/ef6/modeling/code-first/data-annotations.md)， [Fluent API-关系](~/ef6/modeling/code-first/fluent/relationships.md)， [Fluent API 的类型和属性](~/ef6/modeling/code-first/fluent/types-and-properties.md)和[VB.NET使用FluentAPI](~/ef6/modeling/code-first/fluent/vb.md).  

Code First 约定的详细的列表现已推出[API 文档](https://msdn.microsoft.com/library/system.data.entity.modelconfiguration.conventions.aspx)。 本主题概述使用 Code First 的约定。  

## <a name="type-discovery"></a>类型发现  

使用 Code First 开发时通常首先编写.NET Framework 类定义概念 （域） 模型。 除了定义类，您还需要让**DbContext**知道你想要在模型中包含的类型。 若要执行此操作，您定义派生上下文类**DbContext** ，并公开**DbSet**你想要将模型的一部分的类型的属性。 代码首先将包括这些类型，并还会中拉入任何引用的类型，即使不同程序集中定义引用的类型。  

如果您的类型参与继承层次结构，它就足以定义**DbSet**它们是否在同一程序集中作为类的基类将自动包含基类和派生的类型的属性。  

在以下示例中，都只有一个**DbSet**定义的属性**SchoolEntities**类 (**部门**)。 代码首先使用此属性以发现并请求任何引用的类型。  

``` csharp
public class SchoolEntities : DbContext
{
    public DbSet<Department> Departments { get; set; }
}

public class Department
{
    // Primary key
    public int DepartmentID { get; set; }
    public string Name { get; set; }

    // Navigation property
    public virtual ICollection<Course> Courses { get; set; }
}

public class Course
{
    // Primary key
    public int CourseID { get; set; }

    public string Title { get; set; }
    public int Credits { get; set; }

    // Foreign key
    public int DepartmentID { get; set; }

    // Navigation properties
    public virtual Department Department { get; set; }
}

public partial class OnlineCourse : Course
{
    public string URL { get; set; }
}

public partial class OnsiteCourse : Course
{
    public string Location { get; set; }
    public string Days { get; set; }
    public System.DateTime Time { get; set; }
}
```  

如果你想要从模型中排除类型，使用**NotMapped**属性或**DbModelBuilder.Ignore** fluent API。  

```  csharp
modelBuilder.Ignore<Department>();
```  

## <a name="primary-key-convention"></a>主要密钥约定  

代码首先将推断属性为主键，是否一个类上的属性名为"ID"（不区分大小写），或类名后, 跟"ID"。 如果主键属性的类型是数值或 GUID 它就是配置为标识列。  

``` csharp
public class Department
{
    // Primary key
    public int DepartmentID { get; set; }

    . . .  

}
```  

## <a name="relationship-convention"></a>关系约定  

在实体框架中，导航属性提供一种导航两个实体类型之间的关系方法。 针对对象参与到其中的每个关系，各对象均可以具有导航属性。 导航属性，您可以导航和管理两个方向，返回引用对象中的关系 (在重数为两者中任何一个或零或一) 或集合 （如果重数为多个）。 代码首先会推断出关系基于您的类型上定义的导航属性。  

除了导航属性，我们建议您包括表示依赖对象的类型上的外键属性。 使用作为主体的主键属性相同的数据类型和名称遵循以下格式之一的任何属性表示外的键关系: '\<导航属性名称\>\<主体主键属性名称\>'，'\<主体类名\>\<主键属性名称\>，或\<主体主键属性名称\>。 如果找到多个匹配项的优先级有以上所列的顺序。 外键检测不区分大小写。 检测到外键属性时，Code First 推断出关系基于外键的为 null 性的多重性。 如果该属性是可以为 null 关系被注册为可选的;否则注册关系所需的方式。  

如果依赖实体的外键不可为 null，则第一个代码设置级联删除的关系。 如果依赖实体的外键是可以为 null，Code First 不会设置级联删除的关系，并且当删除主体将设置外键为 null。 重数和级联删除检测到的行为可以使用 fluent API 替代约定。  

在下面的示例使用导航属性和外键定义部门和 Course 类之间的关系。  

``` csharp
public class Department
{
    // Primary key
    public int DepartmentID { get; set; }
    public string Name { get; set; }

    // Navigation property
    public virtual ICollection<Course> Courses { get; set; }
}

public class Course
{
    // Primary key
    public int CourseID { get; set; }

    public string Title { get; set; }
    public int Credits { get; set; }

    // Foreign key
    public int DepartmentID { get; set; }

    // Navigation properties
    public virtual Department Department { get; set; }
}
```  

> [!NOTE]
> 如果有多个相同类型之间的关系 (例如，假设您定义**Person**并**书籍**类，其中**人员**类包含**ReviewedBooks**并**AuthoredBooks**导航属性和**书籍**类包含**作者**和**审阅者**导航属性) 需要使用数据注释或 fluent API 手动配置关系。 有关详细信息，请参阅[数据注释-关系](~/ef6/modeling/code-first/data-annotations.md)并[Fluent API-关系](~/ef6/modeling/code-first/fluent/relationships.md)。  

## <a name="complex-types-convention"></a>复杂类型约定  

当 Code First 发现其中主键不能推断出，并通过数据注释或 fluent API 注册没有主键的类定义时，然后该类型自动注册为复杂类型中。 复杂类型检测也需要类型不具有引用实体类型，并且未从另一种类型的集合属性引用的属性。 给定以下类定义 Code First 会推断出的**详细信息**是复杂类型，因为它没有主键。  

``` csharp
public partial class OnsiteCourse : Course
{
    public OnsiteCourse()
    {
        Details = new Details();
    }

    public Details Details { get; set; }
}

public class Details
{
    public System.DateTime Time { get; set; }
    public string Location { get; set; }
    public string Days { get; set; }
}
```  

## <a name="connection-string-convention"></a>连接字符串约定  

若要了解有关约定使用 DbContext 来发现要使用，请参阅的连接[连接和模型](~/ef6/fundamentals/configuring/connection-strings.md)。  

## <a name="removing-conventions"></a>删除约定  

您可以删除任何 System.Data.Entity.ModelConfiguration.Conventions 命名空间中定义的约定。 下面的示例移除**PluralizingTableNameConvention**。  

``` csharp
public class SchoolEntities : DbContext
{
     . . .

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        // Configure Code First to ignore PluralizingTableName convention
        // If you keep this convention, the generated tables  
        // will have pluralized names.
        modelBuilder.Conventions.Remove<PluralizingTableNameConvention>();
    }
}
```  

## <a name="custom-conventions"></a>自定义约定  

自定义约定支持 EF6 及更高版本。 有关详细信息请参阅[自定义代码优先约定](~/ef6/modeling/code-first/conventions/custom.md)。
