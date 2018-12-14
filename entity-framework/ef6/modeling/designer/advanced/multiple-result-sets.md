---
title: 存储过程具有多个结果集的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 1b3797f9-cd3d-4752-a55e-47b84b399dc1
ms.openlocfilehash: 098ed88ba52e211965baf3660f0e51bd74c71efd
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489305"
---
# <a name="stored-procedures-with-multiple-result-sets"></a>具有多个结果集的存储的过程
有时，当使用存储时设置过程将需要返回多个结果。 此方案通常用于减少数据库的所需编写单个屏幕之间的往返。 在 EF5 之前, Entity Framework 将允许要调用的存储的过程，但将仅返回设置为调用代码的第一个结果。

本文将说明可用于从 Entity Framework 中的存储过程访问多个结果集的两种方法。 一个使用只是代码，并首先适用于这两个代码和 EF 设计器，仅可与 EF 设计器使用。 工具和 API 的支持在将来多应提高 Entity Framework 版本。

## <a name="model"></a>模型

这篇文章中的示例使用一个基本博客和所属单一博客文章模型一个博客，其中有许多文章和 post。 返回所有博客和文章，如下所示在数据库中，我们将使用存储的过程：

``` SQL
    CREATE PROCEDURE [dbo].[GetAllBlogsAndPosts]
    AS
        SELECT * FROM dbo.Blogs
        SELECT * FROM dbo.Posts
```

## <a name="accessing-multiple-result-sets-with-code"></a>使用代码访问多个结果集

我们可以执行使用代码以发出原始的 SQL 命令以执行存储的过程。 此方法的优点是，它适用于这两个代码首先和 EF 设计器。

若要获取多个结果设置的工作，我们需要放置到 ObjectContext API 由使用 IObjectContextAdapter 接口。

一旦我们有 ObjectContext，则我们可以使用转换方法来将我们存储过程的结果转换为可以跟踪并像平常一样在 EF 中使用的实体。 下面的代码示例演示此操作在操作中。

``` csharp
    using (var db = new BloggingContext())
    {
        // If using Code First we need to make sure the model is built before we open the connection
        // This isn't required for models created with the EF Designer
        db.Database.Initialize(force: false);

        // Create a SQL command to execute the sproc
        var cmd = db.Database.Connection.CreateCommand();
        cmd.CommandText = "[dbo].[GetAllBlogsAndPosts]";

        try
        {

            db.Database.Connection.Open();
            // Run the sproc
            var reader = cmd.ExecuteReader();

            // Read Blogs from the first result set
            var blogs = ((IObjectContextAdapter)db)
                .ObjectContext
                .Translate<Blog>(reader, "Blogs", MergeOption.AppendOnly);   


            foreach (var item in blogs)
            {
                Console.WriteLine(item.Name);
            }        

            // Move to second result set and read Posts
            reader.NextResult();
            var posts = ((IObjectContextAdapter)db)
                .ObjectContext
                .Translate<Post>(reader, "Posts", MergeOption.AppendOnly);


            foreach (var item in posts)
            {
                Console.WriteLine(item.Title);
            }
        }
        finally
        {
            db.Database.Connection.Close();
        }
    }
```

Translate 方法接受我们执行该过程，一个实体集的名称和 MergeOption 时我们收到的读取器。 EntitySet 名称将是派生上下文的 DbSet 属性相同。 MergeOption 枚举控制如果同一个实体已存在于内存中处理结果的方式。

此处我们循环访问集合的博客然后我们调用 NextResult，这一点很重要给定上面的代码中因为移到下一个结果集之前必须使用第一个结果集。

后两个转换，然后在博客和文章实体被 EF 跟踪方式与任何其他实体相同，因此可以修改或删除并另存为正常调用方法。

>[!NOTE]
> 创建使用转换方法的实体时，EF 不会考虑任何映射。 它只需将匹配的结果集的属性名称与您的类中的列名称。

>[!NOTE]
> 如果已启用，延迟加载访问文章属性上的博客实体之一中的 EF 将连接到数据库延迟加载所有发布的文章，即使我们已加载所有这些也是如此。 这是因为 EF 无法知道已加载所有帖子或如果有多个数据库中。 如果你想要避免这种情况将需要禁用延迟加载。

## <a name="multiple-result-sets-with-configured-in-edmx"></a>使用 EDMX 时配置中的多个结果集

>[!NOTE]
> 必须针对.NET Framework 4.5，以便能够在 EDMX 中配置多个结果集。 如果你正面向.NET 4.0，可以使用在上一部分中所示的基于代码的方法。

