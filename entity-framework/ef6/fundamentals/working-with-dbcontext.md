---
title: 使用 DbContext 的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: b0e6bddc-8a87-4d51-b1cb-7756df938c23
ms.openlocfilehash: d961ffd8bed7f5b2f82dcfa30fc0241b7437be50
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489058"
---
# <a name="working-with-dbcontext"></a>使用 DbContext

若要使用实体框架查询、 插入、 更新和删除使用.NET 对象的数据，需首先[创建模型](~/ef6/modeling/index.md)映射的实体和关系数据库中的表在模型中定义的。

一旦您有一个模型，与你的应用程序进行交互的主类是`System.Data.Entity.DbContext`（通常称为上下文类）。 可以使用与模型关联的 DbContext:
- 编写和执行查询   
- 将查询结果具体化为实体对象
- 跟踪对这些对象所做的更改
- 保存在数据库上返回的对象更改
- 在内存中对象绑定到 UI 控件

此页提供有关如何管理上下文类的一些指导。  

## <a name="defining-a-dbcontext-derived-class"></a>定义派生的 DbContext 类  

使用上下文的建议的方法是定义一个派生自 DbContext，并公开 DbSet 属性所表示的上下文中的指定实体的集合类。 如果您正在使用 EF 设计器，将为您生成上下文。 如果您正在使用 Code First，你将通常上下文自行编写。  

``` csharp
public class ProductContext : DbContext
{
    public DbSet<Category> Categories { get; set; }
    public DbSet<Product> Products { get; set; }
}
```  

上下文后，你将查询，添加 (使用`Add`或`Attach`方法) 或删除 (使用`Remove`) 通过这些属性的上下文中的实体。 访问`DbSet`上下文对象上的属性表示返回指定类型的所有实体的起始查询。 请注意，只需访问属性时，将执行查询。 执行查询时：  

- `foreach` (C#) 或 `For Each` (Visual Basic) 语句枚举对象。  
- 它枚举一组操作如`ToArray`， `ToDictionary`，或`ToList`。  
- LINQ 运算符，如`First`或`Any`查询的最外面部分中指定。  
- 调用以下方法之一：`Load`扩展方法`DbEntityEntry.Reload`， `Database.ExecuteSqlCommand`，和`DbSet<T>.Find`，如果具有指定键的实体找不到已加载的上下文中。  

## <a name="lifetime"></a>生存期  

上下文的生存期开始时创建实例，并结束时释放或垃圾回收该实例。 使用**使用**如果你想在块的末尾释放上下文控制的所有资源。 当你使用**使用**，编译器将自动创建 try/finally 块中调用释放**最后**块。  

``` csharp
public void UseProducts()
{
    using (var context = new ProductContext())
    {     
        // Perform data access using the context
    }
}
```  

在决定对上下文的生存期时，以下是一些通用准则：  

- 使用 Web 应用程序，使用每个请求的上下文实例。  
- 使用 Windows Presentation Foundation (WPF) 或 Windows 窗体，使用每个窗体的上下文实例。 这使您可以使用更改跟踪功能提供了该上下文。  
- 如果上下文实例创建了依赖关系注入容器，它通常是容器的要释放上下文的责任。
- 如果应用程序代码中创建了上下文，请记住在不再需要时释放上下文。  
- 使用长时间运行的上下文时考虑以下方面：  
    - 加载到内存更多对象和它们的引用，如上下文的内存消耗可能会快速增加。 这可能会导致性能问题。  
    - 上下文不是线程安全的因此它不应共享跨多个线程同时执行它的工作。
    - 如果异常会导致上下文处于不可恢复的状态，可能会终止整个应用程序。  
    - 随着查询数据的时间和更新数据的时间的差距增大，出现与并发性相关的问题的可能性将会增加。  

## <a name="connections"></a>连接  

默认情况下，上下文管理与数据库的连接。 上下文将打开和关闭连接，根据需要。 例如，上下文打开连接来执行查询，，并在处理所有结果集时，然后关闭连接。  

在某些情况下，您需要加强控制应在哪些情况下打开和关闭连接。 例如，在使用 SQL Server Compact，通常建议来维护与数据库的应用程序以提高性能的生存期内的单独打开连接。 您可以使用 `Connection` 属性手动管理此过程。  
