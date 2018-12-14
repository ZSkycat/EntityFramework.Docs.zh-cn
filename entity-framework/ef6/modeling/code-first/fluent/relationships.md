---
title: Fluent API 的关系-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: fd73b4f8-16d5-40f1-9640-885ceafe67a1
ms.openlocfilehash: 05f282c02699f8bf3c71197ac5e01000f1855917
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490462"
---
# <a name="fluent-api---relationships"></a>Fluent API 的关系
> [!NOTE]
> 此页提供了有关设置关系中第一个代码模型使用 fluent API 的信息。 有关在 EF 和如何访问和处理数据使用关系的关系的常规信息，请参阅[关系和导航属性](~/ef6/fundamentals/relationships.md)。  

在使用 Code First，可以通过定义域 CLR 类定义您的模型。 默认情况下，实体框架使用 Code First 约定来将您的类映射到数据库架构。 如果使用 Code First 的命名约定，在大多数情况下您可以依赖于 Code First 来设置基于外键和导航属性在类定义的表之间的关系。 如果定义您的类时不遵循约定或约定如果你想要更改的方式工作，可以使用 fluent API 或数据注释来配置您的类，以便代码优先可以映射在表之间的关系。  

## <a name="introduction"></a>介绍  

在通过 fluent API 配置关系时，开始 EntityTypeConfiguration 实例，然后使用 HasRequired、 HasOptional 或 HasMany 方法指定的此实体所参与的关系的类型。 这些 HasRequired 和 HasOptional 方法采用表示引用导航属性的 lambda 表达式。 HasMany 方法采用 lambda 表达式，表示集合导航属性。 然后可以使用 WithRequired、 WithOptional 和 WithMany 方法配置一个反向导航属性。 这些方法具有重载，它们不采用自变量和可用于单向导航使用指定的基数。  

然后可以使用 HasForeignKey 方法来配置外键属性。 此方法采用 lambda 表达式，表示要用作外键的属性。  

## <a name="configuring-a-required-to-optional-relationship-one-tozero-or-one"></a>配置所需-到-可选关系 （一个-到-零或一）  

下面的示例配置对零或一一关系。 OfficeAssignment 具有 primary key 和 foreign key 的 InstructorID 属性，因为该属性的名称不遵循 HasKey 方法用于配置为主键的约定。  

``` csharp
// Configure the primary key for the OfficeAssignment
modelBuilder.Entity<OfficeAssignment>()
    .HasKey(t => t.InstructorID);

// Map one-to-zero or one relationship
modelBuilder.Entity<OfficeAssignment>()
    .HasRequired(t => t.Instructor)
    .WithOptional(t => t.OfficeAssignment);
```  

## <a name="configuring-a-relationship-where-both-ends-are-required-one-to-one"></a>配置两个边界所需 （一对一） 的关系  

在大多数情况下 Entity Framework 可以推断出哪种类型是依赖项以及哪一个是一种关系中的主体。 但是，当同时关系所需或两侧均可选实体框架无法识别依赖和主体可以。 当此关系的两端都是必需的时之后，使用 WithRequiredPrincipal 或 WithRequiredDependent HasRequired 方法。 当此关系的两端都是可选的时之后，使用 WithOptionalPrincipal 或 WithOptionalDependent HasOptional 方法。  

``` csharp
// Configure the primary key for the OfficeAssignment
modelBuilder.Entity<OfficeAssignment>()
    .HasKey(t => t.InstructorID);

modelBuilder.Entity<Instructor>()
    .HasRequired(t => t.OfficeAssignment)
    .WithRequiredPrincipal(t => t.Instructor);
```  

## <a name="configuring-a-many-to-many-relationship"></a>配置多对多关系  

下面的代码配置 Course 和 Instructor 类型之间的多对多关系。 在以下示例中，默认 Code First 约定用于创建联接表。 因此 CourseInstructor 表是使用 Course_CourseID 和 Instructor_InstructorID 列创建。  

``` csharp
modelBuilder.Entity<Course>()
    .HasMany(t => t.Instructors)
    .WithMany(t => t.Courses)
```  

如果你想要在表中指定联接表名称和列的名称需要使用 Map 方法来执行其他配置。 下面的代码生成 CourseID 和 InstructorID 列 CourseInstructor 表。  

``` csharp
modelBuilder.Entity<Course>()
    .HasMany(t => t.Instructors)
    .WithMany(t => t.Courses)
    .Map(m =>
    {
        m.ToTable("CourseInstructor");
        m.MapLeftKey("CourseID");
        m.MapRightKey("InstructorID");
    });
```  

## <a name="configuring-a-relationship-with-one-navigation-property"></a>使用一个导航属性配置关系  

单向 （也称为单向） 仅之一的关系端而不是两者定义导航属性时，关系。 按照约定，代码优先始终解释为一个多单向关系。 例如，如果你想一对一关系之间 Instructor 和 OfficeAssignment，其中只有 Instructor 类型有一个导航属性，需要使用 fluent API 来配置此关系。  

``` csharp
// Configure the primary Key for the OfficeAssignment
modelBuilder.Entity<OfficeAssignment>()
    .HasKey(t => t.InstructorID);

modelBuilder.Entity<Instructor>()
    .HasRequired(t => t.OfficeAssignment)
    .WithRequiredPrincipal();
```  

