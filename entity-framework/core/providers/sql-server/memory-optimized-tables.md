---
title: Microsoft SQL Server 数据库提供程序的内存优化表的 EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 2e007c82-c6e4-45bb-8129-851b79ec1a0a
uid: core/providers/sql-server/memory-optimized-tables
ms.openlocfilehash: 63d2cbf8b69e4f1945ad60914e284fb42c48e8db
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42995797"
---
# <a name="memory-optimized-tables-support-in-sql-server-ef-core-database-provider"></a>内存优化表支持在 SQL Server EF Core数据库提供程序

> [!NOTE]  
>
> EF Core 1.1 中已引入此功能。

[内存优化表](https://docs.microsoft.com/sql/relational-databases/in-memory-oltp/memory-optimized-tables)是整个表在内存中驻留在其中的 SQL Server 的功能。 表数据的第二个副本被保留在磁盘上，但仅用于持续性目的。 内存优化表中的数据只能从磁盘中读取在数据库恢复期间。 例如，服务器后，重新启动。

## <a name="configuring-a-memory-optimized-table"></a>配置内存优化表

你可以指定实体映射到的表是内存优化表。 当使用 EF Core创建和维护的数据库基于你的模型 (通过迁移或`Database.EnsureCreated()`)，将为这些实体创建内存优化表。

``` csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .ForSqlServerIsMemoryOptimized();
}
```
