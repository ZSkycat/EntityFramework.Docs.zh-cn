---
title: 关系、 导航属性和外键的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 8a21ae73-6d9b-4b50-838a-ec1fddffcf37
ms.openlocfilehash: 46c2d11b5704ec7ae82a423ae042b87f5efe436f
ms.sourcegitcommit: 8b42045cd21f80f425a92f5e4e9dd4972a31720b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2018
ms.locfileid: "49315654"
---
# <a name="relationships-navigation-properties-and-foreign-keys"></a>关系、 导航属性和外键
本主题概述了的实体框架如何管理实体之间的关系。 它还提供有关如何将映射和操作关系的一些指导。

## <a name="relationships-in-ef"></a>在 EF 中的关系

在关系数据库中，通过外键来定义表之间关系 （也称为关联）。 外键 (FK) 是列的组合，用于建立和加强两个表中的数据之间的链接。 通常有三种类型的关系： 一对一、 一个一对多和多对多。 在一个对多关系中，表示此关系的多端表上定义外键。 多对多关系包括定义其主键由两个相关表中的外键的第三个表 （称为联接或联接表）。 在一对一关系中，此外作为外键的主键的作用，并且没有任一表没有单独的外键列。

下图显示一个对多关系中两个参与的表。 **课程**表是相关的表，因为它包含**DepartmentID**其链接到的列**部门**表。

![部门和 Course 表](~/ef6/media/database2.png)

在实体框架中，实体可以与其他实体通过关联或关系。 每个关系包含两个端点，用于描述实体类型和在该关系中的两个实体类型 （一个、 零或一个或许多） 的重数。 此关系可能会受引用约束，其中描述了关系中的哪一端为 principal role 以及哪一个是从属角色。

导航属性提供了一种导航两个实体类型之间的关联方法。 针对对象参与到其中的每个关系，各对象均可以具有导航属性。 导航属性，您可以导航和管理两个方向，返回引用对象中的关系 (在重数为两者中任何一个或零或一) 或集合 （如果重数为多个）。 您可能还选择单向导航，这种情况下定义的导航属性，在只有一个参与关系的类型和不在两者上。

建议将映射到数据库中的外键在模型中包含属性。 加入了外键属性，您就可以通过修改依赖对象的外键值来创建或更改关系。 此类关联称为外键关联。 使用断开连接的实体时，使用外键是甚至更加重要。 请注意，当使用 1 到 1 或 1 到 0...1 关系，没有任何单独的外键列，主键属性可用作外键，并始终包括在模型中。

在外键列未包括在模型中，作为独立对象进行管理的关联信息。 通过而不是外键属性的对象引用跟踪的关系。 此类型的关联称为*独立关联*。 若要修改的最常见方式*独立关联*是修改为参与到关联中每个实体生成导航属性。

可以在您的模型中选择使用一种或两种类型的关联。 但是，如果必须仅包含外键的联接表已连接的纯多对多关系，EF 将使用独立关联管理这种多对多关系。   

下图显示了使用实体框架设计器创建的概念模型。 该模型包含两个参与对多关系的实体。 这两个实体都具有导航属性。 **课程**，且是 depend 实体**DepartmentID**定义的外键属性。

![具有导航属性的部门和 Course 表](~/ef6/media/relationshipefdesigner.png)

以下代码片段显示使用 Code First 创建的相同模型。

``` csharp
public class Course
{
  public int CourseID { get; set; }
  public string Title { get; set; }
  public int Credits { get; set; }
  public int DepartmentID { get; set; }
  public virtual Department Department { get; set; }
}

public class Department
{
   public Department()
   {
     this.Course = new HashSet<Course>();
   }  
   public int DepartmentID { get; set; }
   public string Name { get; set; }
   public decimal Budget { get; set; }
   public DateTime StartDate { get; set; }
   public int? Administrator {get ; set; }
   public virtual ICollection<Course> Courses { get; set; }
}
```

## <a name="configuring-or-mapping-relationships"></a>配置或映射关系

此页的其余部分介绍了如何访问和处理使用关系数据。 设置您的模型中的关系的信息，请参阅以下页面。

-   若要配置关系中第一个代码，请参阅[数据批注](~/ef6/modeling/code-first/data-annotations.md)并[Fluent API-关系](~/ef6/modeling/code-first/fluent/relationships.md)。
-   若要配置使用实体框架设计器的关系，请参阅[与 EF 设计器之间的关系](~/ef6/modeling/designer/relationships.md)。

## <a name="creating-and-modifying-relationships"></a>创建和修改关系

在中*外键关联*，当更改关系后，具有的依赖对象的状态`EntityState.Unchanged`的状态更改为`EntityState.Modified`。 在独立关系中，更改该关系不会更新依赖对象的状态。

以下示例演示如何使用外键属性和导航属性关联相关的对象。 通过外键关联，可以使用任一方法来更改、 创建或修改关系。 使用独立关联，则不能使用外键属性。

- 通过将新值分配给外键属性，如以下示例所示。  
  ``` csharp
  course.DepartmentID = newCourse.DepartmentID;
  ```

- 下面的代码中删除通过外键设置为一种关系**null**。 请注意外, 键属性必须是可以为 null。  
  ``` csharp
  course.DepartmentID = null;
  ```

  >[!NOTE]
  > 如果引用处于已添加状态 （在此示例中，课程对象），引用导航属性将不同步新的对象的键值，直到调用 SaveChanges。 由于对象上下文在键值保存前不包含已添加对象的永久键，因此不发生同步。 如果你必须拥有新对象进行完全同步设置关系一样，使用以下 methods.* 之一

