---
title: 验证-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 77d6a095-c0d0-471e-80b9-8f9aea6108b2
ms.openlocfilehash: 3aeb33763819544618c4a3068bb278c9b23409b6
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490619"
---
# <a name="data-validation"></a>数据验证
> [!NOTE]
> **EF4.1 及更高版本仅**的功能，Api，Entity Framework 4.1 中引入了此页所述的等。 如果您使用的是早期版本，部分或全部信息不适用

此页上的内容改编自文章作者： Julie Lerman 最初编写和 ([http://thedatafarm.com](http://thedatafarm.com))。

实体框架提供了多种源通过客户端验证的用户界面或用于服务器端验证的验证功能。 当使用 code first，可以指定使用注释或 fluent API 配置的验证。 其他验证，以及更多复杂，可以在代码中指定和您的模型拥有从代码前，是否将工作模型优先或数据库优先。

## <a name="the-model"></a>模型

我将演示如何使用类的一个简单的对验证： 博客和文章。

``` csharp
    public class Blog
      {
          public int Id { get; set; }
          public string Title { get; set; }
          public string BloggerName { get; set; }
          public DateTime DateCreated { get; set; }
          public virtual ICollection<Post> Posts { get; set; }
          }
      }

      public class Post
      {
          public int Id { get; set; }
          public string Title { get; set; }
          public DateTime DateCreated { get; set; }
          public string Content { get; set; }
          public int BlogId { get; set; }
          public ICollection<Comment> Comments { get; set; }
      }
```
## <a name="data-annotations"></a>数据注释

代码首先使用 System.ComponentModel.DataAnnotations 程序集中的批注作为一种配置代码第一类。 这些批注包括那些提供必需的 MaxLength 和 MinLength 等规则。 大量的.NET 客户端应用程序也能识别这些批注，例如，ASP.NET MVC。 您可以获得这些批注与这两个客户端和服务器端验证。 例如，可以强制博客 Title 属性是必需的属性。

``` csharp
    [Required]
    public string Title { get; set; }
```

与任何其他代码或应用程序中的标记更改时，现有的 MVC 应用程序将执行客户端验证，甚至动态生成使用的属性和批注名称的消息。

![图 1](~/ef6/media/figure01.png)

文中重新创建此视图的方法，实体框架用于将新博客保存到数据库，但 MVC 的客户端验证会触发之前应用程序到达该代码。

客户端验证不但是项目符号经得起考验。 用户可能会影响其浏览器的功能或更糟糕的是，黑客可能会使用一些技巧来避免 UI 验证。 但是，实体框架还将识别所需的批注，并对其进行验证。

对此进行测试的简单方法是禁用 MVC 的客户端验证功能。 在 MVC 应用程序的 web.config 文件中，可以执行此操作。 AppSettings 节的 ClientValidationEnabled 有一个键。 将此项设置为 false 会阻止 UI 执行验证。

``` xml
    <appSettings>
        <add key="ClientValidationEnabled"value="false"/>
        ...
    </appSettings>
```

即使与禁用客户端验证，将在应用程序中获取相同的响应。 错误消息"标题字段是必填"将显示为。 不过，现在它将是服务器端验证的结果。 实体框架将前进行验证所需的注释上 （甚至厌烦与生成和要发送到数据库的 INSERT 命令） 并将错误返回给 MVC 将显示消息。

## <a name="fluent-api"></a>Fluent API

您可以使用代码第一的 fluent API，而不是批注以获取同一个客户端端和服务器端验证。 而不是使用必需的我将介绍此过程通过 MaxLength 验证。

如代码首先构建从类模型应用 Fluent API 配置。 可以通过重写 DbContext 类的 OnModelCreating 方法注入配置。 下面是配置指定 BloggerName 属性可以长度不得超过 10 个字符。

``` csharp
    public class BlogContext : DbContext
      {
          public DbSet<Blog> Blogs { get; set; }
          public DbSet<Post> Posts { get; set; }
          public DbSet<Comment> Comments { get; set; }

          protected override void OnModelCreating(DbModelBuilder modelBuilder)
          {
              modelBuilder.Entity<Blog>().Property(p => p.BloggerName).HasMaxLength(10);
          }
        }
```

引发验证错误基于 Fluent API 配置将不自动 reach UI，但您可以捕获其代码，然后响应中相应地。

下面的一些异常，处理应用程序的 BlogController 类，它捕获该验证错误，当尝试使用超过 10 个字符最大 BloggerName 保存一个博客，实体框架中的错误代码。

``` csharp
    [HttpPost]
    public ActionResult Edit(int id, Blog blog)
    {
        try
        {
            db.Entry(blog).State = EntityState.Modified;
            db.SaveChanges();
            return RedirectToAction("Index");
        }
        catch(DbEntityValidationException ex)
        {
            var error = ex.EntityValidationErrors.First().ValidationErrors.First();
            this.ModelState.AddModelError(error.PropertyName, error.ErrorMessage);
            return View();
        }
    }
```

验证不会自动传递回这正是使用 ModelState.AddModelError 的附加代码正在使用的视图。 这可确保错误详细信息视图将随后使用 ValidationMessageFor Htmlhelper 显示错误使它。

``` csharp
    @Html.ValidationMessageFor(model => model.BloggerName)
```

## <a name="ivalidatableobject"></a>IValidatableObject

IValidatableObject 是位于 System.ComponentModel.DataAnnotations 的接口。 尽管这不是 Entity Framework API 的一部分，你仍然可以利用它进行服务器端验证在实体框架的类中。 IValidatableObject 提供实体框架将调用期间 SaveChanges 或你的验证方法可以调用自己想要验证的类的任何时间。

如所需的配置和 MaxLength 单个字段执行验证。 在验证方法可以具有变得更加复杂的逻辑，例如，比较两个字段。

在下面的示例，博客类已扩展，实现 IValidatableObject，然后提供标题和 BloggerName 无法匹配的规则。

``` csharp
    public class Blog : IValidatableObject
     {
         public int Id { get; set; }
         [Required]
         public string Title { get; set; }
         public string BloggerName { get; set; }
         public DateTime DateCreated { get; set; }
         public virtual ICollection<Post> Posts { get; set; }

         public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
         {
             if (Title == BloggerName)
             {
                 yield return new ValidationResult
                  ("Blog Title cannot match Blogger Name", new[] { "Title", “BloggerName” });
             }
         }
     }
```

ValidationResult 构造函数采用一个字符串，表示错误消息和一个表示与验证相关联的成员名称的字符串数组。 由于此验证检查的标题和 BloggerName，将返回这两个属性名称。

与 Fluent API 提供的验证，此验证结果会被视为由视图和我之前使用将错误添加到 ModelState 的异常处理程序是不必要的。 由于我在 ValidationResult 中设置两个属性名称，MVC HtmlHelpers 这些属性的两个显示的错误消息。

![图 2](~/ef6/media/figure02.png)

## <a name="dbcontextvalidateentity"></a>DbContext.ValidateEntity

DbContext 具有名为 ValidateEntity 的可重写方法。 当调用 SaveChanges 时，实体框架将调用此方法，其状态不是未更改其缓存中每个实体。 可以直接在此处或甚至使用此方法调用，例如，在上一节中添加的 Blog.Validate 方法将验证逻辑。

下面是验证以确保文章标题未已使用的新帖子的 ValidateEntity 重写的示例。 它首先检查以查看实体是否为 post，并添加其状态。 如果是这样，然后它会查找数据库以查看是否已存在具有相同标题的帖子中。 如果已存在现有的帖子，则会创建新 DbEntityValidationResult。

DbEntityValidationResult 存储 DbEntityEntry 和针对单个实体 DbValidationErrors ICollection。 在此方法开始时，实例化 DbEntityValidationResult，然后发现任何错误将被添加到其 ValidationErrors 集合。

``` csharp
    protected override DbEntityValidationResult ValidateEntity (
        System.Data.Entity.Infrastructure.DbEntityEntry entityEntry,
        IDictionary\<object, object> items)
    {
        var result = new DbEntityValidationResult(entityEntry, new List<DbValidationError>());
        if (entityEntry.Entity is Post && entityEntry.State == EntityState.Added)
        {
            Post post = entityEntry.Entity as Post;
            //check for uniqueness of post title
            if (Posts.Where(p => p.Title == post.Title).Count() > 0)
            {
                result.ValidationErrors.Add(
                        new System.Data.Entity.Validation.DbValidationError("Title",
                        "Post title must be unique."));
            }
        }

        if (result.ValidationErrors.Count > 0)
        {
            return result;
        }
        else
        {
         return base.ValidateEntity(entityEntry, items);
        }
    }
```

## <a name="explicitly-triggering-validation"></a>显式触发验证

调用 SaveChanges 触发所有本文中所述的验证。 但无需依赖于 SaveChanges。 您可能想要验证应用程序中的其他位置。

DbContext.GetValidationErrors 将触发的所有验证、 通过注释或 Fluent API 定义的、 在 IValidatableObject (例如，Blog.Validate) 中创建的验证和 DbContext.ValidateEntity 中执行的验证方法。

下面的代码将调用 GetValidationErrors DbContext 的当前实例上。 ValidationErrors 按实体类型到 DbValidationRestuls 分组。 此代码首先循环通过该方法返回 DbValidationResults，然后通过在每个 ValidationError。

``` csharp
    foreach (var validationResults in db.GetValidationErrors())
        {
            foreach (var error in validationResults.ValidationErrors)
            {
                Debug.WriteLine(
                                  "Entity Property: {0}, Error {1}",
                                  error.PropertyName,
                                  error.ErrorMessage);
            }
        }
```

## <a name="other-considerations-when-using-validation"></a>使用验证时的其他注意事项

下面是几个使用实体框架验证时要考虑其他要点：

-   在验证过程中禁用延迟加载。
-   EF 将验证数据批注在非映射属性 （未映射到数据库中的列的属性）。
-   更改在 SaveChanges 期间检测到的后执行验证。 如果在验证过程进行了更改它由你负责通知更改跟踪器。
-   如果在验证过程中发生错误，将引发 DbUnexpectedValidationException。
-   实体框架包括模型 （所需的最大长度等） 中的分面时会进行验证，即使不是您的类上的数据批注和/或使用 EF 设计器来创建模型。
-   优先顺序规则：
    -   Fluent API 调用重写相应的数据注释
-   执行顺序：
    -   属性验证之前类型验证
    -   类型验证时，才出现属性验证成功
-   如果属性是复杂还将包括其验证：
    -   属性级别验证的复杂类型属性
    -   在复杂类型，包括复杂类型上的 IValidatableObject 验证类型级别的验证

## <a name="summary"></a>总结

在实体框架验证 API 非常非常适合使用 MVC 中的客户端验证，但无需依赖于客户端验证。 实体框架将负责在服务器端验证 DataAnnotations 或已使用 code first Fluent API 应用的配置。

您还了解到大量的自定义行为，是否使用 IValidatableObject 接口或利用 DbContet.ValidateEntity 方法的扩展点。 并验证这些最后两个方法可通过 DbContext，无论使用 Code First、 Model First 或 Database First 工作流来描述概念模型。
