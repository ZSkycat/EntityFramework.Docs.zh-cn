---
title: 本地数据-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 2eda668b-1e5d-487d-9a8c-0e3beef03fcb
ms.openlocfilehash: 400b9e1337edac1b9fa4f0ec9e1384ca58aa2fbc
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490449"
---
# <a name="local-data"></a>本地数据
直接对 DbSet 运行 LINQ 查询将始终将查询发送到数据库，但是您可以访问的数据的当前内存中使用的 DbSet.Local 属性。 有关你使用的 DbContext.Entry 和 DbContext.ChangeTracker.Entries 方法的实体，还可以访问 EF 跟踪的额外信息。 本主题所介绍的方法同样适用于查询使用 Code First 和 EF 设计器创建的模型。  

## <a name="using-local-to-look-at-local-data"></a>使用本地查看本地数据  

DbSet 的本地属性提供对实体集的当前由上下文跟踪和未标记为已删除的简单访问。 访问本地属性永远不会导致查询发送到数据库。 这意味着，它通常用于执行查询后。 可以使用 Load 扩展方法来执行查询，以便将上下文跟踪结果。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    // Load all blogs from the database into the context
    context.Blogs.Load();

    // Add a new blog to the context
    context.Blogs.Add(new Blog { Name = "My New Blog" });

    // Mark one of the existing blogs as Deleted
    context.Blogs.Remove(context.Blogs.Find(1));

    // Loop over the blogs in the context.
    Console.WriteLine("In Local: ");
    foreach (var blog in context.Blogs.Local)
    {
        Console.WriteLine(
            "Found {0}: {1} with state {2}",
            blog.BlogId,  
            blog.Name,
            context.Entry(blog).State);
    }

    // Perform a query against the database.
    Console.WriteLine("\nIn DbSet query: ");
    foreach (var blog in context.Blogs)
    {
        Console.WriteLine(
            "Found {0}: {1} with state {2}",
            blog.BlogId,  
            blog.Name,
            context.Entry(blog).State);
    }
}
```  

如果我们有两个博客数据库的 ADO.NET 博客与 1 BlogId-和 Visual Studio 博客与 2 BlogId 中我们可以看到以下输出：  

```  
In Local:
Found 0: My New Blog with state Added
Found 2: The Visual Studio Blog with state Unchanged

In DbSet query:
Found 1: ADO.NET Blog with state Deleted
Found 2: The Visual Studio Blog with state Unchanged
```  

这说明了三个点：  

- 即使它具有尚未保存到数据库，将本地集合中包含新的博客我的新博客。 此博客具有零的主键，因为数据库不生成的实体的实际密钥。  
- 尽管仍正在由上下文跟踪，ADO.NET 博客不包括在本地集合中。 这是因为我们从从而将其标记为删除 DbSet 中删除它。  
- DbSet 用于执行查询时在结果中包含标记为删除 （ADO.NET 博客） 的博客和尚未保存到数据库的新博客 （我的新博客） 不包括在结果中。 这是因为 DbSet 正在执行对数据库进行查询，并且始终返回的结果反映什么是数据库中。  

## <a name="using-local-to-add-and-remove-entities-from-the-context"></a>使用本地添加和删除实体的上下文  

DbSet 上的本地属性返回[ObservableCollection](https://msdn.microsoft.com/library/ms668604.aspx)与事件挂钩，以便它与保持同步的上下文的内容。 这意味着可以添加或从本地集合或 DbSet 中删除实体。 这还意味着到上下文中将新的实体的查询会提供这些实体来更新本地集合。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    // Load some posts from the database into the context
    context.Posts.Where(p => p.Tags.Contains("entity-framework").Load();  

    // Get the local collection and make some changes to it
    var localPosts = context.Posts.Local;
    localPosts.Add(new Post { Name = "What's New in EF" });
    localPosts.Remove(context.Posts.Find(1));  

    // Loop over the posts in the context.
    Console.WriteLine("In Local after entity-framework query: ");
    foreach (var post in context.Posts.Local)
    {
        Console.WriteLine(
            "Found {0}: {1} with state {2}",
            post.Id,  
            post.Title,
            context.Entry(post).State);
    }

    var post1 = context.Posts.Find(1);
    Console.WriteLine(
        "State of post 1: {0} is {1}",
        post1.Name,  
        context.Entry(post1).State);  

    // Query some more posts from the database
    context.Posts.Where(p => p.Tags.Contains("asp.net").Load();  

    // Loop over the posts in the context again.
    Console.WriteLine("\nIn Local after asp.net query: ");
    foreach (var post in context.Posts.Local)
    {
        Console.WriteLine(
            "Found {0}: {1} with state {2}",
            post.Id,  
            post.Title,
            context.Entry(post).State);
    }
}
```  

