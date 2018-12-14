---
title: 从 EF6 移植到 EF Core技术-移植基于代码的模型
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 2dce1a50-7d84-4856-abf6-2763dd9be99d
uid: efcore-and-ef6/porting/port-code
ms.openlocfilehash: 2484b681d71ae8711b1b3a59bc274a0b2e403294
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42997044"
---
# <a name="porting-an-ef6-code-based-model-to-ef-core"></a>移植到 EF Core EF6 基于代码的模型

如果已读取所有需要注意的问题，您就可以到端口，以下是一些准则，以帮助您入门。

## <a name="install-ef-core-nuget-packages"></a>安装 EF Core NuGet 包

若要使用 EF Core，请为你想要使用的数据库提供程序安装 NuGet 包。 例如，当目标 SQL Server，则会安装`Microsoft.EntityFrameworkCore.SqlServer`。 请参阅[数据库提供程序](../../core/providers/index.md)有关详细信息。

如果想要使用迁移，则还应安装`Microsoft.EntityFrameworkCore.Tools`包。

最好将从 EF6 NuGet 包 (EntityFramework) 安装，因为 EF Core 和 ef6 更高版本可以是在同一应用程序并行使用。 但是，如果不打算在您的应用程序的任何区域中使用 EF6，然后卸载包将有助于为提供的代码片段，需要注意的编译错误。

## <a name="swap-namespaces"></a>交换命名空间

在 EF6 中使用的大多数 Api 尚处于`System.Data.Entity`命名空间 （和相关子命名空间）。 第一个代码更改是切换到`Microsoft.EntityFrameworkCore`命名空间。 将使用派生的上下文文件通常会启动，然后计算出在这里，解决了在发生编译错误。

## <a name="context-configuration-connection-etc"></a>上下文配置 （连接等。）

中所述[确保 EF Core将工作为应用程序](ensure-requirements.md)，EF Core具有较少幻解决检测要连接到的数据库。 将需要重写`OnConfiguring`派生的上下文和使用数据库提供程序特定 API 来设置连接到数据库的方法。

大多数 EF6 应用程序的应用程序中存储连接字符串`App/Web.config`文件。 在 EF Core 中，阅读此连接字符串使用`ConfigurationManager`API。 可能需要添加对引用`System.Configuration`framework 程序集，以便能够使用此 API。

``` csharp
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
      optionsBuilder.UseSqlServer(ConfigurationManager.ConnectionStrings["BloggingDatabase"].ConnectionString);
    }
}
```

## <a name="update-your-code"></a>更新你的代码

在此情况下，这是解决编译错误和检查代码，若要查看的行为更改会影响您的问题。

## <a name="existing-migrations"></a>现有迁移

实际上，没有端口现有 ef6 更高版本迁移到 EF Core的可行方法。

如果可能，最好假定从 ef6 更高版本的所有以前迁移已应用到数据库，然后从，迁移架构的开始点使用 EF Core。 若要执行此操作，你将使用`Add-Migration`命令添加迁移后模型移植到 EF Core。 然后，你需要删除所有码`Up`和`Down`的基架生成的迁移方法。 当该初始迁移时已搭建基架时，以后的迁移将与模型进行比较。

## <a name="test-the-port"></a>测试端口

只是因为你的应用程序将编译，并不意味着它成功移植到 EF Core。 将需要测试应用程序，确保没有行为更改产生负面影响你的应用程序中的所有区域。
