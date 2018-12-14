---
title: 使用事务-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 0d0f1824-d781-4cb3-8fda-b7eaefced1cd
ms.openlocfilehash: 7197733ab25c8475746e7863963384730919e3ff
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489773"
---
# <a name="working-with-transactions"></a>使用事务
> [!NOTE]
> **仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。 如果使用的是早期版本，则部分或全部信息不适用。  

本文档将介绍使用 EF6 中的事务包括我们添加了自 EF5 来简化使用事务的增强功能。  

## <a name="what-ef-does-by-default"></a>EF 默认情况下的用途  

在所有版本的实体框架中，每次执行**savechanges （)** 若要插入、 更新或删除对数据库框架将包装该操作在事务中。 此事务持续仅足够长的时间以执行此操作，然后完成。 当您执行另一个此类操作时被启动新事务。  

从 EF6 **Database.ExecuteSqlCommand()** 默认情况下将自动换行命令在事务中如果尚未存在。 没有可用于重写此行为，如果您希望此方法的重载。 此外会在 EF6 执行如通过 Api 模型中包含的存储过程**ObjectContext.ExecuteFunction()** （不过，可以覆盖默认行为不能在时刻） 执行相同的操作。  

在任一情况下，事务的隔离级别是任何隔离级别的数据库提供程序认为其默认设置。 默认情况下，例如，SQL Server 上这是 READ COMMITTED。  

不，实体框架不会将查询包装在事务中。  

此默认功能适合大量用户，如果因此，无需执行任何差异 EF6; 中如往常一样，只需编写的代码。  

但是，一些用户需要更好地控制他们的事务 – 以下各节中对此进行了。  

## <a name="how-the-apis-work"></a>Api 工作原理  

在 EF6 实体框架之前他们坚持认为在打开数据库连接本身 （如果它已处于已打开的连接，它引发了异常）。 由于只能对打开的连接启动事务，这意味着用户可以包装到一个事务的多个操作的唯一方式是使用[TransactionScope](https://msdn.microsoft.com/library/system.transactions.transactionscope.aspx)或使用**ObjectContext.Connection**属性并开始调用**open （)** 并**BeginTransaction()** 直接对返回**EntityConnection**对象。 此外，如果你在自己的基础数据库连接上启动事务，就会失败的联系数据库 API 调用。  

> [!NOTE]
> 仅接受关闭的连接的限制已在 Entity Framework 6 中删除。 有关详细信息，请参阅[连接管理](~/ef6/fundamentals/connection-management.md)。  

从开始 EF6 framework 现在提供：  

1. **Database.BeginTransaction()** ： 用户可以启动和完成事务本身内现有的 DbContext – 来组合在同一事务中的多个操作，从而更简单的方法，因此所有已提交或所有回滚作为其中一个。 它还允许用户更轻松地指定事务的隔离级别。  
2. **Database.UseTransaction()** ： 它允许使用 Entity Framework 外部启动的事务的 DbContext。  

### <a name="combining-several-operations-into-one-transaction-within-the-same-context"></a>将组合到一个事务在同一上下文中的多个操作  

**Database.BeginTransaction()** 具有两个替代 – 其中一个采用显式[IsolationLevel](https://msdn.microsoft.com/library/system.data.isolationlevel.aspx) ，另一个不采用任何参数，并使用默认隔离级别从基础数据库提供程序。 这两个重写返回**DbContextTransaction**对象提供**commit （)** 并**rollback （)** 对基础存储区中执行 commit 和 rollback 方法事务。  

**DbContextTransaction**旨在被释放后提交或回滚。 一种简单方法完成此操作是**using(...){...}** 将自动调用的语法**dispose （)** using 代码块时完成：  

``` csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.SqlClient;
using System.Linq;
using System.Transactions;

namespace TransactionsExamples
{
    class TransactionsExample
    {
        static void StartOwnTransactionWithinContext()
        {
            using (var context = new BloggingContext())
            {
                using (var dbContextTransaction = context.Database.BeginTransaction())
                {
                    try
                    {
                        context.Database.ExecuteSqlCommand(
                            @"UPDATE Blogs SET Rating = 5" +
                                " WHERE Name LIKE '%Entity Framework%'"
                            );

                        var query = context.Posts.Where(p => p.Blog.Rating >= 5);
                        foreach (var post in query)
                        {
                            post.Title += "[Cool Blog]";
                        }

                        context.SaveChanges();

                        dbContextTransaction.Commit();
                    }
                    catch (Exception)
                    {
                        dbContextTransaction.Rollback();
                    }
                }
            }
        }
    }
}
```  

> [!NOTE]
> 开始事务需要基础存储区连接打开。 如果尚未打开，因此调用 Database.BeginTransaction() 将打开连接。 如果 DbContextTransaction 打开连接然后它将关闭它调用 dispose （） 时。  

### <a name="passing-an-existing-transaction-to-the-context"></a>将现有事务传递到上下文  

有时您希望即作用域中更广泛和完全包括在同一数据库，但在 EF 外部操作的事务。 若要完成此操作必须打开的连接和自行启动事务，然后告诉 EF） 若要将已打开数据库连接和 b） 到该连接上使用现有事务。  

若要执行此操作必须定义并在上下文类，该类继承自 DbContext 构造函数采用 i） 现有的连接参数和 ii) contextOwnsConnection 布尔之一上使用的构造函数。  

