---
title: 基于模型的约定的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 0fc4eef8-29b8-4192-9c77-08fd33d3db3a
ms.openlocfilehash: 80b722730b4ca6c9d00a8611b6c9027e8bc9fe61
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283702"
---
# <a name="model-based-conventions"></a>基于模型的约定
> [!NOTE]
> **仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。 如果使用的是早期版本，则部分或全部信息不适用。  

基于模型约定是一种高级的基于约定模型配置方法。 在大多数情况下[自定义 Code First 约定 API，在 DbModelBuilder](~/ef6/modeling/code-first/conventions/custom.md)应使用。 约定 DbModelBuilder API 了解建议使用基于模型约定之前。  

基于模型约定允许创建的约定会影响属性和不能通过标准约定配置的表。 其中的示例为每个层次结构模型表中的鉴别器列和独立关联中的列。  

## <a name="creating-a-convention"></a>创建约定   

创建模型基于约定的第一步选择时需要在管道中应用于模型约定。 有两种类型的模型约定，概念 （C-空间） 和应用商店 （S-空间）。 C 空间约定应用于应用程序生成，而非 S 空间约定应用于表示的数据库的模型的版本和控件等如何自动生成的列名为的模型。  

模型约定是从 IConceptualModelConvention 或 IStoreModelConvention 扩展的类。  两者都接受的泛型类型，可以为这些接口类型 MetadataItem 用于筛选约定适用于的数据类型。  

## <a name="adding-a-convention"></a>添加一种约定   

模型约定添加为正则约定类相同的方式。 在中**OnModelCreating**方法中，将约定添加到模型的约定的列表。  

``` csharp
using System.Data.Entity;
using System.Data.Entity.Core.Metadata.Edm;
using System.Data.Entity.Infrastructure;
using System.Data.Entity.ModelConfiguration.Conventions;

public class BlogContext : DbContext  
{  
    public DbSet<Post> Posts { get; set; }  
    public DbSet<Comment> Comments { get; set; }  

    protected override void OnModelCreating(DbModelBuilder modelBuilder)  
    {  
        modelBuilder.Conventions.Add<MyModelBasedConvention>();  
    }  
}
```  

一种约定还可以添加与另一个惯例使用 Conventions.AddBefore\< \>或 Conventions.AddAfter\< \>方法。 有关实体框架应用的约定的详细信息请参阅注释部分。  

``` csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Conventions.AddAfter<IdKeyDiscoveryConvention>(new MyModelBasedConvention());
}
```  

## <a name="example-discriminator-model-convention"></a>示例： 鉴别器模型约定  

重命名列生成的 EF 是示例就不能与其他约定 Api。  这是一种情况其中使用模型约定是唯一的选项。  

如何使用模型基于约定配置生成的列的示例自定义鉴别器列的命名的方法。  下面是将名为"Discriminator"的模型中的每列重命名为"EntityType"简单的模型基于约定的示例。  这包括开发人员只需命名为"Discriminator"的列。 由于"Discriminator"列是生成的列，这需要在 S 空间中运行。  

``` csharp
using System.Data.Entity;
using System.Data.Entity.Core.Metadata.Edm;
using System.Data.Entity.Infrastructure;
using System.Data.Entity.ModelConfiguration.Conventions;

class DiscriminatorRenamingConvention : IStoreModelConvention<EdmProperty>  
{  
    public void Apply(EdmProperty property, DbModel model)  
    {            
        if (property.Name == "Discriminator")  
        {  
            property.Name = "EntityType";  
        }  
    }  
}
```  

## <a name="example-general-ia-renaming-convention"></a>示例： 常规 IA 重命名约定   

在操作中的模型基于约定的另一个更复杂的示例是配置为独立关联 (IAs) 的方式。  这是模型约定都适用，因为 IAs 都由 EF 且 DbModelBuilder API 可以访问的模型中不存在。  

当 EF 生成 IA 时，它会创建一个名为 EntityType_KeyName 列。 例如，对于键列与名为 Customer 的关联名为客户 id 则会生成一个名为 Customer_CustomerId 列。 下列约定带\_字符超出为 IA.生成的列名称  

