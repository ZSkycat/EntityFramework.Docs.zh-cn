---
title: 连接管理-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: ecaa5a27-b19e-4bf9-8142-a3fb00642270
ms.openlocfilehash: a6352bbbc38c38bd5f30536736ec969056df2c7d
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489331"
---
# <a name="connection-management"></a>连接管理
此页介绍实体框架的行为将连接传递给上下文和功能方面**Database.Connection.Open()** API。  

## <a name="passing-connections-to-the-context"></a>传递到上下文的连接  

### <a name="behavior-for-ef5-and-earlier-versions"></a>EF5 和较早版本的行为  

有两个构造函数接受连接：  

``` csharp
public DbContext(DbConnection existingConnection, bool contextOwnsConnection)
public DbContext(DbConnection existingConnection, DbCompiledModel model, bool contextOwnsConnection)
```  

可以使用这些，但必须先解决了几个限制：  

1. 如果传递的打开连接到其中一种然后第一次该框架将尝试使用它引发 InvalidOperationException 说它无法重新打开已打开的连接。  
2. ContextOwnsConnection 标志被解释为表示在释放上下文时应释放基础存储区连接。 但是，无论启用该设置存储连接始终关闭时释放上下文。 因此如果必须使用相同的连接的多个 DbContext 任何上下文被处置首先将关闭连接 （同样如果具有混合的现有的 ADO.NET 连接，使用 DbContext，DbContext 将始终关闭连接时它已释放）.  

就可以通过传递已关闭的连接，并仅执行已创建的所有上下文后会打开它的代码解决上面的第一个限制：  

``` csharp
using System.Collections.Generic;
using System.Data.Common;
using System.Data.Entity;
using System.Data.Entity.Infrastructure;
using System.Data.EntityClient;
using System.Linq;

namespace ConnectionManagementExamples
{
    class ConnectionManagementExampleEF5
    {         
        public static void TwoDbContextsOneConnection()
        {
            using (var context1 = new BloggingContext())
            {
                var conn =
                    ((EntityConnection)  
                        ((IObjectContextAdapter)context1).ObjectContext.Connection)  
                            .StoreConnection;

                using (var context2 = new BloggingContext(conn, contextOwnsConnection: false))
                {
                    context2.Database.ExecuteSqlCommand(
                        @"UPDATE Blogs SET Rating = 5" +
                        " WHERE Name LIKE '%Entity Framework%'");

                    var query = context1.Posts.Where(p => p.Blog.Rating > 5);
                    foreach (var post in query)
                    {
                        post.Title += "[Cool Blog]";
                    }
                    context1.SaveChanges();
                }
            }
        }
    }
}
```  

第二个限制只是意味着您需要避免在您准备好要关闭的连接之前释放任何 DbContext 对象。  

### <a name="behavior-in-ef6-and-future-versions"></a>EF6 和未来版本中的行为  

在 EF6 和未来版本 DbContext 具有相同的两个构造函数，但不再需要时接收到关闭的连接传递给构造函数。 因此，这是现在可以：  

``` csharp
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.SqlClient;
using System.Linq;
using System.Transactions;

namespace ConnectionManagementExamples
{
    class ConnectionManagementExample
    {
        public static void PassingAnOpenConnection()
        {
            using (var conn = new SqlConnection("{connectionString}"))
            {
                conn.Open();

                var sqlCommand = new SqlCommand();
                sqlCommand.Connection = conn;
                sqlCommand.CommandText =
                    @"UPDATE Blogs SET Rating = 5" +
                     " WHERE Name LIKE '%Entity Framework%'";
                sqlCommand.ExecuteNonQuery();

                using (var context = new BloggingContext(conn, contextOwnsConnection: false))
                {
                    var query = context.Posts.Where(p => p.Blog.Rating > 5);
                    foreach (var post in query)
                    {
                        post.Title += "[Cool Blog]";
                    }
                    context.SaveChanges();
                }

                var sqlCommand2 = new SqlCommand();
                sqlCommand2.Connection = conn;
                sqlCommand2.CommandText =
                    @"UPDATE Blogs SET Rating = 7" +
                     " WHERE Name LIKE '%Entity Framework Rocks%'";
                sqlCommand2.ExecuteNonQuery();
            }
        }
    }
}
```  

