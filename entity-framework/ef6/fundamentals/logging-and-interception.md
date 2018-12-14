---
title: 日志记录和拦截数据库操作的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: b5ee7eb1-88cc-456e-b53c-c67e24c3f8ca
ms.openlocfilehash: 3f06e073f3ab6e46883663620219e302d5db1d60
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490073"
---
# <a name="logging-and-intercepting-database-operations"></a>日志记录和拦截数据库操作
> [!NOTE]
> **仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。 如果使用的是早期版本，则部分或全部信息不适用。  

应用程序代码可以截获从 Entity Framework 6 开始，只要实体框架发出命令到数据库下面的命令。 这最常用于日志记录 SQL，但也可以使用来修改或中止该命令。  

具体而言，EF 包括：  

- 类似于 LINQ to SQL 中 DataContext.Log 上下文日志属性  
- 一种机制，以自定义内容并将输出发送到日志的格式设置  
- 用于侦听提供更大的控制/灵活性的低级构建基块  

## <a name="context-log-property"></a>上下文日志属性  

DbContext.Database.Log 属性可以设置为采用字符串的任何方法的委托。 通常它是与使用任何 TextWriter 通过将其设置为该 TextWriter"Write"方法。 生成的当前上下文的所有 SQL 将都记录到该编写器。 例如，下面的代码会将 SQL 记录到控制台：  

``` csharp
using (var context = new BlogContext())
{
    context.Database.Log = Console.Write;

    // Your code here...
}
```  

请注意，该上下文。Database.Log 设置为 Console.Write。 这是所有所需 SQL 记录到控制台。  

让我们添加一些简单的查询/插入/更新代码，以便我们可以看到一些输出：  

``` csharp
using (var context = new BlogContext())
{
    context.Database.Log = Console.Write;

    var blog = context.Blogs.First(b => b.Title == "One Unicorn");

    blog.Posts.First().Title = "Green Eggs and Ham";

    blog.Posts.Add(new Post { Title = "I do not like them!" });

    context.SaveChangesAsync().Wait();
}
```  

这将生成以下输出：  

``` SQL
SELECT TOP (1)
    [Extent1].[Id] AS [Id],
    [Extent1].[Title] AS [Title]
    FROM [dbo].[Blogs] AS [Extent1]
    WHERE (N'One Unicorn' = [Extent1].[Title]) AND ([Extent1].[Title] IS NOT NULL)
-- Executing at 10/8/2013 10:55:41 AM -07:00
-- Completed in 4 ms with result: SqlDataReader

SELECT
    [Extent1].[Id] AS [Id],
    [Extent1].[Title] AS [Title],
    [Extent1].[BlogId] AS [BlogId]
    FROM [dbo].[Posts] AS [Extent1]
    WHERE [Extent1].[BlogId] = @EntityKeyValue1
-- EntityKeyValue1: '1' (Type = Int32)
-- Executing at 10/8/2013 10:55:41 AM -07:00
-- Completed in 2 ms with result: SqlDataReader

UPDATE [dbo].[Posts]
SET [Title] = @0
WHERE ([Id] = @1)
-- @0: 'Green Eggs and Ham' (Type = String, Size = -1)
-- @1: '1' (Type = Int32)
-- Executing asynchronously at 10/8/2013 10:55:41 AM -07:00
-- Completed in 12 ms with result: 1

INSERT [dbo].[Posts]([Title], [BlogId])
VALUES (@0, @1)
SELECT [Id]
FROM [dbo].[Posts]
WHERE @@ROWCOUNT > 0 AND [Id] = scope_identity()
-- @0: 'I do not like them!' (Type = String, Size = -1)
-- @1: '1' (Type = Int32)
-- Executing asynchronously at 10/8/2013 10:55:41 AM -07:00
-- Completed in 2 ms with result: SqlDataReader
```  

（请注意，这是输出假定已发生的任何数据库初始化。 如果数据库初始化不已发生了则不会有更多输出显示了所有工作迁移实际执行了检查或创建新的数据库。）  

## <a name="what-gets-logged"></a>获取记录内容？  

日志属性时设置以下所有条件，将会记录：  

- SQL 的所有不同类型的命令。 例如：  
    - 查询，包括普通的 LINQ 查询、 eSQL 查询和方法，例如 SqlQuery 从原始查询  
    - 插入、 更新和删除的 SaveChanges 过程中生成的  
    - 加载查询，例如那些由延迟加载生成的关系  
- 参数  
- 在该值指示是否以异步方式执行命令  
- 时间戳，指示该命令时开始执行  
- 指示已成功完成，此命令失败，通过引发异常，或者，对于异步，已取消  
- 结果值的提示  
- 执行命令所用的时间近似量。 请注意，这是将命令发送到取回结果对象的时间。 它不包括读取结果的时间。  

