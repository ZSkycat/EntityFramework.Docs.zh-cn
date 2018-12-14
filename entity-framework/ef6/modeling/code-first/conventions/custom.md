---
title: 自定义代码优先约定-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: dd2bdbd9-ae9e-470a-aeb8-d0ba160499b7
ms.openlocfilehash: cfd7f7cad532dca5227793c04d7d91e977ea5e4e
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489839"
---
# <a name="custom-code-first-conventions"></a>自定义代码优先约定
> [!NOTE]
> **仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。 如果使用的是早期版本，则部分或全部信息不适用。

使用 Code First 时您的模型会计算从您使用的一组约定的类。 默认值[Code First 约定](~/ef6/modeling/code-first/conventions/built-in.md)确定某些事宜等的属性变为实体的主键、 表实体映射到，并在默认情况下哪些精度和小数位数有 decimal 列的名称。

有时不适合您的模型，这些默认约定，必须先解决它们通过配置多个使用数据注释或 Fluent API 的单个实体。 自定义代码优先约定，可以定义您自己提供配置的默认值为您的模型的约定。 在本演练中，我们将了解不同类型的自定义约定以及如何创建每个。


## <a name="model-based-conventions"></a>基于模型的约定

本页介绍如何自定义约定 DbModelBuilder API。 此 API 应足以满足创作大多数自定义约定。 但是，还有创作模型基于约定的操作的最终模型，在创建后的约定-若要处理的高级的方案的功能。 有关详细信息，请参阅[基于模型的约定](~/ef6/modeling/code-first/conventions/model.md)。

 

## <a name="our-model"></a>我们的模型

让我们首先定义一个简单的模型，我们可以使用与我们的约定。 将以下类添加到你的项目。

``` csharp
    using System;
    using System.Collections.Generic;
    using System.Data.Entity;
    using System.Linq;

    public class ProductContext : DbContext
    {
        static ProductContext()
        {
            Database.SetInitializer(new DropCreateDatabaseIfModelChanges<ProductContext>());
        }

        public DbSet<Product> Products { get; set; }
    }

    public class Product
    {
        public int Key { get; set; }
        public string Name { get; set; }
        public decimal? Price { get; set; }
        public DateTime? ReleaseDate { get; set; }
        public ProductCategory Category { get; set; }
    }

    public class ProductCategory
    {
        public int Key { get; set; }
        public string Name { get; set; }
        public List<Product> Products { get; set; }
    }
```

 

## <a name="introducing-custom-conventions"></a>引入了自定义约定

让我们来编写配置已命名密钥为其实体类型的主键的任何属性的约定。

模型生成器中，可通过在上下文中重写 OnModelCreating 上启用了约定。 按如下所示更新 ProductContext 类：

``` csharp
    public class ProductContext : DbContext
    {
        static ProductContext()
        {
            Database.SetInitializer(new DropCreateDatabaseIfModelChanges<ProductContext>());
        }

        public DbSet<Product> Products { get; set; }

        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            modelBuilder.Properties()
                        .Where(p => p.Name == "Key")
                        .Configure(p => p.IsKey());
        }
    }
```

现在，我们已命名密钥的模型中的任何属性将配置为所有实体的主键的一部分。

此外可以使通过筛选我们要配置的属性的类型更具体是将约定：

``` csharp
    modelBuilder.Properties<int>()
                .Where(p => p.Name == "Key")
                .Configure(p => p.IsKey());
```

这会配置其实体的键，但前提是一个整数，这些调用要为主键的所有属性。

IsKey 方法的一个有趣功能是，它是累加性。 这意味着，如果您对多个属性调用 IsKey，它们将成为复合键的一部分。 对于这一点是，指定键的多个属性时，还必须指定这些属性的顺序。 可以通过调用方法如下所示的 HasColumnOrder 来执行此操作：

``` csharp
    modelBuilder.Properties<int>()
                .Where(x => x.Name == "Key")
                .Configure(x => x.IsKey().HasColumnOrder(1));

    modelBuilder.Properties()
                .Where(x => x.Name == "Name")
                .Configure(x => x.IsKey().HasColumnOrder(2));
```

此代码将在模型 int 键列和字符串名称列组成的复合键中配置的类型。 如果我们查看模型设计器中它将如下所示：

![复合键](~/ef6/media/compositekey.png)

属性约定的另一个示例是我要映射到 SQL Server 中而不是日期时间的 datetime2 类型的模型中配置所有日期时间属性。 您可以使用以下实现此目的：

``` csharp
    modelBuilder.Properties<DateTime>()
                .Configure(c => c.HasColumnType("datetime2"));
```

 

## <a name="convention-classes"></a>约定类

定义约定的另一种方法是使用约定类来封装你的约定。 使用约定类时，创建从 System.Data.Entity.ModelConfiguration.Conventions 命名空间中的约定类继承的类型。

我们可以使用我们前面显示通过执行以下的 datetime2 约定创建约定类：

