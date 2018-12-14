---
title: 查询和查找实体 - EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 65bb3db2-2226-44af-8864-caa575cf1b46
ms.openlocfilehash: 29a86817e250a2f53ecaa73e8fa4bf93452f0497
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489786"
---
# <a name="querying-and-finding-entities"></a>查询和查找实体
本主题介绍使用实体框架查询数据的各种方法，包括 LINQ 和 Find 方法。 本主题所介绍的方法同样适用于查询使用 Code First 和 EF 设计器创建的模型。  

## <a name="finding-entities-using-a-query"></a>使用查询查找实体  

DbSet 和 IDbSet 可实现 IQueryable，因此可用作针对数据库编写 LINQ 查询的起点。 在此不适合深入讨论 LINQ，但以下提供了几个简单示例：  

``` csharp
using (var context = new BloggingContext())
{
    // Query for all blogs with names starting with B
    var blogs = from b in context.Blogs
                   where b.Name.StartsWith("B")
                   select b;

    // Query for the Blog named ADO.NET Blog
    var blog = context.Blogs
                    .Where(b => b.Name == "ADO.NET Blog")
                    .FirstOrDefault();
}
```  

请注意，DbSet 和 IDbSet 始终针对数据库创建查询，并且始终会涉及数据库往返，即使返回的实体已存在于上下文中。 出现以下情况时，会针对数据库执行查询：  

- 查询由 foreach (C#) 或 For Each (Visual Basic) 语句枚举。  
- 查询由集合操作（如 [ToArray](https://msdn.microsoft.com/library/bb298736)、[ToDictionary](https://msdn.microsoft.com/library/system.linq.enumerable.todictionary) 或 [ToList](https://msdn.microsoft.com/library/bb342261)）枚举。  
- 在查询最外部指定了 LINQ 运算符，例如 [First](https://msdn.microsoft.com/library/bb291976) 或 [Any](https://msdn.microsoft.com/library/bb337697)。  
- 调用了以下方法：DbSet 上的 [Load](https://msdn.microsoft.com/library/system.data.entity.dbextensions.load) 扩展方法、[DbEntityEntry.Reload](https://msdn.microsoft.com/library/system.data.entity.infrastructure.dbentityentry.reload.aspx) 和 Database.ExecuteSqlCommand。  

从数据库返回结果时，上下文中不存在的对象会附加到上下文。 如果某个对象已存在于上下文中，则不返回现有对象（不会使用数据库值覆盖该对象的属性在对应项中的当前值和原始值）。  

执行查询时，结果集中不回返回已添加到上下文但尚未保存到数据库的实体。 若要获取上下文中的数据，请参阅[本地数据](~/ef6/querying/local-data.md)。  

如果查询未从数据库返回任何行，则结果将是空集合，而不是 NULL。  

## <a name="finding-entities-using-primary-keys"></a>使用主键查找实体  

DbSet 上的 Find 方法使用主键值来尝试查找由上下文跟踪的实体。 如果在上下文中未找到实体，则会向数据库发送查询以在其中查找实体。 如果未在上下文中或数据库中找到实体，则返回 NULL。  

Find 与使用查询有两个重要区别：  

- 只有未在上下文中找到具有给定键的实体时，才会往返数据库。  
- Find 会返回处于“已添加”状态的实体。 即 Find 会返回已添加到上下文但尚未保存到数据库的实体。  
### <a name="finding-an-entity-by-primary-key"></a>通过主键查找实体  

以下代码演示了 Find 的部分用法：  

``` csharp
using (var context = new BloggingContext())
{
    // Will hit the database
    var blog = context.Blogs.Find(3);

    // Will return the same instance without hitting the database
    var blogAgain = context.Blogs.Find(3);

    context.Blogs.Add(new Blog { Id = -1 });

    // Will find the new blog even though it does not exist in the database
    var newBlog = context.Blogs.Find(-1);

    // Will find a User which has a string primary key
    var user = context.Users.Find("johndoe1987");
}
```  

### <a name="finding-an-entity-by-composite-primary-key"></a>通过组合主键查找实体  

实体框架允许实体具有组合键，即由多个属性组成的键。 例如，用户可具有表示特定博客的用户设置的 BlogSettings 实体。 因为用户的每个博客只能拥有一个 BlogSettings，所以可以选择将 BlogId 和用户名结合作为 BlogSettings 的主键。 以下代码尝试查找 BlogId = 3，用户名 =“johndoe1987”的 BlogSettings：  

``` csharp  
using (var context = new BloggingContext())
{
    var settings = context.BlogSettings.Find(3, "johndoe1987");
}
```  

请注意，具有组合键时，需使用 ColumnAttribute 或 Fluent API 来指定组合键属性的顺序。 指定构成键的值时，对 Find 的调用必须使用此顺序。  
