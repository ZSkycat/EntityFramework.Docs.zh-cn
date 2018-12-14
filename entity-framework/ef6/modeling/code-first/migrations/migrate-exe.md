---
title: 使用 migrate.exe-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 989ea862-e936-4c85-926a-8cfbef5df5b8
ms.openlocfilehash: cf6c3a0a256730b24addf1012d6ff53b17035cd4
ms.sourcegitcommit: c568d33214fc25c76e02c8529a29da7a356b37b4
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/30/2018
ms.locfileid: "47459534"
---
# <a name="using-migrateexe"></a>使用 migrate.exe
可以使用 code First 迁移更新数据库从内部 visual studio 中，但也可以通过命令行工具 migrate.exe 执行。 此页将提供有关如何使用 migrate.exe 执行针对数据库迁移的快速概述。

> [!NOTE]
> 本文假定你知道如何在基本方案中使用 Code First 迁移。 如果不这样做，则你将需要读取[Code First 迁移](~/ef6/modeling/code-first/migrations/index.md)然后再继续。

## <a name="copy-migrateexe"></a>复制 migrate.exe

在安装实体框架时使用 NuGet migrate.exe 将下载的包的工具文件夹内。 在中&lt;项目文件夹&gt;\\包\\EntityFramework。&lt;版本&gt;\\工具

一旦有 migrate.exe 然后需要将其复制到包含你迁移的程序集的位置。

如果应用程序面向.NET 4 中，并且不 4.5，然后你将需要复制**Redirect.config**到位置一样以及与将其重命名**migrate.exe.config**。这是以便 migrate.exe 获取正确的绑定重定向，以便能够找到实体框架程序集。

| .NET 4.5                                      | .NET 4.0                                      |
|:----------------------------------------------|:----------------------------------------------|
| ![.NET 4.5 文件](~/ef6/media/net45files.png) | ![.NET 4.0 文件](~/ef6/media/net40files.png) |

> [!NOTE]
> migrate.exe 不支持 x64 的程序集。

一旦已移至正确的文件夹的 migrate.exe 则应能够使用它来执行对数据库的迁移。 该实用程序用于执行操作的就是执行迁移。 它不能生成迁移或创建一个 SQL 脚本。

## <a name="see-options"></a>请参阅选项

``` console
Migrate.exe /?
```

更高版本将显示与此实用程序，请注意，需要能够 EntityFramework.dll 正在 migrate.exe 为了使此功能在同一位置相关联的帮助页。

## <a name="migrate-to-the-latest-migration"></a>迁移到最新的迁移

``` console
Migrate.exe MyMvcApplication.dll /startupConfigurationFile=”..\\web.config”
```

当运行 migrate.exe 唯一必需的参数是程序集，这是包含想要运行的迁移的程序集，但它将使用所有约定基于设置，如果未指定配置文件。

## <a name="migrate-to-a-specific-migration"></a>迁移到特定的迁移

``` console
Migrate.exe MyApp.exe /startupConfigurationFile=”MyApp.exe.config” /targetMigration=”AddTitle”
```

如果你想要运行到特定迁移的迁移，你可以指定迁移的名称。 这将根据需要运行所有以前的迁移之前获取到指定的迁移。

## <a name="specify-working-directory"></a>指定工作目录

``` console
Migrate.exe MyApp.exe /startupConfigurationFile=”MyApp.exe.config” /startupDirectory=”c:\\MyApp”
```

如果您的程序集具有依赖关系或读取文件相对于工作目录，你将需要设置 startupDirectory。

## <a name="specify-migration-configuration-to-use"></a>指定要使用的迁移配置

``` console
Migrate.exe MyAssembly CustomConfig /startupConfigurationFile=”..\\web.config”
```

如果有多个迁移配置类，类继承 DbMigrationConfiguration，然后您需要指定这是要用于此执行。 这是通过提供可选的第二个参数而无需为上面的开关来指定。

## <a name="provide-connection-string"></a>提供连接字符串

