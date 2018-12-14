---
title: 设计时服务的 EF Core
author: bricelam
ms.author: bricelam
ms.date: 10/26/2017
ms.openlocfilehash: e1cacdd4f40f9c395d8c88a91df4a92ef27001a8
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42997526"
---
<a name="design-time-services"></a>设计时服务
====================
工具使用某些服务仅在设计时使用。 这些服务 EF Core运行时服务，以阻止它们不会与你的应用部署分开管理。 若要重写其中一种服务 （例如服务来生成迁移文件），将添加的实现`IDesignTimeServices`到启动项目。

``` csharp
class MyDesignTimeServices : IDesignTimeServices
{
    public void ConfigureDesignTimeServices(IServiceCollection services)
        => services.AddSingleton<IMigrationsCodeGenerator, MyMigrationsCodeGenerator>()
}
```