``` csharp
using System.Data.Entity;
using System.Data.Entity.Core.Metadata.Edm;
using System.Data.Entity.Infrastructure;
using System.Data.Entity.ModelConfiguration.Conventions;

// Provides a convention for fixing the independent association (IA) foreign key column names.  
public class ForeignKeyNamingConvention : IStoreModelConvention<AssociationType>
{

    public void Apply(AssociationType association, DbModel model)
    {
        // Identify ForeignKey properties (including IAs)  
        if (association.IsForeignKey)
        {
            // rename FK columns  
            var constraint = association.Constraint;
            if (DoPropertiesHaveDefaultNames(constraint.FromProperties, constraint.ToRole.Name, constraint.ToProperties))
            {
                NormalizeForeignKeyProperties(constraint.FromProperties);
            }
            if (DoPropertiesHaveDefaultNames(constraint.ToProperties, constraint.FromRole.Name, constraint.FromProperties))
            {
                NormalizeForeignKeyProperties(constraint.ToProperties);
            }
        }
    }

    private bool DoPropertiesHaveDefaultNames(ReadOnlyMetadataCollection<EdmProperty> properties, string roleName, ReadOnlyMetadataCollection<EdmProperty> otherEndProperties)
    {
        if (properties.Count != otherEndProperties.Count)
        {
            return false;
        }

        for (int i = 0; i < properties.Count; ++i)
        {
            if (!properties[i].Name.EndsWith("_" + otherEndProperties[i].Name))
            {
                return false;
            }
        }
        return true;
    }

    private void NormalizeForeignKeyProperties(ReadOnlyMetadataCollection<EdmProperty> properties)
    {
        for (int i = 0; i \< properties.Count; ++i)
        {
            int underscoreIndex = properties[i].Name.IndexOf('_');
            if (underscoreIndex > 0)
            {
                properties[i].Name = properties[i].Name.Remove(underscoreIndex, 1);
            }                 
        }
    }
}
```  

## <a name="extending-existing-conventions"></a>扩展现有约定   

如果您需要编写类似于实体框架已适用于您始终可以扩展这一惯例，以免从头开始重新编写的模型的约定的约定。  出现这种是替换现有的 Id 匹配与自定义的约定。   重写的键的约定的一个好处是，仅当已检测到或显式配置没有键，将会调用重写的方法。 一组约定由实体框架提供了： [ http://msdn.microsoft.com/library/system.data.entity.modelconfiguration.conventions.aspx ](https://msdn.microsoft.com/library/system.data.entity.modelconfiguration.conventions.aspx)。  

``` csharp
using System.Data.Entity;
using System.Data.Entity.Core.Metadata.Edm;
using System.Data.Entity.Infrastructure;
using System.Data.Entity.ModelConfiguration.Conventions;
using System.Linq;  

// Convention to detect primary key properties.
// Recognized naming patterns in order of precedence are:
// 1. 'Key'
// 2. [type name]Key
// Primary key detection is case insensitive.
public class CustomKeyDiscoveryConvention : KeyDiscoveryConvention
{
    private const string Id = "Key";

    protected override IEnumerable<EdmProperty> MatchKeyProperty(
        EntityType entityType, IEnumerable<EdmProperty> primitiveProperties)
    {
        Debug.Assert(entityType != null);
        Debug.Assert(primitiveProperties != null);

        var matches = primitiveProperties
            .Where(p => Id.Equals(p.Name, StringComparison.OrdinalIgnoreCase));

        if (!matches.Any())
       {
            matches = primitiveProperties
                .Where(p => (entityType.Name + Id).Equals(p.Name, StringComparison.OrdinalIgnoreCase));
        }

        // If the number of matches is more than one, then multiple properties matched differing only by
        // case--for example, "Key" and "key".  
        if (matches.Count() > 1)
        {
            throw new InvalidOperationException("Multiple properties match the key convention");
        }

        return matches;
    }
}
```  

然后，我们需要添加的现有密钥约定之前我们新的约定。 我们添加 CustomKeyDiscoveryConvention 后，我们可以删除 IdKeyDiscoveryConvention。  如果我们没有删除现有 IdKeyDiscoveryConvention 此约定将仍优先于 Id 发现约定由于它运行第一次，但在其中找到没有"密钥"属性的情况下，将运行的"id"约定。  我们看到此行为，因为每个约定会看到该模型，为已更新的以前的约定 （而不是独立操作和所有正在组合在一起），以便如果例如上, 一个约定更新要匹配的内容的列名称感兴趣，为你自定义的约定 （如果在之前的名称是不感兴趣的），则它将应用于该列。  

``` csharp
public class BlogContext : DbContext
{
    public DbSet<Post> Posts { get; set; }
    public DbSet<Comment> Comments { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        modelBuilder.Conventions.AddBefore<IdKeyDiscoveryConvention>(new CustomKeyDiscoveryConvention());
        modelBuilder.Conventions.Remove<IdKeyDiscoveryConvention>();
    }
}
```  

## <a name="notes"></a>说明  

此处的 MSDN 文档中提供了当前应用由实体框架的约定的列表： [ http://msdn.microsoft.com/library/system.data.entity.modelconfiguration.conventions.aspx ](https://msdn.microsoft.com/library/system.data.entity.modelconfiguration.conventions.aspx)。  直接从我们的源代码拉取此列表。  Entity Framework 6 的源代码位于[GitHub](https://github.com/aspnet/entityframework6/)和许多实体框架使用的约定是很好的自定义模型起点基于约定。  
