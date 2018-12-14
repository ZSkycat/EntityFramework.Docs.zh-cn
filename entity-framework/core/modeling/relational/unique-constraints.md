---
title: 备用键 （唯一约束） 的 EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 3d419dcf-2b5d-467c-b408-ea03d830721a
uid: core/modeling/relational/unique-constraints
ms.openlocfilehash: 7ec58ee31aac79e15329dc8542f37fd117772fbe
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42994186"
---
# <a name="alternate-keys-unique-constraints"></a>备用键 （唯一约束）

> [!NOTE]  
> 一般而言，本部分中的配置适用于关系数据库。 安装关系数据库提供程序时，此处显示的扩展方法将变为可用（原因在于共享的 Microsoft.EntityFrameworkCore.Relational 包）。

在模型中每个备用键是引入了唯一约束。

## <a name="conventions"></a>约定

按照约定，索引和约束，引入了的备用键将被命名为`AK_<type name>_<property name>`。 用于备用组合键`<property name>`变得下划线分隔属性名称的列表。

## <a name="data-annotations"></a>数据注释

不能使用数据批注配置唯一约束。

## <a name="fluent-api"></a>Fluent API

可以使用 Fluent API 配置备用键的索引和约束名称。

[!code-csharp[Main](../../../../samples/core/Modeling/FluentAPI/Samples/Relational/AlternateKeyName.cs?name=Model&highlight=9)]
