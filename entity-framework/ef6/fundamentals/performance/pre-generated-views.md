---
title: 预生成的映射视图-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 917ba9c8-6ddf-4631-ab8c-c4fb378c2fcd
ms.openlocfilehash: 1fda9fe9638adce9b24a6b81aa081effeb0def81
ms.sourcegitcommit: c568d33214fc25c76e02c8529a29da7a356b37b4
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/30/2018
ms.locfileid: "47459521"
---
# <a name="pre-generated-mapping-views"></a>预生成的映射视图
实体框架可以执行查询或将更改保存到数据源之前，它必须生成一组映射视图来访问数据库。 这些映射视图是一组以抽象方式表示的数据库的实体 SQL 语句，并且每个应用程序域缓存的元数据的一部分。 如果在同一应用程序域中创建相同的上下文的多个实例，这些实例将重用缓存元数据，而无需重新生成它们，从映射视图。 映射视图生成是执行第一个查询的总成本的重要组成部分，实体框架，可预生成映射视图并将其包含在已编译的项目中。 有关详细信息，请参阅[性能注意事项 （实体框架）](~/ef6/fundamentals/performance/perf-whitepaper.md)。

## <a name="generating-mapping-views-with-the-ef-power-tools-community-edition"></a>生成映射视图 with EF Power Tools 社区版本

预生成视图的最简单方法是使用[EF Power Tools Community Edition](https://marketplace.visualstudio.com/items?itemName=ErikEJ.EntityFramework6PowerToolsCommunityEdition)。 已安装 Power 工具将后，到生成的视图，如下所示的菜单选项。

-   有关**Code First**模型右键单击包含您的 DbContext 类的代码文件。
-   有关**EF 设计器**模型右键单击 EDMX 文件。

![生成的视图](~/ef6/media/generateviews.png)

该过程完成后将具有类似于以下生成的类

![生成的视图](~/ef6/media/generatedviews.png)

现在，在运行时 EF 应用程序将使用此类加载所需的视图。 如果模型更改，但不重新生成此类 EF 将引发异常。

## <a name="generating-mapping-views-from-code---ef6-onwards"></a>生成映射代码的 EF6 及更高版本中的视图

生成视图的其他方法是使用 EF 提供的 Api。 使用此方法时您可以自由地序列化视图，但是，但您还需要自行加载视图。

> [!NOTE]
> **EF6 及更高版本仅**-Entity Framework 6 中引入了在本部分中所示的 Api。 如果使用此信息不适用于较早版本。

### <a name="generating-views"></a>生成视图

若要生成的视图的 Api 是 System.Data.Entity.Core.Mapping.StorageMappingItemCollection 类上。 您可以使用 ObjectContext 的 MetadataWorkspace StorageMappingCollection 检索上下文。 如果你使用较新的 DbContext API，则可以使用访问这 IObjectContextAdapter 如下所示，我们在此代码中具有名为 dbContext 派生 DbContext 的实例：

``` csharp
    var objectContext = ((IObjectContextAdapter) dbContext).ObjectContext;
    var  mappingCollection = (StorageMappingItemCollection)objectContext.MetadataWorkspace
                                                                        .GetItemCollection(DataSpace.CSSpace);
```

一旦你有 StorageMappingItemCollection 您可获得 GenerateViews 和 ComputeMappingHashValue 方法的访问权限。

``` csharp
    public Dictionary\<EntitySetBase, DbMappingView> GenerateViews(IList<EdmSchemaError> errors)
    public string ComputeMappingHashValue()
```

第一种方法创建容器映射中每个视图的条目的字典。 第二种方法计算单个容器映射的哈希值，并用于在运行时验证，该模型后未发生更改的预生成视图。 对于复杂的方案涉及多个容器映射提供两种方法的重写。

生成视图时将调用 GenerateViews 方法并写出生成 EntitySetBase 和 DbMappingView。 您还需要用于存储由 ComputeMappingHashValue 方法生成的哈希。

### <a name="loading-views"></a>正在加载视图

若要加载的 GenerateViews 方法生成的视图，可以从 DbMappingViewCache 抽象类继承的类提供 EF。 DbMappingViewCache 指定必须实现的两个方法：

``` csharp
    public abstract string MappingHashValue { get; }
    public abstract DbMappingView GetView(EntitySetBase extent);
```

MappingHashValue 属性必须返回 ComputeMappingHashValue 方法生成的哈希。 EF 时将它寻求视图首先将生成并与此属性返回的哈希值进行比较的模型哈希值。 如果它们不匹配，EF 将引发 EntityCommandCompilationException 异常。

GetView 方法将接受 EntitySetBase 并且您需要返回包含已生成的 EntitySql DbMappingVIew 已给定 EntitySetBase GenerateViews 方法生成字典中与相关联。 如果 EF 要求提供，你没有然后 GetView 的视图应返回 null。

以下是上文所述，在其中我们可以看到存储和检索所需 EntitySql 的一种方法将生成带有 Power Tools DbMappingViewCache 年期间提取。

``` csharp
    public override string MappingHashValue
    {
        get { return "a0b843f03dd29abee99789e190a6fb70ce8e93dc97945d437d9a58fb8e2afd2e"; }
    }

    public override DbMappingView GetView(EntitySetBase extent)
    {
        if (extent == null)
        {
            throw new ArgumentNullException("extent");
        }

        var extentName = extent.EntityContainer.Name + "." + extent.Name;

        if (extentName == "BlogContext.Blogs")
        {
            return GetView2();
        }

        if (extentName == "BlogContext.Posts")
        {
            return GetView3();
        }

        return null;
    }

    private static DbMappingView GetView2()
    {
        return new DbMappingView(@"
            SELECT VALUE -- Constructing Blogs
            [BlogApp.Models.Blog](T1.Blog_BlogId, T1.Blog_Test, T1.Blog_title, T1.Blog_Active, T1.Blog_SomeDecimal)
            FROM (
            SELECT
                T.BlogId AS Blog_BlogId,
                T.Test AS Blog_Test,
                T.title AS Blog_title,
                T.Active AS Blog_Active,
                T.SomeDecimal AS Blog_SomeDecimal,
                True AS _from0
            FROM CodeFirstDatabase.Blog AS T
            ) AS T1");
    }
```

若要让 EF 使用您添加您 DbMappingViewCache 使用 DbMappingViewCacheTypeAttribute，指定为创建的上下文。 下面的代码中我们将与 MyMappingViewCache 类关联 BlogContext。

``` csharp
    [assembly: DbMappingViewCacheType(typeof(BlogContext), typeof(MyMappingViewCache))]
```

对于更复杂的方案，可以通过指定映射视图缓存工厂提供映射视图缓存实例。 这可以通过实现抽象类 System.Data.Entity.Infrastructure.MappingViews.DbMappingViewCacheFactory。 可以检索或设置使用 StorageMappingItemCollection.MappingViewCacheFactoryproperty 映射视图缓存工厂使用的实例。
