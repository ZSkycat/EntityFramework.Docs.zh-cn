---
title: 使用构造函数的 EF Core 的实体类型
author: ajcvickers
ms.date: 02/23/2018
ms.assetid: 420AFFE7-B709-4A68-9149-F06F8746FB33
uid: core/modeling/constructors
ms.openlocfilehash: 1b36197465fb9a6571a306d36eb1e9d885a5399e
ms.sourcegitcommit: 0cef7d448e1e47bdb333002e2254ed42d57b45b6
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/29/2018
ms.locfileid: "43152460"
---
# <a name="entity-types-with-constructors"></a>使用构造函数的实体类型

> [!NOTE]  
> 此功能是在 EF Core 2.1 的新增功能。

从开始 EF Core 2.1，它则现在可以定义参数的构造函数，并让 EF Core 创建实体的实例时调用此构造函数。 构造函数参数可以绑定到映射的属性，或到各种类型的服务，以促进行为 like 延迟加载。

> [!NOTE]  
> 截至 EF Core 2.1，按照约定将为所有构造函数绑定。 配置特定的构造函数使用计划在将来的版本。

## <a name="binding-to-mapped-properties"></a>绑定到映射的属性

请考虑一个典型博客/文章模型：

``` csharp
public class Blog
{
    public int Id { get; set; }

    public string Name { get; set; }
    public string Author { get; set; }

    public ICollection<Post> Posts { get; } = new List<Post>();
}

public class Post
{
    public int Id { get; set; }

    public string Title { get; set; }
    public string Content { get; set; }
    public DateTime PostedOn { get; set; }

    public Blog Blog { get; set; }
}
```

当 EF Core 创建这些类型的实例时，如的结果的查询，它将首先调用默认的无参数构造函数，然后设置每个属性的值从数据库。 但是，如果 EF Core 查找的参数化构造函数参数名称和类型相匹配的映射属性，则它将改为调用这些属性具有值的参数化构造函数，然后将未显式设置每个属性。 例如：

``` csharp
public class Blog
{
    public Blog(int id, string name, string author)
    {
        Id = id;
        Name = name;
        Author = author;
    }

    public int Id { get; set; }

    public string Name { get; set; }
    public string Author { get; set; }

    public ICollection<Post> Posts { get; } = new List<Post>();
}

public class Post
{
    public Post(int id, string title, DateTime postedOn)
    {
        Id = id;
        Title = title;
        PostedOn = postedOn;
    }

    public int Id { get; set; }

    public string Title { get; set; }
    public string Content { get; set; }
    public DateTime PostedOn { get; set; }

    public Blog Blog { get; set; }
}
```
需要注意一些事项：
* 需要具有构造函数参数不是所有属性。 例如，由任何构造函数参数，因此 EF Core 将以正常方式调用的构造函数后设置未设置 Post.Content 属性。
* 参数类型和名称必须与匹配的属性类型和名称，只不过属性可以是 Pascal 大小写而参数 camel 大小写。
* 无法设置 （如博客或更高版本的文章） 的导航属性，EF Core 使用构造函数。
* 构造函数可以是公共、 私有的或具有任何其他可访问性。

### <a name="read-only-properties"></a>只读属性

通过构造函数中设置属性之后它以使其中一些只读可以很有意义。 EF Core 支持此功能，但有一些需要注意的事项：
* 按照约定未映射不包含 setter 的属性。 （执行此操作通常不应将映射，如计算属性的属性映射。）
* 使用自动生成的密钥值需要是读写的因为需要插入新实体时，密钥生成器设置密钥值的键属性。

避免这些事情的简单方法是使用私有资源库。 例如：
``` csharp
public class Blog
{
    public Blog(int id, string name, string author)
    {
        Id = id;
        Name = name;
        Author = author;
    }

    public int Id { get; private set; }

    public string Name { get; private set; }
    public string Author { get; private set; }

    public ICollection<Post> Posts { get; } = new List<Post>();
}

public class Post
{
    public Post(int id, string title, DateTime postedOn)
    {
        Id = id;
        Title = title;
        PostedOn = postedOn;
    }

    public int Id { get; private set; }

    public string Title { get; private set; }
    public string Content { get; set; }
    public DateTime PostedOn { get; private set; }

    public Blog Blog { get; set; }
}
```
EF Core 看到为读写模式，这意味着，像以前那样映射所有属性并密钥可能仍会由存储生成的具有专用 setter 的属性。