``` console
Migrate.exe BlogDemo.dll /connectionString=”Data Source=localhost;Initial Catalog=BlogDemo;Integrated Security=SSPI” /connectionProviderName=”System.Data.SqlClient”
```

如果你想要指定连接字符串在命令行，则还必须提供的提供程序名称。 未指定提供程序名称将导致异常。

## <a name="common-problems"></a>常见问题

| 错误消息                                                                                                                                                                                                                                                                                                                      | 解决方案                                                                                                                                                                                                                                                                                             |
|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 未经处理的异常： System.IO.FileLoadException： 无法加载文件或程序集 EntityFramework，版本 = 5.0.0.0，区域性 = 中性，PublicKeyToken = b77a5c561934e089 或其某个依赖项。 找到的程序集清单定义与程序集引用不匹配。 (异常来自 HRESULT: 0x80131040)         | 这通常意味着没有 Redirect.config 文件的情况下运行的.NET 4 应用程序。 您需要将 Redirect.config 复制到与 migrate.exe 相同的位置，并重命名为 migrate.exe.config。                                                                                       |
| 未经处理的异常： System.IO.FileLoadException： 无法加载文件或程序集 EntityFramework，版本 = 4.4.0.0，区域性 = 中性，PublicKeyToken = b77a5c561934e089 或其某个依赖项。 找到的程序集清单定义与程序集引用不匹配。 (异常来自 HRESULT: 0x80131040)          | 此异常表示正在使用 Redirect.config 应用程序复制到 migrate.exe 位置.NET 4.5。 如果您的应用程序是.NET 4.5 然后不需要使用内部重定向配置文件。 删除 migrate.exe.config 文件。                                    |
| 错误： 无法更新数据库以匹配当前模型，因为有挂起的更改和禁用自动迁移。 将处理的模型更改写入到基于代码的迁移，或者启用自动迁移。 设置为 true，则启用自动迁移到 DbMigrationsConfiguration.AutomaticMigrationsEnabled。 | 如果尚未创建迁移，以应对到模型中，所做的更改和数据库的模型不匹配时，将迁移正在运行，将发生此错误。 将属性添加到 model 类，然后运行而无需创建迁移来升级数据库 migrate.exe 是此示例。 |
| 错误： 未解析为成员类型 System.Data.Entity.Migrations.Design.ToolingFacade+UpdateRunner,EntityFramework，版本 = 5.0.0.0，区域性 = 中性，PublicKeyToken = b77a5c561934e089。                                                                                                                                       | 可以通过指定一个不正确的启动目录导致此错误。 这必须是 migrate.exe 的位置                                                                                                                                                                                      |
| 未经处理的异常： System.NullReferenceException： 对象引用未设置为某个对象的实例。 <br/>   在 System.Data.Entity.Migrations.Console.Program.Main (String [] args)                                                                                                                                             | 这可能引起不指定正在使用的方案所需的参数。 无需指定提供程序名称，例如指定的连接字符串。                                                                                                                        |
| 错误： 在程序集中 ClassLibrary1' 找多个迁移配置类型。 指定是要使用的名称。                                                                                                                                                                                                  | 该错误指出，给定的程序集没有多个配置类。 必须使用 /configurationType 开关来指定要使用。                                                                                                                                           |
| 错误： 无法加载文件或程序集 '&lt;assemblyName&gt;或其某个依赖项。 给定的程序集名称或基本代码无效。 (异常来自 HRESULT: 0x80131047)                                                                                                                                                    | 原因可能是通过指定程序集名称不正确或不具有                                                                                                                                                                                                                          |
| 错误： 无法加载文件或程序集 '&lt;assemblyName&gt;或其某个依赖项。 试图加载的程序的格式不正确。                                                                                                                                                                          | 发生这种情况是如果你尝试运行 migrate.exe 针对 x64 应用程序。 EF 5.0 和 x86 上才会起下方。                                                                                                                                                                                |
