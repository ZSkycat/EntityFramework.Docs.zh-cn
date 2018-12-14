---
title: 索引的 EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 85b92003-b692-417d-ac1d-76d40dce664b
uid: core/modeling/indexes
ms.openlocfilehash: 87fe893243377e3ab83d419ae9bedf813ca50c3f
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42995475"
---
# <a name="indexes"></a>索引

索引是在许多数据存储之间的一个常见概念。 而其数据存储区中的实现可能会有所不同，它们用于使基于列 （或一组列） 上查找更高效。

## <a name="conventions"></a>约定

按照约定，用作外键每个属性 （或组的属性） 中创建的索引。

## <a name="data-annotations"></a>数据注释

不使用数据批注创建索引。

## <a name="fluent-api"></a>Fluent API

Fluent API 可用于根据单个属性中指定的索引。 默认情况下，索引是非唯一的。

<!-- [!code-csharp[Main](samples/core/Modeling/FluentAPI/Samples/Index.cs?highlight=7,8)] -->
``` csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>()
            .HasIndex(b => b.Url);
    }
}

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
}
```

此外可以指定索引应是唯一的这意味着任何两个实体可以具有给定属性的相同值。

<!-- [!code-csharp[Main](samples/core/Modeling/FluentAPI/Samples/IndexUnique.cs?highlight=3)] -->
``` csharp
        modelBuilder.Entity<Blog>()
            .HasIndex(b => b.Url)
            .IsUnique();
```

此外可以通过多个列指定一个索引。

<!-- [!code-csharp[Main](samples/core/Modeling/FluentAPI/Samples/IndexComposite.cs?highlight=7,8)] -->
``` csharp
class MyContext : DbContext
{
    public DbSet<Person> People { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Person>()
            .HasIndex(p => new { p.FirstName, p.LastName });
    }
}

public class Person
{
    public int PersonId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
}
```

> [!TIP]  
> 没有每个非重复的属性集只有一个索引。 如果 Fluent API 用于在一组已定义了索引，不论是通过约定或以前配置的属性上配置索引然后你将会更改该索引定义。 这是很有用，如果你想要进一步配置按约定创建的索引。
