---
title: EF Core 2.0 中的新增功能 - EF Core
author: divega
ms.date: 02/20/2018
ms.assetid: 2CB5809E-0EFB-44F6-AF14-9D5BFFFBFF9D
uid: core/what-is-new/ef-core-2.0
ms.openlocfilehash: b52b1fe6b2d5a585f4d55b0299891f61cbc968a3
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42997565"
---
# <a name="new-features-in-ef-core-20"></a>EF Core 2.0 中的新增功能

## <a name="net-standard-20"></a>.NET Standard 2.0
EF Core 现面向 .NET Standard 2.0，这意味着它可用于 .NET Core 2.0、.NET Framework 4.6.1 以及其他实现 .NET Standard 2.0 的库。
有关支持功能的更多详细信息，请参阅[支持的 .NET 实现](../platforms/index.md)。

## <a name="modeling"></a>建模

### <a name="table-splitting"></a>表拆分

现可将两个或多个实体类型映射到同一个表，其中主键列处于共享状态，每行对应两个或多个实体。

要使用表拆分，必须在共享该表的所有实体类型之间配置识别关系（其中外键属性构成主键）：

``` csharp
modelBuilder.Entity<Product>()
    .HasOne(e => e.Details).WithOne(e => e.Product)
    .HasForeignKey<ProductDetails>(e => e.Id);
modelBuilder.Entity<Product>().ToTable("Products");
modelBuilder.Entity<ProductDetails>().ToTable("Products");
```

### <a name="owned-types"></a>固有类型

固有实体类型可与另一个固有实体类型共享同一 CLR 类型，但是由于它不能仅由 CLR 类型标识，因此必须从另一个实体类型导航到该类型。 包含定义导航的实体是所有者。 查询所有者时，固有类型将默认包含在内。

依照约定，将为固有类型创建一个阴影主键，并通过表拆分将其映射到与所有者相同的表。 这样就可以通过类似于 EF6 中复杂类型的用法来使用固有类型：

``` csharp
modelBuilder.Entity<Order>().OwnsOne(p => p.OrderDetails, cb =>
    {
        cb.OwnsOne(c => c.BillingAddress);
        cb.OwnsOne(c => c.ShippingAddress);
    });

public class Order
{
    public int Id { get; set; }
    public OrderDetails OrderDetails { get; set; }
}

public class OrderDetails
{
    public StreetAddress BillingAddress { get; set; }
    public StreetAddress ShippingAddress { get; set; }
}

public class StreetAddress
{
    public string Street { get; set; }
    public string City { get; set; }
}
```
阅读[有关固有实体类型的部分](xref:core/modeling/owned-entities)，详细了解此功能。

### <a name="model-level-query-filters"></a>模型级别查询筛选器

EF Core 2.0 包含一个称为“模型级别查询筛选器”的新功能。 凭借此功能，可在元数据模型（通常为 OnModelCreating）的实体类型上直接定义 LINQ 查询谓词（通常传递给 LINQ Where 查询运算符的布尔表达式）。 此类筛选器自动应用于涉及这些实体类型（包括通过使用 Include 或直接导航属性引用等方式间接引用的实体类型）的所有 LINQ 查询。 此功能的一些常见应用如下：

- 软删除 - 实体类型定义 IsDeleted 属性。
- 多租户 - 实体类型定义 TenantId 属性。

以下简单示例演示了此功能在上述两种方案中的应用：

``` csharp
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }

    public int TenantId { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Post>().HasQueryFilter(
            p => !p.IsDeleted
            && p.TenantId == this.TenantId );
    }
}
```
我们为 ```Post``` 实体类型的实例定义一个实现多租户和软删除的模型级别筛选器。 请留意 DbContext 实例级别属性的使用：```TenantId```。 模型级筛选器将使用正确上下文实例（即执行查询的上下文实例）中的值。

可使用 IgnoreQueryFilters() 运算符对各个 LINQ 查询禁用筛选器。

#### <a name="limitations"></a>限制

- 不允许导航引用。 可根据反馈添加此功能。
- 仅可在层次结构的根实体类型上定义筛选器。

### <a name="database-scalar-function-mapping"></a>数据库标量函数映射

