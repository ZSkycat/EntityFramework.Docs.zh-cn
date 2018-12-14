---
title: 从 EF Core 1.0 RC1 升级到 RC2-EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 6d75b229-cc79-4d08-88cd-3a1c1b24d88f
uid: core/miscellaneous/rc1-rc2-upgrade
ms.openlocfilehash: 83b98fda5ac9491994b5b3fb333c9951ec01188a
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42996892"
---
# <a name="upgrading-from-ef-core-10-rc1-to-10-rc2"></a>从 EF Core 1.0 RC1 升级到 1.0 RC2

本文提供了用于移动到 RC2 的 RC1 包生成的应用程序的指南。

## <a name="package-names-and-versions"></a>包名称和版本

之间 RC1 和 RC2，我们从更改"Entity Framework 7"为"Entity Framework Core"。 你可以阅读更多的更改的原因[Scott hanselman 撰写的此文章](http://www.hanselman.com/blog/ASPNET5IsDeadIntroducingASPNETCore10AndNETCore10.aspx)。 由于此更改，我们的包名称有所不同`EntityFramework.*`到`Microsoft.EntityFrameworkCore.*`以及从我们版本`7.0.0-rc1-final`到`1.0.0-rc2-final`(或`1.0.0-preview1-final`工具)。

**将需要 RC1 包中完全删除，然后安装 RC2 的。** 下面是一些常见的包的映射。

| RC1 程序包                                               | RC2 等效项                                                       |
|:----------------------------------------------------------|:---------------------------------------------------------------------|
| EntityFramework.MicrosoftSqlServer        7.0.0-rc1-final | Microsoft.EntityFrameworkCore.SqlServer         1.0.0-rc2-final      |
| EntityFramework.SQLite                    7.0.0-rc1-final | Microsoft.EntityFrameworkCore.Sqlite            1.0.0-rc2-final      |
| EntityFramework7.Npgsql                   3.1.0-rc1-3     | NpgSql.EntityFrameworkCore.Postgres             <to be advised>      |
| EntityFramework.SqlServerCompact35        7.0.0-rc1-final | EntityFrameworkCore.SqlServerCompact35          1.0.0-rc2-final      |
| EntityFramework.SqlServerCompact40        7.0.0-rc1-final | EntityFrameworkCore.SqlServerCompact40          1.0.0-rc2-final      |
| EntityFramework.InMemory 7.0.0-rc1-final | Microsoft.EntityFrameworkCore.InMemory          1.0.0-rc2-final      |
| EntityFramework.IBMDataServer             7.0.0-beta1     | 尚不可用于 RC2                                            |
| EntityFramework.Commands                  7.0.0-rc1-final | Microsoft.EntityFrameworkCore.Tools             1.0.0-preview1-final |
| EntityFramework.MicrosoftSqlServer.Design 7.0.0-rc1-final | Microsoft.EntityFrameworkCore.SqlServer.Design  1.0.0-rc2-final      |

## <a name="namespaces"></a>命名空间

包名称，以及从更改命名空间`Microsoft.Data.Entity.*`到`Microsoft.EntityFrameworkCore.*`。 您可以处理此更改的查找/替换`using Microsoft.Data.Entity`与`using Microsoft.EntityFrameworkCore`。

## <a name="table-naming-convention-changes"></a>表命名约定更改

在 RC2 中，我们采取的重要功能更改是使用的名称`DbSet<TEntity>`作为表名称为给定实体的属性则映射为，而不是只是类名称。 你可以阅读更多有关这一更改[相关的公告问题](https://github.com/aspnet/Announcements/issues/167)。

对于现有 RC1 应用程序，我们建议将以下代码添加到开始您`OnModelCreating`方法以保持 RC1 命名策略：

``` csharp
foreach (var entity in modelBuilder.Model.GetEntityTypes())
{
    entity.Relational().TableName = entity.DisplayName();
}
```

如果你想要采用新的命名策略，我们建议已成功完成余下的升级步骤，然后删除代码，然后创建要应用的表的迁移将重命名。

## <a name="adddbcontext--startupcs-changes-aspnet-core-projects-only"></a>AddDbContext / Startup.cs 更改 （仅适用于 ASP.NET Core 项目）

在 RC1 中，您必须在将实体框架服务添加到应用程序服务提供程序- `Startup.ConfigureServices(...)`:

``` csharp
services.AddEntityFramework()
  .AddSqlServer()
  .AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(Configuration["ConnectionStrings:DefaultConnection"]));
```

在 RC2 中，你可以删除对调用`AddEntityFramework()`， `AddSqlServer()`，等等。:

``` csharp
services.AddDbContext<ApplicationDbContext>(options =>
  options.UseSqlServer(Configuration["ConnectionStrings:DefaultConnection"]));
```

此外需要添加一个构造函数，为您派生上下文，它接受上下文选项，并将它们传递到基构造函数。 这需要，因为我们删除了一些可怕的潜入中对其在后台的奇妙的作用：

``` csharp
public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
    : base(options)
{
}
```

## <a name="passing-in-an-iserviceprovider"></a>传入 IServiceProvider

如果你有 RC1 代码传递`IServiceProvider`到上下文，这现在已移到`DbContextOptions`，而不是单独的构造函数参数。 使用`DbContextOptionsBuilder.UseInternalServiceProvider(...)`设置服务提供程序。

### <a name="testing"></a>正在测试

执行此操作的最常见方案是在测试时控制的 InMemory 数据库作用域。 请参阅更新[测试](testing/index.md)一文，了解执行此操作与 RC2 的示例。

### <a name="resolving-internal-services-from-application-service-provider-aspnet-core-projects-only"></a>解析内部服务从应用程序服务提供商 （仅适用于 ASP.NET Core 项目）

如果你有一个 ASP.NET Core 应用并且想 EF 若要解决应用程序服务提供商提供的内部服务，则的重载`AddDbContext`，可以对此进行配置：

``` csharp
services.AddEntityFrameworkSqlServer()
  .AddDbContext<ApplicationDbContext>((serviceProvider, options) =>
    options.UseSqlServer(Configuration["ConnectionStrings:DefaultConnection"])
           .UseInternalServiceProvider(serviceProvider)); );
```

> [!WARNING]  
> 我们建议让 EF 在内部管理其自己的服务，除非有其他原因，将内部的 EF 服务合并到应用程序服务提供商。 您可能想要执行此操作的主要原因是使用应用程序服务提供商来替换 EF 在内部使用的服务

## <a name="dnx-commands--net-cli-aspnet-core-projects-only"></a>DNX 命令 = >.NET CLI （仅适用于 ASP.NET Core 项目）

如果以前使用过`dnx ef`ASP.NET 5 项目的命令，这些现都已移到`dotnet ef`命令。 相同的命令语法仍适用。 可以使用`dotnet ef --help`语法的信息。

在 RC2 中，由于 DNX 替换为.NET CLI，注册命令的方式已更改。 现已在注册命令`tools`主题中`project.json`:

``` json
"tools": {
  "Microsoft.EntityFrameworkCore.Tools": {
    "version": "1.0.0-preview1-final",
    "imports": [
      "portable-net45+win8+dnxcore50",
      "portable-net45+win8"
    ]
  }
}
```

> [!TIP]  
> 如果你使用 Visual Studio，你现在可以使用程序包管理器控制台运行 ASP.NET Core 项目 （这不支持在 RC1 中） 的 EF 命令。 你仍需要注册中的命令`tools`一部分`project.json`若要执行此操作。

## <a name="package-manager-commands-require-powershell-5"></a>包管理器命令需要 PowerShell 5

如果在 Visual Studio 中的包管理器控制台中使用 Entity Framework 命令，你将需要确保已安装的 PowerShell 5。 这是临时的要求，将在下一版本中删除 (请参阅[发出 #5327](https://github.com/aspnet/EntityFramework/issues/5327)的更多详细信息)。

## <a name="using-imports-in-projectjson"></a>在 project.json 中使用"导入"

一些 EF Core 依赖关系不支持.NET 标准尚未。 标准.NET 和.NET Core 项目中的 EF Core 可能会要求添加"导入"到 project.json 临时的解决方法。

在添加 EF，NuGet 还原将显示此错误消息：

``` Console
Package Ix-Async 1.2.5 is not compatible with netcoreapp1.0 (.NETCoreApp,Version=v1.0). Package Ix-Async 1.2.5 supports:
  - net40 (.NETFramework,Version=v4.0)
  - net45 (.NETFramework,Version=v4.5)
  - portable-net45+win8+wp8 (.NETPortable,Version=v0.0,Profile=Profile78)
Package Remotion.Linq 2.0.2 is not compatible with netcoreapp1.0 (.NETCoreApp,Version=v1.0). Package Remotion.Linq 2.0.2 supports:
  - net35 (.NETFramework,Version=v3.5)
  - net40 (.NETFramework,Version=v4.0)
  - net45 (.NETFramework,Version=v4.5)
  - portable-net45+win8+wp8+wpa81 (.NETPortable,Version=v0.0,Profile=Profile259)
```

解决方法是手动导入"portable-net451 + win8"的可移植配置文件。 此强制将处理与此二进制文件的 NuGet 提供作为使用.NET Standard 兼容框架，即使它们不是。 虽然"portable-net451 + win8"不是 100%兼容.NET 标准，是足够的兼容性有关转换到.NET Standard 的 PCL。 EF 的依赖项最终升级到.NET Standard 时，可以删除导入。

在数组语法中，多个框架可以添加到"导入"。 如果将其他库添加到你的项目，则可能需要采用其他导入。

``` json
{
  "frameworks": {
    "netcoreapp1.0": {
      "imports": ["dnxcore50", "portable-net451+win8"]
    }
  }
}
```

请参阅[发出 #5176](https://github.com/aspnet/EntityFramework/issues/5176)。
