---
title: 定义 Dbset 的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 4528a509-ace7-4dfb-8065-1b833f5e03a0
ms.openlocfilehash: 045b22d2b9d26804948689dd7c9dd694baadda7e
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45488993"
---
# <a name="defining-dbsets"></a>定义 Dbset
使用代码优先工作流进行开发时定义一个派生的 DbContext 来表示与数据库的会话，并公开您的模型中每个类型的 DbSet。 本主题介绍您可以定义 DbSet 属性的各种方法。  

## <a name="dbcontext-with-dbset-properties"></a>使用 DbSet 属性的 DbContext  

第一个代码示例所示的常见情况是模型的拥有与您的实体类型的公共自动 DbSet 属性 DbContext。 例如：  

``` csharp
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
}
```  

Code First 模式中使用时，这会配置为实体类型，以及配置可从这些其他类型的博客和文章。 此外 DbContext 自动将这些属性来设置相应的 DbSet 的实例的每个调用 setter。  

## <a name="dbcontext-with-idbset-properties"></a>DbContext IDbSet 属性  

有些情况下，例如当创建模拟或 fakes，其中很多有用声明您使用的接口的组属性。 在这种情况下 IDbSet 接口可以代替 DbSet。 例如：  

``` csharp
public class BloggingContext : DbContext
{
    public IDbSet<Blog> Blogs { get; set; }
    public IDbSet<Post> Posts { get; set; }
}
```  

此上下文的工作方式完全相同的方式为使用 DbSet 类对于其设置的属性的上下文中。  

## <a name="dbcontext-with-read-only-set-properties"></a>使用只读的 DbContext 设置属性  

如果不希望公开 DbSet 或 IDbSet 属性而是可以创建只读属性，然后自行创建集实例的公共 setter。 例如：  

``` csharp
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs
    {
        get { return Set<Blog>(); }
    }

    public DbSet<Post> Posts
    {
        get { return Set<Post>(); }
    }
}
```  

请注意，DbContext 缓存，以便每个属性将返回相同的实例，每次调用时它从 Set 方法返回的 DbSet 的实例。  

相同的方式中的 Code First 工作原理的实体类型的发现像它那样具有公共 getter 和 setter 的属性。  