``` csharp
    public class DateTime2Convention : Convention
    {
        public DateTime2Convention()
        {
            this.Properties<DateTime>()
                .Configure(c => c.HasColumnType("datetime2"));        
        }
    }
```

若要告诉 EF 使用此约定，将其添加到约定集合在 OnModelCreating，如果您已以及本演练将如下所示：

``` csharp
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        modelBuilder.Properties<int>()
                    .Where(p => p.Name.EndsWith("Key"))
                    .Configure(p => p.IsKey());

        modelBuilder.Conventions.Add(new DateTime2Convention());
    }
```

您可以看到我们将我们约定的实例添加到约定集合。 从约定继承提供了分组和在团队或项目之间共享约定的简便方法。 例如，可以令使用一组通用的您所在组织的所有项目使用的约定。

 

## <a name="custom-attributes"></a>自定义特性

约定的另一个出色用途是启用要在配置模型时使用的新属性。 为了说明这一点，让我们创建一个我们可以使用标记为非 Unicode 字符串属性的属性。

``` csharp
    [AttributeUsage(AttributeTargets.Property, AllowMultiple = false)]
    public class NonUnicode : Attribute
    {
    }
```

现在，让我们创建的约定，若要将此特性应用于我们的模型：

``` csharp
    modelBuilder.Properties()
                .Where(x => x.GetCustomAttributes(false).OfType<NonUnicode>().Any())
                .Configure(c => c.IsUnicode(false));
```

此约定可添加到任何字符串属性，这意味着数据库中的列使用非 Unicode 属性将存储为 varchar，而不是 nvarchar。

关于此约定需要注意的一点是，如果将使用非 Unicode 属性放在一个字符串属性以外的任何它将引发异常。 这是因为您不能配置 IsUnicode 字符串以外的其他任何类型上。 如果发生这种情况，然后即可在约定更具体，以便筛选出任何内容不是字符串。

尽管上述约定适用于定义自定义属性还有另一个 API，可以是更易于使用，尤其是当你想要使用从特性类的属性。

对于此示例中我们将更新我们属性并将其更改为 IsUnicode 属性，以便它如下所示：

``` csharp
    [AttributeUsage(AttributeTargets.Property, AllowMultiple = false)]
    internal class IsUnicode : Attribute
    {
        public bool Unicode { get; set; }

        public IsUnicode(bool isUnicode)
        {
            Unicode = isUnicode;
        }
    }
```

一旦我们有这，我们可以告诉约定属性应为 Unicode 我们属性上设置一个布尔值。 我们无法在我们已通过访问此类配置类的 ClrProperty 的约定来执行此操作：

``` csharp
    modelBuilder.Properties()
                .Where(x => x.GetCustomAttributes(false).OfType<IsUnicode>().Any())
                .Configure(c => c.IsUnicode(c.ClrPropertyInfo.GetCustomAttribute<IsUnicode>().Unicode));
```

这非常简单，但没有实现此目的通过使用具有更简洁的方式的约定 API 的方法。 无方法具有的参数键入 Func&lt;PropertyInfo、 T&gt; ，它接受 PropertyInfo Where 相同方法，但应返回一个对象。 如果返回的对象为 null，则将不会配置属性，这意味着您可以筛选出属性与之类似的位置，但它的不同在于它还将捕获返回的对象，并将其传递给配置方法。 此工作原理如下所示：

``` csharp
    modelBuilder.Properties()
                .Having(x =>x.GetCustomAttributes(false).OfType<IsUnicode>().FirstOrDefault())
                .Configure((config, att) => config.IsUnicode(att.Unicode));
```

自定义特性不是唯一的原因，若要使用 Having 方法，它可推论内容对其进行筛选配置类型或属性时所需的任何位置。

 

## <a name="configuring-types"></a>配置类型

到目前为止所有约定已对于属性，但在您的模型中配置类型的约定 API 的另一个区域。 体验是到目前为止，我们已了解的约定类似但内部配置的选项将在实体而不是属性级别。

类型级别约定可以是非常有用的事情之一更改表命名约定，以将映射到现有的架构不同于的 EF 默认设置，或创建新的数据库具有不同的命名约定。 若要执行此操作我们首先需要一个方法，可以接受我们的模型中类型的类型信息并返回该类型的表名称应该是什么：

``` csharp
    private string GetTableName(Type type)
    {
        var result = Regex.Replace(type.Name, ".[A-Z]", m => m.Value[0] + "_" + m.Value[1]);

        return result.ToLower();
    }
```

此方法接受一个类型，并返回而不是驼峰式大小写的下划线开头使用小写的字符串。 在我们的模型，这意味着 ProductCategory 类将其映射到名为 product 表\_而不是 ProductCategories 的类别。

我们有了该方法后我们可以在此类约定中调用它：

``` csharp
    modelBuilder.Types()
                .Configure(c => c.ToTable(GetTableName(c.ClrType)));
```

