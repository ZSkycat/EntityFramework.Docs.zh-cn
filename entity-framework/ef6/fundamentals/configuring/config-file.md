---
title: 配置文件设置的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 000044c6-1d32-4cf7-ae1f-ea21d86ebf8f
ms.openlocfilehash: faba4e406b9f26f5bed6149f75c59da362d84692
ms.sourcegitcommit: 15022dd06d919c29b1189c82611ea32f9fdc6617
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2018
ms.locfileid: "47415778"
---
# <a name="configuration-file-settings"></a>配置文件设置
实体框架允许要从配置文件指定设置的数量。 一般情况下 EF 遵循惯例优先于配置原则： 此帖子中讨论的所有设置都具有默认行为，只需担心如何更改设置时默认值就不再满足您的要求。  

## <a name="a-code-based-alternative"></a>基于代码的替代方法  

所有这些设置还可使用代码。 从我们引入了的 EF6[基于代码的配置](code-based.md)，其中提供了一种重要方法的应用代码中的配置。 在 EF6 之前仍可以从代码应用配置，但需要使用各种 Api 来配置不同的区域。 配置文件选项，而无需更新代码，在部署过程中轻松地更改这些设置。

## <a name="the-entity-framework-configuration-section"></a>实体框架配置部分  

启动 EF4.1 后，可以设置为上下文使用的数据库初始值设定项**appSettings**配置文件的部分。 在 EF 4.3 我们引入了自定义**entityFramework**部分，以处理新的设置。 实体框架仍将识别数据库初始值设定项使用旧格式设置，但我们建议移到新的格式，在可能的情况。

**EntityFramework**部分为自动添加到你的项目的配置文件时安装 EntityFramework NuGet 包。  

``` xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <configSections>
    <!-- For more information on Entity Framework configuration, visit http://go.microsoft.com/fwlink/?LinkID=237468 -->
    <section name="entityFramework"
       type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=4.3.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" />
  </configSections>
</configuration>
```  

## <a name="connection-strings"></a>连接字符串  

[此页](~/ef6/fundamentals/configuring/connection-strings.md)实体框架如何确定要使用的数据库提供更多详细信息包括在配置文件中的连接字符串。  

标准中的连接字符串转**connectionStrings**元素，并且不需要**entityFramework**部分。  

代码首先基于模型使用普通 ADO.NET 连接字符串。 例如：  

``` xml
<connectionStrings>
  <add name="BlogContext"  
        providerName="System.Data.SqlClient"  
        connectionString="Server=.\SQLEXPRESS;Database=Blogging;Integrated Security=True;"/>
</connectionStrings>
```  

EF 设计器基于模型使用特殊 EF 连接字符串。 例如：  

``` xml  
<connectionStrings>
  <add name="BlogContext"  
    connectionString=
      "metadata=
        res://*/BloggingModel.csdl|
        res://*/BloggingModel.ssdl|
        res://*/BloggingModel.msl;
      provider=System.Data.SqlClient
      provider connection string=
        &quot;data source=(localdb)\mssqllocaldb;
        initial catalog=Blogging;
        integrated security=True;
        multipleactiveresultsets=True;&quot;"
     providerName="System.Data.EntityClient" />
</connectionStrings>
```

## <a name="code-based-configuration-type-ef6-onwards"></a>基于代码的配置类型 (EF6 及更高版本)  

从 EF6 开始，可以指定要用于 ef DbConfiguration[基于代码的配置](code-based.md)在应用程序中。 在大多数情况下不需要指定此设置，如 EF 将自动发现你 DbConfiguration。 有关你可能需要在配置文件中指定 DbConfiguration 的详细信息，请参阅**移动 DbConfiguration**一部分[基于代码的配置](code-based.md)。  

若要设置 DbConfiguration 类型，您指定的程序集限定的类型名称在**codeConfigurationType**元素。  

> [!NOTE]
> 程序集限定的名是命名空间限定的名称后, 跟一个逗号，然后该类型所在的程序集。 您可以选择指定程序集版本、 区域性和公钥标记。  

