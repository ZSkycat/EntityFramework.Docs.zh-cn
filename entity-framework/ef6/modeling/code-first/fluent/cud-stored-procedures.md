---
title: 代码的第一个 Insert、 Update 和 Delete 存储的过程-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 9a7ae7f9-4072-4843-877d-506dd7eef576
ms.openlocfilehash: bfc56671814aec1965ac054ff901297e5cdbbecb
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489617"
---
# <a name="code-first-insert-update-and-delete-stored-procedures"></a>代码的第一个 Insert、 Update 和 Delete 存储的过程
> [!NOTE]
> **仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。 如果使用的是早期版本，则部分或全部信息不适用。  

默认情况下，Code First 将配置所有实体，以执行插入、 更新和删除命令使用表的直接访问。 开始在 EF6 中可以配置 Code First 模型的存储的过程用于在模型中的某些或所有实体。  

## <a name="basic-entity-mapping"></a>基本实体映射  

您可以选择使用存储的过程为插入、 更新和删除使用 Fluent API。  

``` csharp
modelBuilder
  .Entity<Blog>()
  .MapToStoredProcedures();
```  

执行此操作将导致代码优先，一些约定用于生成的数据库中的存储过程预期的形状。  

- 三个存储过程名为 **\<type_name\>（_i)**，  **\<type_name\>（_u)** 并**\<得以名称\>（_d)** （例如，Blog_Insert、 Blog_Update 和 Blog_Delete）。  
- 参数名称对应于属性名称。  
  > [!NOTE]
  > 如果使用 HasColumnName() 或列属性设置为给定属性列重命名此名称用于而不是属性名称的参数。  
- **Insert 存储过程**将具有每个属性，除了那些已标记为存储生成的参数 (标识或计算)。 存储的过程应返回包含每个生成的存储属性的列的结果集。  
- **更新存储过程**将具有每个属性，除了那些标有 Computed 生成的存储模式的参数。 一些并发标记的原始值需要一个参数，请参阅*并发标记*部分获取详细信息。 存储的过程应返回包含每个计算属性的列的结果集。  
- **删除存储过程**应具有的键值的实体 （或者如果该实体具有复合键的多个参数） 的参数。 此外，删除过程还应为所有独立关联中的外键的参数对目标表 （没有声明的实体中的相应外键属性的关系）。 一些并发标记的原始值需要一个参数，请参阅*并发标记*部分获取详细信息。  

例如，使用以下类：  

``` csharp
public class Blog  
{  
  public int BlogId { get; set; }  
  public string Name { get; set; }  
  public string Url { get; set; }  
}
```  

默认值是存储的过程：  

``` SQL
CREATE PROCEDURE [dbo].[Blog_Insert]  
  @Name nvarchar(max),  
  @Url nvarchar(max)  
AS  
BEGIN
  INSERT INTO [dbo].[Blogs] ([Name], [Url])
  VALUES (@Name, @Url)

  SELECT SCOPE_IDENTITY() AS BlogId
END
CREATE PROCEDURE [dbo].[Blog_Update]  
  @BlogId int,  
  @Name nvarchar(max),  
  @Url nvarchar(max)  
AS  
  UPDATE [dbo].[Blogs]
  SET [Name] = @Name, [Url] = @Url     
  WHERE BlogId = @BlogId;
CREATE PROCEDURE [dbo].[Blog_Delete]  
  @BlogId int  
AS  
  DELETE FROM [dbo].[Blogs]
  WHERE BlogId = @BlogId
```  

### <a name="overriding-the-defaults"></a>重写默认值  

您可以重写部分或全部默认情况下已配置的内容。  

您可以更改一个或多个存储过程的名称。 此示例将仅更新存储过程。  

``` csharp
modelBuilder  
  .Entity<Blog>()  
  .MapToStoredProcedures(s =>  
    s.Update(u => u.HasName("modify_blog")));
```  

此示例将所有三个存储的过程。  

``` csharp
modelBuilder  
  .Entity<Blog>()  
  .MapToStoredProcedures(s =>  
    s.Update(u => u.HasName("modify_blog"))  
     .Delete(d => d.HasName("delete_blog"))  
     .Insert(i => i.HasName("insert_blog")));
```  

在这些示例中调用链接在一起，但您也可以使用 lambda 块语法。  

``` csharp
modelBuilder  
  .Entity<Blog>()  
  .MapToStoredProcedures(s =>  
    {  
      s.Update(u => u.HasName("modify_blog"));  
      s.Delete(d => d.HasName("delete_blog"));  
      s.Insert(i => i.HasName("insert_blog"));  
    });
```  

