---
title: 使用属性值的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: e3278b4b-9378-4fdb-923d-f64d80aaae70
ms.openlocfilehash: 97902021a671dea9854a365dc2f10eaecb9e5ab8
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45488824"
---
# <a name="working-with-property-values"></a>使用属性值
大多数情况下 Entity Framework 将负责跟踪状态、 原始值和当前值的实体实例的属性。 但是，可能有某些情况下-例如断开连接的场景-你想要查看或操作的属性信息 EF 拥有。 本主题所介绍的方法同样适用于查询使用 Code First 和 EF 设计器创建的模型。  

实体框架会跟踪的跟踪实体的每个属性的两个值。 当前值是，顾名思义，实体中属性的当前值。 如果实体已从数据库中查询或附加到上下文，该属性具有值的原始值。  

有两种常规机制，用于处理属性值：  

- 可以使用属性方法以强类型化方式获取的单个属性的值。  
- 实体的所有属性的值可以读入 DbPropertyValues 对象。 DbPropertyValues 然后充当类似于字典的对象，以允许读取和设置属性值。 可以从另一个 DbPropertyValues 对象中的值或其他某个对象，如实体或简单的数据传输对象 (DTO) 的另一个副本中的值设置 DbPropertyValues 对象中的值。  

以下部分说明了使用这两个以上的机制的示例。  

## <a name="getting-and-setting-the-current-or-original-value-of-an-individual-property"></a>获取和设置的个别属性的当前或原始值  

下面的示例说明如何可以读取，然后将设置为新值的属性的当前值：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(3);

    // Read the current value of the Name property
    string currentName1 = context.Entry(blog).Property(u => u.Name).CurrentValue;

    // Set the Name property to a new value
    context.Entry(name).Property(u => u.Name).CurrentValue = "My Fancy Blog";

    // Read the current value of the Name property using a string for the property name
    object currentName2 = context.Entry(blog).Property("Name").CurrentValue;

    // Set the Name property to a new value using a string for the property name
    context.Entry(blog).Property("Name").CurrentValue = "My Boring Blog";
}
```  

使用而不是 CurrentValue 属性的 OriginalValue 属性来读取或设置原始值。  

请注意，返回的值的类型为"对象"用于指定属性名称的字符串。 但是，强类型返回的值，如果使用了 lambda 表达式。  

设置此类的属性值将仅将该属性标记为已修改的新值是否不同于旧值。  

以这种方式设置属性值可以自动检测到更改即使 AutoDetectChanges 处于关闭状态。  

## <a name="getting-and-setting-the-current-value-of-an-unmapped-property"></a>获取和设置未映射属性的当前值  

此外可以阅读不映射到数据库的属性的当前值。 未映射属性的一个示例可能是博客上的 RssLink 属性。 此值可能根据 BlogId，计算，因此不需要存储在数据库中。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);
    // Read the current value of an unmapped property
    var rssLink = context.Entry(blog).Property(p => p.RssLink).CurrentValue;

    // Use a string to specify the property name
    var rssLinkAgain = context.Entry(blog).Property("RssLink").CurrentValue;
}
```  

如果该属性公开 setter，还可以设置的当前值。  

执行未映射属性的实体框架验证时，读取未映射的属性的值非常有用。 出于同一原因可以读取和设置的属性不当前正在由上下文跟踪的实体的当前值。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    // Create an entity that is not being tracked
    var blog = new Blog { Name = "ADO.NET Blog" };

    // Read and set the current value of Name as before
    var currentName1 = context.Entry(blog).Property(u => u.Name).CurrentValue;
    context.Entry(blog).Property(u => u.Name).CurrentValue = "My Fancy Blog";
    var currentName2 = context.Entry(blog).Property("Name").CurrentValue;
    context.Entry(blog).Property("Name").CurrentValue = "My Boring Blog";
}
```  

请注意，原始值不能用于未映射的属性或未正在由上下文跟踪的实体的属性。  

## <a name="checking-whether-a-property-is-marked-as-modified"></a>检查属性标记为已修改  

下面的示例演示如何检查的个别属性标记为已修改：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);

    var nameIsModified1 = context.Entry(blog).Property(u => u.Name).IsModified;

    // Use a string for the property name
    var nameIsModified2 = context.Entry(blog).Property("Name").IsModified;
}
```  

当调用 SaveChanges 时，已修改属性的值是作为更新发送到数据库中。  

##  <a name="marking-a-property-as-modified"></a>将标记为已修改的属性  

