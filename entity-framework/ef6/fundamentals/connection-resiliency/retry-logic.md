---
title: 连接复原和重试逻辑-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 47d68ac1-927e-4842-ab8c-ed8c8698dff2
ms.openlocfilehash: 09ebed18b43f864af36b6931f45638f3a3056229
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490800"
---
# <a name="connection-resiliency-and-retry-logic"></a>连接复原和重试逻辑
> [!NOTE]
> **仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。 如果使用的是早期版本，则部分或全部信息不适用。  

连接到数据库服务器应用程序始终都容易受到由于后端故障和网络不稳定连接中断。 但是，在基于 LAN 环境中使用专用的数据库服务器上的这些错误很少，额外的逻辑来处理这些故障通常是不需要。 与云的兴起现在是更常见的连接中断发生不可靠网络上的基于数据库服务器，例如 Windows Azure SQL 数据库和连接。 这可能是由于防御技术，云数据库使用，以确保服务，如连接限制，或导致间歇性超时和其他暂时性错误的网络中的不稳定的公平性。  

连接复原能力是指 EF 会自动重试会由于这些连接中断而失败的任何命令的能力。  

## <a name="execution-strategies"></a>执行策略  

连接重试是由负责 IDbExecutionStrategy 接口的实现。 IDbExecutionStrategy 的实现将负责接受操作，如果并发生异常，确定是否适合重试它是否正在重试。 有四种随 EF 执行策略：  

1. **DefaultExecutionStrategy**： 此执行策略不会重试任何操作，它是 sql server 以外的数据库的默认值。  
2. **DefaultSqlExecutionStrategy**： 这是默认情况下使用的内部执行策略。 此策略不会重根本试，但是，它会换行可能是暂时性通知，告知他们可能想要启用连接复原能力的用户的任何异常。  
3. **DbExecutionStrategy**： 此类是适用于其他执行策略，包括你自己的自定义的基类。 它实现的指数重试策略，其中初始重试之前不会发生与零延迟和延迟会增加指数级增长达到最大重试计数。 此类具有可以在派生的执行策略以控制哪些异常应重试中实现的抽象 ShouldRetryOn 方法。  
4. **SqlAzureExecutionStrategy**： 此执行策略继承自 DbExecutionStrategy，在已知可使用 Azure SQL 数据库时可能是暂时性异常时将重试。

> [!NOTE]
> 附带了 EF，位于 EntityFramework.SqlServer 程序集中的 Sql Server 提供程序中包含的执行策略 2 和 4 和用于处理 SQL Server。  

## <a name="enabling-an-execution-strategy"></a>启用执行策略  

告诉 EF 使用执行策略的最简单方法是使用 SetExecutionStrategy 方法的[DbConfiguration](~/ef6/fundamentals/configuring/code-based.md)类：  

``` csharp
public class MyConfiguration : DbConfiguration
{
    public MyConfiguration()
    {
        SetExecutionStrategy("System.Data.SqlClient", () => new SqlAzureExecutionStrategy());
    }
}
```  

此代码告知 EF 时要使用 SqlAzureExecutionStrategy 连接到 SQL Server。  

## <a name="configuring-the-execution-strategy"></a>配置执行策略  

SqlAzureExecutionStrategy 的构造函数可以接受两个参数，MaxRetryCount 和 MaxDelay。 MaxRetry 计数为该策略将重试的最大次数。 MaxDelay 是表示将使用执行策略的重试之间的最大延迟时间跨度。  

若要设置重试的最大数目为 1 且为 30 秒的最大延迟应像 execue 以下：  

``` csharp
public class MyConfiguration : DbConfiguration
{
    public MyConfiguration()
    {
        SetExecutionStrategy(
            "System.Data.SqlClient",
            () => new SqlAzureExecutionStrategy(1, TimeSpan.FromSeconds(30)));
    }
}
```  

第一次一个暂时性的错误发生，但将时间间隔再每次重试，直到任一的最大重试次数限制超过了或的总时间达到的最大延迟时间立即，SqlAzureExecutionStrategy 将重试。  

将仅在有限的数量的异常通常是 tansient 重试执行策略，您将仍需要处理其他错误，以及所捕获这种情况的错误不是暂时性或花很长时间来解决 RetryLimitExceeded 异常本身。  

使用重试执行策略时，有一些已知限制：  

## <a name="streaming-queries-are-not-supported"></a>不支持流式处理查询  

默认情况下，EF6 和更高版本进行缓冲处理的查询结果，而不是流式处理它们。 如果想要的结果流式传输你可以使用 AsStreaming 方法更改 LINQ to Entities 查询到流式处理。  

``` csharp
using (var db = new BloggingContext())
{
    var query = (from b in db.Blogs
                orderby b.Url
                select b).AsStreaming();
    }
}
```  

注册正在重试执行策略时，不支持流式处理。 存在此限制，因为连接无法期间删除通过所返回的结果。 此操作时，EF 需要重新运行整个查询，但具有可靠的方法知道已返回的结果 （数据可能已更改自初始查询的发送，结果可能会返回不同的顺序，结果可能不具有唯一标识符等等。)。  