看一下上面的示例输出，每个记录的四个命令是：  

- 从上下文调用生成查询。Blogs.First  
    - 请注意，不会因为此查询起作用的 ToString 方法获取 SQL 的"First"不提供可以在其调用 ToString IQueryable  
- 导致博客延迟加载的查询。文章  
    - 请注意发生的延迟加载的密钥值的参数详细信息  
    - 记录仅设置为非默认值的参数的属性。 例如，大小属性仅显示是否为非零。  
- 两个命令因 SaveChangesAsync;一个用于更新更改文章标题，另一个用于插入操作将添加新文章  
    - 请注意的 FK 和 Title 属性的参数详细信息  
    - 请注意，以异步方式执行这些命令  

## <a name="logging-to-different-places"></a>日志记录到不同的位置  

日志记录到如上所示控制台是非常容易。 它也很容易使用不同登录到内存、 文件等的[TextWriter](https://msdn.microsoft.com/library/system.io.textwriter.aspx)。  

如果您熟悉 LINQ to SQL 可能注意在 LINQ to SQL 日志属性设置为实际 TextWriter 中的对象 (例如，Console.Out) 时的日志属性设置为一个方法接受字符串 （例如 EFConsole.Write 或 Console.Out.Write)。 这样做的原因是 EF 从 TextWriter 分离通过接受任何可以作为字符串的接收器的委托。 例如，假设已有一些日志记录框架，它定义了日志记录方法如下所示：  

``` csharp
public class MyLogger
{
    public void Log(string component, string message)
    {
        Console.WriteLine("Component: {0} Message: {1} ", component, message);
    }
}
```  

这可以挂接到 EF Log 属性如下：  

``` csharp
var logger = new MyLogger();
context.Database.Log = s => logger.Log("EFApp", s);
```  

## <a name="result-logging"></a>结果日志记录  

默认记录器命令文本 (SQL)、 参数和"正在执行"行使用记录时间戳之前的命令发送到数据库。 包含已用时间的"已完成"行是记录的执行以下命令。  

请注意，对于异步命令的"已完成"的行不被记录直到异步任务实际完成、 失败，或被取消。  

在"已完成"的行包含不同的信息，具体取决于命令和已成功执行的类型。  

### <a name="successful-execution"></a>成功执行  

