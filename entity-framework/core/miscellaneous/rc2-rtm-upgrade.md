---
title: 从 EF 升级核心到 RTM-1.0 RC2 EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: c3c1940b-136d-45d8-aa4f-cb5040f8980a
uid: core/miscellaneous/rc2-rtm-upgrade
ms.openlocfilehash: 1b95b2ab1943dfb541b3a7c873cff3cb4c16d9c1
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42998314"
---
# <a name="upgrading-from-ef-core-10-rc2-to-rtm"></a>从 EF Core 1.0 RC2 升级到 RTM

本文介绍了使用为 1.0.0 RC2 包生成应用程序迁移指南 RTM。

## <a name="package-versions"></a>包版本

RC2 和 RTM 之间未更改通常会安装到的应用程序的最高级别包的名称。

**需要升级到 RTM 版本的已安装的包：**

* 运行时包 (例如， `Microsoft.EntityFrameworkCore.SqlServer`) 从更改`1.0.0-rc2-final`到`1.0.0`。

* `Microsoft.EntityFrameworkCore.Tools`包中更改`1.0.0-preview1-final`到`1.0.0-preview2-final`。 请注意，工具仍预发行版。

## <a name="existing-migrations-may-need-maxlength-added"></a>现有迁移可能需要添加的 maxLength

在 RC2 中，在迁移中的列定义看起来像`table.Column<string>(nullable: true)`和中我们将存储在代码隐藏迁移一些元数据中列的长度进行查找。 在 RTM，长度现在包含在已搭建基架代码`table.Column<string>(maxLength: 450, nullable: true)`。

已在使用 RTM 之前的基架的任何现有迁移不会`maxLength`指定的参数。 这意味着，将使用数据库支持的最大长度 (`nvarchar(max)` SQL Server 上)。 这可能是相当不错的某些列，但包含一个键外, 键列或索引需要更新，以包含最大长度。 按照约定，450 是最大长度用于密钥外, 键和索引列。 如果在模型中，具有显式配置长度，然后应改为使用该长度。

**ASP.NET 标识**

此更改会影响项目的早期版本中创建和使用 ASP.NET 标识的 RTM 项目模板。 项目模板包括用于创建数据库的迁移。 必须编辑此迁移，以指定最大长度为`256`为以下列。

*  **AspNetRoles**

    * name

    * NormalizedName

*  **AspNetUsers**

   * 电子邮件

   * NormalizedEmail

   * NormalizedUserName

   * UserName

初始迁移应用到数据库时，无法进行此更改将导致出现以下异常。

    System.Data.SqlClient.SqlException (0x80131904): Column 'NormalizedName' in table 'AspNetRoles' is of a type that is invalid for use as a key column in an index.

## <a name="net-core-remove-imports-in-projectjson"></a>.NET Core： project.json 中删除"导入"

如果你已面向.NET Core 与 RC2，需要添加`imports`到临时的解决方法不支持.NET 标准的 EF Core 依赖关系的某些 project.json。 可以立即删除这些。

``` json
{
  "frameworks": {
    "netcoreapp1.0": {
      "imports": ["dnxcore50", "portable-net451+win8"]
    }
  }
}
```

> [!NOTE]  
> 截至版本 1.0 RTM， [.NET Core SDK](https://www.microsoft.com/net/download/core)不再支持`project.json`或开发.NET Core 应用程序使用 Visual Studio 2015。 我们建议你[从 project.json 迁移到 csproj](https://docs.microsoft.com/dotnet/articles/core/migration/)。 如果使用的 Visual Studio，我们建议升级到[Visual Studio 2017](https://www.visualstudio.com/downloads/)。

## <a name="uwp-add-binding-redirects"></a>UWP： 添加绑定重定向

尝试在以下的错误导致的通用 Windows 平台 (UWP) 项目上运行 EF 命令：

    System.IO.FileLoadException: Could not load file or assembly 'System.IO.FileSystem.Primitives, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' or one of its dependencies. The located assembly's manifest definition does not match the assembly reference.

您需要将绑定重定向手动添加到 UWP 项目。 创建名为`App.config`在项目根文件夹并将重定向添加到正确的程序集版本。

``` xml
<configuration>
 <runtime>
   <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
     <dependentAssembly>
       <assemblyIdentity name="System.IO.FileSystem.Primitives"
                         publicKeyToken="b03f5f7f11d50a3a"
                         culture="neutral" />
       <bindingRedirect oldVersion="4.0.0.0"
                        newVersion="4.0.1.0"/>
     </dependentAssembly>
     <dependentAssembly>
       <assemblyIdentity name="System.Threading.Overlapped"
                         publicKeyToken="b03f5f7f11d50a3a"
                         culture="neutral" />
       <bindingRedirect oldVersion="4.0.0.0"
                        newVersion="4.0.1.0"/>
     </dependentAssembly>
   </assemblyBinding>
 </runtime>
</configuration>
```
