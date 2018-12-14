---
title: 值的转换的 EF Core
author: ajcvickers
ms.date: 02/19/2018
ms.assetid: 3154BF3C-1749-4C60-8D51-AE86773AA116
uid: core/modeling/value-conversions
ms.openlocfilehash: 2a1956221ecc920feba796e4d95cc97259e89c53
ms.sourcegitcommit: 0cef7d448e1e47bdb333002e2254ed42d57b45b6
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/29/2018
ms.locfileid: "43152505"
---
# <a name="value-conversions"></a>值的转换

> [!NOTE]  
> 此功能是在 EF Core 2.1 的新增功能。

值转换器允许读取或写入数据库时要转换的属性值。 此转换可以是从某个值与另一个相同类型 （例如，加密字符串） 或值为一种类型的值的另一种类型 （例如，转换的枚举值和从数据库中的字符串。）

## <a name="fundamentals"></a>基础知识

指定的值转换器`ModelClrType`和一个`ProviderClrType`。 模型类型是属性的实体类型中的.NET 类型。 提供程序类型是理解的数据库提供程序的.NET 类型。 例如，若要保存为数据库中的字符串的枚举，模型类型是枚举的类型和提供程序类型是`String`。 这两种类型可以是相同的。

使用两个定义的转换`Func`表达式树： 一个来自`ModelClrType`到`ProviderClrType`以及从其他`ProviderClrType`到`ModelClrType`。 使用表达式树，这样它们可以编译到数据库访问代码的有效转换。 对于复杂的转换，表达式树可能是对执行转换的方法的简单调用。

## <a name="configuring-a-value-converter"></a>配置值转换器

中的属性定义值的转换`OnModelCreating`的你`DbContext`。 有关示例，请定义为枚举和实体类型：
``` csharp
public class Rider
{
    public int Id { get; set; }
    public EquineBeast Mount { get; set; }
}

public enum EquineBeast
{
    Donkey,
    Mule,
    Horse,
    Unicorn
}
```
然后可以在中定义的转换`OnModelCreating`存储为字符串 （例如，"驴女子"、"帮凶"，...） 在数据库中的枚举值：
``` csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder
        .Entity<Rider>()
        .Property(e => e.Mount)
        .HasConversion(
            v => v.ToString(),
            v => (EquineBeast)Enum.Parse(typeof(EquineBeast), v));
}
```
> [!NOTE]  
> 一个`null`永远不会将该值传递到值转换器。 这使转换的实现更轻松，并允许他们在可以为 null 和不可为 null 属性之间共享。

## <a name="the-valueconverter-class"></a>ValueConverter 类

调用`HasConversion`如上所示将创建`ValueConverter`实例，并将其设置的属性。 `ValueConverter`改为显式创建。 例如：
``` csharp
var converter = new ValueConverter<EquineBeast, string>(
    v => v.ToString(),
    v => (EquineBeast)Enum.Parse(typeof(EquineBeast), v));

modelBuilder
    .Entity<Rider>()
    .Property(e => e.Mount)
    .HasConversion(converter);
```
当多个属性使用相同的转换时，这很有用。

> [!NOTE]  
> 目前没有办法在一个位置指定给定类型的每个属性必须使用相同的值转换器。 此功能将被视为在将来的版本。

## <a name="built-in-converters"></a>内置的转换器

EF Core附带了一组预定义`ValueConverter`类，在中找到`Microsoft.EntityFrameworkCore.Storage.ValueConversion`命名空间。 这些是：
* `BoolToZeroOneConverter` -Bool 零和一到
* `BoolToStringConverter` -布尔值为"Y"和"N"等字符串
* `BoolToTwoValuesConverter` -Bool 到任意两个值
* `BytesToStringConverter` 为 Base64 编码的字符串的字节数组
* `CastingConverter` -转换需要类型转换
* `CharToStringConverter` -Char 到单字符字符串
* `DateTimeOffsetToBinaryConverter` 的为二进制编码的 64 位值 DateTimeOffset
* `DateTimeOffsetToBytesConverter` -DateTimeOffset 到字节数组
* `DateTimeOffsetToStringConverter` 的为字符串 DateTimeOffset
* `DateTimeToBinaryConverter` 的为包括 DateTimeKind 的 64 位值日期时间
* `DateTimeToStringConverter` -从 DateTime 到 string
* `DateTimeToTicksConverter` 的为刻度日期时间
* `EnumToNumberConverter` 的枚举基础数字
* `EnumToStringConverter` 的为字符串枚举
* `GuidToBytesConverter` -Guid 传递给字节数组
* `GuidToStringConverter` -Guid 传递给字符串
* `NumberToBytesConverter` 的指向字节数组任意数字值
* `NumberToStringConverter` 的为字符串任意数字值
* `StringToBytesConverter` -为 UTF8 字节字符串
* `TimeSpanToStringConverter` 的为字符串时间跨度
* `TimeSpanToTicksConverter` 的为刻度时间跨度

请注意，`EnumToStringConverter`包含在此列表。 这意味着，，无需转换显式指定，如上所示。 相反，只需使用内置的转换器：
``` csharp
var converter = new EnumToStringConverter<EquineBeast>();

modelBuilder
    .Entity<Rider>()
    .Property(e => e.Mount)
    .HasConversion(converter);
```
请注意，所有内置的转换器是无状态，因此单个实例可以安全地共享多个属性。

## <a name="pre-defined-conversions"></a>预定义的转换

对于存在内置的转换器的常见转换是无需显式指定转换器。 相反，只需配置应使用哪种提供程序类型和 EF 将自动使用适当的内置转换器。 更高版本，例如使用到字符串的转换的枚举，但 EF 将实际执行此操作自动如果配置的提供程序类型：
``` csharp
modelBuilder
    .Entity<Rider>()
    .Property(e => e.Mount)
    .HasConversion<string>();
```
可以通过显式指定列类型来实现相同的操作。 例如，如果实体类型的定义类似于这样：
``` csharp
public class Rider
{
    public int Id { get; set; }

    [Column(TypeName = "nvarchar(24)")]
    public EquineBeast Mount { get; set; }
}
```
枚举值将保存为没有任何进一步的配置，在数据库中的字符串`OnModelCreating`。

## <a name="limitations"></a>限制

有几个已知的当前限制的值转换系统：
* 如上所述，`null`无法转换。
* 目前没有办法分散到多个列或进行相反转换的转换的一个属性。
* 使用的值转换可能会影响 EF Core能够把表达式转换到 SQL。 对于这种情况下，将记录警告。
删除这些限制正在考虑针对将来的版本。
