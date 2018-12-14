---
title: 连接字符串和模型的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 294bb138-978f-4fe2-8491-fdf3cd3c60c4
ms.openlocfilehash: 2c9f084107e4de7f5439bf0082b46a3b538496e0
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490736"
---
# <a name="connection-strings-and-models"></a>连接字符串和模型
本主题介绍实体框架如何发现哪个数据库连接，若要使用，以及如何更改它。 同时涉及本主题中使用 Code First 和 EF 设计器创建的模型。  

通常的实体框架应用程序使用从 DbContext 派生的类。 此派生的类将调用的构造函数之一来控制 DbContext 基类上：  

- 上下文将如何连接到数据库，也就是说，连接字符串的方式找到使用  
- 是否将使用上下文计算使用 Code First 的模型或加载使用 EF 设计器创建模型  
- 其他高级的选项  

以下片段显示可以使用几种方式的 DbContext 构造函数。  

## <a name="use-code-first-with-connection-by-convention"></a>按照约定使用 Code First 与连接  

如果你尚未在应用程序中做任何其他配置，然后对 DbContext 调用无参数构造函数将导致 DbContext 在 Code First 模式下运行与创建的约定的数据库连接。 例如：  

``` csharp  
namespace Demo.EF
{
    public class BloggingContext : DbContext
    {
        public BloggingContext()
        // C# will call base class parameterless constructor by default
        {
        }
    }
}
```  

在此示例中 DbContext 使用你派生的上下文 class—Demo.EF.BloggingContext—as 数据库名称的命名空间限定的名称，并创建使用 SQL Express 或 LocalDB 此数据库的连接字符串。 如果安装了，则将使用 SQL Express。  

Visual Studio 2010 包括 SQL Express 默认情况下和 Visual Studio 2012 和更高版本包含 LocalDB。 在安装期间，EntityFramework NuGet 包检查的数据库服务器可用。 NuGet 包然后将更新的配置文件，通过设置时创建连接的约定，代码优先使用的默认数据库服务器。 如果正在运行 SQL Express，将使用它。 如果不可用 SQL Express 然后 LocalDB 将注册为默认值改为。 如果它已包含的默认连接工厂的设置，将配置文件不进行任何更改。  

## <a name="use-code-first-with-connection-by-convention-and-specified-database-name"></a>按约定和指定的数据库名称使用 Code First 与连接  

如果你尚未在应用程序中做任何其他配置，使用你想要使用的数据库名称，然后调用 DbContext 上字符串构造函数将导致 DbContext 在 Code First 模式下运行与创建的约定的数据库的数据库连接该名称。 例如：  

``` csharp  
public class BloggingContext : DbContext
{
    public BloggingContext()
        : base("BloggingDatabase")
    {
    }
}
```  

在此示例中 DbContext"BloggingDatabase"用作数据库名称，并创建此数据库使用 SQL Express （随 Visual Studio 2010 一起安装） 或 LocalDB （随 Visual Studio 2012 一起安装） 的连接字符串。 如果安装了，则将使用 SQL Express。  

## <a name="use-code-first-with-connection-string-in-appconfigwebconfig-file"></a>使用 Code First 与 app.config/web.config 文件中的连接字符串  

您可以选择将连接字符串放在你的 app.config 或 web.config 文件中。 例如：  

``` xml  
<configuration>
  <connectionStrings>
    <add name="BloggingCompactDatabase"
         providerName="System.Data.SqlServerCe.4.0"
         connectionString="Data Source=Blogging.sdf"/>
  </connectionStrings>
</configuration>
```  

这是告知 DbContext，若要使用 SQL Express 或 LocalDB 以外的数据库服务器的简单办法，上面的示例中指定的 SQL Server Compact Edition 数据库。  

如果连接字符串的名称匹配的上下文 （有或没有命名空间限定） 名称然后它将找到通过 DbContext 时，使用无参数构造函数。 如果连接字符串名称不同于您的上下文的名称，然后可以告知 DbContext 通过将连接字符串名称传递给 DbContext 构造函数在代码优先模式下使用此连接。 例如：  

``` csharp  
public class BloggingContext : DbContext
{
    public BloggingContext()
        : base("BloggingCompactDatabase")
    {
    }
}
```  

或者，可以使用窗体"名称 =\<连接字符串名称\>"字符串传递给 DbContext 构造函数。 例如：  

``` csharp  
public class BloggingContext : DbContext
{
    public BloggingContext()
        : base("name=BloggingCompactDatabase")
    {
    }
}
```  

这种形式使显式预期要在配置文件中找到的连接字符串。 如果找不到具有给定名称的连接字符串，将引发异常。  

## <a name="databasemodel-first-with-connection-string-in-appconfigwebconfig-file"></a>Database first 或 Model First app.config/web.config 文件中的连接字符串  

使用 EF 设计器创建的模型的不同于 Code First，在于您的模型已存在且应用程序运行时不生成代码。 模型通常作为你的项目中的 EDMX 文件存在。  

在设计器会将 EF 连接字符串添加到 app.config 或 web.config 文件。 此连接字符串很特殊，因为它包含有关如何在 EDMX 文件中查找信息的信息。 例如：  

``` xml  
<configuration>  
  <connectionStrings>  
    <add name="Northwind_Entities"  
         connectionString="metadata=res://*/Northwind.csdl|  
                                    res://*/Northwind.ssdl|  
                                    res://*/Northwind.msl;  
                           provider=System.Data.SqlClient;  
                           provider connection string=  
                               &quot;Data Source=.\sqlexpress;  
                                     Initial Catalog=Northwind;  
                                     Integrated Security=True;  
                                     MultipleActiveResultSets=True&quot;"  
         providerName="System.Data.EntityClient"/>  
  </connectionStrings>  
</configuration>
```  

在 EF 设计器还会生成告知 DbContext 通过将连接字符串名称传递给 DbContext 构造函数使用此连接的代码。 例如：  

``` csharp  
public class NorthwindContext : DbContext
{
    public NorthwindContext()
        : base("name=Northwind_Entities")
    {
    }
}
```  

DbContext 知道要加载现有模型 （而不是使用 Code First 计算从代码） 的连接字符串是包含要使用的模型的详细信息的 EF 连接字符串。  

## <a name="other-dbcontext-constructor-options"></a>DbContext 构造函数的其他选项  

DbContext 类包含其他构造函数和启用一些更高级的方案的使用模式。 其中一些是：  

- DbModelBuilder 类可用于生成 Code First 模型，而实例化 DbContext 实例。 此操作的结果是 DbModel 对象。 然后可以将此 DbModel 对象传递给 DbContext 构造函数之一时已准备好创建 DbContext 实例。  
- 而不是只是数据库或连接字符串名称，可以将完整的连接字符串传递到 DbContext。 默认情况下使用 System.Data.SqlClient 的提供程序; 使用此连接字符串这可以通过设置 IConnectionFactory 到上下文上的不同实现更改。Database.DefaultConnectionFactory。  
- 通过将它传递到 DbContext 构造函数，可以使用现有的 DbConnection 对象。 如果连接对象实例的 EntityConnection，则在连接中指定的模型，会使用，而不是计算模型使用代码优先。 如果对象是其他类型的实例 — 例如，SqlConnection，则上下文将使用它的 Code First 模式。  
- 可以将现有的 ObjectContext 传递给 DbContext 构造函数创建包装现有上下文的 DbContext。 这可以用于现有应用程序使用 ObjectContext，但其想要利用应用程序的某些部分中的 DbContext。  