假设我们有几个帖子标记为实体框架和 asp.net 输出可能如下所示：  

```  
In Local after entity-framework query:
Found 3: EF Designer Basics with state Unchanged
Found 5: EF Code First Basics with state Unchanged
Found 0: What's New in EF with state Added
State of post 1: EF Beginners Guide is Deleted

In Local after asp.net query:
Found 3: EF Designer Basics with state Unchanged
Found 5: EF Code First Basics with state Unchanged
Found 0: What's New in EF with state Added
Found 4: ASP.NET Beginners Guide with state Unchanged
```  

这说明了三个点：  

- 新文章 What's New in EF 已添加到本地集合将成为由上下文跟踪处于已添加状态。 它将因此插入到数据库时调用 SaveChanges。  
- 已从本地集合 （EF 初学者指南） 中删除的文章现在标记为已删除的上下文中。 它将因此从数据库中删除时调用 SaveChanges。  
- 加载到上下文中与第二个查询的其他文章 （ASP.NET 初学者指南） 自动添加到本地集合。  

最后一件要注意有关本地的事情是，因为它是 ObservableCollection 性能不适合大量实体。 因此如果您的上下文中正在处理数千个实体可能不会建议使用本地。  

## <a name="using-local-for-wpf-data-binding"></a>使用 WPF 数据绑定的本地  

DbSet 上的本地属性可直接用于 WPF 应用程序中的数据绑定，因为它是 ObservableCollection 实例。 这意味着它会自动将前面部分中所述的上下文的内容与保持同步和上下文的内容将自动保持与其同步。 请注意，您需要预填充的数据才会将绑定到，由于本地永远不会导致数据库查询的任何内容的本地集合。  

这不是完整的 WPF 数据绑定示例的合适位置，但的关键要素是：  

- 设置绑定源  
- 将其绑定到你的集的本地属性  
- 填充本地使用对数据库查询。  

## <a name="wpf-binding-to-navigation-properties"></a>WPF 绑定到导航属性  

如果你正在母版/详细信息数据绑定您可能想要绑定到一个实体的导航属性的详细信息视图。 实现此目的的简单方法是使用一个 ObservableCollection 的导航属性。 例如：  

``` csharp
public class Blog
{
    private readonly ObservableCollection<Post> _posts =
        new ObservableCollection<Post>();

    public int BlogId { get; set; }
    public string Name { get; set; }

    public virtual ObservableCollection<Post> Posts
    {
        get { return _posts; }
    }
}
```  

## <a name="using-local-to-clean-up-entities-in-savechanges"></a>使用本地清理 SaveChanges 中的实体  

在大多数情况下从导航属性中移除的实体将不会自动标记为已删除的上下文中。 例如，如果删除 Post 对象从 Blog.Posts 集合，文章将不会自动删除调用 SaveChanges 的时候。 如果您需要它来删除然后可能需要找到这些无关联的实体并标记为已删除，然后再调用 SaveChanges 或重写 SaveChanges 的一部分。 例如：  

``` csharp
public override int SaveChanges()
{
    foreach (var post in this.Posts.Local.ToList())
    {
        if (post.Blog == null)
        {
            this.Posts.Remove(post);
        }
    }

    return base.SaveChanges();
}
```  

上面的代码使用本地集合来查找所有文章和标记为已删除不具有的博客引用任何的。 ToList 调用是必需的因为删除将否则修改该集合正在枚举时调用。 在大多数其他情况下可以查询直接针对本地属性而无需先使用 ToList。  

## <a name="using-local-and-tobindinglist-for-windows-forms-data-binding"></a>使用本地和 ToBindingList 为 Windows 窗体数据绑定  

