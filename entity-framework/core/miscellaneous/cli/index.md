---
title: Entity Framework Core 工具参考 - EF Core
author: bricelam
ms.author: bricelam
ms.date: 09/19/2018
uid: core/miscellaneous/cli/index
ms.openlocfilehash: 9fcb452c2798a3d07e39cbcc3c34629dca4394ff
ms.sourcegitcommit: ad1bdea58ed35d0f19791044efe9f72f94189c18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/28/2018
ms.locfileid: "47447113"
---
# <a name="entity-framework-core-tools-reference"></a>Entity Framework Core 工具参考

Entity Framework Core 工具可帮助完成设计时的开发任务。 它们主要用于通过对数据库架构进行反向工程来管理迁移和搭建 `DbContext` 和实体类型的基架。

* 在 Visual Studio 中，[EF Core 程序包管理器控制台工具](powershell.md)) 在[程序包管理器控制台](https://docs.microsoft.com/nuget/tools/package-manager-console)中运行。 这些工具同时适用于 .NET Framework 和 .NET Core 项目。

* [EF Core.NET 命令行接口 (CLI) 工具](dotnet.md)是对跨平台 [.NET Core CLI 工具](https://docs.microsoft.com/dotnet/core/tools/)的扩展。 这些工具需要 .NET Core SDK 项目（具有 `Sdk="Microsoft.NET.Sdk"` 的项目或项目文件中的相似项目）。

这两种工具提供相同的功能。 如果使用 Visual Studio 进行开发，我们建议使用**程序包管理器控制台**工具，因为这些工具提供更完整的体验。

## <a name="next-steps"></a>后续步骤

* [EF Core 程序包管理器控制台工具参考](powershell.md)
* [EF Core.NET CLI 工具参考](dotnet.md)