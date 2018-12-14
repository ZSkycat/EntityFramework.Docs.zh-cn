---
title: 数据类型的 EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 9d2e647f-29e4-483b-af00-74269eb06e8f
uid: core/modeling/relational/data-types
ms.openlocfilehash: 9060f66c752be01090ce40be6bf3a32f348ce571
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42993516"
---
# <a name="data-types"></a>数据类型

> [!NOTE]  
> 一般而言，本部分中的配置适用于关系数据库。 安装关系数据库提供程序时，此处显示的扩展方法将变为可用（原因在于共享的 Microsoft.EntityFrameworkCore.Relational 包）。

数据类型是指数据库特定类型的属性映射到其中的列。

## <a name="conventions"></a>约定

按照约定，数据库提供程序选择基于属性的 CLR 类型的数据类型。 它还会考虑其他元数据，例如配置[最大长度](../max-length.md)，该属性是否属于主键，等等。

例如，使用 SQL Server`datetime2(7)`有关`DateTime`属性，并`nvarchar(max)`的`string`属性 (或`nvarchar(450)`的`string`用作键的属性)。

## <a name="data-annotations"></a>数据注释

可以使用数据注释来指定精确的数据类型的列。

例如下面的代码中将配置`Url`为非 unicode 字符串，最大长度为`200`并`Rating`用作小数点的精度`5`和缩放的`2`。

[!code-csharp[Main](../../../../samples/core/Modeling/DataAnnotations/Samples/Relational/DataType.cs?name=Entities&highlight=4,6)]

## <a name="fluent-api"></a>Fluent API

Fluent API 还可用于指定相同的数据类型的列。

[!code-csharp[Main](../../../../samples/core/Modeling/FluentAPI/Samples/Relational/DataType.cs?name=Model&highlight=9-10)]