Windows 窗体不支持直接使用 ObservableCollection 完全保真的数据绑定。 但是，仍可以使用数据绑定的 DbSet 本地属性以获取前面各节中所述的所有权益。 这通过 ToBindingList 扩展方法创建[IBindingList](https://msdn.microsoft.com/library/system.componentmodel.ibindinglist.aspx)实现支持的本地 ObservableCollection。  

这不是完整的 Windows 窗体数据绑定示例的合适位置，但的关键要素是：  

- 设置对象绑定源  
- 将其绑定到使用 Local.ToBindingList() 集的本地属性  
- 填充本地使用对数据库查询  

## <a name="getting-detailed-information-about-tracked-entities"></a>获取有关跟踪的实体的详细的信息  

许多本系列中的示例使用入口方法以返回实体的 DbEntityEntry 实例。 此条目对象然后充当用于收集有关其当前状态，例如实体的信息以及在如显式加载相关的实体的实体上执行操作的起始点。  

条目方法返回多个或所有实体正在由上下文跟踪的 DbEntityEntry 对象。 这允许你收集的信息或执行对多个实体而不是只需单个条目的操作。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    // Load some entities into the context
    context.Blogs.Load();
    context.Authors.Load();
    context.Readers.Load();

    // Make some changes
    context.Blogs.Find(1).Title = "The New ADO.NET Blog";
    context.Blogs.Remove(context.Blogs.Find(2));
    context.Authors.Add(new Author { Name = "Jane Doe" });
    context.Readers.Find(1).Username = "johndoe1987";

    // Look at the state of all entities in the context
    Console.WriteLine("All tracked entities: ");
    foreach (var entry in context.ChangeTracker.Entries())
    {
        Console.WriteLine(
            "Found entity of type {0} with state {1}",
            ObjectContext.GetObjectType(entry.Entity.GetType()).Name,
            entry.State);
    }

    // Find modified entities of any type
    Console.WriteLine("\nAll modified entities: ");
    foreach (var entry in context.ChangeTracker.Entries()
                              .Where(e => e.State == EntityState.Modified))
    {
        Console.WriteLine(
            "Found entity of type {0} with state {1}",
            ObjectContext.GetObjectType(entry.Entity.GetType()).Name,
            entry.State);
    }

    // Get some information about just the tracked blogs
    Console.WriteLine("\nTracked blogs: ");
    foreach (var entry in context.ChangeTracker.Entries<Blog>())
    {
        Console.WriteLine(
            "Found Blog {0}: {1} with original Name {2}",
            entry.Entity.BlogId,  
            entry.Entity.Name,
            entry.Property(p => p.Name).OriginalValue);
    }

    // Find all people (author or reader)
    Console.WriteLine("\nPeople: ");
    foreach (var entry in context.ChangeTracker.Entries<IPerson>())
    {
        Console.WriteLine("Found Person {0}", entry.Entity.Name);
    }
}
```  

您会注意到，我们引入的作者和读者类到示例-这两种类均实现 IPerson 接口。  

``` csharp
public class Author : IPerson
{
    public int AuthorId { get; set; }
    public string Name { get; set; }
    public string Biography { get; set; }
}

public class Reader : IPerson
{
    public int ReaderId { get; set; }
    public string Name { get; set; }
    public string Username { get; set; }
}

public interface IPerson
{
    string Name { get; }
}
```  

让我们假设我们在数据库中具有以下数据：

使用 BlogId 博客 = 1 且名称 = ADO.NET 博客  
使用 BlogId 博客 = 2，名称 = Visual Studio 博客  
使用 BlogId 博客 = 3 且名称 =.NET Framework 博客  
作者与作者 Id = 1 且名称 = ' Joe Bloggs  
读取器与 ReaderId = 1 且名称 = John Doe  

将运行代码的输出：  

```  
All tracked entities:
Found entity of type Blog with state Modified
Found entity of type Blog with state Deleted
Found entity of type Blog with state Unchanged
Found entity of type Author with state Unchanged
Found entity of type Author with state Added
Found entity of type Reader with state Modified

All modified entities:
Found entity of type Blog with state Modified
Found entity of type Reader with state Modified

Tracked blogs:
Found Blog 1: The New ADO.NET Blog with original Name ADO.NET Blog
Found Blog 2: The Visual Studio Blog with original Name The Visual Studio Blog
Found Blog 3: .NET Framework Blog with original Name .NET Framework Blog

People:
Found Person John Doe
Found Person Joe Bloggs
Found Person Jane Doe
```  

这些示例阐释了几个点：  

- 条目方法返回的所有状态，包括已删除的实体的条目。 与此比较不包括本地删除的实体。  
- 使用非泛型条目方法时，则返回所有实体类型的项。 使用泛型条目方法时则仅会返回的实体是泛型类型的实例的项。 这用于上面获取的所有博客条目。 它还用于获得实现 IPerson 的所有实体的条目。 此示例演示泛型类型不必是实际的实体类型。  
- LINQ 到对象可用来筛选返回的结果。 这用于上面查找任何类型的实体，只要进行修改。  

请注意 DbEntityEntry 实例始终包含一个非 null 的实体。 这样就无需这些筛选器作为 DbEntityEntry 实例不表示关系条目与存根 （stub） 条目。