> [!NOTE]
> ContextOwnsConnection 标志必须设置为 false 时调用在此方案中。 这是非常重要，因为它会通知 Entity Framework，它不应关闭连接时做了 （有关示例，请参阅以下 4 行）：  

``` csharp
using (var conn = new SqlConnection("..."))
{
    conn.Open();
    using (var context = new BloggingContext(conn, contextOwnsConnection: false))
    {
    }
}
```  

此外，您必须自己 （如果你想要避免的默认设置，包括 IsolationLevel） 启动事务，并允许实体框架知道已启动的连接上的现有事务 （请参阅下面的 33 行）。  

然后，你可以随意执行数据库操作，直接对 SqlConnection 本身，或在 DbContext。 在一个事务内执行所有此类操作。 你可以提交或回滚事务和调用 dispose （），以及关闭和释放数据库连接。 例如：  

``` csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.SqlClient;
using System.Linq;
sing System.Transactions;

namespace TransactionsExamples
{
     class TransactionsExample
     {
        static void UsingExternalTransaction()
        {
            using (var conn = new SqlConnection("..."))
            {
               conn.Open();

               using (var sqlTxn = conn.BeginTransaction(System.Data.IsolationLevel.Snapshot))
               {
                   try
                   {
                       var sqlCommand = new SqlCommand();
                       sqlCommand.Connection = conn;
                       sqlCommand.Transaction = sqlTxn;
                       sqlCommand.CommandText =
                           @"UPDATE Blogs SET Rating = 5" +
                            " WHERE Name LIKE '%Entity Framework%'";
                       sqlCommand.ExecuteNonQuery();

                       using (var context =  
                         new BloggingContext(conn, contextOwnsConnection: false))
                        {
                            context.Database.UseTransaction(sqlTxn);

                            var query =  context.Posts.Where(p => p.Blog.Rating >= 5);
                            foreach (var post in query)
                            {
                                post.Title += "[Cool Blog]";
                            }
                           context.SaveChanges();
                        }

                        sqlTxn.Commit();
                    }
                    catch (Exception)
                    {
                        sqlTxn.Rollback();
                    }
                }
            }
        }
    }
}
```  

### <a name="clearing-up-the-transaction"></a>清理事务

您可以将 null 传递到 Database.UseTransaction() 以清除当前事务的实体框架的知识。 实体框架将既不提交或回滚现有事务时执行此操作，因此请谨慎使用，并仅当您确定这是你想要执行操作。  

### <a name="errors-in-usetransaction"></a>UseTransaction 中的错误

如果传递的事务，则会看到从 Database.UseTransaction() 异常时：  
- 实体框架已具有现有事务  
- 在 TransactionScope 内已运行实体框架  
- 传入的事务中的连接对象为 null。 也就是说，该事务不是与连接相关联 – 通常这是一个信号，该事务已完成  
- 传入的事务中的连接对象与实体框架的连接不匹配。  

## <a name="using-transactions-with-other-features"></a>与其他功能配合使用事务  

本部分详细介绍上述的事务与交互的方式：  

- 连接复原  
- 异步方法  
- TransactionScope 事务  

### <a name="connection-resiliency"></a>连接弹性  

