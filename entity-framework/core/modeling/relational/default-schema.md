---
title: 默认架构的 EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: e6e58473-9f5e-4a1f-ac0f-b87d2cbb667e
uid: core/modeling/relational/default-schema
ms.openlocfilehash: 800551bbadd0a9e8b5eb7070a8ccf6ed2407e3d2
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42995361"
---
# <a name="default-schema"></a>默认架构

> [!NOTE]  
> 一般而言，本部分中的配置适用于关系数据库。 安装关系数据库提供程序时，此处显示的扩展方法将变为可用（原因在于共享的 Microsoft.EntityFrameworkCore.Relational 包）。

默认架构是将在中创建对象，如果该对象未显式配置架构的数据库架构。

## <a name="conventions"></a>约定

按照约定，数据库提供程序将选择最合适的默认架构。 例如，将使用 Microsoft SQL Server`dbo`架构和 SQLite 将不使用架构 （因为在 SQLite 中不支持架构）。

## <a name="data-annotations"></a>数据注释

不可以设置使用数据注释的默认架构。

## <a name="fluent-api"></a>Fluent API

Fluent API 可用于指定默认架构。

<!-- [!code-csharp[Main](samples/core/relational/Modeling/FluentAPI/Samples/Relational/DefaultSchema.cs?highlight=7)] -->
``` csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.HasDefaultSchema("blogging");
    }
}
```
