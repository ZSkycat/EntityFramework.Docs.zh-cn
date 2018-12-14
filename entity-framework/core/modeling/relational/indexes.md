---
title: 索引的 EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 4581e7ba-5e7f-452c-9937-0aaf790ba10a
uid: core/modeling/relational/indexes
ms.openlocfilehash: 605b30ce710d9034deab97f695496ec66a576565
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42993212"
---
# <a name="indexes"></a>索引

> [!NOTE]  
> 一般而言，本部分中的配置适用于关系数据库。 安装关系数据库提供程序时，此处显示的扩展方法将变为可用（原因在于共享的 Microsoft.EntityFrameworkCore.Relational 包）。

关系数据库中的索引映射到相同的概念的 Entity Framework core 中的索引。

## <a name="conventions"></a>约定

按照约定，索引名为`IX_<type name>_<property name>`。 对于复合索引`<property name>`变得下划线分隔属性名称的列表。

## <a name="data-annotations"></a>数据注释

索引不使用数据批注配置。

## <a name="fluent-api"></a>Fluent API

可以使用 Fluent API 来配置索引的名称。

[!code-csharp[Main](../../../../samples/core/Modeling/FluentAPI/Samples/Relational/IndexName.cs?name=Model&highlight=9)]

此外可以指定筛选器。

[!code-csharp[Main](../../../../samples/core/Modeling/FluentAPI/Samples/Relational/IndexFilter.cs?name=Model&highlight=9)]

当使用 EF 的 SQL Server 提供程序将添加 IS NOT NULL' 筛选所有唯一索引的一部分的可以为 null 列。 若要重写可以提供此约定`null`值。

[!code-csharp[Main](../../../../samples/core/Modeling/FluentAPI/Samples/Relational/IndexNoFilter.cs?name=Model&highlight=10)]
