---
title: 设计时 DbContext 创建-EF Core
author: bricelam
ms.author: bricelam
ms.date: 10/27/2017
uid: core/miscellaneous/cli/dbcontext-creation
ms.openlocfilehash: 66fec7605b6ac2da0af1e801f8a1dca0789aea35
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42993713"
---
<a name="design-time-dbcontext-creation"></a>设计时 DbContext 创建
==============================
某些 EF Core 工具命令 (例如，[迁移][ 1]命令) 需要派生`DbContext`实例以收集有关应用程序的详细信息在设计时创建实体类型和它们如何映射到数据库架构。 在大多数情况下，最好的`DbContext`从而创建到如何将类似的方式配置[在运行时配置][2]。

有的多种工具会尝试创建`DbContext`:

<a name="from-application-services"></a>从应用程序服务
-------------------------
如果你的启动项目，ASP.NET Core 应用工具将尝试从应用程序的服务提供程序获取 DbContext 对象。

工具会首先尝试通过调用获取服务提供商`Program.BuildWebHost()`和访问`IWebHost.Services`属性。

> [!NOTE]
> 在创建新的 ASP.NET Core 2.0 应用程序时，默认情况下包含此挂钩。 在以前版本的 EF Core 和 ASP.NET Core，工具尝试调用`Startup.ConfigureServices`直接以获取应用程序的服务提供商，但此模式不能再正常 ASP.NET Core 2.0 应用程序中。 如果您正在升级到 2.0 ASP.NET Core 1.x 应用程序，则可以[修改你`Program`类遵循新模式][3]。

`DbContext`本身和其构造函数中的任何依赖项需要注册为应用程序的服务提供程序中的服务。 这可以轻松地实现通过让[上的构造函数`DbContext`采用的实例`DbContextOptions<TContext>`作为参数][ 4]并使用[ `AddDbContext<TContext>` 方法][5].

<a name="using-a-constructor-with-no-parameters"></a>使用不带任何参数的构造函数
--------------------------------------
如果不能从应用程序服务提供商获取 DbContext，这些工具查找派生`DbContext`项目内的类型。 然后，他们尝试使用不带任何参数的构造函数创建实例。 这可以是默认构造函数，如果`DbContext`使用配置[ `OnConfiguring` ] [ 6]方法。

<a name="from-a-design-time-factory"></a>从设计时工厂
--------------------------
此外可以告知工具如何通过实现来创建 DbContext`IDesignTimeDbContextFactory<TContext>`接口： 如果找到实现此接口的类中派生的同一项目`DbContext`或在应用程序的启动项目中，这些工具绕过改为创建 DbContext 和使用的设计时工厂的其他方法。

``` csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Design;
using Microsoft.EntityFrameworkCore.Infrastructure;

namespace MyProject
{
    public class BloggingContextFactory : IDesignTimeDbContextFactory<BloggingContext>
    {
        public BloggingContext CreateDbContext(string[] args)
        {
            var optionsBuilder = new DbContextOptionsBuilder<BloggingContext>();
            optionsBuilder.UseSqlite("Data Source=blog.db");

            return new BloggingContext(optionsBuilder.Options);
        }
    }
}
```

> [!NOTE]
> `args`参数是当前未使用。 没有[问题][ 7]跟踪指定的工具的设计时参数的功能。

设计时工厂可能非常有用，如果您需要以不同的方式为比在运行时的设计时配置 DbContext 如果`DbContext`构造函数采用其他参数不在中注册 DI，如果不使用 DI，或者对于某些原因都不希望`BuildWebHost`方法在 ASP.NET Core 应用程序的`Main`类。

  [1]: xref:core/managing-schemas/migrations/index
  [2]: xref:core/miscellaneous/configuring-dbcontext
  [3]: https://docs.microsoft.com/aspnet/core/migration/1x-to-2x/#update-main-method-in-programcs
  [4]: xref:core/miscellaneous/configuring-dbcontext#constructor-argument
  [5]: xref:core/miscellaneous/configuring-dbcontext#using-dbcontext-with-dependency-injection
  [6]: xref:core/miscellaneous/configuring-dbcontext#onconfiguring
  [7]: https://github.com/aspnet/EntityFrameworkCore/issues/8332