EF Core 2.0 包含来自 [Paul Middleton](https://github.com/pmiddleton) 的一个重要贡献功能，该功能支持将数据库标量函数映射到方法存根，使其可用于 LINQ 查询并转换为 SQL。

下面是如何使用该功能的简要说明：

在 `DbContext` 上声明一种静态方法，并使用 `DbFunctionAttribute` 对其批注：

``` csharp
public class BloggingContext : DbContext
{
    [DbFunction]
    public static int PostReadCount(int blogId)
    {
        throw new Exception();
    }
}
```

此类方法会自动注册。 注册后，对 LINQ 查询中方法的调用可转换为 SQL 中的函数调用：

``` csharp
var query =
    from p in context.Posts
    where BloggingContext.PostReadCount(p.Id) > 5
    select p;
```

需要注意以下事项：

- 依照约定，方法名称在生成 SQL 时会用作函数（此情况下为用户定义的函数）名称，但是你可以在方法注册期间替代名称和架构
- 当前仅支持标量函数
- 必须在数据库中创建映射函数。 EF Core 迁移不创建此函数

### <a name="self-contained-type-configuration-for-code-first"></a>Code First 的自包含类型配置

在 EF6 中可以通过从 EntityTypeConfiguration 派生来封装特定实体类型的 Code First 配置。 我们在 EF Core 2.0 中重新添加了此模式：

``` csharp
class CustomerConfiguration : IEntityTypeConfiguration<Customer>
{
  public void Configure(EntityTypeBuilder<Customer> builder)
  {
     builder.HasKey(c => c.AlternateKey);
     builder.Property(c => c.Name).HasMaxLength(200);
   }
}

...
// OnModelCreating
builder.ApplyConfiguration(new CustomerConfiguration());
```

## <a name="high-performance"></a>高性能

### <a name="dbcontext-pooling"></a>DbContext 池

在 ASP.NET Core 应用程序中使用 EF Core 的基本模式通常是先将自定义 DbContext 类型注册到依赖关系注入系统，然后通过控制器中的构造函数参数获取该类型的实例。 这会为各个请求创建一个 DbContext 新实例。

在版本 2.0 中，我们引入了一种在依赖关系注入中注册自定义 DbContext 类型的新方法，即以透明形式引入可重用 DbContext 实例的池。 要使用 DbContext 池，请在服务注册期间使用 ```AddDbContextPool``` 而不是 ```AddDbContext```：

``` csharp
services.AddDbContextPool<BloggingContext>(
    options => options.UseSqlServer(connectionString));
```

如果使用此方法，那么在控制器请求 DbContext 实例时，我们会首先检查池中有无可用的实例。 请求处理完成后，实例的任何状态都将被重置，并且实例本身会返回池中。

从概念上讲，此方法类似于连接池在 ADO.NET 提供程序中的运行原理，并具有节约 DbContext 实例初始化成本的优势。

### <a name="limitations"></a>限制

此新方法对使用 DbContext 的 ```OnConfiguring()``` 方法可执行的操作带来了一些限制。

> [!WARNING]  
> 如果要在不能在请求间共享的派生的 DbContext 类中保留自己的状态（例如私有字段），请避免使用 DbContext 池。 EF Core 仅会重置将 DbContext 实例添加到池之前所识别的状态。

### <a name="explicitly-compiled-queries"></a>显式编译的查询

这是第二个选择加入性能功能，旨在为大规模方案提供优势。

EF 早期版本以及 LINQ to SQL 中已经提供手动或显式编译的查询 API，允许应用程序缓存查询转换，使其可仅被计算一次并执行多次。

尽管 EF Core 通常可基于查询表达式的哈希表示法自动编译和缓存查询，但是使用此机制可绕过哈希计算和缓存查询，允许应用程序通过调用委托来使用已编译查询，从而实现性能小幅提升。

``` csharp
// Create an explicitly compiled query
private static Func<CustomerContext, int, Customer> _customerById =
    EF.CompileQuery((CustomerContext db, int id) =>
        db.Customers
            .Include(c => c.Address)
            .Single(c => c.Id == id));

// Use the compiled query by invoking it
using (var db = new CustomerContext())
{
   var customer = _customerById(db, 147);
}
```

## <a name="change-tracking"></a>更改跟踪

### <a name="attach-can-track-a-graph-of-new-and-existing-entities"></a>Attach 可跟踪新实体和现有实体的关系图。

EF Core 支持通过多种机制自动生成键值。 使用此功能时，如果键属性为 CLR 默认值，则会生成一个值（通常为零或 null）。 这意味着实体的关系图可传递到 `DbContext.Attach` 或 `DbSet.Attach`，并且 EF Core 会标记键已设置为 `Unchanged` 的实体，同时将没有键集的实体标记为 `Added`。 这样就可以轻松地在使用生成键时，附加混合了新实体和现有实体的关系图。 `DbContext.Update` 和 `DbSet.Update` 的工作原理相同，但是有键集的实体会被标记为 `Modified` 而不是 `Unchanged`。

## <a name="query"></a>查询

### <a name="improved-linq-translation"></a>改进的 LINQ 转换

可成功执行的查询更多，在数据库中（而不是内存中）计算的逻辑也更多，并且从数据库中检索的不必要数据更少。

### <a name="groupjoin-improvements"></a>GroupJoin 改进

这改进了为组联接生成的 SQL。 组联接通常是可选导航属性上子查询的结果。

### <a name="string-interpolation-in-fromsql-and-executesqlcommand"></a>FromSql 和 ExecuteSqlCommand 中的字符串内插

C# 6 引入了字符串内插功能，此功能允许将 C# 表达式直接嵌入字符串文本，从而提供了一种很适合在运行时生成字符串的方法。 在 EF Core 2.0 中，我们为两个主要 API 添加了对内插字符串的特殊支持，这两个 API 用于接收原始 SQL 字符串：```FromSql``` 和 ```ExecuteSqlCommand```。 这项新支持允许以“安全”方式使用 C# 字符串内插。 即，采用此方式可防止在运行时动态构造 SQL 时可能发生的常见 SQL 注入错误。

下面是一个示例：

``` csharp
var city = "London";
var contactTitle = "Sales Representative";

using (var context = CreateContext())
{
    context.Set<Customer>()
        .FromSql($@"
            SELECT *
            FROM ""Customers""
            WHERE ""City"" = {city} AND
                ""ContactTitle"" = {contactTitle}")
            .ToArray();
  }
```

本示例在 SQL 格式字符串中嵌入了两个变量。 EF Core 会生成如下 SQL：

```sql
@p0='London' (Size = 4000)
@p1='Sales Representative' (Size = 4000)

SELECT *
FROM ""Customers""
WHERE ""City"" = @p0
    AND ""ContactTitle"" = @p1
```

### <a name="effunctionslike"></a>EF.Functions.Like()

我们已添加 EF.Functions 属性，EF Core 或提供程序可使用该属性定义映射到数据库函数或运算符的方法，从而可在 LINQ 查询中调用它们。 此类方法的第一个示例是 Like()：

``` csharp
var aCustomers =
    from c in context.Customers
    where EF.Functions.Like(c.Name, "a%")
    select c;
```

请注意，Like() 附带一个内存中实现，处理内存中数据库时或需要在客户端上进行谓词计算时，此实现可能非常方便。

## <a name="database-management"></a>数据库管理

### <a name="pluralization-hook-for-dbcontext-scaffolding"></a>DbContext 基架的复数化挂钩

EF Core 2.0 引入了一种新的 IPluralizer 服务，用于单数化实体类型名称和复数化 DbSet 名称。 默认实现为 no-op，因此这仅仅是开发人员可轻松插入自己的复数化程序的挂钩。

下面是开发人员挂入自己的复数化程序的示例：

``` csharp
public class MyDesignTimeServices : IDesignTimeServices
{
    public void ConfigureDesignTimeServices(IServiceCollection services)
    {
        services.AddSingleton<IPluralizer, MyPluralizer>();
    }
}

public class MyPluralizer : IPluralizer
{
    public string Pluralize(string name)
    {
        return Inflector.Inflector.Pluralize(name) ?? name;
    }

    public string Singularize(string name)
    {
        return Inflector.Inflector.Singularize(name) ?? name;
    }
}
```

## <a name="others"></a>其他

### <a name="move-adonet-sqlite-provider-to-sqlitepclraw"></a>将 ADO.NET SQLite 提供程序移动到 SQLitePCL.raw
这为我们提供了一个 Microsoft.Data.Sqlite 中的更强大解决方案，用以在不同平台分发本机 SQLite 二进制文件。

### <a name="only-one-provider-per-model"></a>每个模型仅有一个提供程序
显著增强了提供程序与模型的交互方式，并简化了约定、注释和 Fluent API 用于不同提供程序的方法。

EF Core 2.0 现将对所用的每个不同提供程序生成不同的 [IModel](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Metadata/IModel.cs)。 这对应用程序而言通常是透明的。 这有助于简化较低级别的元数据 API，从而始终通过调用 `.Relational`（而不是 `.SqlServer`、`.Sqlite` 等）来访问常见关系元数据概念。

### <a name="consolidated-logging-and-diagnostics"></a>增强的日志记录和诊断

日志记录（基于 ILogger）和诊断（基于 DiagnosticSource）机制现可共享更多代码。

发送给 ILogger 的消息的事件 ID 在 2.0 中已更改。 现在，事件 ID 在 EF Core 代码内具有唯一性。 这些消息现在还遵循 MVC 等所用的结构化日志记录的标准模式。

记录器类别也已更改。 现提供通过 [DbLoggerCategory](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/DbLoggerCategory.cs) 访问的熟知类别集。

DiagnosticSource 事件现使用与相应 `ILogger` 消息相同的事件 ID 名称。
