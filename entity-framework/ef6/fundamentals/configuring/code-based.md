---
title: 基于代码的配置-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 13886d24-2c74-4a00-89eb-aa0dee328d83
ms.openlocfilehash: 7c7da8992fd1b29f998e08c13d445c1d2d8cc2ce
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490813"
---
# <a name="code-based-configuration"></a>基于代码的配置
> [!NOTE]
> **仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。 如果使用的是早期版本，则部分或全部信息不适用。  

在配置文件 (app.config/web.config) 或通过代码，可以指定的实体框架应用程序配置。 后一种称为基于代码的配置。  

配置配置文件中的所述[单独的文章](config-file.md)。 配置文件将优先于基于代码的配置。 换而言之，如果在这两个代码和配置文件中设置的配置选项，则使用配置文件中的设置。  

## <a name="using-dbconfiguration"></a>使用 DbConfiguration  

基于代码的配置在 EF6 及更高版本是通过创建 System.Data.Entity.Config.DbConfiguration 的子类来实现的。 子类化 DbConfiguration 时，应遵循以下准则：  

- 创建应用程序只有一个 DbConfiguration 类。 此类指定应用程序域范围的设置。  
- 将 DbConfiguration 类放在 DbContext 类相同的程序集中。 (请参阅*移动 DbConfiguration*部分如果想要更改此设置。)  
- 为 DbConfiguration 类提供公共的无参数构造函数。  
- 通过调用此构造函数中的受保护的 DbConfiguration 方法从设置配置选项。  

遵循以下准则使得 EF 可发现和需要访问您的模型和运行你的应用程序时这两个工具通过自动使用你的配置。  

## <a name="example"></a>示例  

从 DbConfiguration 派生的类可能如下所示：  

``` csharp
using System.Data.Entity;
using System.Data.Entity.Infrastructure;
using System.Data.Entity.SqlServer;

namespace MyNamespace
{
    public class MyConfiguration : DbConfiguration
    {
        public MyConfiguration()
        {
            SetExecutionStrategy("System.Data.SqlClient", () => new SqlAzureExecutionStrategy());
            SetDefaultConnectionFactory(new LocalDBConnectionFactory("mssqllocaldb"));
        }
    }
}
```  

此类设置 EF，若要使用 SQL Azure 执行策略的自动重试失败的数据库操作-和 Local DB 用于通过从代码优先约定创建的数据库。  

## <a name="moving-dbconfiguration"></a>移动 DbConfiguration  

有些情况下，不能将 DbConfiguration 类放在 DbContext 类相同的程序集中。 例如，你可能有两个 DbContext 类每个不同程序集中。 有两个选项用于实现此操作。  

第一个选项是使用配置文件来指定要使用的 DbConfiguration 实例。 若要执行此操作，设置 entityFramework 部分的 codeConfigurationType 属性。 例如：  

``` xml
<entityFramework codeConfigurationType="MyNamespace.MyDbConfiguration, MyAssembly">
    ...Your EF config...
</entityFramework>
```  

CodeConfigurationType 的值必须是程序集和 DbConfiguration 类的命名空间限定的名称。  

第二个选项是 DbConfigurationTypeAttribute 置于您的上下文类。 例如：  

``` csharp  
[DbConfigurationType(typeof(MyDbConfiguration))]
public class MyContextContext : DbContext
{
}
```  

值传递给该属性可以是你的 DbConfiguration 类型--如上所示，或程序集和命名空间限定类型名称字符串。 例如：  

``` csharp
[DbConfigurationType("MyNamespace.MyDbConfiguration, MyAssembly")]
public class MyContextContext : DbContext
{
}
```  

## <a name="setting-dbconfiguration-explicitly"></a>显式设置 DbConfiguration  

有某些情况下，已使用任何 DbContext 类型之前可能需要配置。 这样的示例包括：  

- 使用 DbModelBuilder 生成没有上下文的模型  
- 使用使用的 DbContext 之前使用应用程序上下文则使用该上下文的其他一些框架/实用程序代码  

在这种情况下 EF 是无法自动发现配置，而是必须执行以下项之一：  

- DbConfiguration 中的类型设置配置文件中，如中所述*移动 DbConfiguration*上面部分
- 在应用程序启动过程中调用静态 DbConfiguration.SetConfiguration 方法  

## <a name="overriding-dbconfiguration"></a>重写 DbConfiguration  

有某些情况下，您需要重写中 DbConfiguration 设置的配置。 这不通常是由应用程序开发人员，而是由第三方提供程序和不能使用一个 DbConfiguration 类，派生的插件。  

为此，EntityFramework 允许事件处理程序注册的向下锁定之前可以修改现有配置。  它还提供专用于替换任何服务返回了由 EF 服务定位器的 sugar 方法。 这是如何它旨在使用：  

- 在应用启动时 （之前使用 EF） 的插件或提供程序应注册此事件的事件处理程序方法。 （请注意这必须发生之前应用程序使用 EF）。  
- 事件处理程序调用到 ReplaceService 需要替换每个服务。  

例如下 IDbConnectionFactory, 和 DbProviderService 会注册一个处理程序如下所示：  

``` csharp
DbConfiguration.Loaded += (_, a) =>
   {
       a.ReplaceService<DbProviderServices>((s, k) => new MyProviderServices(s));
       a.ReplaceService<IDbConnectionFactory>((s, k) => new MyConnectionFactory(s));
   };
```  

在代码中更高版本的 MyProviderServices MyConnectionFactory 表示你的服务的实现。  

此外可以添加其他依赖项的处理程序来获取相同的效果。  

请注意，您还可以封装 DbProviderFactory 在这样一来，但这样做因此只会影响 EF 而不使用在 EF 外部 DbProviderFactory。 出于此原因您可能需要继续为之前已包装 DbProviderFactory。  

您应该还请记住您运行外部到你的应用程序-例如，从包管理器控制台运行迁移时的服务。 在运行时将从它将尝试查找你 DbConfiguration 控制台迁移。 但是，无论有无它将获取已包装的服务依赖于 where 它注册的事件处理程序。 如果你 DbConfiguration 的构造过程中注册然后应执行的代码和服务应获取包装。 通常这不会出现这种情况，这意味着工具不会获取已包装的服务。  
