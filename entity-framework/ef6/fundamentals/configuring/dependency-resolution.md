---
title: 依赖项解析-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 32d19ac6-9186-4ae1-8655-64ee49da55d0
ms.openlocfilehash: 6082124481f5795bbcb62fff2bb6a58ecdcb48e4
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490956"
---
# <a name="dependency-resolution"></a>依赖项解析
> [!NOTE]
> **仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。 如果使用的是早期版本，则部分或全部信息不适用。  

从 EF6 开始，实体框架包含用于获取它需要的服务的实现的通用机制。 也就是说，EF 使用一些接口或基类的实例时它将要求提供的接口或基类的类的具体实现使用。 这可以通过使用 IDbDependencyResolver 接口的实现：  

``` csharp
public interface IDbDependencyResolver
{
    object GetService(Type type, object key);
}
```  

GetService 方法通常由 EF 调用和由 IDbDependencyResolver EF 或应用程序提供的实现。 类型参数调用时，是所请求的服务的接口或基类的类类型和键的对象为 null 或对象，用于提供有关所请求服务的上下文信息。  

除非另有说明返回任何对象必须是线程安全，因为它可以用作单一实例。 在许多情况下，在这种情况下是一个工厂返回的对象工厂本身必须是线程安全，但从工厂返回的对象不需要是线程安全，因为从每次使用工厂请求的新实例。

这篇文章不包含有关如何实现 IDbDependencyResolver，完整的详细信息，但充当其 EF 调用 GetService 和键的对象的语义的每个服务类型 （即，接口和基类类类型） 的引用调用。

## <a name="systemdataentityidatabaseinitializertcontext"></a>System.Data.Entity.IDatabaseInitializer < TContext\>  

**版本引入了**: EF6.0.0  

**返回对象**： 给定的上下文类型的数据库初始值设定项  

**密钥**： 未使用; 将为 null  

## <a name="funcsystemdataentitymigrationssqlmigrationsqlgenerator"></a>Func < System.Data.Entity.Migrations.Sql.MigrationSqlGenerator\>  

**版本引入了**: EF6.0.0

**返回对象**： 用于创建可用于迁移和其他操作，会导致创建，例如数据库初始值设定项使用的数据库创建数据库的 SQL 生成器工厂。  

**密钥**： 包含指定的类型将为其生成 SQL 数据库的 ADO.NET 提供程序固定名称的字符串。 例如，SQL Server SQL 生成器返回密钥"System.Data.SqlClient"。  

>[!NOTE]
> 有关在 EF6 中与提供程序相关的服务的详细信息，请参阅[EF6 提供程序模型](~/ef6/fundamentals/providers/provider-model.md)部分。  

## <a name="systemdataentitycorecommondbproviderservices"></a>System.Data.Entity.Core.Common.DbProviderServices  

**版本引入了**: EF6.0.0  

**返回对象**： 要用于给定提供程序固定名称的 EF 提供程序  

**密钥**： 包含指定的类型为其提供程序所需的数据库的 ADO.NET 提供程序固定名称的字符串。 例如，SQL Server 提供程序返回密钥"System.Data.SqlClient"。  

>[!NOTE]
> 有关在 EF6 中与提供程序相关的服务的详细信息，请参阅[EF6 提供程序模型](~/ef6/fundamentals/providers/provider-model.md)部分。  

## <a name="systemdataentityinfrastructureidbconnectionfactory"></a>System.Data.Entity.Infrastructure.IDbConnectionFactory  

**版本引入了**: EF6.0.0  

**返回对象**： 将按照惯例 EF 创建数据库连接时使用的连接工厂。 也就是说时没有连接或连接字符串被授予对 EF，并且可以在 app.config 或 web.config 中找到任何连接字符串，, 然后此服务用于创建连接按照约定。 默认情况下，更改连接工厂可以允许 EF 使用不同类型的数据库 (例如，SQL Server Compact Edition)。  

**密钥**： 未使用; 将为 null  

>[!NOTE]
> 有关在 EF6 中与提供程序相关的服务的详细信息，请参阅[EF6 提供程序模型](~/ef6/fundamentals/providers/provider-model.md)部分。  

## <a name="systemdataentityinfrastructureimanifesttokenservice"></a>System.Data.Entity.Infrastructure.IManifestTokenService  

**版本引入了**: EF6.0.0  

