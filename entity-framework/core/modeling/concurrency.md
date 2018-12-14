---
title: 并发标记的 EF Core
author: rowanmiller
ms.date: 03/03/2018
ms.assetid: bc8b1cb0-befe-4b67-8004-26e6c5f69385
uid: core/modeling/concurrency
ms.openlocfilehash: 0051d416544a11385f99d36e45843c5b20725af7
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42994221"
---
# <a name="concurrency-tokens"></a>并发标记

> [!NOTE]
> 此页介绍了如何配置并发标记。 请参阅[处理并发冲突](../saving/concurrency.md)并发控制 EF Core 以及如何处理应用程序中的并发冲突的示例中的工作原理的详细说明。

属性配置为并发标记用于实现乐观并发控制。

## <a name="conventions"></a>约定

按照约定，属性永远不会配置为并发标记。

## <a name="data-annotations"></a>数据注释

可以使用数据注释属性配置为并发标记。

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/Samples/Concurrency.cs#ConfigureConcurrencyAnnotations)]

## <a name="fluent-api"></a>Fluent API

Fluent API 可用于将属性配置为并发标记。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Samples/Concurrency.cs#ConfigureConcurrencyFluent)]

## <a name="timestamprow-version"></a>时间戳/行版本

时间戳是每次插入或更新行时，由数据库生成一个新的值是其中一个属性。 该属性也被视为并发标记。 这可确保您将收到异常，如果任何其他人已修改的行，你尝试更新，因为查询的数据。

这如何实现由正在使用的数据库提供程序。 对于 SQL Server，时间戳通常用于在*byte []* 属性，它将设置为*ROWVERSION*数据库中的列。

### <a name="conventions"></a>约定

按照约定，永远不会作为时间戳配置属性。

### <a name="data-annotations"></a>数据注释

可以使用数据注释属性配置为时间戳。

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/Samples/Timestamp.cs#ConfigureTimestampAnnotations)]

### <a name="fluent-api"></a>Fluent API

Fluent API 可用于将属性配置为时间戳。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Samples/Timestamp.cs#ConfigureTimestampFluent)]