使用私有资源库的替代方法是使属性实际上是只读的并在 OnModelCreating 中添加更多显式映射。 同样，某些属性可以彻底删除并替换为只有字段。 例如，考虑这些实体类型：

``` csharp
public class Blog
{
    private int _id;

    public Blog(string name, string author)
    {
        Name = name;
        Author = author;
    }

    public string Name { get; }
    public string Author { get; }

    public ICollection<Post> Posts { get; } = new List<Post>();
}

public class Post
{
    private int _id;

    public Post(string title, DateTime postedOn)
    {
        Title = title;
        PostedOn = postedOn;
    }

    public string Title { get; }
    public string Content { get; set; }
    public DateTime PostedOn { get; }

    public Blog Blog { get; set; }
}
```
与此配置在 OnModelCreating 中：
``` csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>(
        b =>
        {
            b.HasKey("_id");
            b.Property(e => e.Author);
            b.Property(e => e.Name);
        });

    modelBuilder.Entity<Post>(
        b =>
        {
            b.HasKey("_id");
            b.Property(e => e.Title);
            b.Property(e => e.PostedOn);
        });
}
```
需要注意的事项：
* 键"属性"现在是一个字段。 它不是`readonly`字段，以便可以使用应用商店生成的键。
* 其他属性是只读的只能在构造函数中设置的属性。
* 如果主键值是只通过 EF 设置或从数据库读取，则无需将其包括在构造函数。 这为简单字段离开密钥"属性"，并使用户清楚地知道，它不应设置显式创建新的博客或帖子时。

> [!NOTE]  
> 此代码将导致编译器警告"169"，永远不会使用该字段指示。 由于在现实中 EF Core extralinguistic 的方式使用该字段，这可予以忽视。

## <a name="injecting-services"></a>注入服务

EF Core 还可以将"服务"注入到实体类型的构造函数。 例如，下面可以注入：
* `DbContext` -当前上下文实例，也可以作为派生 DbContext 类型类型化
* `ILazyLoader` -延迟加载服务中--请参阅[延迟加载文档](../querying/related-data.md)的更多详细信息
* `Action<object, string>` -延迟加载委托-请参阅[延迟加载文档](../querying/related-data.md)的更多详细信息
* `IEntityType` -与此实体类型关联的 EF Core 元数据

> [!NOTE]  
> 截至 EF Core 2.1，可插入仅通过 EF Core 已知的服务。 注入应用程序服务的支持正在考虑针对将来的版本。

例如，注入的 DbContext 可用于有选择地访问数据库以获取有关相关实体的信息不加载所有这些情况下。 在下面的示例这用于获取在博客中的帖子数而不加载发布的文章：

``` csharp
public class Blog
{
    public Blog()
    {
    }

    private Blog(BloggingContext context)
    {
        Context = context;
    }

    private BloggingContext Context { get; set; }

    public int Id { get; set; }
    public string Name { get; set; }
    public string Author { get; set; }

    public ICollection<Post> Posts { get; set; }

    public int PostsCount
        => Posts?.Count
           ?? Context?.Set<Post>().Count(p => Id == EF.Property<int?>(p, "BlogId"))
           ?? 0;
}

public class Post
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }
    public DateTime PostedOn { get; set; }

    public Blog Blog { get; set; }
}
```
请注意，有关这几个注意事项：
* 构造函数是专用容器，因为它只能由 EF Core，并且没有用于常规用途的另一个公共构造函数。
* 使用注入的服务 （上下文） 的代码是针对该防御性正在`null`处理情况下，其中 EF Core 不创建实例。
* 由于服务存储在读/写属性将重置时该实体附加到新的上下文实例。

> [!WARNING]  
> 因为它将直接与 EF Core 的实体类型，将注入如下 DbContext 通常被视为反模式。 使用此类服务注入之前仔细考虑所有选项。