**返回对象**： 一种服务，可以从连接生成提供程序清单标记。 通常通过两种方式使用此服务。 首先，它可以用于避免 Code First 生成模型时，连接到数据库。 其次，它可以用于强制代码优先，若要生成的模型特定的数据库版本--例如，若要强制进行 SQL Server 2005 的一个模型，即使有时使用 SQL Server 2008。  

**对象生存期**： 单一实例-相同的对象可能是使用多个时间同时由不同的线程  

**密钥**： 未使用; 将为 null  

## <a name="systemdataentityinfrastructureidbproviderfactoryservice"></a>System.Data.Entity.Infrastructure.IDbProviderFactoryService  

**版本引入了**: EF6.0.0  

**返回对象**： 一种服务，可以从给定的连接获取提供程序工厂。 在.NET 4.5 提供程序是从连接可公开访问。 .NET 4 上的此服务的默认实现使用某些试探方法查找匹配的提供程序。 如果这些然后失败，此服务的新实现可以注册为提供相应的解决方法。  

**密钥**： 未使用; 将为 null  

## <a name="funcdbcontext-systemdataentityinfrastructureidbmodelcachekey"></a>Func < DbContext，System.Data.Entity.Infrastructure.IDbModelCacheKey\>  

**版本引入了**: EF6.0.0  

**返回对象**： 一个工厂，它将生成给定上下文的模型缓存键。 默认情况下，EF 缓存每个 DbContext 类型，每个提供程序的一个模型。 此服务的不同实现可以用于将其他信息，如架构名称添加到缓存键。  

**密钥**： 未使用; 将为 null  

## <a name="systemdataentityspatialdbspatialservices"></a>System.Data.Entity.Spatial.DbSpatialServices  

**版本引入了**: EF6.0.0  

**返回对象**： 向 geography 和 geometry 空间类型的基本 EF 提供程序支持的 EF 空间提供程序。  

**密钥**: DbSptialServices 要求的两种方式。 首先，提供程序特定的空间服务请求使用 DbProviderInfo 对象 (其中包含固定条件名称和清单标记) 作为键。 其次，DbSpatialServices 可以要求不含键。 这用于解决"全局空间提供程序"在创建独立的 DbGeography 或 DbGeometry 类型时使用。  

>[!NOTE]
> 有关在 EF6 中与提供程序相关的服务的详细信息，请参阅[EF6 提供程序模型](~/ef6/fundamentals/providers/provider-model.md)部分。  

## <a name="funcsystemdataentityinfrastructureidbexecutionstrategy"></a>Func < System.Data.Entity.Infrastructure.IDbExecutionStrategy\>  

**版本引入了**: EF6.0.0  

**返回对象**： 工厂可以创建允许提供程序时对数据库执行查询和命令实现重试或其他行为的服务。 如果未不提供任何实现，然后 EF 将只需执行命令并引发任何异常传播。 SQL Server 的此服务用于提供的重试策略，这是针对基于云的数据库服务器，例如 SQL Azure 运行时特别有用。  

**密钥**: ExecutionStrategyKey 对象，其中包含提供程序固定名称和 （可选） 将用于执行策略的服务器名称。  

>[!NOTE]
> 有关在 EF6 中与提供程序相关的服务的详细信息，请参阅[EF6 提供程序模型](~/ef6/fundamentals/providers/provider-model.md)部分。  

## <a name="funcdbconnection-string-systemdataentitymigrationshistoryhistorycontext"></a>Func < DbConnection，字符串，System.Data.Entity.Migrations.History.HistoryContext\>  

**版本引入了**: EF6.0.0  

**返回对象**： 一个工厂，它允许一个提供程序来配置映射到 HistoryContext`__MigrationHistory`使用 EF 迁移的表。 HistoryContext 是代码的第一个 DbContext，并且可以使用正常的 fluent API 来更改像表和列映射规范的名称进行配置。  

**密钥**： 未使用; 将为 null  

>[!NOTE]
> 有关在 EF6 中与提供程序相关的服务的详细信息，请参阅[EF6 提供程序模型](~/ef6/fundamentals/providers/provider-model.md)部分。  

## <a name="systemdatacommondbproviderfactory"></a>System.Data.Common.DbProviderFactory  

**版本引入了**: EF6.0.0  

