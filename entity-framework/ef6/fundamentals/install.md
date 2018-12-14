---
title: 获取实体框架的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 122c38a2-f9e8-4ecc-9c72-a83bc9af7814
ms.openlocfilehash: 7f840a4f9e437ec12f699184339e386976e1528b
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490623"
---
# <a name="get-entity-framework"></a>获取实体框架
实体框架的 Visual Studio 和 EF 运行时由组成的 EF 工具。

## <a name="ef-tools-for-visual-studio"></a>用于 Visual Studio 的 EF 工具

适用于 Visual Studio 的实体框架工具包括在 EF 设计器和 EF 模型向导和所需的数据库首次和第一个工作流的模型。 在所有最新版本的 Visual Studio 包含 EF 工具。 如果执行自定义安装的 Visual Studio 将需要确保该项目通过选择包含它的工作负荷或通过选择作为单个组件，则选择"Entity Framework 6 工具"。

对于某些以前的 Visual Studio 版本，已更新的 EF 工具是可通过下载获得。 请参阅[Visual Studio 版本](~/ef6/what-is-new/visual-studio.md)有关如何获取你的 Visual Studio 版本的 EF 工具可用的最新版本的指南。

## <a name="ef-runtime"></a>EF 运行时

实体框架的最新版本是可用作[EntityFramework NuGet 包](http://nuget.org/packages/EntityFramework/)。 如果您不熟悉使用 NuGet 程序包管理器中，我们建议您阅读[NuGet 概述](https://docs.microsoft.com/nuget/consume-packages/overview-and-workflow)。

### <a name="installing-the-ef-nuget-package"></a>安装 EF NuGet 包

可以通过右键单击安装 EntityFramework 包**引用**项目文件夹并选择**管理 NuGet 包...**

![管理 NuGet 包](~/ef6/media/managenugetpackages.png)

### <a name="installing-from-package-manager-console"></a>从包管理器控制台安装

或者，可以通过在运行以下命令安装 EntityFramework[程序包管理器控制台](http://docs.nuget.org/docs/start-here/using-the-package-manager-console)。

``` powershell
Install-Package EntityFramework
```

## <a name="installing-a-specific-version-of-ef"></a>安装特定版本的 EF

从 EF 4.1 及更高版本，已作为发布新版本的 EF 运行时[EntityFramework NuGet 包](https://www.nuget.org/packages/EntityFramework/)。 任何这些版本可以向基于.NET Framework 的项目添加，通过在 Visual Studio 中运行以下命令[程序包管理器控制台](http://docs.nuget.org/docs/start-here/using-the-package-manager-console):

``` powershell
Install-Package EntityFramework -Version <number>
```

请注意，`<number>`表示的特定版本的 EF 安装。 例如，6.2.0 是 EF 6.2 的版本号。   

EF 4.1 之前的运行时是.NET Framework 的一部分，并且不能单独安装。

### <a name="installing-the-latest-preview"></a>安装最新预览版

上述方法将为您提供最新版本完全支持 Entity Framework 的版本。 通常是实体框架可用，我们很高兴您尝试并向我们提供有关反馈的预发布版本。

若要安装的 EntityFramework 可以选择最新预览版**包括预发行版**管理 NuGet 包窗口中。 如果不有任何预发布版本将自动获取最新的实体框架的完全受支持的版本。

![包括预发行版](~/ef6/media/includeprerelease.png)

或者，可以运行以下命令[程序包管理器控制台](http://docs.nuget.org/docs/start-here/using-the-package-manager-console)。

``` powershell
Install-Package EntityFramework -Pre
```