## <a name="enabling-cascade-delete"></a>启用级联删除  

可以通过使用 WillCascadeOnDelete 方法在关系上配置级联删除。 如果依赖实体的外键不可为 null，则第一个代码设置级联删除的关系。 如果依赖实体的外键是可以为 null，Code First 不会设置级联删除的关系，并且当删除主体将设置外键为 null。  

可通过删除以下级联删除约定：  

modelBuilder.Conventions.Remove\<OneToManyCascadeDeleteConvention\>（)  
modelBuilder.Conventions.Remove\<ManyToManyCascadeDeleteConvention\>（)  

下面的代码将关系是必需的配置，然后禁用级联删除。  

``` csharp
modelBuilder.Entity<Course>()
    .HasRequired(t => t.Department)
    .WithMany(t => t.Courses)
    .HasForeignKey(d => d.DepartmentID)
    .WillCascadeOnDelete(false);
```  

## <a name="configuring-a-composite-foreign-key"></a>配置复合外键  

如果部门类型的主键包含 DepartmentID 和 Name 属性，则配置为主键部门和课程类型上的外键，如下所示：  

``` csharp
// Composite primary key
modelBuilder.Entity<Department>()
.HasKey(d => new { d.DepartmentID, d.Name });

// Composite foreign key
modelBuilder.Entity<Course>()  
    .HasRequired(c => c.Department)  
    .WithMany(d => d.Courses)
    .HasForeignKey(d => new { d.DepartmentID, d.DepartmentName });
```  

## <a name="renaming-a-foreign-key-that-is-not-defined-in-the-model"></a>重命名模型中未定义的外键  

如果您选择不在 CLR 类型上定义外键，但想要在数据库中指定它应具有什么名称，请执行以下操作：  

``` csharp
modelBuilder.Entity<Course>()
    .HasRequired(c => c.Department)
    .WithMany(t => t.Courses)
    .Map(m => m.MapKey("ChangedDepartmentID"));
```  

## <a name="configuring-a-foreign-key-name-that-does-not-follow-the-code-first-convention"></a>配置不遵循代码的第一个约定的外键名称  

如果外的键属性课程类上调用而不是 DepartmentID SomeDepartmentID，需要执行以下操作来指定你想要作为外键的 SomeDepartmentID:  

``` csharp
modelBuilder.Entity<Course>()
         .HasRequired(c => c.Department)
         .WithMany(d => d.Courses)
         .HasForeignKey(c => c.SomeDepartmentID);
```  

## <a name="model-used-in-samples"></a>示例中所用的模型  

下面的代码优先模型用于在此页上的示例。  

``` csharp
using System.Data.Entity;
using System.Data.Entity.ModelConfiguration.Conventions;
// add a reference to System.ComponentModel.DataAnnotations DLL
using System.ComponentModel.DataAnnotations;
using System.Collections.Generic;
using System;

public class SchoolEntities : DbContext
{
    public DbSet<Course> Courses { get; set; }
    public DbSet<Department> Departments { get; set; }
    public DbSet<Instructor> Instructors { get; set; }
    public DbSet<OfficeAssignment> OfficeAssignments { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        // Configure Code First to ignore PluralizingTableName convention
        // If you keep this convention then the generated tables will have pluralized names.
        modelBuilder.Conventions.Remove<PluralizingTableNameConvention>();
    }
}

public class Department
{
    public Department()
    {
        this.Courses = new HashSet<Course>();
    }
    // Primary key
    public int DepartmentID { get; set; }
    public string Name { get; set; }
    public decimal Budget { get; set; }
    public System.DateTime StartDate { get; set; }
    public int? Administrator { get; set; }

    // Navigation property
    public virtual ICollection<Course> Courses { get; private set; }
}

public class Course
{
    public Course()
    {
        this.Instructors = new HashSet<Instructor>();
    }
    // Primary key
    public int CourseID { get; set; }

    public string Title { get; set; }
    public int Credits { get; set; }

    // Foreign key
    public int DepartmentID { get; set; }

    // Navigation properties
    public virtual Department Department { get; set; }
    public virtual ICollection<Instructor> Instructors { get; private set; }
}

public partial class OnlineCourse : Course
{
    public string URL { get; set; }
}

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

public class Instructor
{
    public Instructor()
    {
        this.Courses = new List<Course>();
    }

    // Primary key
    public int InstructorID { get; set; }
    public string LastName { get; set; }
    public string FirstName { get; set; }
    public System.DateTime HireDate { get; set; }

    // Navigation properties
    public virtual ICollection<Course> Courses { get; private set; }
}

public class OfficeAssignment
{
    // Specifying InstructorID as a primary
    [Key()]
    public Int32 InstructorID { get; set; }

    public string Location { get; set; }

    // When Entity Framework sees Timestamp attribute
    // it configures ConcurrencyCheck and DatabaseGeneratedPattern=Computed.
    [Timestamp]
    public Byte[] Timestamp { get; set; }

    // Navigation property
    public virtual Instructor Instructor { get; set; }
}
```  