**返回对象**： 要用于给定提供程序固定名称的 ADO.NET 提供程序。  

**密钥**： 包含 ADO.NET 提供程序固定名称的字符串  

>[!NOTE]
> 此服务不通常会更改直接由于的默认实现使用普通 ADO.NET 提供程序注册。 有关在 EF6 中与提供程序相关的服务的详细信息，请参阅[EF6 提供程序模型](~/ef6/fundamentals/providers/provider-model.md)部分。  

## <a name="systemdataentityinfrastructureiproviderinvariantname"></a>System.Data.Entity.Infrastructure.IProviderInvariantName  

**版本引入了**: EF6.0.0  

**返回对象**： 一种服务，用于确定给定类型的 DbProviderFactory 的提供程序固定名称。 此服务的默认实现使用 ADO.NET 提供程序注册。 这意味着，如果因为 DbProviderFactory 正在解析由 EF，ADO.NET 提供程序未注册以正常方式，然后它将还需要解析此服务。  

**密钥**: DbProviderFactory 的固定名称是必需的实例。  

>[!NOTE]
> 有关在 EF6 中与提供程序相关的服务的详细信息，请参阅[EF6 提供程序模型](~/ef6/fundamentals/providers/provider-model.md)部分。  

## <a name="systemdataentitycoremappingviewgenerationiviewassemblycache"></a>System.Data.Entity.Core.Mapping.ViewGeneration.IViewAssemblyCache  

**版本引入了**: EF6.0.0  

**返回对象**： 包含预生成的视图的程序集的缓存。 替换通常用于让 EF 知道哪些程序集而无需执行任何发现包含预生成的视图。  

**密钥**： 未使用; 将为 null  

## <a name="systemdataentityinfrastructurepluralizationipluralizationservice"></a>System.Data.Entity.Infrastructure.Pluralization.IPluralizationService

**版本引入了**: EF6.0.0  

**返回对象**： 由 EF 来改为复数形式和所名称的服务。 默认情况下使用英语复数化服务。  

**密钥**： 未使用; 将为 null  

## <a name="systemdataentityinfrastructureinterceptionidbinterceptor"></a>System.Data.Entity.Infrastructure.Interception.IDbInterceptor  

**版本引入了**: EF6.0.0

**返回的对象**: 应用程序启动时应注册任何侦听器。 请注意，这些对象使用 getservices 进行提取调用请求和返回的任何依赖关系解析程序的所有侦听器都会注册。

**密钥**： 未使用; 将为 null。  

## <a name="funcsystemdataentitydbcontext-actionstring-systemdataentityinfrastructureinterceptiondatabaselogformatter"></a>Func < System.Data.Entity.DbContext、 操作 < 字符串\>，System.Data.Entity.Infrastructure.Interception.DatabaseLogFormatter\>  

**版本引入了**: EF6.0.0  

**返回对象**： 将用于创建的数据库日志格式化程序将是一个工厂时使用的上下文。Database.Log 属性设置在给定的上下文。  

**密钥**： 未使用; 将为 null。  

## <a name="funcsystemdataentitydbcontext"></a>Func < System.Data.Entity.DbContext\>  

**版本引入了**: EF6.1.0  

**返回对象**： 将用于当上下文不具有可访问的无参数构造函数时为迁移创建上下文实例的工厂。  

**密钥**: 工厂为其所需的派生 dbcontext 类型的类型对象。  

## <a name="funcsystemdataentitycoremetadataedmimetadataannotationserializer"></a>Func < System.Data.Entity.Core.Metadata.Edm.IMetadataAnnotationSerializer\>  

**版本引入了**: EF6.1.0  

**返回对象**： 一个工厂，它将用于创建序列化程序进行序列化强类型化自定义批注，以便它们能够序列化并在 Code First 迁移中使用 desterilized 为 XML。  

**密钥**： 正在批注的名称序列化或反序列化。  

## <a name="funcsystemdataentityinfrastructuretransactionhandler"></a>Func < System.Data.Entity.Infrastructure.TransactionHandler\>  

**版本引入了**: EF6.1.0  

**返回对象**： 一个工厂，它将用于创建事务的处理程序，以便可以如处理提交失败的情况下应用特殊处理。  

**密钥**: ExecutionStrategyKey 对象，其中包含提供程序固定名称和 （可选） 将使用事务处理程序的服务器名称。  
