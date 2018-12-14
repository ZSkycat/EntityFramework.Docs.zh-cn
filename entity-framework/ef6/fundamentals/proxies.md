---
title: 使用代理的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 869ee4dc-06f1-471d-8e0e-0a1a2bc59c30
ms.openlocfilehash: 8f7d2e8b41ece28efe8d1df3b0679e6e4510d64a
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489812"
---
# <a name="working-with-proxies"></a>使用代理
创建 POCO 实体类型的实例时，实体框架通常会创建动态生成的派生类型的代理的实体的实例。 此代理将覆盖要插入挂钩，以便访问属性时自动执行的操作的实体的某些虚拟属性。 例如，这种机制用于支持延迟加载的关系。 本主题所介绍的方法同样适用于查询使用 Code First 和 EF 设计器创建的模型。  

## <a name="disabling-proxy-creation"></a>禁用代理创建  

有时，它可用于防止 Entity Framework 创建代理实例了。 例如，序列化非代理实例比更容易得多序列化代理实例。 可以通过清除 ProxyCreationEnabled 标志关闭代理创建。 无法执行此操作的一个位置是上下文的您的构造函数中。 例如：  

``` csharp
public class BloggingContext : DbContext
{
    public BloggingContext()
    {
        this.Configuration.ProxyCreationEnabled = false;
    }  

    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
}
```  

请注意，EF 将不会创建代理类型没有任何关于要执行的代理的信息。 这意味着您可以通过采用类型都密封的和/或不包含虚拟属性中还避免代理。  

## <a name="explicitly-creating-an-instance-of-a-proxy"></a>显式创建代理的实例  

如果创建的实体使用 new 运算符实例，将无法创建代理实例。 这可能不是问题，但是如果您需要创建代理实例 （例如，以便延迟加载或代理更改跟踪将起作用），可以实现使用 DbSet 的 Create 方法。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Create();
}
```  

可以使用创建的泛型版本，如果你想要创建派生的实体类型的实例。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var admin = context.Users.Create<Administrator>();
}
```  

请注意，Create 方法不会不添加或创建的实体附加到上下文。  

请注意，如果创建代理类型的实体，Create 方法将只需创建实体类型本身的实例必须没有值，因为它不会执行任何操作。 例如，如果实体类型密封的并且/或者没有虚拟属性创建只需将创建实体类型的实例。  

## <a name="getting-the-actual-entity-type-from-a-proxy-type"></a>从代理类型中获取的实际实体类型  

代理类型具有名称如下所示：  

System.Data.Entity.DynamicProxies.Blog_5E43C6C196972BF0754973E48C9C941092D86818CD94005E9A759B70BF6E48E6  

您可以找到使用 GetObjectType 方法从 ObjectContext 此代理类型的实体类型。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);
    var entityType = ObjectContext.GetObjectType(blog.GetType());
}
```  

请注意，如果将类型传递给 GetObjectType 仍返回不是代理类型的实体类型的实体类型的一个实例。 这意味着你始终可以使用此方法以获取实际的实体类型，而无需任何其他检查以查看是否类型为代理类型。  