对于成功完成输出的命令是"已完成中的使用结果毫秒 x:"跟以及结果的提示。 指示返回的结果的数据读取器的命令是一种[DbDataReader](https://msdn.microsoft.com/library/system.data.common.dbdatareader.aspx)返回。 返回一个整数值，例如更新的命令对于上面所示的结果所示的命令是该整数。  

### <a name="failed-execution"></a>执行失败  

对于失败的命令都通过引发异常，输出包含来自异常的消息。 例如，使用 SqlQuery 对存在的表的查询将导致日志输出如下：  

``` SQL
SELECT * from ThisTableIsMissing
-- Executing at 5/13/2013 10:19:05 AM
-- Failed in 1 ms with error: Invalid object name 'ThisTableIsMissing'.
```  

### <a name="canceled-execution"></a>已取消的执行  

对于已取消任务的异步命令结果可能会失败并引发异常，因为这是基础 ADO.NET 提供程序通常的用途时尝试取消。 如果不会出现此问题，则输出将如下所示完全取消任务：  

```  
update Blogs set Title = 'No' where Id = -1
-- Executing asynchronously at 5/13/2013 10:21:10 AM
-- Canceled in 1 ms
```  

## <a name="changing-log-content-and-formatting"></a>更改日志内容和格式设置  

在后台 Database.Log 属性，则可以使用的 DatabaseLogFormatter 对象。 此对象有效地将 IDbCommandInterceptor 实现 （见下文） 绑定到一个委托，它接受字符串和 DbContext。 这意味着 DatabaseLogFormatter 上的方法通过 EF 调用之前和之后执行命令。 这些 DatabaseLogFormatter 方法收集和设置日志输出的格式并将其发送给该委托。  

### <a name="customizing-databaselogformatter"></a>自定义 DatabaseLogFormatter  

更改记录的内容和它的格式可以通过创建从 DatabaseLogFormatter 派生而来并重写相应的方法的新类来实现。 若要重写的最常见方法是：  

- LogCommand – 替代此设置以更改在执行前如何记录命令。 默认情况下 LogCommand LogParameter 调用为每个参数;您可以选择执行重写中相同的操作或改为以不同方式处理参数。  
- LogResult – 替代此设置以更改如何记录从执行命令的结果。  
- LogParameter – 替代此设置以更改格式设置和参数日志记录的内容。  

例如，假设我们想要记录单个行，每个命令发送到数据库之前。 这可以通过两个替代：  

- 重写 LogCommand 要格式化和写入单个 SQL 行  
- 重写 LogResult 不执行任何操作。  

该代码将如下所示：

``` csharp
public class OneLineFormatter : DatabaseLogFormatter
{
    public OneLineFormatter(DbContext context, Action<string> writeAction)
        : base(context, writeAction)
    {
    }

    public override void LogCommand<TResult>(
        DbCommand command, DbCommandInterceptionContext<TResult> interceptionContext)
    {
        Write(string.Format(
            "Context '{0}' is executing command '{1}'{2}",
            Context.GetType().Name,
            command.CommandText.Replace(Environment.NewLine, ""),
            Environment.NewLine));
    }

    public override void LogResult<TResult>(
        DbCommand command, DbCommandInterceptionContext<TResult> interceptionContext)
    {
    }
}
```  

若要记录输出只需调用 Write 方法，这会将输出发送到已配置的写入委托。  

（请注意，此代码的作用只是为了举例简单删除换行符。 它可能不会适合查看复杂的 SQL。）  

### <a name="setting-the-databaselogformatter"></a>设置 DatabaseLogFormatter  

新的 DatabaseLogFormatter 类具有创建它后需要使用 EF 进行注册。 这是使用基于代码的配置。 简而言之，这意味着创建从 DbConfiguration 派生 DbContext 类为在同一程序集中的新类并在此新类的构造函数中调用 SetDatabaseLogFormatter。 例如：  

``` csharp
public class MyDbConfiguration : DbConfiguration
{
    public MyDbConfiguration()
    {
        SetDatabaseLogFormatter(
            (context, writeAction) => new OneLineFormatter(context, writeAction));
    }
}
```  

### <a name="using-the-new-databaselogformatter"></a>使用新 DatabaseLogFormatter  

只要设置 Database.Log，现在将使用此新 DatabaseLogFormatter。 因此，第 1 部分中运行的代码现在将产生以下输出：  

```  
Context 'BlogContext' is executing command 'SELECT TOP (1) [Extent1].[Id] AS [Id], [Extent1].[Title] AS [Title]FROM [dbo].[Blogs] AS [Extent1]WHERE (N'One Unicorn' = [Extent1].[Title]) AND ([Extent1].[Title] IS NOT NULL)'
Context 'BlogContext' is executing command 'SELECT [Extent1].[Id] AS [Id], [Extent1].[Title] AS [Title], [Extent1].[BlogId] AS [BlogId]FROM [dbo].[Posts] AS [Extent1]WHERE [Extent1].[BlogId] = @EntityKeyValue1'
Context 'BlogContext' is executing command 'update [dbo].[Posts]set [Title] = @0where ([Id] = @1)'
Context 'BlogContext' is executing command 'insert [dbo].[Posts]([Title], [BlogId])values (@0, @1)select [Id]from [dbo].[Posts]where @@rowcount > 0 and [Id] = scope_identity()'
```  

## <a name="interception-building-blocks"></a>拦截构建基块  

到目前为止，我们已经探讨如何使用 DbContext.Database.Log 记录生成的 EF 的 SQL。 但此代码通过用于更多常规侦听某些低级别构建基块是实际的相对较薄的外观。  

### <a name="interception-interfaces"></a>拦截接口  

拦截代码设计思路的拦截接口。 这些接口从 IDbInterceptor 继承，并定义当 EF 执行某些操作时调用的方法。 目的是对象的每个所拦截的类型有一个接口。 例如，IDbCommandInterceptor 接口定义 EF 进行对 ExecuteNonQuery、 ExecuteScalar、 ExecuteReader 和相关的方法的调用之前调用的方法。 同样，该接口定义这些操作完成时调用的方法。 我们在上面的 DatabaseLogFormatter 类实现此接口以记录的命令。  

### <a name="the-interception-context"></a>拦截上下文  

它查看任何拦截器接口上定义的方法是很明显，每次调用给定的对象类型 DbInterceptionContext 或某种类型派生自这如 DbCommandInterceptionContext\<\>。 此对象包含有关 EF 所用的操作的上下文信息。 例如，如果代表 DbContext 采取此操作后，然后在 DbContext 中包括 DbInterceptionContext。 同样，对于正在以异步方式执行的命令，则上 DbCommandInterceptionContext 设置 IsAsync 标志。  

### <a name="result-handling"></a>结果处理  

DbCommandInterceptionContext\< \>类包含名为结果、 OriginalResult、 异常和 OriginalException 属性。 这些属性设置为 null/零值对之前将执行此操作调用的拦截方法的调用 — 即，用于...正在执行的方法。 如果该操作执行，并且成功，则结果和 OriginalResult 将设置为运算的结果。 然后可以在执行该操作后调用的拦截方法中观察到这些值 — 也就是说，在...执行的方法。 同样，如果该操作将引发，然后异常和 OriginalException 属性将设置。  

#### <a name="suppressing-execution"></a>取消执行  

如果侦听器设置的 Result 属性之前执行了该命令 (在其中一个...然后 EF 执行方法） 不会尝试实际执行该命令，但将只需使用结果集。 换而言之，侦听器可以禁止显示命令的执行，但已有 EF 继续执行，如同已执行命令。  

这可能会如何使用的一个示例是使用包装提供程序完成传统上了批处理，该命令。 侦听器将存储为一批的更高版本执行的命令，但将"假设"到该命令必须作为正常执行的 EF。 请注意，它比这更多需要实现批处理，但这是可能会如何使用更改拦截结果的示例。  

此外可以通过将异常属性之一设置禁止显示执行...正在执行的方法。 这将导致 EF 来继续执行，如同通过引发给定的异常失败执行操作。 这可能，当然，会导致应用程序崩溃，但它也可能是暂时性异常或某种其他由 EF 处理的异常。 例如，这可以用于在测试环境中测试应用程序的行为，命令执行失败时。  

#### <a name="changing-the-result-after-execution"></a>在执行之后更改结果  

如果侦听器设置的 Result 属性执行该命令后 (在其中一个...执行方法） 则 EF 将使用已更改的结果，而不是从操作中实际返回的结果。 同样，如果执行该命令后，侦听器将设置异常属性，然后 EF 将引发集异常就像操作引发了异常的异常。  

拦截器还可以将异常属性设置为 null 以指示应引发任何异常。 如果执行该操作失败，但拦截器希望 EF 来继续执行，如同该操作已经成功，这会很有用。 这通常还需要将结果设置，以便 EF 有一些结果值，因为它将继续使用。  

#### <a name="originalresult-and-originalexception"></a>OriginalResult 和 OriginalException  

EF 执行某个操作后它会设置的结果和 OriginalResult 属性如果执行未失败或异常和 OriginalException 属性，如果执行失败，出现异常。  

OriginalResult 和 OriginalException 属性是只读的并且实际执行某项操作后仅由 EF 设置。 这些属性不能通过侦听器设置。 这意味着任何侦听器可区分的异常或已由某些其他侦听器而不是实际异常设置的结果或执行操作时出现的结果。  

### <a name="registering-interceptors"></a>注册侦听器  

创建实现一个或多个侦听接口的类后可使用 EF 使用 DbInterception 类注册它。 例如：  

``` csharp
DbInterception.Add(new NLogCommandInterceptor());
```  

此外可以在使用 DbConfiguration 基于代码的配置机制的应用程序域级别注册侦听器。  

### <a name="example-logging-to-nlog"></a>示例： 将日志写入 NLog  

让我们整理了所有这些示例到该使用 IDbCommandInterceptor 并[NLog](http://nlog-project.org/)到：  

- 记录非异步执行任何命令一条警告  
- 记录有关执行时，会引发任何命令的错误  

下面是执行的日志记录，应注册，如上所示的类：  

``` csharp
public class NLogCommandInterceptor : IDbCommandInterceptor
{
    private static readonly Logger Logger = LogManager.GetCurrentClassLogger();

    public void NonQueryExecuting(
        DbCommand command, DbCommandInterceptionContext<int> interceptionContext)
    {
        LogIfNonAsync(command, interceptionContext);
    }

    public void NonQueryExecuted(
        DbCommand command, DbCommandInterceptionContext<int> interceptionContext)
    {
        LogIfError(command, interceptionContext);
    }

    public void ReaderExecuting(
        DbCommand command, DbCommandInterceptionContext<DbDataReader> interceptionContext)
    {
        LogIfNonAsync(command, interceptionContext);
    }

    public void ReaderExecuted(
        DbCommand command, DbCommandInterceptionContext<DbDataReader> interceptionContext)
    {
        LogIfError(command, interceptionContext);
    }

    public void ScalarExecuting(
        DbCommand command, DbCommandInterceptionContext<object> interceptionContext)
    {
        LogIfNonAsync(command, interceptionContext);
    }

    public void ScalarExecuted(
        DbCommand command, DbCommandInterceptionContext<object> interceptionContext)
    {
        LogIfError(command, interceptionContext);
    }

    private void LogIfNonAsync<TResult>(
        DbCommand command, DbCommandInterceptionContext<TResult> interceptionContext)
    {
        if (!interceptionContext.IsAsync)
        {
            Logger.Warn("Non-async command used: {0}", command.CommandText);
        }
    }

    private void LogIfError<TResult>(
        DbCommand command, DbCommandInterceptionContext<TResult> interceptionContext)
    {
        if (interceptionContext.Exception != null)
        {
            Logger.Error("Command {0} failed with exception {1}",
                command.CommandText, interceptionContext.Exception);
        }
    }
}
```  

请注意此代码以发现命令非异步执行并发现时出错执行命令时如何使用拦截上下文。  
