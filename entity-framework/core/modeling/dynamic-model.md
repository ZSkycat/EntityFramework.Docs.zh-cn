---
title: 使用相同的 DbContext 类型的 EF Core 的多个模型之间交替
author: AndriySvyryd
ms.date: 12/10/2017
ms.assetid: 3154BF3C-1749-4C60-8D51-AE86773AA116
uid: core/modeling/dynamic-model
ms.openlocfilehash: 1d87efb668c12a2862583fba16a6c444b3cda9de
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42994981"
---
# <a name="alternating-between-multiple-models-with-the-same-dbcontext-type"></a>具有相同 DbContext 类型的多个模型之间切换

生成的模型`OnModelCreating`无法使用属性的上下文更改如何构建的模型。 例如它可用来排除某些属性：

[!code-csharp[Main](../../../samples/core/DynamicModel/DynamicContext.cs?name=Class)]

## <a name="imodelcachekeyfactory"></a>IModelCacheKeyFactory
但是如果您尝试执行上述无需进行其他更改则会得到相同的模型每次新的上下文创建的任何值的`IgnoreIntProperty`。 这由于缓存的机制 EF 使用通过仅调用来提高性能的模型`OnModelCreating`一次，并且缓存模型。

默认情况下 EF 假定任何给定的上下文类型的模型都将是相同。 若要实现此目的的默认实现`IModelCacheKeyFactory`返回只包含上下文类型的密钥。 若要更改此你需要更换`IModelCacheKeyFactory`服务。 新的实现需要对使用其他模型项返回一个对象进行比较，`Equals`会考虑会影响模型的所有变量的方法：

[!code-csharp[Main](../../../samples/core/DynamicModel/DynamicModelCacheKeyFactory.cs?name=Class)]