此示例将更新存储过程的 BlogId 属性参数。  

``` csharp
modelBuilder  
  .Entity<Blog>()  
  .MapToStoredProcedures(s =>  
    s.Update(u => u.Parameter(b => b.BlogId, "blog_id")));
```  

这些调用是所有可链接和可组合。 下面是示例重命名所有三个存储的过程和其参数。  

``` csharp
modelBuilder  
  .Entity<Blog>()  
  .MapToStoredProcedures(s =>  
    s.Update(u => u.HasName("modify_blog")  
                   .Parameter(b => b.BlogId, "blog_id")  
                   .Parameter(b => b.Name, "blog_name")  
                   .Parameter(b => b.Url, "blog_url"))  
     .Delete(d => d.HasName("delete_blog")  
                   .Parameter(b => b.BlogId, "blog_id"))  
     .Insert(i => i.HasName("insert_blog")  
                   .Parameter(b => b.Name, "blog_name")  
                   .Parameter(b => b.Url, "blog_url")));
```  

此外可以更改中包含的数据库生成值的结果集列的名称。  

``` csharp
modelBuilder
  .Entity<Blog>()
  .MapToStoredProcedures(s =>
    s.Insert(i => i.Result(b => b.BlogId, "generated_blog_identity")));
```

``` SQL
CREATE PROCEDURE [dbo].[Blog_Insert]  
  @Name nvarchar(max),  
  @Url nvarchar(max)  
AS  
BEGIN
  INSERT INTO [dbo].[Blogs] ([Name], [Url])
  VALUES (@Name, @Url)

  SELECT SCOPE_IDENTITY() AS generated_blog_id
END
```  

## <a name="relationships-without-a-foreign-key-in-the-class-independent-associations"></a>而无需类 （独立关联） 中的外键关系  

当在类定义中包括外键属性时，可以为任何其他属性相同的方式重命名的相应参数。 当没有外键属性的类中存在的关系时，默认参数名称是 **\<navigation_property_name\>_\<primary_key_name\>**。  

例如，以下类定义将导致存储过程以插入和更新文章中所预期的 Blog_BlogId 参数。  

``` csharp
public class Blog  
{  
  public int BlogId { get; set; }  
  public string Name { get; set; }  
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

### <a name="overriding-the-defaults"></a>重写默认值  

可以更改参数的情况下提供对参数方法主键属性的路径不包括在类中的外键。  

``` csharp
modelBuilder
  .Entity<Post>()  
  .MapToStoredProcedures(s =>  
    s.Insert(i => i.Parameter(p => p.Blog.BlogId, "blog_id")));
```  

如果依赖实体 （即没有导航属性 没有 Post.Blog 属性） 则可以使用关联方法来确定此关系的另一端，然后配置参数对应于每个键属性。  

``` csharp
modelBuilder
  .Entity<Post>()  
  .MapToStoredProcedures(s =>  
    s.Insert(i => i.Navigation<Blog>(  
      b => b.Posts,  
      c => c.Parameter(b => b.BlogId, "blog_id"))));
```  

## <a name="concurrency-tokens"></a>并发标记  

Update 和 delete 存储过程可能还需要并发处理：  

- 如果该实体包含并发标记，该存储的过程可以具有一个输出参数，它返回的行更新/删除 （受影响的行） 数。 此类参数必须使用 RowsAffectedParameter 方法进行配置。  
默认情况下 EF 使用 ExecuteNonQuery 的返回值来确定受影响行数。 指定受影响的行的输出参数非常有用，如果你将导致 ExecuteNonQuery 正在 （从 EF 的角度来看） 不正确的返回值的存储过程中执行的任何逻辑执行结束时。  
- 对于每个并发令牌存在将名为的参数 **\<property_name\>_Original** (例如，Timestamp_Original)。 这将传递此属性-值时从数据库中查询的原始值。  
    - 由数据库，– 例如，时间戳 – 计算的并发标记将只能有一个原始值参数。  
    - 非计算的属性设置为并发标记还必须在更新过程中的新值的参数。 这将使用已讨论过的新值的命名约定。 此类令牌的一个示例将使用博客的 URL 作为并发标记，新值是必需的因为这可以为新值更新，你的代码 （与不同的时间戳标记仅更新一次数据库）。  

这是一个示例类，并使用时间戳并发标记更新存储的过程。  

``` csharp
public class Blog  
{  
  public int BlogId { get; set; }  
  public string Name { get; set; }  
  public string Url { get; set; }  
  [Timestamp]
  public byte[] Timestamp { get; set; }
}
```

``` SQL
CREATE PROCEDURE [dbo].[Blog_Update]  
  @BlogId int,  
  @Name nvarchar(max),  
  @Url nvarchar(max),
  @Timestamp_Original rowversion  
