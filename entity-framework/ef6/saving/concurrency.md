---
title: 处理并发冲突-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 2318e4d3-f561-4720-bbc3-921556806476
ms.openlocfilehash: 81ae186201fdfac331b1d4e7836b222545fe78b5
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489149"
---
# <a name="handling-concurrency-conflicts"></a>处理并发冲突
乐观并发涉及到乐观地尝试将实体保存到数据库中的实体不更改那里的数据希望加载。 事实证明，如果数据已更改，则会引发异常并尝试再次保存之前，必须解决该冲突。 本主题介绍如何处理实体框架中的此类异常。 本主题所介绍的方法同样适用于查询使用 Code First 和 EF 设计器创建的模型。  

此帖子不是开放式并发的完整讨论的适当位置。 以下各节假定并发解决方法的一些知识和显示模式的常见任务。  

许多这些模式都使用的讨论中的主题[属性值使用方面](~/ef6/saving/change-tracking/property-values.md)。  

使用独立关联 （其中外键不映射到你的实体中的属性） 时解决并发问题是比使用外键关联时要困难得多。 因此如果您打算执行并发应用程序中的解决方法建议始终将外键映射到你的实体。 下面的所有示例均都假定使用外键关联。  

尝试保存使用外键关联的实体时检测到开放式并发异常时，将通过 SaveChanges 引发 DbUpdateConcurrencyException。  

## <a name="resolving-optimistic-concurrency-exceptions-with-reload-database-wins"></a>解决重新加载 （数据库优先） 的开放式并发异常  

Reload 方法可以用于使用现在在数据库中的值覆盖该实体的当前值。 实体然后通常送回给中某种形式的用户，他们必须尝试再次进行更改并重新保存。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);
    blog.Name = "The New ADO.NET Blog";

    bool saveFailed;
    do
    {
        saveFailed = false;

        try
        {
            context.SaveChanges();
        }
        catch (DbUpdateConcurrencyException ex)
        {
            saveFailed = true;

            // Update the values of the entity that failed to save from the store
            ex.Entries.Single().Reload();
        }

    } while (saveFailed);
}
```  

模拟并发异常的好方法是在 SaveChanges 调用处设置断点，然后修改正在使用另一个 SQL Management Studio 等工具在数据库中保存的实体。 您还可以插入行之前 SaveChanges 以直接使用 SqlCommand 更新数据库。 例如：  

``` csharp
context.Database.SqlCommand(
    "UPDATE dbo.Blogs SET Name = 'Another Name' WHERE BlogId = 1");
```  

DbUpdateConcurrencyException 条目方法返回实体的更新失败的 DbEntityEntry 的实例。 （此属性当前始终返回单个值的并发问题。 它可能返回多个值的一般性的更新异常。）在某些情况下的替代方法可能是以获得可能需要从数据库重新加载的所有实体的条目并调用重新加载的每个。  

## <a name="resolving-optimistic-concurrency-exceptions-as-client-wins"></a>作为客户端 wins 解决开放式并发异常  

上面的示例使用重新加载有时称为数据库优先或存储优先因为的数据库中的值将覆盖该实体中的值。 有时您可能希望执行相反的操作并使用该实体中的当前值覆盖该数据库中的值。 这有时称为客户端 wins，可以通过获取当前的数据库值并将它们设置为实体的原始值。 (请参阅[属性值使用方面](~/ef6/saving/change-tracking/property-values.md)有关当前和原始值的信息。)例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);
    blog.Name = "The New ADO.NET Blog";

    bool saveFailed;
    do
    {
        saveFailed = false;
        try
        {
            context.SaveChanges();
        }
        catch (DbUpdateConcurrencyException ex)
        {
            saveFailed = true;

            // Update original values from the database
            var entry = ex.Entries.Single();
            entry.OriginalValues.SetValues(entry.GetDatabaseValues());
        }

    } while (saveFailed);
}
```  

## <a name="custom-resolution-of-optimistic-concurrency-exceptions"></a>自定义解析的开放式并发异常  

有时你可能想要将数据库中的当前值与实体中的当前值相结合。 这通常需要一些自定义逻辑或用户交互。 例如，可能会给用户，其中包含当前值，在数据库中的值呈现窗体和一组默认的解析的值。 然后，用户将编辑根据需要解析的值并会保存到数据库中获取这些解析的值。 这可以使用 DbPropertyValues 对象从当前值和 GetDatabaseValues 返回实体的条目。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);
    blog.Name = "The New ADO.NET Blog";

    bool saveFailed;
    do
    {
        saveFailed = false;
        try
        {
            context.SaveChanges();
        }
        catch (DbUpdateConcurrencyException ex)
        {
            saveFailed = true;

            // Get the current entity values and the values in the database
            var entry = ex.Entries.Single();
            var currentValues = entry.CurrentValues;
            var databaseValues = entry.GetDatabaseValues();

            // Choose an initial set of resolved values. In this case we
            // make the default be the values currently in the database.
            var resolvedValues = databaseValues.Clone();

            // Have the user choose what the resolved values should be
            HaveUserResolveConcurrency(currentValues, databaseValues, resolvedValues);

            // Update the original values with the database values and
            // the current values with whatever the user choose.
            entry.OriginalValues.SetValues(databaseValues);
            entry.CurrentValues.SetValues(resolvedValues);
        }
    } while (saveFailed);
}

public void HaveUserResolveConcurrency(DbPropertyValues currentValues,
                                       DbPropertyValues databaseValues,
                                       DbPropertyValues resolvedValues)
{
    // Show the current, database, and resolved values to the user and have
    // them edit the resolved values to get the correct resolution.
}
```  

## <a name="custom-resolution-of-optimistic-concurrency-exceptions-using-objects"></a>使用对象的开放式并发异常的自定义解决  

上面的代码使用 DbPropertyValues 实例的当前传递、 数据库和解析的值。 有时可能会更轻松地使用此实体类型的实例。 这可以使用 DbPropertyValues ToObject 和 SetValues 方法。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);
    blog.Name = "The New ADO.NET Blog";

    bool saveFailed;
    do
    {
        saveFailed = false;
        try
        {
            context.SaveChanges();
        }
        catch (DbUpdateConcurrencyException ex)
        {
            saveFailed = true;

            // Get the current entity values and the values in the database
            // as instances of the entity type
            var entry = ex.Entries.Single();
            var databaseValues = entry.GetDatabaseValues();
            var databaseValuesAsBlog = (Blog)databaseValues.ToObject();

            // Choose an initial set of resolved values. In this case we
            // make the default be the values currently in the database.
            var resolvedValuesAsBlog = (Blog)databaseValues.ToObject();

            // Have the user choose what the resolved values should be
            HaveUserResolveConcurrency((Blog)entry.Entity,
                                       databaseValuesAsBlog,
                                       resolvedValuesAsBlog);

            // Update the original values with the database values and
            // the current values with whatever the user choose.
            entry.OriginalValues.SetValues(databaseValues);
            entry.CurrentValues.SetValues(resolvedValuesAsBlog);
        }

    } while (saveFailed);
}

public void HaveUserResolveConcurrency(Blog entity,
                                       Blog databaseValues,
                                       Blog resolvedValues)
{
    // Show the current, database, and resolved values to the user and have
    // them update the resolved values to get the correct resolution.
}
```  