``` xml
<entityFramework codeConfigurationType="MyNamespace.MyConfiguration, MyAssembly">
</entityFramework>
```  

## <a name="ef-database-providers-ef6-onwards"></a>EF 数据库提供程序 (EF6 及更高版本)  

在 EF6 之前，数据库提供程序的特定于实体框架的部分必须是作为核心 ADO.NET 提供程序的一部分。 从 EF6 开始，EF 特定部分现在管理，并单独注册。  

通常情况下不需要将自己注册提供程序。 通常这会在提供程序时安装它。  

通过包括注册提供程序**提供程序**元素下的**提供程序**的子部分**entityFramework**部分。 有两个所需的属性提供程序条目：  

- **invariantName**标识核心 ADO.NET 提供程序，此 EF 提供程序目标  
- **类型**是 EF 提供程序实现的程序集限定的类型名称  

> [!NOTE]
> 程序集限定的名是命名空间限定的名称后, 跟一个逗号，然后该类型所在的程序集。 您可以选择指定程序集版本、 区域性和公钥标记。  

例如，下面是创建安装实体框架时注册的默认 SQL Server 提供程序的条目。  

``` xml  
<providers>
  <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
</providers>
```  

## <a name="interceptors-ef61-onwards"></a>侦听器 (EF6.1 及更高版本)  

启动与 ef6.1 结合使用可以在配置文件中注册侦听器。 拦截器，可以运行其他逻辑，当 EF 执行某些操作，如执行数据库查询打开连接，等等。  

通过包括注册侦听器**侦听器**元素下的**侦听器**的子部分**entityFramework**部分。 例如，下面的配置注册内置**DatabaseLogger**将记录到控制台的所有数据库操作的侦听器。  

``` xml  
<interceptors>
  <interceptor type="System.Data.Entity.Infrastructure.Interception.DatabaseLogger, EntityFramework"/>
</interceptors>
```  

### <a name="logging-database-operations-to-a-file-ef61-onwards"></a>日志记录到文件 (及更高版本 EF6.1) 的数据库操作  

注册侦听器通过配置文件时，尤其是你想要将日志记录添加到现有应用程序来帮助调试问题。 **DatabaseLogger**支持通过提供作为构造函数参数的文件名称的文件的日志记录。  

``` xml  
<interceptors>
  <interceptor type="System.Data.Entity.Infrastructure.Interception.DatabaseLogger, EntityFramework">
    <parameters>
      <parameter value="C:\Temp\LogOutput.txt"/>
    </parameters>
  </interceptor>
</interceptors>
```  

默认情况下，这将导致日志文件以启动该应用程序每次用新文件覆盖。 若要改为追加到日志文件已存在使用类似于：  

``` xml  
<interceptors>
  <interceptor type="System.Data.Entity.Infrastructure.Interception.DatabaseLogger, EntityFramework">
    <parameters>
      <parameter value="C:\Temp\LogOutput.txt"/>
      <parameter value="true" type="System.Boolean"/>
    </parameters>
  </interceptor>
</interceptors>
```  

