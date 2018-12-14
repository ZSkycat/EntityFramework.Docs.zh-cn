---
title: 原始 SQL 查询的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 9e1ee76e-2499-408c-81e8-9b6c5d1945a0
ms.openlocfilehash: 168aee67186535bf2a50705e86bfc5a88147e369
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283779"
---
# <a name="raw-sql-queries"></a>原生 SQL 查询
实体框架，可与实体类使用 LINQ 进行查询。 但是，可能有时你想要使用原始 SQL 直接针对数据库运行查询。 这包括调用存储的过程的当前不支持映射到存储过程的 Code First 模型会很有用。 本主题所介绍的方法同样适用于查询使用 Code First 和 EF 设计器创建的模型。  

## <a name="writing-sql-queries-for-entities"></a>编写实体 SQL 查询  

DbSet 的 SqlQuery 方法允许原始 SQL 查询，将返回实体实例写入。 返回的对象将由上下文跟踪，就像它们是如果它们未返回 LINQ 查询。 例如：  

``` csharp  
using (var context = new BloggingContext())
{
    var blogs = context.Blogs.SqlQuery("SELECT * FROM dbo.Blogs").ToList();
}
```  

请注意，就像 LINQ 查询，不执行查询之前枚举结果-这在上面的示例通过调用 ToList。  

每当有原始 SQL 查询写入以下两个原因，应格外小心。 首先，应编写查询以确保它仅返回所请求类型的新技术确实的实体。 例如，使用功能，例如继承时很容易编写查询将创建的错误的 CLR 类型的实体。  

第二，某些类型的原始 SQL 查询公开潜在的安全风险，尤其是在方面 SQL 注入式攻击。 请确保正确的方式来防止此类攻击的参数在查询中使用。  

### <a name="loading-entities-from-stored-procedures"></a>从存储过程加载实体  

DbSet.SqlQuery 可用于从存储过程的结果加载实体。 例如，以下代码调用 dbo。GetBlogs 数据库中的过程：  

``` csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs.SqlQuery("dbo.GetBlogs").ToList();
}
```  

此外可以将参数传递到存储过程，请使用以下语法：  

``` csharp
using (var context = new BloggingContext())
{
    var blogId = 1;

    var blogs = context.Blogs.SqlQuery("dbo.GetBlogById @p0", blogId).Single();
}
```  

## <a name="writing-sql-queries-for-non-entity-types"></a>为非实体类型编写 SQL 查询  

可以使用数据库类中的 SqlQuery 方法创建返回任意类型，包括基元类型的实例的 SQL 查询。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blogNames = context.Database.SqlQuery<string>(
                       "SELECT Name FROM dbo.Blogs").ToList();
}
```  

即使对象是实体类型的实例，则永远不会将由上下文跟踪从 SqlQuery 返回在数据库上的结果。  

## <a name="sending-raw-commands-to-the-database"></a>将原始命令发送到数据库  

可以在数据库上使用 ExecuteSqlCommand 方法向数据库发送非查询命令。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    context.Database.ExecuteSqlCommand(
        "UPDATE dbo.Blogs SET Name = 'Another Name' WHERE BlogId = 1");
}
```  

请注意使用 ExecuteSqlCommand 在数据库中的数据所做的任何更改都不透明的上下文，直到加载或从数据库重新加载实体。  

### <a name="output-parameters"></a>输出参数  

如果使用输出参数，其值将不可用，直到已完全读取结果。 这是由于 DbDataReader 的基础行为，请参阅[使用 DataReader 检索数据](https://go.microsoft.com/fwlink/?LinkID=398589)的更多详细信息。  
