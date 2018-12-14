---
title: 异步查询和保存的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: d56e6f1d-4bd1-4b50-9558-9a30e04a8ec3
ms.openlocfilehash: 4ed4f5c13341f33ccff8325a5ddacd8f7b195a76
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283818"
---
# <a name="async-query-and-save"></a>异步查询和保存
> [!NOTE]
> **仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。 如果使用的是早期版本，则部分或全部信息不适用。

EF6 引入了异步查询和保存使用的支持[async 和 await 关键字](https://msdn.microsoft.com/library/vstudio/hh191443.aspx).NET 4.5 中引入的。 虽然并非所有应用程序可能会受益于异步功能，它可以用于处理长时间运行、 网络或 O 绑定任务时提高客户端响应能力和服务器可伸缩性。

## <a name="when-to-really-use-async"></a>何时真正使用异步

本演练的目的是介绍了异步概念的方式，轻松地观察异步和同步计划执行之间的差异。 本演练不是为了说明的重要方案的任何异步编程优势。

异步编程是主要侧重于释放当前托管线程 （线程正在运行的.NET 代码） 来执行其他工作，等待不需要任何计算时间的操作时从托管线程。 例如，尽管数据库引擎处理查询时没有要通过.NET 代码。

在客户端应用程序 （WinForms、 WPF 等） 中可以使用当前线程来执行异步操作时保持 UI 响应能力。 在服务器应用程序 （ASP.NET 等） 可以使用线程以处理其他传入的请求-这可以减少内存使用量和/或增加服务器的吞吐量。

在大多数应用程序使用 async 会有任何明显的好处，甚至可能是造成不利影响。 使用测试、 分析和常识来提交前测量在特定方案的 async 的影响。

下面是一些更多资源，以了解异步：

-   [Brandon Bray 的 async/await.NET 4.5 中的概述](https://blogs.msdn.com/b/dotnet/archive/2012/04/03/async-in-4-5-worth-the-await.aspx)
-   [异步编程](https://msdn.microsoft.com/library/hh191443.aspx)MSDN Library 中的页
-   [如何生成 ASP.NET Web 应用程序使用 Async](http://channel9.msdn.com/events/teched/northamerica/2013/dev-b337) （包括提高的服务器的吞吐量的演示）

## <a name="create-the-model"></a>创建模型

我们将使用[Code First 工作流](~/ef6/modeling/code-first/workflows/new-database.md)来创建我们的模型，但异步功能将使用包括那些使用 EF 设计器创建的所有 EF 模型生成数据库。

-   创建一个控制台应用程序并对其**AsyncDemo**
-   添加 EntityFramework NuGet 包
    -   在解决方案资源管理器，右键单击**AsyncDemo**项目
    -   选择**管理 NuGet 包...**
    -   在管理 NuGet 包对话框中，选择**联机**选项卡，选择**EntityFramework**包
    -   单击**安装**
-   添加**Model.cs**类具有以下实现

``` csharp
    using System.Collections.Generic;
    using System.Data.Entity;

    namespace AsyncDemo
    {
        public class BloggingContext : DbContext
        {
            public DbSet<Blog> Blogs { get; set; }
            public DbSet<Post> Posts { get; set; }
        }

        public class Blog
        {
            public int BlogId { get; set; }
            public string Name { get; set; }

            public virtual List<Post> Posts { get; set; }
        }

        public class Post
        {
            public int PostId { get; set; }
            public string Title { get; set; }
            public string Content { get; set; }

            public int BlogId { get; set; }
            public virtual Blog Blog { get; set; }
        }
    }
```

 

## <a name="create-a-synchronous-program"></a>创建一个同步程序，

现在，我们已有 EF 模型，让我们编写一些代码来使用它来执行某些数据访问。

-   内容替换为**Program.cs**用下面的代码

``` csharp
    using System;
    using System.Linq;

    namespace AsyncDemo
    {
        class Program
        {
            static void Main(string[] args)
            {
                PerformDatabaseOperations();

                Console.WriteLine();
                Console.WriteLine("Quote of the day");
                Console.WriteLine(" Don't worry about the world coming to an end today... ");
                Console.WriteLine(" It's already tomorrow in Australia.");

                Console.WriteLine();
                Console.WriteLine("Press any key to exit...");
                Console.ReadKey();
            }

            public static void PerformDatabaseOperations()
            {
                using (var db = new BloggingContext())
                {
                    // Create a new blog and save it
                    db.Blogs.Add(new Blog
                    {
                        Name = "Test Blog #" + (db.Blogs.Count() + 1)
                    });
                    db.SaveChanges();

                    // Query for all blogs ordered by name
                    var blogs = (from b in db.Blogs
                                orderby b.Name
                                select b).ToList();

                    // Write all blogs out to Console
                    Console.WriteLine();
                    Console.WriteLine("All blogs:");
                    foreach (var blog in blogs)
                    {
                        Console.WriteLine(" " + blog.Name);
                    }
                }
            }
        }
    }
```

此代码将调用**PerformDatabaseOperations**方法，从而节省了一个新**博客**到数据库，然后检索所有**博客**从数据库，并输出到**控制台**。 在此之后，程序写入到一天中的引号**控制台**。

因为代码是同步的我们可以观察到以下的执行流，当我们运行程序：

1.  **SaveChanges**开始推送新**博客**到数据库
2.  **SaveChanges**完成
3.  所有查询**博客**发送到数据库
4.  查询返回，而结果将写入到**控制台**
5.  一天中的引号写入到**控制台**

![同步输出](~/ef6/media/syncoutput.png) 

 

## <a name="making-it-asynchronous"></a>使其异步

现在，我们将我们的程序启动并运行，我们可以开始使用新的 async 和 await 关键字。 我们已对 Program.cs 所做以下更改

1.  第 2 行： using 语句**System.Data.Entity**命名空间提供了我们访问到 EF 异步扩展方法。
2.  第 4 行： using 语句**System.Threading.Tasks**命名空间，我们便使用**任务**类型。
3.  第 12 和 18 行： 我们捕获作为任务，用于监视的进度**PerformSomeDatabaseOperations** （第 12 行），然后阻止此程序执行任务为完成一次的所有工作程序完成 （第 18 行）。
4.  第 25 行： 我们已更新**PerformSomeDatabaseOperations**要标记为已**异步**并返回**任务**。
5.  行 35： 我们现在调用 SaveChanges 的异步版本和等待其完成。
6.  行 42： 我们现在调用 ToList 的异步版本和正等待的结果。

有关 system.data.entity 的引用命名空间中可用的扩展方法的完整列表，请参阅 QueryableExtensions 类。 *您还需要添加"using System.Data.Entity"到语句。*

``` csharp
    using System;
    using System.Data.Entity;
    using System.Linq;
    using System.Threading.Tasks;

    namespace AsyncDemo
    {
        class Program
        {
            static void Main(string[] args)
            {
                var task = PerformDatabaseOperations();

                Console.WriteLine("Quote of the day");
                Console.WriteLine(" Don't worry about the world coming to an end today... ");
                Console.WriteLine(" It's already tomorrow in Australia.");

                task.Wait();

                Console.WriteLine();
                Console.WriteLine("Press any key to exit...");
                Console.ReadKey();
            }

            public static async Task PerformDatabaseOperations()
            {
                using (var db = new BloggingContext())
                {
                    // Create a new blog and save it
                    db.Blogs.Add(new Blog
                    {
                        Name = "Test Blog #" + (db.Blogs.Count() + 1)
                    });
                    Console.WriteLine("Calling SaveChanges.");
                    await db.SaveChangesAsync();
                    Console.WriteLine("SaveChanges completed.");

                    // Query for all blogs ordered by name
                    Console.WriteLine("Executing query.");
                    var blogs = await (from b in db.Blogs
                                orderby b.Name
                                select b).ToListAsync();

                    // Write all blogs out to Console
                    Console.WriteLine("Query completed with following results:");
                    foreach (var blog in blogs)
                    {
                        Console.WriteLine(" - " + blog.Name);
                    }
                }
            }
        }
    }
```

现在，代码为异步，我们可以观察到不同的执行流，当我们运行程序：

1.  **SaveChanges**开始推送新**博客**到数据库*后将命令发送到该数据库没有更多计算当前托管线程上所需时间。**PerformDatabaseOperations**方法将返回 （即使它尚未完成执行），在 Main 方法中的程序流将继续。*
2.  **一天中的引号写入到控制台**
    *由于没有更多工作要做 Main 方法中，托管的线程被阻止在等待上调用，直到数据库操作完成。完成后，其余我们**PerformDatabaseOperations** *将执行。
3.  **SaveChanges**完成
4.  所有查询**博客**发送到数据库*同样，托管的线程是免费的数据库中处理查询时执行其他工作。由于所有其他执行完成后，该线程将只需暂停等待调用不过。*
5.  查询返回，而结果将写入到**控制台**

![异步输出](~/ef6/media/asyncoutput.png) 

 

## <a name="the-takeaway"></a>要点

我们现在已了解以使是多么的 EF 的异步方法使用。 尽管异步的优点可能不能使用简单的控制台应用程序非常明显，但可以在其中长时间运行或网络密集型活动可能会否则为阻止应用程序，或者导致大量的线程的情况下应用这些相同的策略增加内存占用量。