如果使用 EF 设计器，您还可以修改您的模型，以便它了解将返回不同的结果集。 一件事之前，此工具不是多个结果指针是要了解设置注意，因此将需要手动编辑 edmx 文件。 此操作才有效，但它还会在 VS 中中断模型的验证，请编辑 edmx 文件。 因此如果您验证您的模型，始终会收到错误。

-   若要执行此操作需要将存储的过程添加到您的模型，就像为单个结果集查询。
-   当您获得此则需要您在模型上右键单击并选择**打开方式...** 然后**Xml**

    ![打开为](~/ef6/media/openas.png)

后的模型以 XML 形式打开，则需要执行以下步骤：

-   在您的模型中找到的复杂类型和函数导入：

``` xml
    <!-- CSDL content -->
    <edmx:ConceptualModels>

    ...

      <FunctionImport Name="GetAllBlogsAndPosts" ReturnType="Collection(BlogModel.GetAllBlogsAndPosts_Result)" />

    ...

      <ComplexType Name="GetAllBlogsAndPosts_Result">
        <Property Type="Int32" Name="BlogId" Nullable="false" />
        <Property Type="String" Name="Name" Nullable="false" MaxLength="255" />
        <Property Type="String" Name="Description" Nullable="true" />
      </ComplexType>

    ...

    </edmx:ConceptualModels>
```

 

-   删除复杂类型
-   更新函数导入，以便它映射到实体，在本例中它看起来如下所示：

``` xml
    <FunctionImport Name="GetAllBlogsAndPosts">
      <ReturnType EntitySet="Blogs" Type="Collection(BlogModel.Blog)" />
      <ReturnType EntitySet="Posts" Type="Collection(BlogModel.Post)" />
    </FunctionImport>
```

这将告知该模型的存储的过程将返回两个集合，一个博客条目和文章条目之一。

-   查找函数映射元素：

``` xml
    <!-- C-S mapping content -->
    <edmx:Mappings>

    ...

      <FunctionImportMapping FunctionImportName="GetAllBlogsAndPosts" FunctionName="BlogModel.Store.GetAllBlogsAndPosts">
        <ResultMapping>
          <ComplexTypeMapping TypeName="BlogModel.GetAllBlogsAndPosts_Result">
            <ScalarProperty Name="BlogId" ColumnName="BlogId" />
            <ScalarProperty Name="Name" ColumnName="Name" />
            <ScalarProperty Name="Description" ColumnName="Description" />
          </ComplexTypeMapping>
        </ResultMapping>
      </FunctionImportMapping>

    ...

    </edmx:Mappings>
```

-   将结果映射为与另一个用于每个实体返回，如下所示：

``` xml
    <ResultMapping>
      <EntityTypeMapping TypeName ="BlogModel.Blog">
        <ScalarProperty Name="BlogId" ColumnName="BlogId" />
        <ScalarProperty Name="Name" ColumnName="Name" />
        <ScalarProperty Name="Description" ColumnName="Description" />
      </EntityTypeMapping>
    </ResultMapping>
    <ResultMapping>
      <EntityTypeMapping TypeName="BlogModel.Post">
        <ScalarProperty Name="BlogId" ColumnName="BlogId" />
        <ScalarProperty Name="PostId" ColumnName="PostId"/>
        <ScalarProperty Name="Title" ColumnName="Title" />
        <ScalarProperty Name="Text" ColumnName="Text" />
      </EntityTypeMapping>
    </ResultMapping>
```

还有可能要将结果集映射到复杂类型，例如默认情况下创建一个。 若要执行此操作可以创建新的复杂类型，而不是删除它们，并使用复杂类型无处不在您获得上面的示例中使用的实体名称。

一旦这些映射已更改，则可以将模型保存并执行以下代码以使用存储的过程：

``` csharp
    using (var db = new BlogEntities())
    {
        var results = db.GetAllBlogsAndPosts();

        foreach (var result in results)
        {
            Console.WriteLine("Blog: " + result.Name);
        }

        var posts = results.GetNextResult<Post>();

        foreach (var result in posts)
        {
            Console.WriteLine("Post: " + result.Title);
        }

        Console.ReadLine();
    }
```

>[!NOTE]
> 如果你手动编辑 edmx 文件为您的模型将被覆盖，如果曾经重新生成数据库模型。

## <a name="summary"></a>总结

此处我们已经演示了使用实体框架访问多个结果的两个不同的方法集。 这两个文件都同样有效，具体取决于您的具体情况和首选项和您应选择一种似乎是最适合您的环境。 计划对多个结果集将会支持在将来改进 Entity Framework 版本和，执行本文档中的步骤将不再有必要。