新的连接复原功能并不适用于用户启动事务。 有关详细信息，请参阅[重试执行策略](~/ef6/fundamentals/connection-resiliency/retry-logic.md#user-initiated-transactions-are-not-supported)。  

### <a name="asynchronous-programming"></a>异步编程  

前面几节中列出的方法需要任何进一步的选项或设置，以使用[异步查询和保存方法](~/ef6/fundamentals/async.md
)。 但请注意，具体取决于您做什么异步方法中，这可能会导致长时间运行的事务 – 这又可能导致死锁或阻止这错误的整个应用程序的性能。  

### <a name="transactionscope-transactions"></a>TransactionScope 事务  

在 EF6 之前提供较大范围的事务的推荐的方法是使用 TransactionScope 对象：  

``` csharp
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.SqlClient;
using System.Linq;
using System.Transactions;

namespace TransactionsExamples
{
    class TransactionsExample
    {
        static void UsingTransactionScope()
        {
            using (var scope = new TransactionScope(TransactionScopeOption.Required))
            {
                using (var conn = new SqlConnection("..."))
                {
                    conn.Open();

                    var sqlCommand = new SqlCommand();
                    sqlCommand.Connection = conn;
                    sqlCommand.CommandText =
                        @"UPDATE Blogs SET Rating = 5" +
                            " WHERE Name LIKE '%Entity Framework%'";
                    sqlCommand.ExecuteNonQuery();

                    using (var context =
                        new BloggingContext(conn, contextOwnsConnection: false))
                    {
                        var query = context.Posts.Where(p => p.Blog.Rating > 5);
                        foreach (var post in query)
                        {
                            post.Title += "[Cool Blog]";
                        }
                        context.SaveChanges();
                    }
                }

                scope.Complete();
            }
        }
    }
}
```  

SqlConnection 和 Entity Framework 会同时使用环境的 TransactionScope 事务，因此是已提交到一起。  

从.NET 4.5.1 TransactionScope 已更新为也适用于通过使用异步方法[TransactionScopeAsyncFlowOption](https://msdn.microsoft.com/library/system.transactions.transactionscopeasyncflowoption.aspx)枚举：  

``` csharp
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.SqlClient;
using System.Linq;
using System.Transactions;

namespace TransactionsExamples
{
    class TransactionsExample
    {
        public static void AsyncTransactionScope()
        {
            using (var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
            {
                using (var conn = new SqlConnection("..."))
                {
                    await conn.OpenAsync();

                    var sqlCommand = new SqlCommand();
                    sqlCommand.Connection = conn;
                    sqlCommand.CommandText =
                        @"UPDATE Blogs SET Rating = 5" +
                            " WHERE Name LIKE '%Entity Framework%'";
                    await sqlCommand.ExecuteNonQueryAsync();

                    using (var context = new BloggingContext(conn, contextOwnsConnection: false))
                    {
                        var query = context.Posts.Where(p => p.Blog.Rating > 5);
                        foreach (var post in query)
                        {
                            post.Title += "[Cool Blog]";
                        }

                        await context.SaveChangesAsync();
                    }
                }
            }
        }
    }
}
```  

仍有一些限制 TransactionScope 方法：  

- 需要.NET 4.5.1 或更高版本才能使用异步方法。  
- 它不能使用云方案除非您是确保你具有一个且只有一个连接 （云方案不支持分布式的事务）。  
- 它不能结合前面部分的 Database.UseTransaction() 方法。  
- 如果发出任何 DDL，未启用通过 MSDTC 服务的分布式的事务，它将引发异常。  

TransactionScope 方法的优点：  

- 它将自动升级本地事务为分布式事务如果建立到给定数据库的多个连接或连接到不同的数据库在同一事务中合并到一个数据库的连接 (请注意： 必须具有MSDTC 服务配置为允许为实现此目的的分布式的事务）。  
- 编码难度。 如果想要环境和已发的牌与后台中隐式事务，而不是显式下控制然后 TransactionScope 方法可能比较适合更好。  

总之，使用新的 Database.BeginTransaction() 和 Database.UseTransaction() Api 更高版本，TransactionScope 方法不再需要对于大多数用户。 如果不要继续使用 TransactionScope 然后请注意上述限制。 我们建议使用列出的方法前面各节中改为在可能的情况。  
