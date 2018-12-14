---
title: 卷影属性-EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 75369266-d2b9-4416-b118-ed238f81f599
uid: core/modeling/shadow-properties
ms.openlocfilehash: b7b7b10642564dfa3dbc05755188b5b5c63e0d03
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42993797"
---
# <a name="shadow-properties"></a>隐藏属性

卷影属性是指你.NET 实体类中未定义但 EF Core模型中该实体类型定义。 完全在更改跟踪器中维护的值和这些属性的状态。

不应在映射的实体类型公开数据库中没有数据时，卷影属性非常有用。 它们通常用于外键属性，其中两个实体之间的关系表示通过在数据库中的外键值，但此关系管理使用实体类型之间的导航属性的实体类型上。

阴影属性值可以获取并通过更改`ChangeTracker`API。

``` csharp
   context.Entry(myBlog).Property("LastUpdated").CurrentValue = DateTime.Now;
```

可以通过 LINQ 查询中引用隐藏属性`EF.Property`静态方法。

``` csharp
var blogs = context.Blogs
    .OrderBy(b => EF.Property<DateTime>(b, "LastUpdated"));
```

## <a name="conventions"></a>约定

当发现一种关系，但依赖实体类中找到任何外键属性时，可以通过约定创建隐藏属性。 在这种情况下，将引入了卷影的外键属性。 卷影外键属性将被命名为`<navigation property name><principal key property name>`（指向主体实体，依赖实体的导航用于命名）。 如果主体键属性名称包含导航属性的名称，则名称只是将`<principal key property name>`。 如果依赖实体上没有任何导航属性，则主体类型名称使用在其原位置。

例如，以下代码列表将导致`BlogId`影子属性引入到`Post`实体。

<!-- [!code-csharp[Main](samples/core/Modeling/Conventions/Samples/ShadowForeignKey.cs)] -->
``` csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
}

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

    public List<Post> Posts { get; set; }
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    public Blog Blog { get; set; }
}
```

## <a name="data-annotations"></a>数据注释

使用数据注释，可以创建隐藏属性。

## <a name="fluent-api"></a>Fluent API

可以使用 Fluent API 配置卷影属性。 一旦调用的字符串重载`Property`可以链接的任何其他属性会配置调用。

如果名称提供给`Property`方法与现有属性 （的影子属性或一个实体类中定义） 的名称相匹配，则代码将配置该现有属性，而不是引入的新的影子属性。

<!-- [!code-csharp[Main](samples/core/Modeling/FluentAPI/Samples/ShadowProperty.cs?highlight=7,8)] -->
``` csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>()
            .Property<DateTime>("LastUpdated");
    }
}

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
}
```
