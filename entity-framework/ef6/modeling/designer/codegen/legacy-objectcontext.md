---
title: 恢复到 ObjectContext 中实体框架设计器的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 36550569-a1de-47cb-ba6d-544794ffd500
ms.openlocfilehash: 3e436f0d9cf94720be9c424b327816438d571ae8
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45488930"
---
# <a name="reverting-to-objectcontext-in-entity-framework-designer"></a>恢复到 ObjectContext 中实体框架设计器
与以前版本的 Entity Framework 使用 EF 设计器创建的模型将生成派生自 ObjectContext 的上下文和从 entityobject 继承派生的实体类。

从 EF4.1 建议交换到生成派生自 DbContext 和 POCO 实体类的上下文的代码生成模板。

在 Visual Studio 2012 中获取默认情况下，所有新创建的模型使用 EF 设计器生成的 DbContext 代码。 现有模型将继续生成基于 ObjectContext 代码，除非你决定切换到 DbContext 基于代码生成器。

## <a name="reverting-back-to-objectcontext-code-generation"></a>恢复到 ObjectContext 代码生成

### <a name="1-disable-dbcontext-code-generation"></a>1.禁用 DbContext 代码生成

派生的 DbContext 和 POCO 类生成由你的项目中的两个.tt 文件，如果展开.edmx 文件在解决方案资源管理器中的会看到这些文件。 从项目中删除这两个文件。

![常规文件](~/ef6/media/codegenfiles.png)

如果使用 VB.NET 将需要选择**显示所有文件**按钮以查看嵌套的文件。

![显示所有文件](~/ef6/media/showallfiles.png)

### <a name="2-re-enable-objectcontext-code-generation"></a>2.重新启用 ObjectContext 代码生成

打开模型在 EF 设计器中，右击设计图面并选择一个空白部分**属性**。

在属性窗口中更改**代码生成策略**从**None**到**默认**。

![代码常规策略](~/ef6/media/codegenstrategy.png)
