---
title: .NET Framework 入门 - 新数据库 - EF Core
author: rowanmiller
ms.date: 08/06/2018
ms.assetid: 52b69727-ded9-4a7b-b8d5-73f3acfbbad3
uid: core/get-started/full-dotnet/new-db
ms.openlocfilehash: 66d9011a5978fc3d8253a963bdb9df27848e9ff9
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42997582"
---
# <a name="getting-started-with-ef-core-on-net-framework-with-a-new-database"></a>使用新数据库在 .NET Framework 上开始使用 EF Core

在本教程中，你将构建一个控制台应用程序，它使用 Entity Framework 对 Microsoft SQL Server 数据库执行基本数据访问。 你将通过迁移基于模型创建数据库。

[在 GitHub 上查看此文章的示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/GetStarted/FullNet/ConsoleApp.NewDb)。

## <a name="prerequisites"></a>系统必备

* [Visual Studio 2017 版本 15.7 或更高版本](https://www.visualstudio.com/downloads/)

## <a name="create-a-new-project"></a>创建新项目

* 打开 Visual Studio 2017

* “文件”>“新建”>“项目...”

* 从左侧菜单中选择“已安装”>“Visual C#”->“Windows Desktop”

* 选择“控制台应用(.NET Framework)”项目模板

* 确保项目面向 **.NET Framework 4.6.1** 或更高版本

* 将项目命名为 ConsoleApp.NewDb，并单击“确定”

## <a name="install-entity-framework"></a>安装 Entity Framework

要使用 EF Core，请为要作为目标对象的数据库提供程序安装程序包。 本教程使用 SQL Server。 有关可用提供程序的列表，请参阅[数据库提供程序](../../providers/index.md)。

* “工具”>“NuGet 包管理器”>“包管理器控制台”

* 运行 `Install-Package Microsoft.EntityFrameworkCore.SqlServer`

在本教程的后面部分，你会使用某些 Entity Framework Tools 维护数据库。 因此，请同时安装该工具包。

* 运行 `Install-Package Microsoft.EntityFrameworkCore.Tools`

## <a name="create-the-model"></a>创建模型

现在是时候定义构成模型的上下文和实体类了。

* “项目”>“添加类...”

* 输入“Model.cs”作为名称，然后单击“确定”

* 将此文件的内容替换为以下代码

  [!code-csharp[Main](../../../../samples/core/GetStarted/FullNet/ConsoleApp.NewDb/Model.cs)] 

> [!TIP]  
> 在实际的应用程序中，可以将每个类放在单独的文件中，并将连接字符串放在配置文件或环境变量中。 为简单起见，在本教程中，所有内容都在单个代码文件中。

## <a name="create-the-database"></a>创建数据库

现在有了模型，就可以使用迁移创建数据库。

* “工具”>“NuGet 包管理器”>“包管理器控制台”

* 运行 `Add-Migration InitialCreate` 为迁移搭建基架，以便为模型创建一组初始表。

* 运行 `Update-Database` 以将新迁移应用到数据库。 由于数据库尚不存在，因此将在应用迁移之前进行创建。

> [!TIP]  
> 如果对模型进行更改，可以使用 `Add-Migration` 命令为新迁移搭建基架，以便对数据库进行相应的架构更改。 检查已搭建基架的代码（并进行了必要更改）后，就可以使用 `Update-Database` 命令将更改应用到数据库。
>
> EF 使用数据库中的 `__EFMigrationsHistory` 表来跟踪哪些迁移已经应用到数据库。

## <a name="use-the-model"></a>使用模型

现在可以使用模型执行数据访问。

* 打开 Program.cs

* 将此文件的内容替换为以下代码

  [!code-csharp[Main](../../../../samples/core/GetStarted/FullNet/ConsoleApp.NewDb/Program.cs)]

* “调试”>“开始执行(不调试)”

  可以看到，把一个博客保存到数据库后，所有博客的详细信息都将显示在控制台中。

  ![图像](_static/output-new-db.png)

## <a name="additional-resources"></a>其他资源

* [通过现有数据库在 .NET Framework 上使用 EF Core](xref:core/get-started/full-dotnet/existing-db)
* [通过新数据库在 .NET Core 上使用 EF Core - SQLite](xref:core/get-started/netcore/new-db-sqlite) - 跨平台控制台 EF 教程。
