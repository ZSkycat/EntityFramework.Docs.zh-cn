---
title: 测试以及 InMemory 的 EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 0d0590f1-1ea3-4d5c-8f44-db17395cd3f3
uid: core/miscellaneous/testing/in-memory
ms.openlocfilehash: 2754d1deba98fcee0eb88669293b2197545c8874
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42997887"
---
# <a name="testing-with-inmemory"></a>使用 InMemory 进行测试

InMemory 提供程序时，你想要测试组件使用类似于连接真实数据库，而无需实际的数据库操作的开销。

> [!TIP]  
> 可在 GitHub 上查看此文章的[示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Testing)。

## <a name="inmemory-is-not-a-relational-database"></a>InMemory 不是关系数据库

EF Core 数据库提供程序不需要是关系数据库。 InMemory 设计为通用数据库进行测试，而不是要模拟关系数据库。

包括一些示例：

* InMemory 将允许您将会违反引用完整性约束关系数据库中的数据保存。
* 如果在模型中的属性使用 DefaultValueSql(string)，这是一个关系数据库 API，不会影响针对 InMemory 运行时。
* [通过时间戳/行版本的并发](xref:core/modeling/concurrency#timestamprow-version)(`[Timestamp]`或`IsRowVersion`) 不受支持。 否[DbUpdateConcurrencyException](https://docs.microsoft.com/dotnet/api/microsoft.entityframeworkcore.dbupdateconcurrencyexception)如果更新执行此操作将引发使用旧的并发标记。

> [!TIP]  
> 对于许多测试目的这些差异将不起作用。 但是，如果你想要针对的表现得更像真正的关系数据库进行测试，请考虑使用[SQLite 内存中模式](sqlite.md)。

## <a name="example-testing-scenario"></a>示例测试方案

请考虑以下服务，允许应用程序代码来执行某些与博客相关的操作。 在内部使用`DbContext`连接到 SQL Server 数据库。 可以用来交换此上下文，以便我们可以编写此服务的有效测试，而无需修改代码，或执行大量工作来创建测试连接到 InMemory 数据库上下文的双精度。

[!code-csharp[Main](../../../../samples/core/Miscellaneous/Testing/BusinessLogic/BlogService.cs)]

## <a name="get-your-context-ready"></a>准备您的上下文

### <a name="avoid-configuring-two-database-providers"></a>避免配置两个数据库提供程序

在测试中要从外部配置要使用 InMemory 提供程序的上下文。 如果要配置数据库提供程序通过重写`OnConfiguring`在上下文中，则需要添加一些条件的代码，以确保，仅当其中一个已尚未配置配置的数据库提供程序。

[!code-csharp[Main](../../../../samples/core/Miscellaneous/Testing/BusinessLogic/BloggingContext.cs#OnConfiguring)]

> [!TIP]  
> 如果你使用的 ASP.NET Core，则应不需要此代码由于外部 （会在 Startup.cs) 的上下文已配置数据库提供程序。

### <a name="add-a-constructor-for-testing"></a>添加用于测试的构造函数

若要启用针对不同的数据库的测试的最简单方法是修改上下文，以公开一个构造函数接受`DbContextOptions<TContext>`。

[!code-csharp[Main](../../../../samples/core/Miscellaneous/Testing/BusinessLogic/BloggingContext.cs#Constructors)]

> [!TIP]  
> `DbContextOptions<TContext>` 告诉所有其设置，例如要连接到的数据库上下文。 这是通过在您的上下文中运行 OnConfiguring 方法生成的相同对象。

## <a name="writing-tests"></a>编写测试

使用此提供程序进行测试的关键是可以让使用 InMemory 提供程序，并控制内存中数据库的作用域的上下文。 通常情况下想干净的数据库的每个测试方法。

下面是使用 InMemory 数据库的测试类的示例。 每个测试方法指定一个唯一的数据库名称，表示每个方法都有其自己的 InMemory 数据库。

>[!TIP]
> 若要使用`.UseInMemoryDatabase()`扩展方法，引用的 NuGet 包`Microsoft.EntityFrameworkCore.InMemory`。

[!code-csharp[Main](../../../../samples/core/Miscellaneous/Testing/TestProject/InMemory/BlogServiceTests.cs)]