有关其他信息**DatabaseLogger**和注册侦听器，请参阅博客文章[EF 6.1： 打开日志记录，无需重新编译](https://blog.oneunicorn.com/2014/02/09/ef-6-1-turning-on-logging-without-recompiling/)。  

## <a name="code-first-default-connection-factory"></a>代码第一个默认连接工厂  

配置部分，可指定代码优先应用来定位要用于上下文的数据库的默认连接工厂。 没有连接字符串已添加到上下文的配置文件时，才使用默认连接工厂。  

安装 EF NuGet 包时默认连接工厂注册了指向 SQL Express 或 LocalDB，具体取决于哪一个已安装。  

若要设置一个连接工厂，你指定的程序集限定的类型名称在**defaultConnectionFactory**元素。  

> [!NOTE]
> 程序集限定的名是命名空间限定的名称后, 跟一个逗号，然后该类型所在的程序集。 您可以选择指定程序集版本、 区域性和公钥标记。  

下面是设置自己的默认连接工厂的示例：  

``` xml  
<entityFramework>
  <defaultConnectionFactory type="MyNamespace.MyCustomFactory, MyAssembly"/>
</entityFramework>
```  

上面的示例要求具有无参数构造函数的自定义工厂。 如果需要可以指定使用的构造函数参数**参数**元素。  

例如，SqlCeConnectionFactory，包含在实体框架中，要求您提供给构造函数提供程序固定名称。 提供程序固定名称标识的 SQL Compact 你想要使用的版本。 下面的配置将导致要使用默认的 SQL Compact 版本 4.0 的上下文。  

``` xml  
<entityFramework>
  <defaultConnectionFactory type="System.Data.Entity.Infrastructure.SqlCeConnectionFactory, EntityFramework">
    <parameters>
      <parameter value="System.Data.SqlServerCe.4.0" />
    </parameters>
  </defaultConnectionFactory>
</entityFramework>
```  

如果未设置默认连接工厂，Code First 将使用指向 SqlConnectionFactory `.\SQLEXPRESS`。 SqlConnectionFactory 还具有构造函数，您可以重写连接字符串的部分。 如果你想要使用的 SQL Server 实例而不`.\SQLEXPRESS`可以使用此构造函数来设置服务器。  

下面的配置将导致代码优先，使用**我**没有明确的连接字符串设置的上下文。  

``` xml  
<entityFramework>
  <defaultConnectionFactory type="System.Data.Entity.Infrastructure.SqlConnectionFactory, EntityFramework">
    <parameters>
      <parameter value="Data Source=MyDatabaseServer; Integrated Security=True; MultipleActiveResultSets=True" />
    </parameters>
  </defaultConnectionFactory>
</entityFramework>
```  

默认情况下，假定构造函数自变量是字符串类型。 类型属性可用于更改此设置。  

``` xml
<parameter value="2" type="System.Int32" />
```  

## <a name="database-initializers"></a>数据库初始值设定项  

在每个上下文的基础上配置数据库初始值设定项。 可以在该配置文件中设置它们**上下文**元素。 此元素使用的程序集限定的名称来标识正在配置的上下文。  

默认情况下，Code First 上下文配置为使用 CreateDatabaseIfNotExists 初始值设定项。 没有**disableDatabaseInitialization**特性，可以在**上下文**可用于禁用数据库初始化的元素。  

例如，下面的配置禁用数据库初始化 Blogging.BlogContext 上下文 MyAssembly.dll 中定义。  

``` xml  
<contexts>
  <context type=" Blogging.BlogContext, MyAssembly" disableDatabaseInitialization="true" />
</contexts>
```  

可以使用**databaseInitializer**要设置自定义初始值设定项元素。  

``` xml
<contexts>
  <context type=" Blogging.BlogContext, MyAssembly">
    <databaseInitializer type="Blogging.MyCustomBlogInitializer, MyAssembly" />
  </context>
</contexts>
```  

构造函数参数作为默认连接工厂使用相同的语法。  

``` xml  
<contexts>
  <context type=" Blogging.BlogContext, MyAssembly">
    <databaseInitializer type="Blogging.MyCustomBlogInitializer, MyAssembly">
      <parameters>
        <parameter value="MyConstructorParameter" />
      </parameters>
    </databaseInitializer>
  </context>
</contexts>
```  

您可以配置一个包含实体框架中的通用数据库初始值设定项。 **类型**属性为泛型类型使用.NET Framework 格式。  

例如，如果使用 Code First 迁移，你可以配置要使用自动迁移的数据库`MigrateDatabaseToLatestVersion<TContext, TMigrationsConfiguration>`初始值设定项。  

``` xml
<contexts>
  <context type="Blogging.BlogContext, MyAssembly">
    <databaseInitializer type="System.Data.Entity.MigrateDatabaseToLatestVersion`2[[Blogging.BlogContext, MyAssembly], [Blogging.Migrations.Configuration, MyAssembly]], EntityFramework" />
  </context>
</contexts>
```
