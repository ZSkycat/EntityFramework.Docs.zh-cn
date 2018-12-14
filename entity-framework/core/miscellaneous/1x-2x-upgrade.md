---
title: 从以前的版本升级到 EF Core 2-EF Core
author: divega
ms.date: 08/13/2017
ms.assetid: 8BD43C8C-63D9-4F3A-B954-7BC518A1B7DB
uid: core/miscellaneous/1x-2x-upgrade
ms.openlocfilehash: 5371c8f3b7c6102c621296bbae145d13779e0c6e
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283766"
---
# <a name="upgrading-applications-from-previous-versions-to-ef-core-20"></a>从以前版本的应用程序升级到 EF Core 2.0

我们已经采取了显著改进我们的现有 Api 和 2.0 中的行为的机会。 可能需要修改现有应用程序代码的一些改进，尽管我们认为大多数应用程序的影响很低，在大多数情况下需要只需重新编译和最小的引导式的更改来替换已过时的 Api。

更新现有应用程序到 EF Core 2.0 可能还需要：

1. 升级为支持.NET Standard 2.0 应用程序的目标.NET 实现。 请参阅[支持的.NET 实现](../platforms/index.md)的更多详细信息。

2. 标识的提供程序与 EF Core 2.0 兼容的目标数据库。 请参阅[EF Core 2.0 需要 2.0 版数据库提供程序](#ef-core-20-requires-a-20-database-provider)下面。

3. 升级到 2.0 所有 EF Core 包 （运行时和工具）。 请参阅[安装 EF Core](../get-started/install/index.md) 有关详细信息。

4. 进行任何必要的代码更改，以弥补此文档的其余部分中所述的重大更改。

## <a name="aspnet-core-now-includes-ef-core"></a>ASP.NET Core 现在包括 EF Core

面向 ASP.NET Core 2.0 的应用程序可以使用 EF Core 2.0，而不需要第三方数据库提供程序以外的其他依赖项。 但是，面向以前版本的 ASP.NET Core 应用程序需要为了使用 EF Core 2.0 升级到 ASP.NET Core 2.0。 有关升级到 2.0 的 ASP.NET Core 应用程序的详细信息，请参阅[有关该主题的 ASP.NET Core 文档](https://docs.microsoft.com/aspnet/core/migration/1x-to-2x/)。

## <a name="new-way-of-getting-application-services-in-aspnet-core"></a>获取在 ASP.NET Core 中的应用程序服务的新方法

已为 2.0 中断 1.x 中使用的 EF Core 的设计时逻辑的方式更新的建议的模式为 ASP.NET Core web 应用程序。 以前在设计时，EF Core 会尝试调用`Startup.ConfigureServices`直接才能访问应用程序的服务提供程序。 在 ASP.NET Core 2.0 中，配置初始化外部`Startup`类。 通常使用 EF Core 应用程序配置中，从访问其连接字符串因此`Startup`本身已不再够用。 如果升级 ASP.NET Core 1.x 应用程序，你可能会收到以下错误，使用 EF Core 工具时。

> 没有无参数构造函数 ApplicationContext 上找到。 将无参数构造函数添加到 ApplicationContext 或添加的实现 IDesignTimeDbContextFactory&lt;ApplicationContext&gt;作为 ApplicationContext 在同一程序集中

ASP.NET Core 2.0 的默认模板中已添加新的设计时挂钩。 静态`Program.BuildWebHost`方法使 EF Core 以在设计时访问应用程序的服务提供程序。 如果要升级的 ASP.NET Core 1.x 应用程序，你将需要更新`Program`类如下所示。

``` csharp
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;

namespace AspNetCoreDotNetCore2._0App
{
    public class Program
    {
        public static void Main(string[] args)
        {
            BuildWebHost(args).Run();
        }

        public static IWebHost BuildWebHost(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>()
                .Build();
    }
}
```

此新模式时正在更新应用程序到 2.0 强烈建议和 Entity Framework Core 迁移等产品功能才能工作所需的采用。 其他常见的替代方法是[实现*IDesignTimeDbContextFactory\<TContext >*](xref:core/miscellaneous/cli/dbcontext-creation#from-a-design-time-factory)。

## <a name="idbcontextfactory-renamed"></a>IDbContextFactory 重命名

为了支持不同的应用程序模式，并为用户提供更好地控制如何他们`DbContext`用于在设计时，我们，在过去，提供了`IDbContextFactory<TContext>`接口。 EF Core 工具将在设计时发现你的项目中的此接口的实现并使用它来创建`DbContext`对象。

此接口有一个非常常规名称，这会误导某些用户，请尝试重新将其用于其他`DbContext`-创建方案。 它们时 EF 工具，则尝试在设计时使用它们的实现和导致之类的命令`Update-Database`或`dotnet ef database update`失败。

为了进行通信的强的设计时语义的此接口，我们已重命名到`IDesignTimeDbContextFactory<TContext>`。

对于在 2.0 发行`IDbContextFactory<TContext>`仍然存在，但已标记为过时。

## <a name="dbcontextfactoryoptions-removed"></a>DbContextFactoryOptions 删除

由于上面所述的 ASP.NET Core 2.0 更改，我们发现，`DbContextFactoryOptions`已不再需要在新`IDesignTimeDbContextFactory<TContext>`接口。 以下是应改为使用你的备选方案。

| DbContextFactoryOptions | 替代项                                                  |
|:------------------------|:-------------------------------------------------------------|
| ApplicationBasePath     | AppContext.BaseDirectory                                     |
| ContentRootPath         | Directory.GetCurrentDirectory()                              |
| EnvironmentName         | Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") |

## <a name="design-time-working-directory-changed"></a>设计时更改的工作目录

ASP.NET Core 2.0 更改也需要使用的工作目录`dotnet ef`为了符合运行你的应用程序时，由 Visual Studio 使用的工作目录。 这一个可观察的副作用是像以前一样，文件名现相对于项目目录中并不是输出目录的 SQLite。

## <a name="ef-core-20-requires-a-20-database-provider"></a>EF Core 2.0 需要 2.0 版数据库提供程序

为使用 EF Core 2.0 我们所做工作许多简化和方式的数据库提供程序中的改进。 这意味着 1.0.x 版和 1.1.x 提供程序不会使用 EF Core 2.0。

EF 团队提供的 SQL Server 和 SQLite 提供程序和 2.0 版本将可作为 2.0 版的一部分发布。 开放源代码第三方提供程序[SQL Compact](https://github.com/ErikEJ/EntityFramework.SqlServerCompact)， [PostgreSQL](https://github.com/npgsql/Npgsql.EntityFrameworkCore.PostgreSQL)，并[MySQL](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql)正在更新 2.0。 对于所有其他提供程序，请联系提供程序编写器。

## <a name="logging-and-diagnostics-events-have-changed"></a>日志记录和诊断事件已更改

注意： 这些更改不应影响大多数应用程序代码。

事件 Id 发送到的消息[ILogger](https://github.com/aspnet/Logging/blob/dev/src/Microsoft.Extensions.Logging.Abstractions/ILogger.cs) 2.0 中已更改。 现在，事件 ID 在 EF Core 代码内具有唯一性。 这些消息现在还遵循 MVC 等所用的结构化日志记录的标准模式。

记录器类别也已更改。 现提供通过 [DbLoggerCategory](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/DbLoggerCategory.cs) 访问的熟知类别集。

[DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md)事件现在使用相同的事件 ID 名称与相应`ILogger`消息。 事件负载是所有的名义类型派生自[EventData](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Diagnostics/EventData.cs)。

在中记录事件 Id、 有效负载类型和类别[CoreEventId](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Diagnostics/CoreEventId.cs)并[RelationalEventId](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore.Relational/Diagnostics/RelationalEventId.cs)类。

Id 也已从 Microsoft.EntityFrameworkCore.Infraestructure 到新 Microsoft.EntityFrameworkCore.Diagnostics 命名空间。

## <a name="ef-core-relational-metadata-api-changes"></a>EF Core 关系元数据 API 更改

EF Core 2.0 现将对所用的每个不同提供程序生成不同的 [IModel](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Metadata/IModel.cs)。 这对应用程序而言通常是透明的。 这有助于简化较低级别的元数据 API，从而始终通过调用 `.Relational`（而不是 `.SqlServer`、`.Sqlite` 等）来访问常见关系元数据概念。例如，1.1.x 代码如下：

``` csharp
var tableName = context.Model.FindEntityType(typeof(User)).SqlServer().TableName;
```

现在应编写如下代码：

``` csharp
var tableName = context.Model.FindEntityType(typeof(User)).Relational().TableName;
```

而不是使用方法，如`ForSqlServerToTable`，扩展方法现在都可用于编写基于当前提供程序中使用的条件代码。 例如：

```C#
modelBuilder.Entity<User>().ToTable(
    Database.IsSqlServer() ? "SqlServerName" : "OtherName");
```

请注意，此更改仅适用于为定义的 Api/metadata_所有_关系提供程序。 API 和元数据保持不变，当它是特定于单个提供程序。 例如，聚集的索引都是特定于 SQL Sever，因此`ForSqlServerIsClustered`和`.SqlServer().IsClustered()`仍必须使用。

## <a name="dont-take-control-of-the-ef-service-provider"></a>不控制 EF 服务提供程序

EF Core 使用内部`IServiceProvider`（依赖关系注入容器） 为其内部实现。 应用程序应允许 EF Core 以创建和管理在特殊情况下此提供程序除外。 强烈建议考虑删除对任何调用`UseInternalServiceProvider`。 如果应用程序确实需要调用`UseInternalServiceProvider`，请考虑[上报](https://github.com/aspnet/EntityFramework/Issues)以便我们可以调查其他方式来处理你的方案。

调用`AddEntityFramework`， `AddEntityFrameworkSqlServer`，等不需要由应用程序代码，除非`UseInternalServiceProvider`也称为。 删除对任何现有调用`AddEntityFramework`或`AddEntityFrameworkSqlServer`等`AddDbContext`仍需使用相同的方式与前面。

## <a name="in-memory-databases-must-be-named"></a>内存中数据库必须命名为

未命名的全局内存中数据库已删除，而是必须命名为内存中的所有数据库。 例如：

``` csharp
optionsBuilder.UseInMemoryDatabase("MyDatabase");
```

这将创建/使用具有名称"MyDatabase"的数据库。 如果`UseInMemoryDatabase`会再次调用具有相同的名称，则将使用相同的内存中数据库，从而使其可以由多个上下文实例共享。

## <a name="read-only-api-changes"></a>只读的 API 更改

`IsReadOnlyBeforeSave``IsReadOnlyAfterSave`，并`IsStoreGeneratedAlways`已过时，替换[BeforeSaveBehavior](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Metadata/IProperty.cs#L39)并[AfterSaveBehavior](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Metadata/IProperty.cs#L55)。 这些行为将应用于任何属性 （不只存储生成的属性），并确定将插入到数据库行时应如何使用属性的值 (`BeforeSaveBehavior`) 或更新现有数据库行时 (`AfterSaveBehavior`)。

属性标记为[ValueGenerated.OnAddOrUpdate](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Metadata/ValueGenerated.cs) （例如，对于计算列） 将默认情况下忽略当前的属性上设置任何值。 这意味着，而不考虑任何值已设置或修改所跟踪实体上是否将始终获取由存储生成的值。 这可以通过设置其他更改`Before\AfterSaveBehavior`。

## <a name="new-clientsetnull-delete-behavior"></a>新 ClientSetNull 删除行为

在上一版本中， [DeleteBehavior.Restrict](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Metadata/DeleteBehavior.cs)必须为实体行为由上下文跟踪的详细信息已关闭的匹配`SetNull`语义。 在 EF Core 2.0 中，新`ClientSetNull`作为的默认值为可选关系引入了行为。 此行为具有`SetNull`语义跟踪的实体和`Restrict`创建使用 EF Core 的数据库的行为。 在我们的经验，它们是跟踪的实体和数据库的最预期/有用行为。 `DeleteBehavior.Restrict` 现在为跟踪的实体的可选关系设置时采用。

## <a name="provider-design-time-packages-removed"></a>删除提供程序设计时包

`Microsoft.EntityFrameworkCore.Relational.Design`删除包。 它的内容已合并到`Microsoft.EntityFrameworkCore.Relational`和`Microsoft.EntityFrameworkCore.Design`。

这将传播到提供程序设计时包。 这些包 (`Microsoft.EntityFrameworkCore.Sqlite.Design`，`Microsoft.EntityFrameworkCore.SqlServer.Design`等) 已删除和其内容合并到主要提供程序包。

若要启用`Scaffold-DbContext`或`dotnet ef dbcontext scaffold`在 EF Core 2.0 中，你只需引用单个提供程序包：

``` xml
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer"
    Version="2.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools"
    Version="2.0.0"
    PrivateAssets="All" />
<DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet"
    Version="2.0.0" />
```