此约定在我们的模型将映射到从我们 GetTableName 方法返回的表名中配置的每个类型。 此约定是等效于调用 ToTable 方法使用 Fluent API 在模型中每个实体。

有一点需要注意相关信息是在调用时 ToTable EF 将需要提供作为确切的表名，而无需任何复数形式，它通常执行的操作时确定表名称的字符串。 这就是原因我们约定中的表名称是产品\_而不是产品类别\_类别。 我们可以在我们的约定中解析它，从而对复数化服务的调用自己。

下面的代码中我们将使用[依赖项解析](~/ef6/fundamentals/configuring/dependency-resolution.md)EF6 检索会使用 EF 复数化服务中添加的功能和复数化我们表的名称。

``` csharp
    private string GetTableName(Type type)
    {
        var pluralizationService = DbConfiguration.DependencyResolver.GetService<IPluralizationService>();

        var result = pluralizationService.Pluralize(type.Name);

        result = Regex.Replace(result, ".[A-Z]", m => m.Value[0] + "_" + m.Value[1]);

        return result.ToLower();
    }
```

> [!NOTE]
> GetService 的泛型版本是 System.Data.Entity.Infrastructure.DependencyResolution 命名空间中的扩展方法，将需要添加一个 using 语句到上下文中才能使用它。

### <a name="totable-and-inheritance"></a>ToTable 和继承

ToTable 的另一个重要方面是，如果显式将类型映射到给定的表，则可以更改将使用 EF 映射策略。 如果对于继承层次结构中每个类型调用 ToTable，将类型名称作为表的名称传递，我们更高版本，然后您将更改默认的每个层次结构一个表 (TPH) 映射策略为每种类型一个表 (TPT)。 描述此的最佳方法是有具体的示例：

``` csharp
    public class Employee
    {
        public int Id { get; set; }
        public string Name { get; set; }
    }

    public class Manager : Employee
    {
        public string SectionManaged { get; set; }
    }
```

默认情况下 employee 和 manager 映射到数据库中的同一个表 （员工）。 表将包含员工和管理员都会告诉您哪些类型的实例存储在每行一个鉴别器列。 这是 TPH 映射，因为没有为层次结构一个表。 但是，如果在这两个类上调用 ToTable 然后每种类型将改为将映射到自己的表，也称为 TPT 由于每个类型都有其自己的表。

``` csharp
    modelBuilder.Types()
                .Configure(c=>c.ToTable(c.ClrType.Name));
```

上面的代码将映射到类似于下面的表结构：

![tpt 示例](~/ef6/media/tptexample.jpg)

可以避免此问题，同时保持几种方法中的默认 TPH 映射：

1.  调用层次结构中每个类型一个表同名 ToTable。
2.  仅在层次结构，在本例中，将为员工的基本类上调用 ToTable。

 

## <a name="execution-order"></a>执行顺序

在最后一个 wins 方式，与 Fluent API 相同操作约定。 这意味着，即如果您编写两个约定的配置相同的选项将相同的属性，则最后一个执行 wins。 例如，下面的代码中的所有字符串的最大长度设置为 500，但接下来我们配置的最大长度为 250 的模型中名为 Name 的所有属性。

``` csharp
    modelBuilder.Properties<string>()
                .Configure(c => c.HasMaxLength(500));

    modelBuilder.Properties<string>()
                .Where(x => x.Name == "Name")
                .Configure(c => c.HasMaxLength(250));
```

由于要设置最大长度为 250 的惯例是之后的所有字符串都设置为 500，在我们的模型名为 Name 的所有属性将具有最大为 250 任何其他字符串，如说明时，为 500。 以这种方式使用约定意味着您可以为类型或模型和然后覆盖属性提供常规约定以不同的子集。

Fluent API 和数据批注还可用来重写中特定用例的约定。 在上例如果我们已使用 Fluent API 来设置属性的最大长度然后我们可能已将它放之前或之后约定，因为更具体的 Fluent API 可以赢取更多常规配置约定。

 

## <a name="built-in-conventions"></a>内置约定

自定义约定可能会影响默认 Code First 约定，因为它可用于添加运行之前或之后另一个约定的约定。 若要执行此操作可以在派生 DbContext 上使用约定集合的 AddBefore 和 AddAfter 方法。 下面的代码将添加我们之前创建，以便它将运行之前的内置约定类密钥发现约定中。

``` csharp
    modelBuilder.Conventions.AddBefore<IdKeyDiscoveryConvention>(new DateTime2Convention());
```

这要充分利用的添加需要运行之前或之后的内置约定的约定时，一组内置的约定可在此处找到： [System.Data.Entity.ModelConfiguration.Conventions Namespace](https://msdn.microsoft.com/library/system.data.entity.modelconfiguration.conventions.aspx).

此外可以删除不希望应用于您的模型的约定。 若要删除一个约定，请使用 Remove 方法。 下面是删除 PluralizingTableNameConvention 的示例。

``` csharp
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        modelBuilder.Conventions.Remove<PluralizingTableNameConvention>();
    }
```