## <a name="user-initiated-transactions-are-not-supported"></a>不支持用户启动事务  

已配置导致重试执行策略，时使用的事务存在一些限制。  

默认情况下，EF 将执行的事务中任何数据库更新。 无需执行任何操作即可启用此功能，EF 始终自动执行此操作。  

例如，下面的代码中 SaveChanges 自动在事务内执行。 如果 SaveChanges 后插入一个新的站点，则会回滚事务并不更改应用到数据库失败。 上下文还处于允许 SaveChanges 以再次调用以重试将更改应用的状态。  

``` csharp
using (var db = new BloggingContext())
{
    db.Blogs.Add(new Site { Url = "http://msdn.com/data/ef" });
    db.Blogs.Add(new Site { Url = "http://blogs.msdn.com/adonet" });
    db.SaveChanges();
}
```  

不使用重试执行策略时可以在单个事务中包装多个操作。 例如，下面的代码将两个 SaveChanges 调用包装在单个事务中。 如果这两种操作的任何部分然后失败的任何更改会应用。  

``` csharp
using (var db = new BloggingContext())
{
    using (var trn = db.Database.BeginTransaction())
    {
        db.Blogs.Add(new Site { Url = "http://msdn.com/data/ef" });
        db.Blogs.Add(new Site { Url = "http://blogs.msdn.com/adonet" });
        db.SaveChanges();

        db.Blogs.Add(new Site { Url = "http://twitter.com/efmagicunicorns" });
        db.SaveChanges();

        trn.Commit();
    }
}
```  

仅使用重试执行策略，因为 EF 不知道的任何先前的操作以及如何重试时，不支持此操作。 例如，如果第二个 SaveChanges 未通过，则 EF 无法再具有所需的信息以第一次的 SaveChanges 调用重试。  

### <a name="workaround-suspend-execution-strategy"></a>解决方法： 挂起执行策略  

一种解决方法是代码的挂起的需要使用用户段的重试执行策略启动的事务。 若要执行此操作的最简单方法是添加到你的代码的 SuspendExecutionStrategy 标志基于配置类和更改执行策略 lambda 时要返回的默认 (非 retying) 执行策略设置的标志。  

``` csharp
using System.Data.Entity;
using System.Data.Entity.Infrastructure;
using System.Data.Entity.SqlServer;
using System.Runtime.Remoting.Messaging;

namespace Demo
{
    public class MyConfiguration : DbConfiguration
    {
        public MyConfiguration()
        {
            this.SetExecutionStrategy("System.Data.SqlClient", () => SuspendExecutionStrategy
              ? (IDbExecutionStrategy)new DefaultExecutionStrategy()
              : new SqlAzureExecutionStrategy());
        }

        public static bool SuspendExecutionStrategy
        {
            get
            {
                return (bool?)CallContext.LogicalGetData("SuspendExecutionStrategy")  false;
            }
            set
            {
                CallContext.LogicalSetData("SuspendExecutionStrategy", value);
            }
        }
    }
}
```  

请注意，我们将使用 CallContext 来存储标志值。 这提供了线程本地存储与类似的功能，但安全地使用与异步代码-包括异步查询和保存与实体框架。  

我们现在可以暂停使用用户启动的事务的代码部分的执行策略。  

``` csharp
using (var db = new BloggingContext())
{
    MyConfiguration.SuspendExecutionStrategy = true;

    using (var trn = db.Database.BeginTransaction())
    {
        db.Blogs.Add(new Blog { Url = "http://msdn.com/data/ef" });
        db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/adonet" });
        db.SaveChanges();

        db.Blogs.Add(new Blog { Url = "http://twitter.com/efmagicunicorns" });
        db.SaveChanges();

        trn.Commit();
    }

    MyConfiguration.SuspendExecutionStrategy = false;
}
```  

### <a name="workaround-manually-call-execution-strategy"></a>解决方法： 手动调用执行策略  

另一个选项是手动使用执行策略并为其提供逻辑来运行，整个集，以便它可以重试的所有内容如果其中一个操作失败。 我们仍需要挂起执行策略的使用方法，以便在可重试代码块内使用任何上下文不尝试重试-上面所示。  

请注意，应在代码块中将重试构造任何上下文。 这可确保我们开始使用干净状态，以便每次重试。  

``` csharp
var executionStrategy = new SqlAzureExecutionStrategy();

MyConfiguration.SuspendExecutionStrategy = true;

executionStrategy.Execute(
    () =>
    {
        using (var db = new BloggingContext())
        {
            using (var trn = db.Database.BeginTransaction())
            {
                db.Blogs.Add(new Blog { Url = "http://msdn.com/data/ef" });
                db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/adonet" });
                db.SaveChanges();

                db.Blogs.Add(new Blog { Url = "http://twitter.com/efmagicunicorns" });
                db.SaveChanges();

                trn.Commit();
            }
        }
    });

MyConfiguration.SuspendExecutionStrategy = false;
```  