此外 contextOwnsConnection 标志现在可以控制连接是同时关闭和释放时释放 DbContext。 因此在上面的示例连接时不会关闭的上下文是重要的是在以前版本的 EF，但而不是连接本身被释放后释放 （第 32 行） （第 40 行）。  

当然，这是连接 (只需设置 contextOwnsConnection 为 true，或使用其他构造函数之一) 控制对 dbcontext 仍可以根据需要。  

> [!NOTE]
> 使用此新模型中使用事务时，有一些其他注意事项。 有关详细信息，请参阅[使用事务](~/ef6/saving/transactions.md)。  

## <a name="databaseconnectionopen"></a>Database.Connection.Open()  

### <a name="behavior-for-ef5-and-earlier-versions"></a>EF5 和较早版本的行为  

在 EF5 和早期版本中是一个 bug，以便**ObjectContext.Connection.State**未更新以反映基础存储区连接的真实状态。 例如，如果执行下面的代码，可以返回状态**Closed**尽管事实上基础存储连接**打开**。  

``` csharp
((IObjectContextAdapter)context).ObjectContext.Connection.State
```  

另外，如果通过调用 Database.Connection.Open() 打开数据库连接将打开直到下一次执行查询或调用任何内容需要数据库连接 (例如，但在 SaveChanges())，基础存储连接将关闭。 上下文将然后重新打开并重新关闭连接另一个数据库操作是必需的任何时间：  

``` csharp
using System;
using System.Data;
using System.Data.Entity;
using System.Data.Entity.Infrastructure;
using System.Data.EntityClient;

namespace ConnectionManagementExamples
{
    public class DatabaseOpenConnectionBehaviorEF5
    {
        public static void DatabaseOpenConnectionBehavior()
        {
            using (var context = new BloggingContext())
            {
                // At this point the underlying store connection is closed

                context.Database.Connection.Open();

                // Now the underlying store connection is open  
                // (though ObjectContext.Connection.State will report closed)

                var blog = new Blog { /* Blog’s properties */ };
                context.Blogs.Add(blog);

                // The underlying store connection is still open  

                context.SaveChanges();

                // After SaveChanges() the underlying store connection is closed  
                // Each SaveChanges() / query etc now opens and immediately closes
                // the underlying store connection

                blog = new Blog { /* Blog’s properties */ };
                context.Blogs.Add(blog);
                context.SaveChanges();
            }
        }
    }
}
```  

### <a name="behavior-in-ef6-and-future-versions"></a>EF6 和未来版本中的行为  

EF6 和未来版本我们已采取的方法，如果调用代码选择通过调用上下文打开连接。Database.Connection.Open()，则具有充分的理由这样做，框架将假定它想要控制打开和关闭的连接，并将不再自动关闭连接。  

> [!NOTE]
> 这会导致连接这种情况对于长时间，因此使用谨慎处于打开状态。  

我们还更新了代码，以便 ObjectContext.Connection.State 现在跟踪的基础连接的状态正确。  

``` csharp
using System;
using System.Data;
using System.Data.Entity;
using System.Data.Entity.Core.EntityClient;
using System.Data.Entity.Infrastructure;

namespace ConnectionManagementExamples
{
    internal class DatabaseOpenConnectionBehaviorEF6
    {
        public static void DatabaseOpenConnectionBehavior()
        {
            using (var context = new BloggingContext())
            {
                // At this point the underlying store connection is closed

                context.Database.Connection.Open();

                // Now the underlying store connection is open and the
                // ObjectContext.Connection.State correctly reports open too

                var blog = new Blog { /* Blog’s properties */ };
                context.Blogs.Add(blog);
                context.SaveChanges();

                // The underlying store connection remains open for the next operation  

                blog = new Blog { /* Blog’s properties */ };
                context.Blogs.Add(blog);
                context.SaveChanges();

                // The underlying store connection is still open

           } // The context is disposed – so now the underlying store connection is closed
        }
    }
}
```  
