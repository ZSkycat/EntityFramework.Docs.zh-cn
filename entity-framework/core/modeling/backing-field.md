---
title: 备份 EF Core的字段-
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: a628795e-64df-4f24-a5e8-76bc261e7ed8
uid: core/modeling/backing-field
ms.openlocfilehash: 79221b6f7968675ff10f80d5df181b674b6a20c9
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42994088"
---
# <a name="backing-fields"></a>支持字段

> [!NOTE]  
> 此功能是 EF Core 1.1 中的新增功能。

支持字段允许 EF 读取和/或写入字段而不是一个属性。 在类中的封装用于限制的使用和/或增强围绕访问数据的语义由应用程序代码，但值应进行读取和/或写入到数据库而无需使用这些限制时这很有用 /增强功能。

## <a name="conventions"></a>约定

按照约定，将为支持字段 （按优先顺序列出） 的给定属性发现以下字段。 对于模型中包含的属性仅发现字段。 在模型中包括属性的详细信息，请参阅[包括和排除属性](included-properties.md)。

* `_<camel-cased property name>`
* `_<property name>`
* `m_<camel-cased property name>`
* `m_<property name>`

[!code-csharp[Main](../../../samples/core/Modeling/Conventions/Samples/BackingField.cs#Sample)]

如果配置支持字段，EF 将直接写入到该字段具体化实体实例从数据库 （而不是使用属性 setter） 时。 如果 EF 需要读取或写入的值在其他时候，它将如有可能使用该属性。 例如，如果 EF 需要更新的属性的值，它将使用属性 setter，如果已定义。 如果该属性是只读的然后它将写入到该字段。

## <a name="data-annotations"></a>数据注释

支持字段不能使用数据批注进行配置。

## <a name="fluent-api"></a>Fluent API

可以使用 Fluent API 配置属性的支持字段。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Samples/BackingField.cs#Sample)]

### <a name="controlling-when-the-field-is-used"></a>控制时使用的字段

您可以配置时 EF 使用的字段或属性。 请参阅[PropertyAccessMode 枚举](https://docs.microsoft.com/dotnet/api/microsoft.entityframeworkcore.propertyaccessmode)有关受支持的选项。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Samples/BackingFieldAccessMode.cs#Sample)]

### <a name="fields-without-a-property"></a>没有一个属性的字段

此外可以在实体类中，没有相应的 CLR 属性，而是使用字段来存储实体中的数据在模型中创建的概念的属性。 这与不同[隐藏属性](shadow-properties.md)、 数据更改跟踪器中的存储位置。 这通常可在实体类使用的方法来获取/设置值。

可以让 EF 中的字段名称`Property(...)`API。 如果不存在具有给定名称属性，EF 将查找字段。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Samples/BackingFieldNoProperty.cs#Sample)]

您还可以选择为提供的属性之外的字段名称的名称。 创建模型时，然后使用此名称，最值得注意的是它将使用映射到数据库中的列名称。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Samples/BackingFieldConceptualProperty.cs#Sample)]

当在实体类中没有任何属性时，可以使用`EF.Property(...)`LINQ 查询来指从概念上讲是模型的一部分的属性中的方法。

``` csharp
var blogs = db.blogs.OrderBy(b => EF.Property<string>(b, "Url"));
```
