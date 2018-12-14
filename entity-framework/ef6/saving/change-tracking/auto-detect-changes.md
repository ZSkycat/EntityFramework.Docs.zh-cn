---
title: 自动检测到更改-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: a8d1488d-9a54-4623-a76b-e81329ff2756
ms.openlocfilehash: 9af85fd7ca48a14432a1f33c59079fc438ef8810
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490982"
---
# <a name="automatic-detect-changes"></a>自动检测更改
使用大多数的 POCO 实体时由检测更改算法处理的实体的更改方式 （并因此需要发送到数据库的更新） 确定。 通过检测该实体的当前属性值和当查询或附加该实体已在快照中存储的原始属性值之间的差异来检测更改的工作原理。 本主题所介绍的方法同样适用于查询使用 Code First 和 EF 设计器创建的模型。  

默认情况下，实体框架会自动执行检测更改，当调用以下方法：  

- DbSet.Find  
- DbSet.Local  
- DbSet.Add  
- DbSet.AddRange
- DbSet.Remove  
- DbSet.RemoveRange
- DbSet.Attach  
- DbContext.SaveChanges  
- DbContext.GetValidationErrors  
- DbContext.Entry  
- DbChangeTracker.Entries  

## <a name="disabling-automatic-detection-of-changes"></a>禁用自动检测更改  

如果您正在跟踪大量实体在上下文中并调用下列方法之一很多时候在循环中，然后可能会显著的性能改进通过关闭更改检测循环的持续时间。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    try
    {
        context.Configuration.AutoDetectChangesEnabled = false;

        // Make many calls in a loop
        foreach (var blog in aLotOfBlogs)
        {
            context.Blogs.Add(blog);
        }
    }
    finally
    {
        context.Configuration.AutoDetectChangesEnabled = true;
    }
}
```  

不要忘记重新启用更改检测循环后的，我们使用了 try/finally 来确保始终重新启用即使循环中的代码引发异常。  

除了禁用和重新启用是将自动在所有时间和任一调用上下文已关闭的更改检测。ChangeTracker.DetectChanges 显式或使用更改跟踪代理努力。 这两种选项都高级和可以轻松地细微 bug 引入你的应用程序因此请谨慎使用它们。  

如果需要添加或删除许多对象上下文中，请考虑使用 DbSet.AddRange 和 DbSet.RemoveRange。 此方法会自动检测更改一次后添加或删除操作都已完成。 