下面的示例演示如何强制执行标记为已修改的个别属性：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);

    context.Entry(blog).Property(u => u.Name).IsModified = true;

    // Use a string for the property name
    context.Entry(blog).Property("Name").IsModified = true;
}
```  

将属性标记为已修改强制更新为将发送到该属性的数据库，当调用 SaveChanges 时，即使该属性的当前值等同于其原始值。  

不是目前无法重置后标记为已修改未修改的单个属性。 这是我们计划在将来的版本中支持。  

## <a name="reading-current-original-and-database-values-for-all-properties-of-an-entity"></a>读取当前、 原始值和所有属性的实体的数据库值  

下面的示例演示如何读取的当前值、 原始值，以及数据库中的所有映射属性的实体的数据库的实际值。  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);

    // Make a modification to Name in the tracked entity
    blog.Name = "My Cool Blog";

    // Make a modification to Name in the database
    context.Database.SqlCommand("update dbo.Blogs set Name = 'My Boring Blog' where Id = 1");

    // Print out current, original, and database values
    Console.WriteLine("Current values:");
    PrintValues(context.Entry(blog).CurrentValues);

    Console.WriteLine("\nOriginal values:");
    PrintValues(context.Entry(blog).OriginalValues);

    Console.WriteLine("\nDatabase values:");
    PrintValues(context.Entry(blog).GetDatabaseValues());
}

public static void PrintValues(DbPropertyValues values)
{
    foreach (var propertyName in values.PropertyNames)
    {
        Console.WriteLine("Property {0} has value {1}",
                          propertyName, values[propertyName]);
    }
}
```  

当前值是实体的当前包含的属性的值。 原始值是该实体已查询时从数据库读取的值。 数据库值是当前数据库中存储的值。 在数据库中的值可能已更改，因为如时并发编辑，以另一个用户已发出对数据库进行了查询实体时，获取数据库值很有用。  

## <a name="setting-current-or-original-values-from-another-object"></a>从另一个对象设置当前或原始值  

可以通过从另一个对象复制的值更新被跟踪实体的当前或原始值。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);
    var coolBlog = new Blog { Id = 1, Name = "My Cool Blog" };
    var boringBlog = new BlogDto { Id = 1, Name = "My Boring Blog" };

    // Change the current and original values by copying the values from other objects
    var entry = context.Entry(blog);
    entry.CurrentValues.SetValues(coolBlog);
    entry.OriginalValues.SetValues(boringBlog);

    // Print out current and original values
    Console.WriteLine("Current values:");
    PrintValues(entry.CurrentValues);

    Console.WriteLine("\nOriginal values:");
    PrintValues(entry.OriginalValues);
}

public class BlogDto
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```  

运行上面的代码将打印出：  

```  
Current values:
Property Id has value 1
Property Name has value My Cool Blog

Original values:
Property Id has value 1
Property Name has value My Boring Blog
```  

使用从服务调用或 n 层应用程序中的客户端获取的值更新实体时，有时会使用此方法。 请注意，使用对象不必是实体类型相同的只要它具有名称匹配的实体的属性。 在上面的示例中，BlogDTO 实例用于更新原始值。  

请注意为不同的值时从另一个对象复制设置的属性标记为已修改。  

## <a name="setting-current-or-original-values-from-a-dictionary"></a>从字典中设置当前或原始值  

可以通过将值从字典或某些其他数据结构中复制更新被跟踪实体的当前或原始值。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);

    var newValues = new Dictionary\<string, object>
    {
        { "Name", "The New ADO.NET Blog" },
        { "Url", "blogs.msdn.com/adonet" },
    };

    var currentValues = context.Entry(blog).CurrentValues;

    foreach (var propertyName in newValues.Keys)
    {
        currentValues[propertyName] = newValues[propertyName];
    }

    PrintValues(currentValues);
}
```  

使用引入了属性而不是当前值属性来设置原始值。  

## <a name="setting-current-or-original-values-from-a-dictionary-using-property"></a>使用属性字典中设置当前或原始值  