AS  
  UPDATE [dbo].[Blogs]
  SET [Name] = @Name, [Url] = @Url     
  WHERE BlogId = @BlogId AND [Timestamp] = @Timestamp_Original
```  

下面是一个示例类，并使用非计算并发标记更新存储的过程。  

``` csharp
public class Blog  
{  
  public int BlogId { get; set; }  
  public string Name { get; set; }  
  [ConcurrencyCheck]
  public string Url { get; set; }  
}
```

``` SQL
CREATE PROCEDURE [dbo].[Blog_Update]  
  @BlogId int,  
  @Name nvarchar(max),  
  @Url nvarchar(max),
  @Url_Original nvarchar(max),
AS  
  UPDATE [dbo].[Blogs]
  SET [Name] = @Name, [Url] = @Url     
  WHERE BlogId = @BlogId AND [Url] = @Url_Original
```  

### <a name="overriding-the-defaults"></a>重写默认值  

您可以根据需要引入受影响的行数参数。  

``` csharp
modelBuilder  
  .Entity<Blog>()  
  .MapToStoredProcedures(s =>  
    s.Update(u => u.RowsAffectedParameter("rows_affected")));
```  

对计算的数据库并发令牌 – 其中仅的原始值将传递 – 只需可以使用重命名机制的标准参数重命名的原始值的参数。  

``` csharp
modelBuilder  
  .Entity<Blog>()  
  .MapToStoredProcedures(s =>  
    s.Update(u => u.Parameter(b => b.Timestamp, "blog_timestamp")));
```  

为 – 个同时的原始和新值传递 – 非计算并发令牌可以使用可用于提供每个参数的名称的参数的重载。  

``` csharp
modelBuilder
 .Entity<Blog>()
 .MapToStoredProcedures(s => s.Update(u => u.Parameter(b => b.Url, "blog_url", "blog_original_url")));
```  

## <a name="many-to-many-relationships"></a>多对多关系  

例如，在本部分中，我们将使用以下类。  

``` csharp
public class Post  
{  
  public int PostId { get; set; }  
  public string Title { get; set; }  
  public string Content { get; set; }  

  public List<Tag> Tags { get; set; }  
}  

public class Tag  
{  
  public int TagId { get; set; }  
  public string TagName { get; set; }  

  public List<Post> Posts { get; set; }  
}
```  

多对多个关系可以映射到具有以下语法的存储过程。  

``` csharp
modelBuilder  
  .Entity<Post>()  
  .HasMany(p => p.Tags)  
  .WithMany(t => t.Posts)  
  .MapToStoredProcedures();
```  

如果不提供任何其他配置，则默认情况下使用以下存储的过程形式。  

- 两个存储过程名为**\<类型 — —\>\<type_two\>（_i)** 并**\<类型 — —\>\<type_two\>（_d)** （例如，PostTag_Insert 和 PostTag_Delete）。  
- 参数将为每种类型的密钥值。 每个参数的名称**\<type_name\>_\<property_name\>** （例如，Post_PostId 和 Tag_TagId）。

下面是示例插入和更新存储的过程。  

``` SQL
CREATE PROCEDURE [dbo].[PostTag_Insert]  
  @Post_PostId int,  
  @Tag_TagId int  
AS  
  INSERT INTO [dbo].[Post_Tags] (Post_PostId, Tag_TagId)   
  VALUES (@Post_PostId, @Tag_TagId)
CREATE PROCEDURE [dbo].[PostTag_Delete]  
  @Post_PostId int,  
  @Tag_TagId int  
AS  
  DELETE FROM [dbo].[Post_Tags]    
  WHERE Post_PostId = @Post_PostId AND Tag_TagId = @Tag_TagId
```  

### <a name="overriding-the-defaults"></a>重写默认值  

可以在类似于实体存储过程中配置的过程和参数名称。  

``` csharp
modelBuilder  
  .Entity<Post>()  
  .HasMany(p => p.Tags)  
  .WithMany(t => t.Posts)  
  .MapToStoredProcedures(s =>  
    s.Insert(i => i.HasName("add_post_tag")  
                   .LeftKeyParameter(p => p.PostId, "post_id")  
                   .RightKeyParameter(t => t.TagId, "tag_id"))  
     .Delete(d => d.HasName("remove_post_tag")  
                   .LeftKeyParameter(p => p.PostId, "post_id")  
                   .RightKeyParameter(t => t.TagId, "tag_id")));
```  
