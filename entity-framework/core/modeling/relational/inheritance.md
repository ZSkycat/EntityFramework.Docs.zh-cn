---
title: 继承 （关系数据库） 的 EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 9a7c5488-aaf4-4b40-b1ff-f435ff30f6ec
uid: core/modeling/relational/inheritance
ms.openlocfilehash: 019893ec8268ef9e59d581799a13d63610c80616
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42996317"
---
# <a name="inheritance-relational-database"></a>继承 （关系数据库）

> [!NOTE]  
> 一般而言，本部分中的配置适用于关系数据库。 安装关系数据库提供程序时，此处显示的扩展方法将变为可用（原因在于共享的 Microsoft.EntityFrameworkCore.Relational 包）。

EF 模型中的继承用于控制如何在数据库中表示实体类中的继承。

> [!NOTE]  
> 目前，每个层次结构一个表的 (TPH) 模式在 EF Core 实现。 每种具体的类型一个表 (TPC) 尚不可用和其他常用模式等每种类型一个表 (TPT)。

## <a name="conventions"></a>约定

按照约定，将使用每个层次结构一张表 (TPH) 模式映射继承。 TPH 使用单个表来存储所有类型的数据层次结构中。 鉴别器列用于标识每行表示的类型。

在模型中显式包含两个或多个继承的类型时，EF Core 将仅安装程序继承 (请参阅[继承](../inheritance.md)有关详细信息)。

下面是一个示例，演示一个简单的继承方案和使用 TPH 模式对关系数据库表中存储的数据。 *鉴别器*列标识哪种类型的*博客*存储在每个行。

<!-- [!code-csharp[Main](samples/core/relational/Modeling/Conventions/Samples/InheritanceDbSets.cs)] -->
``` csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<RssBlog> RssBlogs { get; set; }
}

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
}

public class RssBlog : Blog
{
    public string RssUrl { get; set; }
}
```

![图像](_static/inheritance-tph-data.png)

## <a name="data-annotations"></a>数据注释

不能使用数据注释来配置继承。

## <a name="fluent-api"></a>Fluent API

可以使用 Fluent API 配置的名称和类型的鉴别器列以及用于标识层次结构中的每种类型的值。

<!-- [!code-csharp[Main](samples/core/relational/Modeling/FluentAPI/Samples/InheritanceTPHDiscriminator.cs?highlight=7,8,9,10)] -->
``` csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>()
            .HasDiscriminator<string>("blog_type")
            .HasValue<Blog>("blog_base")
            .HasValue<RssBlog>("blog_rss");
    }
}

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
}

public class RssBlog : Blog
{
    public string RssUrl { get; set; }
}
```

## <a name="configuring-the-discriminator-property"></a>配置鉴别器属性

在上面的示例中，鉴别器创建作为[阴影属性](xref:core/modeling/shadow-properties)上层次结构的基实体。 由于它是模型中的属性，可以将它配置其他属性一样。 例如，若要使用的默认值，通过约定鉴别器时设置的最大长度：

```C#
modelBuilder.Entity<Blog>()
    .Property("Discriminator")
    .HasMaxLength(200);
```

鉴别器还可以映射到你的实体中的实际 CLR 属性。 例如：
```C#
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>()
            .HasDiscriminator<string>("BlogType");
    }
}

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
    public string BlogType { get; set; }
}

public class RssBlog : Blog
{
    public string RssUrl { get; set; }
}
```

将这两个操作组合在一起，可以同时将鉴别器映射到一个真正的属性，并将其配置：
```C#
modelBuilder.Entity<Blog>(b =>
{
    b.HasDiscriminator<string>("BlogType");

    b.Property(e => e.BlogType)
        .HasMaxLength(200)
        .HasColumnName("blog_type");
});
```
