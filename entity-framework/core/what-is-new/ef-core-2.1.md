---
title: EF Core 2.1 中的新增功能 - EF Core
author: divega
ms.date: 02/20/2018
ms.assetid: 585F90A3-4D5A-4DD1-92D8-5243B14E0FEC
uid: core/what-is-new/ef-core-2.1
ms.openlocfilehash: f67f2e695d269e2dde11d396f9a67fd137600f56
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489396"
---
# <a name="new-features-in-ef-core-21"></a>EF Core 2.1 中的新增功能

除了大量缺陷修复以及小规模功能增强和性能增强之外，EF Core 2.1 还新增了一些有吸引力的功能：

## <a name="lazy-loading"></a>延迟加载
EF Core 现包含创作可按需加载导航属性的实体类所必需的构建基块。 我们还创建了一个新包 - Microsoft.EntityFrameworkCore.Proxies，它可利用这些构建基块基于最小修改实体类（例如具有虚拟导航属性的类）来生成延迟加载代理类。

阅读[有关延迟加载的部分](xref:core/querying/related-data#lazy-loading)详细了解本主题。

## <a name="parameters-in-entity-constructors"></a>实体构造函数中的参数
作为延迟加载所必需的一个构建基块，我们启用了实体创建，实体可将参数纳入其构造函数中。 可使用参数来注入属性值、延迟加载委托和服务。

阅读[有关含参数的实体构造函数部分](xref:core/modeling/constructors)详细了解本主题。

## <a name="value-conversions"></a>值转换
到目前为止，EF Core 只能映射底层数据库提供程序本机支持的属性类型。 在列和属性之间来回复制值，无需进行任何转换。 自 EF Core 2.1 起，可通过值转换来转换从列获取的值，之后再将其应用到属性，反之亦然。 我们具有可以根据约定按需应用的大量转换，以及显式配置 API，后者允许在列和属性之间注册自定义转换。 此功能有如下一些用法：

- 将枚举存储为字符串
- 使用 SQL Server 映射无符号整数
- 自动加密和解密属性值

阅读[有关值转换的部分](xref:core/modeling/value-conversions)详细了解本主题。  

## <a name="linq-groupby-translation"></a>LINQ GroupBy 转换
在 2.1 版之前，EF Core 中的 GroupBy LINQ 运算符将始终在内存中进行计算。 在大多数情况下，我们现在支持将其转换为 SQL GROUP BY 子句。

此示例显示了一个用 GroupBy 来计算各种聚合函数的查询：

``` csharp
var query = context.Orders
    .GroupBy(o => new { o.CustomerId, o.EmployeeId })
    .Select(g => new
        {
          g.Key.CustomerId,
          g.Key.EmployeeId,
          Sum = g.Sum(o => o.Amount),
          Min = g.Min(o => o.Amount),
          Max = g.Max(o => o.Amount),
          Avg = g.Average(o => Amount)
        });
```

相应的 SQL 转化如下所示：

``` SQL
SELECT [o].[CustomerId], [o].[EmployeeId],
    SUM([o].[Amount]), MIN([o].[Amount]), MAX([o].[Amount]), AVG([o].[Amount])
FROM [Orders] AS [o]
GROUP BY [o].[CustomerId], [o].[EmployeeId];
```

## <a name="data-seeding"></a>数据种子设定
新版本可提供初始数据来填充数据库。 与 EF6 不同，种子设定数据作为模型配置的一部分与实体类型相关联。 随后将数据库升级为新版本模型时，EF Core 迁移会自动计算需要应用的插入、更新或删除操作。

如示例所示，可使用它在 `OnModelCreating` 中为 Post 配置种子数据：

``` csharp
modelBuilder.Entity<Post>().HasData(new Post{ Id = 1, Text = "Hello World!" });
```

阅读[有关数据种子设定的部分](xref:core/modeling/data-seeding)详细了解本主题。  

## <a name="query-types"></a>查询类型
EF Core 模型现可包含查询类型。 与实体类型不同，查询类型上未定义键，也不能插入、删除或更新查询类型（即它们为只读），但查询可直接返回查询类型。 以下是查询类型的一些用法：

- 映射到没有主键的视图
- 映射到没有主键的表
- 映射到模型中定义的查询
- 用作 `FromSql()` 查询的返回类型

阅读[有关查询类型的部分](xref:core/modeling/query-types)详细了解本主题。

## <a name="include-for-derived-types"></a>针对派生类型的 Include
现可在编写 `Include` 方法的表达式时指定仅在派生类型上定义的导航属性。 对于 `Include` 的强类型版本，我们支持使用显式强制转换或 `as` 运算符。 我们现在还支持在 `Include` 的字符串版本中引用在派生类型上定义的导航属性的名称：

``` csharp
var option1 = context.People.Include(p => ((Student)p).School);
var option2 = context.People.Include(p => (p as Student).School);
var option3 = context.People.Include("School");
```

阅读[有关派生类型的 Include 部分](xref:core/querying/related-data#include-on-derived-types)详细了解本主题。

## <a name="systemtransactions-support"></a>System.Transactions 支持
我们增加了对 System.Transactions 功能（如 TransactionScope）的应用。 使用支持该功能的数据库提供程序时，这将适用于 .NET Framework 和 .NET Core。

阅读[有关 System.Transactions 的部分](xref:core/saving/transactions#using-systemtransactions)详细了解本主题。

## <a name="better-column-ordering-in-initial-migration"></a>初始迁移时生成更好的列顺序
根据客户反馈，我们对迁移进行了更新，使得先以与类中声明的属性相同的顺序为表生成列。 请注意，在创建初始表后，添加新成员时，EF Core 不能更改顺序。

## <a name="optimization-of-correlated-subqueries"></a>相关子查询优化
我们改进了查询转换，避免在许多常见情况下执行“N + 1”SQL 查询，一般情况下，在投影中使用导航属性后，来自根查询的数据会与来自相关子查询的数据相连接。 进行优化需要缓冲子查询的结果，且我们要求修改查询，选择新行为。

例如，以下查询通常会转换为：一个“客户”查询，加上 N（其中“N”是返回的客户数量）个单独的“订单”查询：

``` csharp
var query = context.Customers.Select(
    c => c.Orders.Where(o => o.Amount  > 100).Select(o => o.Amount));
```

将 `ToList()` 放入正确的位置，指示缓冲适用于订单，即可启用优化：

``` csharp
var query = context.Customers.Select(
    c => c.Orders.Where(o => o.Amount  > 100).Select(o => o.Amount).ToList());
```

请注意，此查询只会被转换为两个 SQL 查询：一个“客户”查询，一个“订单”查询。

## <a name="owned-attribute"></a>[Owned] 属性

现只需使用 `[Owned]` 注释类型，并确保所有者实体添加到了模型中，即可配置[固有实体类型](xref:core/modeling/owned-entities)：

``` csharp
[Owned]
public class StreetAddress
{
    public string Street { get; set; }
    public string City { get; set; }
}

public class Order
{
    public int Id { get; set; }
    public StreetAddress ShippingAddress { get; set; }
}
```

## <a name="command-line-tool-dotnet-ef-included-in-net-core-sdk"></a>.NET Core SDK 中包含的命令行工具 dotnet-ef

dotnet-ef 命令现在是 .NET Core SDK 的一部分，因此无须在项目中使用 DotNetCliToolReference 即可使用各项迁移，或通过现有数据库搭建 DbContext 基架。

有关如何为不同版本的 .NET Core SDK 和 EF Core 启用命令行工具的详细信息，请参阅[安装工具](xref:core/miscellaneous/cli/dotnet#installing-the-tools)的相关部分。

## <a name="microsoftentityframeworkcoreabstractions-package"></a>Microsoft.EntityFrameworkCore.Abstractions 包
可以在项目中使用新包内的一些属性和接口，从而启用 EF Core 功能，而无需依赖 EF Core 整体。 例如，[Owned] 属性和 ILazyLoader 接口位于此处。

## <a name="state-change-events"></a>状态更改事件

`ChangeTracker` 中新增的 `Tracked` 和 `StateChanged` 事件可用于编写逻辑，以响应进入 DbContext 或状态更改的实体。

## <a name="raw-sql-parameter-analyzer"></a>原始 SQL 参数分析器

EF Core 随附新增一个代码分析器，用于检测原始 SQL API（如 `FromSql` 或 `ExecuteSqlCommand`）的潜在不安全用法。 例如，对于下面的查询，将会看到一条警告，因为 minAge 未参数化：

``` csharp
var sql = $"SELECT * FROM People WHERE Age > {minAge}";
var query = context.People.FromSql(sql);
```

## <a name="database-provider-compatibility"></a>数据库提供程序兼容性

建议配合使用 EF Core 2.1 以及已更新或至少已经过测试可用于 EF Core 2.1 的提供程序。

> [!TIP]
> 如果新功能出现任何意外的不兼容或问题，或你有任何相关反馈，请使用[我们的问题跟踪器](https://github.com/aspnet/EntityFrameworkCore/issues/new)进行报告。
