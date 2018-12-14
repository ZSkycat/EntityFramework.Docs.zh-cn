---
title: 加载相关实体的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: c8417e18-a2ee-499c-9ce9-2a48cc5b468a
ms.openlocfilehash: 2d33d9db8acc61f7d556e3eca46b1ea90198723e
ms.sourcegitcommit: 15022dd06d919c29b1189c82611ea32f9fdc6617
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2018
ms.locfileid: "47415752"
---
# <a name="loading-related-entities"></a>加载相关的实体
Entity Framework 支持三种方法来加载相关的数据的预先加载、 延迟加载和显式加载。 本主题所介绍的方法同样适用于查询使用 Code First 和 EF 设计器创建的模型。  

## <a name="eagerly-loading"></a>预先加载  

预先加载是实体的一种类型的查询在查询的一部分，并加载相关的实体的过程。 预先加载被通过使用 Include 方法。 例如，以下查询将加载博客和与每个博客相关的所有文章。  

``` csharp
using (var context = new BloggingContext())
{
    // Load all blogs and related posts
    var blogs1 = context.Blogs
                        .Include(b => b.Posts)
                        .ToList();

    // Load one blogs and its related posts
    var blog1 = context.Blogs
                       .Where(b => b.Name == "ADO.NET Blog")
                       .Include(b => b.Posts)
                       .FirstOrDefault();

    // Load all blogs and related posts  
    // using a string to specify the relationship
    var blogs2 = context.Blogs
                        .Include("Posts")
                        .ToList();

    // Load one blog and its related posts  
    // using a string to specify the relationship
    var blog2 = context.Blogs
                       .Where(b => b.Name == "ADO.NET Blog")
                       .Include("Posts")
                       .FirstOrDefault();
}
```  

请注意，Include system.data.entity 的引用命名空间中的扩展方法因此请确保使用该命名空间。  

### <a name="eagerly-loading-multiple-levels"></a>积极加载多个级别  

还有可能自愿加载相关实体的多个级别。 以下查询显示如何为集合和引用导航属性执行此操作的示例。  

``` csharp
using (var context = new BloggingContext())
{
    // Load all blogs, all related posts, and all related comments
    var blogs1 = context.Blogs
                        .Include(b => b.Posts.Select(p => p.Comments))
                        .ToList();

    // Load all users, their related profiles, and related avatar
    var users1 = context.Users
                        .Include(u => u.Profile.Avatar)
                        .ToList();

    // Load all blogs, all related posts, and all related comments  
    // using a string to specify the relationships
    var blogs2 = context.Blogs
                        .Include("Posts.Comments")
                        .ToList();

    // Load all users, their related profiles, and related avatar  
    // using a string to specify the relationships
    var users2 = context.Users
                        .Include("Profile.Avatar")
                        .ToList();
}
```  

请注意，不当前可以对加载哪些相关的实体进行筛选。 包括将始终会将所有相关实体中。  

## <a name="lazy-loading"></a>延迟加载  

延迟加载是凭此实体的集合自动从数据库加载第一次访问指的实体/实体属性的过程。 使用 POCO 实体类型时，延迟加载被通过创建派生的代理类型的实例，然后重写虚拟属性添加加载挂钩。 例如，使用下面定义的网络日志实体类时，相关的帖子将会加载在首次访问文章导航属性：  

``` csharp
public class Blog
{  
    public int BlogId { get; set; }  
    public string Name { get; set; }  
    public string Url { get; set; }  
    public string Tags { get; set; }  

    public virtual ICollection<Post> Posts { get; set; }  
}
```  

### <a name="turn-lazy-loading-off-for-serialization"></a>启用延迟加载关闭以进行序列化  

延迟加载和序列化不要混合在一起，和如果您不小心最终查询为整个数据库，只是因为启用了延迟加载。 大多数序列化程序通过访问每个属性类型的实例上工作。 属性访问触发延迟加载更多实体进行序列化。 对这些实体访问属性，并且加载更多实体。 它是启用延迟加载关闭之前序列化实体的好办法。 以下部分说明如何执行此操作。  

