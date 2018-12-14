---
title: 创建模型 - EF6
author: divega
ms.date: 07/05/2018
ms.assetid: 4890228E-CEA1-4595-B8AD-CA81253F8767
ms.openlocfilehash: c02cdf0550116b703fb6436f8b0c6b064b5d1408
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283571"
---
# <a name="creating-a-model"></a>创建模型

EF 模型可存储有关应用程序类和属性如何映射到数据库表和列的详细信息。 创建 EF 模型有两种主要方式：

- **使用 Code First**：开发者编写代码来指定模型。 EF 基于实体类和开发者提供的其他模型配置，在运行时生成模型和映射。

- **使用 EF 设计器**：开发者使用 EF 设计器进行绘制以指定模型。 生成的模型以 XML 格式存储在具有 EDMX 扩展名的文件中。 应用程序域对象通常从概念模型中自动生成。

## <a name="ef-workflows"></a>EF 工作流

这两种方式都可用于定位现有数据库或创建新数据库，最终会产生 4 种不同的工作流。
了解哪一种最适合：  

|                                           | 我只想编写代码...                                                                                                                   | 我想要使用设计器...                                                                                                                        |
|:------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------|
| **我正在创建新数据库**          | [使用 Code First 在代码中定义模型，然后生成数据库。](~/ef6/modeling/code-first/workflows/new-database.md)           | [通过 Model First 使用框和线定义模型，然后生成数据库。](~/ef6/modeling/designer/workflows/model-first.md)   |
| **我需要访问现有数据库** | [使用 Code First 创建映射到现有数据库的基于代码的模型。](~/ef6/modeling/code-first/workflows/existing-database.md) | [使用 Database First 创建映射到现有数据库的框和线模型。](~/ef6/modeling/designer/workflows/database-first.md) |

### <a name="watch-the-video-what-ef-workflow-should-i-use"></a>观看视频：我应该使用哪种 EF 工作流？

这段短片介绍了其中的差异，以及如何找到适合的工作流。

**主讲人**：[Rowan Miller](http://romiller.com/)

![Which Workflow Thumb](../media/whichworkflow-thumb.png) [WMV](https://download.microsoft.com/download/8/F/8/8F81F4CD-3678-4229-8D79-0C63FFA3C595/HDI_ITPro_Technet_winvideo_ChoseYourWorkflow.wmv) | [MP4](https://download.microsoft.com/download/8/F/8/8F81F4CD-3678-4229-8D79-0C63FFA3C595/HDI_ITPro_Technet_mp4video_ChoseYourWorkflow.m4v) | [WMV (ZIP)](https://download.microsoft.com/download/8/F/8/8F81F4CD-3678-4229-8D79-0C63FFA3C595/HDI_ITPro_Technet_winvideo_ChoseYourWorkflow.zip)

如果在观看视频后仍无法顺利决定使用 EF 设计器还是 Code First，那么分别请了解两者！

## <a name="a-look-under-the-hood"></a>查看二者的本质区别

无论是使用 Code First 还是 EF 设计器，EF 模型始终具有以下几个组件：

- 应用程序域对象或实体类型本身。 这通常称为对象层

- 使用[实体数据模型](~/ef6/resources/glossary.md#entity-data-model)描述的，由特定于域的实体类型和关系组成的概念模型。 该层通常用字母“C”表示概念。

- 表示数据库中定义的表、列和关系的存储模型。 该层通常用字母“S”表示存储。  

- 概念模型和数据库架构之间的映射。 该映射通常称为“C-S”映射。

EF 的映射引擎利用“C-S”映射将针对实体的操作（例如创建、读取、更新和删除）转换为针对数据库中的表的等效操作。

概念模型和应用程序的对象之间的映射通常称为“O-C”映射。 与“C-S”映射相比，“O-C”映射是一对一的隐式映射：概念模型中定义的实体、属性和关系需要与 .NET 对象的形状和类型相匹配。 从 EF4 及更高版本开始，对象层可以由具有属性的简单对象组成，无需任何 EF 依赖关系。 这些通常称为简单传统 CLR 对象 (POCO)，同时类型和属性映射以名称匹配约定为基础进行。 以前，在 EF 3.5 中，对象层存在特定限制，例如实体必须派生自 EntityObject 类，并且必须带有 EF 属性以实现“O-C”映射。