- 通过将一个新对象分配给导航属性。 以下代码将创建一门课程之间的关系和`department`。 如果这些对象附加到上下文中，`course`还将添加到`department.Courses`集合，并且对应的外键属性上`course`对象设置为部门的键属性值。  
  ``` csharp
  course.Department = department;
  ```

- 若要删除此关系，请将导航属性设置为`null`。 如果您正在使用实体框架，基于.NET 4.0，相关的端需要在加载之前将其设置为 null。 例如：   
  ``` csharp
  context.Entry(course).Reference(c => c.Department).Load();
  course.Department = null;
  ```

  从实体框架 5.0，基于.NET 4.5，可以设置为 null 关系而不加载相关的端。 此外可以设置的当前值为 null，使用以下方法。   
  ``` csharp
  context.Entry(course).Reference(c => c.Department).CurrentValue = null;
  ```

- 通过在实体集合中删除或添加对象。 例如，可以添加类型的对象`Course`到`department.Courses`集合。 此操作将创建特定之间的关系**课程**和特定`department`。 如果对象附加到上下文、 部门引用和外键属性上**课程**对象将被设置为相应`department`。  
  ``` csharp
  department.Courses.Add(newCourse);
  ```

- 通过使用`ChangeRelationshipState`方法以更改指定两个实体对象之间关系的状态。 使用 N 层应用程序时最常使用此方法和一个*独立关联*（与外键关联不能使用）。 此外，若要使用此方法必须删除直至`ObjectContext`，下面的示例中所示。  
以下示例中，在没有讲师和课程之间的多对多关系。 调用`ChangeRelationshipState`方法并传入`EntityState.Added`参数，允许`SchoolContext`知道是否两个对象之间添加关系：
  ``` csharp

  ((IObjectContextAdapter)context).ObjectContext.
    ObjectStateManager.
    ChangeRelationshipState(course, instructor, c => c.Instructor, EntityState.Added);
  ```

  请注意，如果你要更新 （而不仅仅添加） 关系，您必须添加新后删除旧的关系：

  ``` csharp
  ((IObjectContextAdapter)context).ObjectContext.
    ObjectStateManager.
    ChangeRelationshipState(course, oldInstructor, c => c.Instructor, EntityState.Deleted);
  ```

## <a name="synchronizing-the-changes-between-the-foreign-keys-and-navigation-properties"></a>同步之间的外键和导航属性的更改

使用上面所述的方法之一附加到上下文的对象的关系更改时，实体框架需要使外键、 引用以及集合保持同步。实体框架会自动管理此同步 （也称为关系修复-最多） 与代理的 POCO 实体。 有关详细信息，请参阅[使用代理](~/ef6/fundamentals/proxies.md)。

如果要使用不含代理的 POCO 实体，必须确保**DetectChanges**调用方法来同步上下文中的相关的对象。 请注意，以下 Api 会自动触发**DetectChanges**调用。

-   `DbSet.Add`
-   `DbSet.AddRange`
-   `DbSet.Remove`
-   `DbSet.RemoveRange`
-   `DbSet.Find`
-   `DbSet.Local`
-   `DbContext.SaveChanges`
-   `DbSet.Attach`
-   `DbContext.GetValidationErrors`
-   `DbContext.Entry`
-   `DbChangeTracker.Entries`
-   对执行 LINQ 查询 `DbSet`

## <a name="loading-related-objects"></a>加载相关对象

最常使用的实体框架中使用的导航属性加载通过定义的关联返回的实体相关的实体。 有关详细信息，请参阅[加载相关对象](~/ef6/querying/related-data.md)。

> [!NOTE]
> 在外键关联中，加载依赖对象的相关端时，将会基于当前位于内存中的依赖对象的外键值加载相关对象：

``` csharp
    // Get the course where currently DepartmentID = 2.
    Course course2 = context.Courses.First(c=>c.DepartmentID == 2);

    // Use DepartmentID foreign key property
    // to change the association.
    course2.DepartmentID = 3;

    // Load the related Department where DepartmentID = 3
    context.Entry(course).Reference(c => c.Department).Load();
```

在独立关联中，基于当前数据库中的外键值查询依赖对象的相关端。 但是，如果关系已修改，并对它加载到对象上下文，实体框架中的不同主体对象的依赖对象点上的引用属性将尝试创建的关系为它定义在客户端上。

## <a name="managing-concurrency"></a>管理并发

在外键和独立关联中，并发检查基于实体键和其他模型中定义的实体属性。 当使用 EF 设计器创建模型时，设置`ConcurrencyMode`属性为**修复**以指定应针对并发检查属性。 在使用 Code First 定义的模型，使用`ConcurrencyCheck`上想要检查并发的属性批注。 使用 Code First 时还可以使用`TimeStamp`批注来指定应针对并发检查属性。 您可以只有一个时间戳属性中给定的类。 代码首先将此属性映射到数据库中的不可以为 null 的字段。

我们建议使用参与并发检查和解析实体时始终使用外键关联。

有关详细信息，请参阅[处理并发冲突](~/ef6/saving/concurrency.md)。

## <a name="working-with-overlapping-keys"></a>使用重叠键

重叠键是指键中某些属性亦是实体中其他键的一部分的那些组合键。 在独立关联中不能包含重叠键。 若要更改包含重叠键的外键关联，我们建议您修改外键值而不要使用对象引用。