### <a name="turning-off-lazy-loading-for-specific-navigation-properties"></a>关闭延迟加载的特定导航属性  

可以通过使文章属性非虚拟关闭延迟加载的 Posts 集合：  

``` csharp
public class Blog
{  
    public int BlogId { get; set; }  
    public string Name { get; set; }  
    public string Url { get; set; }  
    public string Tags { get; set; }  

    public ICollection<Post> Posts { get; set; }  
}
```  

加载的帖子集合仍可以使用预先加载 (请参阅*积极加载*上方) 或 Load 方法 (请参阅*显式加载*下面)。  

### <a name="turn-off-lazy-loading-for-all-entities"></a>关闭延迟加载的所有实体  

延迟加载可以关闭的所有实体的上下文中设置配置属性的标志。 例如：  

``` csharp
public class BloggingContext : DbContext
{
    public BloggingContext()
    {
        this.Configuration.LazyLoadingEnabled = false;
    }
}
```  

仍可以使用预先加载实现加载相关实体 (请参阅*积极加载*上方) 或 Load 方法 (请参阅*显式加载*下面)。  

## <a name="explicitly-loading"></a>显式加载  

即使使用禁用延迟加载就仍然可以延迟加载相关的实体，但它必须使用显式调用完成。 若要执行此操作的相关的实体的条目使用 Load 方法。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var post = context.Posts.Find(2);

    // Load the blog related to a given post
    context.Entry(post).Reference(p => p.Blog).Load();

    // Load the blog related to a given post using a string  
    context.Entry(post).Reference("Blog").Load();

    var blog = context.Blogs.Find(1);

    // Load the posts related to a given blog
    context.Entry(blog).Collection(p => p.Posts).Load();

    // Load the posts related to a given blog  
    // using a string to specify the relationship
    context.Entry(blog).Collection("Posts").Load();
}
```  

请注意当实体具有与另一个单一实体的导航属性时，应使用的引用方法。 另一方面，当实体具有的一系列其他实体的导航属性时，应使用的收集方法。  

### <a name="applying-filters-when-explicitly-loading-related-entities"></a>显式加载相关的实体时应用筛选器  

查询方法提供对加载相关的实体时，将使用实体框架的基础查询访问。 然后可以使用 LINQ 来执行通过 ToList、 负载等之类的 LINQ 扩展方法调用前对查询应用筛选器。查询方法可用于引用和集合导航属性，但是最有用的集合，其中使用它来加载仅集合的一部分。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);

    // Load the posts with the 'entity-framework' tag related to a given blog
    context.Entry(blog)
           .Collection(b => b.Posts)
           .Query()
           .Where(p => p.Tags.Contains("entity-framework"))
           .Load();

    // Load the posts with the 'entity-framework' tag related to a given blog  
    // using a string to specify the relationship  
    context.Entry(blog)
           .Collection("Posts")
           .Query()
           .Where(p => p.Tags.Contains("entity-framework"))
           .Load();
}
```  

使用查询方法时，通常最好关闭延迟加载的导航属性。 这是因为否则整个集合可能会获取自动加载被延迟加载机制之前或之后执行筛选的查询。  

请注意，虽然可以为字符串而不是 lambda 表达式指定的关系，返回的 IQueryable 不泛型时使用了字符串，因此可通过它完成任何有用的操作之前，通常需要强制转换方法。  

## <a name="using-query-to-count-related-entities-without-loading-them"></a>使用查询进行计数而不加载这些相关的实体  

有时最好要知道多少个实体与数据库中的另一个实体而不实际产生加载这些实体的费用。 使用 LINQ 计数方法的查询方法可用来执行此操作。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);

    // Count how many posts the blog has  
    var postCount = context.Entry(blog)
                           .Collection(b => b.Posts)
                           .Query()
                           .Count();
}
```  