使用当前值或引入了如上所示的替代方法是使用属性方法以设置每个属性的值。 当您需要设置复杂属性的值时，这可能是更可取。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var user = context.Users.Find("johndoe1987");

    var newValues = new Dictionary\<string, object>
    {
        { "Name", "John Doe" },
        { "Location.City", "Redmond" },
        { "Location.State.Name", "Washington" },
        { "Location.State.Code", "WA" },
    };

    var entry = context.Entry(user);

    foreach (var propertyName in newValues.Keys)
    {
        entry.Property(propertyName).CurrentValue = newValues[propertyName];
    }
}
```  

在上面的复杂属性的示例使用访问包含点的名称。 有关访问复杂属性的其他方法请参阅后面本主题专门针对复杂属性的两个部分。  

## <a name="creating-a-cloned-object-containing-current-original-or-database-values"></a>创建一个克隆的对象包含当前、 原始路径或数据库值  

从当前值，引入了，返回的 DbPropertyValues 对象或 GetDatabaseValues 可用于创建实体的克隆。 此克隆将包含从 DbPropertyValues 对象用来创建它的属性值。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);

    var clonedBlog = context.Entry(blog).GetDatabaseValues().ToObject();
}
```  

请注意，返回的对象不是实体，并且未被跟踪上下文。 返回的对象还没有设置为其他对象的任何关系。  

克隆的对象可用于解决与并发更新到数据库，尤其是涉及数据绑定到特定类型的对象的用户界面正在使用的位置相关的问题。  

## <a name="getting-and-setting-the-current-or-original-values-of-complex-properties"></a>获取和设置的复杂属性的当前或原始值  

读取和使用属性方法，就像它可以是基元属性集，可以是整个复杂对象的值。 此外，可以向下钻取到复杂的对象，该对象或甚至嵌套的对象的读取或设置属性。 下面是一些可能的恶意活动：  

``` csharp
using (var context = new BloggingContext())
{
    var user = context.Users.Find("johndoe1987");

    // Get the Location complex object
    var location = context.Entry(user)
                       .Property(u => u.Location)
                       .CurrentValue;

    // Get the nested State complex object using chained calls
    var state1 = context.Entry(user)
                     .ComplexProperty(u => u.Location)
                     .Property(l => l.State)
                     .CurrentValue;

    // Get the nested State complex object using a single lambda expression
    var state2 = context.Entry(user)
                     .Property(u => u.Location.State)
                     .CurrentValue;

    // Get the nested State complex object using a dotted string
    var state3 = context.Entry(user)
                     .Property("Location.State")
                     .CurrentValue;

    // Get the value of the Name property on the nested State complex object using chained calls
    var name1 = context.Entry(user)
                       .ComplexProperty(u => u.Location)
                       .ComplexProperty(l => l.State)
                       .Property(s => s.Name)
                       .CurrentValue;

    // Get the value of the Name property on the nested State complex object using a single lambda expression
    var name2 = context.Entry(user)
                       .Property(u => u.Location.State.Name)
                       .CurrentValue;

    // Get the value of the Name property on the nested State complex object using a dotted string
    var name3 = context.Entry(user)
                       .Property("Location.State.Name")
                       .CurrentValue;
}
```  

使用而不是 CurrentValue 属性的 OriginalValue 属性获取或设置原始值。  

请注意该属性或 ComplexProperty 方法可用于访问复杂属性。 但是，如果你想要向下钻取到具有其他属性的复杂对象或调用的 ComplexProperty，则必须使用 ComplexProperty 方法。  

## <a name="using-dbpropertyvalues-to-access-complex-properties"></a>使用 DbPropertyValues 访问复杂属性  

当您使用当前值，引入了或 GetDatabaseValues 来获取所有当前原始，或数据库实体值，为嵌套 DbPropertyValues 对象返回的任何复杂属性的值。 这些嵌套对象可以然后使用来获取复杂对象的值。 例如，以下方法将打印出的所有属性，包括任何复杂属性和嵌套的复杂属性的值的值。  

``` csharp
public static void WritePropertyValues(string parentPropertyName, DbPropertyValues propertyValues)
{
    foreach (var propertyName in propertyValues.PropertyNames)
    {
        var nestedValues = propertyValues[propertyName] as DbPropertyValues;
        if (nestedValues != null)
        {
            WritePropertyValues(parentPropertyName + propertyName + ".", nestedValues);
        }
        else
        {
            Console.WriteLine("Property {0}{1} has value {2}",
                              parentPropertyName, propertyName,
                              propertyValues[propertyName]);
        }
    }
}
```  

打印出所有当前属性值的方法将调用如下：  

``` csharp
using (var context = new BloggingContext())
{
    var user = context.Users.Find("johndoe1987");

    WritePropertyValues("", context.Entry(user).CurrentValues);
}
```  
