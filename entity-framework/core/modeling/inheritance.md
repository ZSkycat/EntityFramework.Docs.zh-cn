---
title: 继承的 EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 754be334-dd21-450e-9d22-2591e80012a2
uid: core/modeling/inheritance
ms.openlocfilehash: c5fa9d13dec8cfc3e1cac69e471f509cbbb9e4c5
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42995891"
---
# <a name="inheritance"></a>继承

EF 模型中的继承用于控制如何在数据库中表示实体类中的继承。

## <a name="conventions"></a>约定

按照约定，负责数据库提供程序，以确定如何继承将表示在数据库中。 请参阅[继承 （关系数据库）](relational/inheritance.md)这与关系数据库提供程序的处理方式。

如果两个或多个继承的类型显式包含在模型中，EF 将仅设置继承。 EF 不会扫描的基类或派生类型，否则不包含在模型中。 可以在模型中包含类型，通过公开*DbSet<TEntity>* 继承层次结构中每个类型。

[!code-csharp[Main](../../../samples/core/Modeling/Conventions/Samples/InheritanceDbSets.cs?highlight=3-4&name=Model)]

如果不想要公开*DbSet<TEntity>* 对于层次结构中的一个或多个实体，你可以使用 Fluent API 以确保它们都包含在模型中。
如果您不依赖于约定可以指定使用显式的基类型`HasBaseType`。

[!code-csharp[Main](../../../samples/core/Modeling/Conventions/Samples/InheritanceModelBuilder.cs?highlight=7&name=Context)]

> [!NOTE]
> 可以使用`.HasBaseType((Type)null)`从层次结构中删除某个实体类型。

## <a name="data-annotations"></a>数据注释

不能使用数据注释来配置继承。

## <a name="fluent-api"></a>Fluent API

用于继承的 Fluent API 取决于正在使用的数据库提供程序。 请参阅[继承 （关系数据库）](relational/inheritance.md)可以针对关系数据库提供程序执行的配置。
