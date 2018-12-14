---
title: 从 EF6 移植到 EF Core 技术-移植了基于 EDMX 的模型
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 63003709-f1ec-4bdc-8083-65a60c4826d2
uid: efcore-and-ef6/porting/port-edmx
ms.openlocfilehash: 2c3336ac675a830566001a0ddb3777839f52db18
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42997406"
---
# <a name="porting-an-ef6-edmx-based-model-to-ef-core"></a>移植到 EF Core 了从 EF6 基于 EDMX 的模型

EF Core 不支持的模型的 EDMX 文件格式。 移植这些模型的最佳选项是从你的应用程序的数据库生成一个新的基于代码的模型。

## <a name="install-ef-core-nuget-packages"></a>安装 EF Core NuGet 包

安装`Microsoft.EntityFrameworkCore.Tools`NuGet 包。

## <a name="regenerate-the-model"></a>重新生成模型

反向工程功能现在可用于创建基于现有数据库的模型。

包管理器控制台中运行以下命令 (工具 –> NuGet 包管理器 –> 包管理器控制台)。 请参阅[程序包管理器控制台 (Visual Studio)](../../core/miscellaneous/cli/powershell.md)命令选项搭建基架的表等的子集。

``` powershell
Scaffold-DbContext "<connection string>" <database provider name>
```

例如，下面是要从 SQL Server LocalDB 实例上的博客数据库模型创建基架的命令。

``` powershell
Scaffold-DbContext "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer
```

## <a name="remove-ef6-model"></a>删除 EF6 模型

现在，您将从你的应用程序删除 EF6 模型。

最好将从 EF6 NuGet 包 (EntityFramework) 安装，因为 EF Core 和 ef6 更高版本可以是在同一应用程序并行使用。 但是，如果不打算在您的应用程序的任何区域中使用 EF6，然后卸载包将有助于为提供的代码片段，需要注意的编译错误。

## <a name="update-your-code"></a>更新你的代码

此时，它是一个寻址编译错误和查看代码，以查看你会影响从 EF6 和 EF Core 之间的行为更改。

## <a name="test-the-port"></a>测试端口

只是因为你的应用程序将编译，并不意味着它成功移植到 EF Core。 将需要测试应用程序，确保没有行为更改产生负面影响你的应用程序中的所有区域。

> [!TIP]
> 请参阅[Getting Started with ASP.NET Core 使用现有的数据库上的 EF Core](xref:core/get-started/aspnetcore/existing-db)有关现有数据库，使用方式的其他参考 
