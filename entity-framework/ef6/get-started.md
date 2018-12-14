---
title: 开始使用 Entity Framework 6 - EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 66ce9113-81d2-480f-8c16-d00ec405b2f7
ms.openlocfilehash: c25bf16bd2c39530d54b286b7743ceb83c941e4d
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489280"
---
# <a name="get-started-with-entity-framework-6"></a>开始使用 Entity Framework 6

本指南包含指向精选文章、演练和视频的链接集合，这些内容可帮助你快速入门。

## <a name="fundamentals"></a>基础知识

* [获取 Entity Framework](~/ef6/fundamentals/install.md)

  此处可学习如何将 Entity Framework 添加到应用程序。如果要使用 EF 设计器，请确保在 Visual Studio 中安装它。

* [创建模型：Code First、EF 设计器和 EF 工作流](~/ef6/modeling/index.md)

  是否希望指定 EF 模型编写代码或绘制方框和线条？
是否要使用 EF 来将对象映射到现有数据库，或希望 EF 创建为对象量身打造的数据库？
此处可学习使用 EF6 的两种不同方法：EF 设计器和 Code First。
请确保关注讨论内容并查看有关不同之处的视频。

* [使用 DbContext](~/ef6/fundamentals/working-with-dbcontext.md)

  DbContext 是需要学习其使用方法的第一个也是最重要的一个 EF 类型。 它可用作数据库查询的启动板，并可跟踪对对象作出的更改，以便持续存回数据库。

* [提出问题](~/ef6/resources/get-help.md)

  了解如何获取专家的帮助，并向社区贡献自己的答案。

* [参与](http://github.com/aspnet/EntityFramework6/)

  Entity Framework 6 采用开放式开发模型。 访问我们的 GitHub 存储库，了解如何帮助改进 EF。

## <a name="code-first-resources"></a>Code First 资源

  - [对现有数据库工作流采用 Code First](~/ef6/modeling/code-first/workflows/existing-database.md)
  - [对新的数据库工作流采用 Code First](~/ef6/modeling/code-first/workflows/new-database.md)
  - [使用 Code First 映射枚举](~/ef6/modeling/code-first/data-types/enums.md)
  - [使用 Code First 映射空间类型](~/ef6/modeling/code-first/data-types/spatial.md)
  - [编写自定义 Code First 约定](~/ef6/modeling/code-first/conventions/custom.md)
  - [将 Code First Fluent 配置与 Visual Basic 配合使用](~/ef6/modeling/code-first/fluent/vb.md)
  - [Code First 迁移](~/ef6/modeling/code-first/migrations/index.md)
  - [团队环境中的 Code First 迁移](~/ef6/modeling/code-first/migrations/teams.md)
  - [自动 Code First 迁移](~/ef6/modeling/code-first/migrations/automatic.md)（不再推荐）

## <a name="ef-designer-resources"></a>EF 设计器资源
  - [Database First 工作流](~/ef6/modeling/designer/workflows/database-first.md)
  - [Model First 工作流](~/ef6/modeling/designer/workflows/model-first.md)
  - [映射枚举](~/ef6/modeling/designer/data-types/enums.md)
  - [映射空间类型](~/ef6/modeling/designer/data-types/spatial.md)
  - [每个层次结构一张表继承映射](~/ef6/modeling/designer/inheritance/tph.md)
  - [每个类型一张表继承映射](~/ef6/modeling/designer/inheritance/tpt.md)
  - [用于更新的存储过程映射](~/ef6/modeling/designer/stored-procedures/cud.md)
  - [用于查询的存储过程映射](~/ef6/modeling/designer/stored-procedures/query.md)
  - [实体拆分](~/ef6/modeling/designer/entity-splitting.md)
  - [表拆分](~/ef6/modeling/designer/table-splitting.md)
  - [定义查询](~/ef6/modeling/designer/advanced/defining-query.md)（高级）
  - [表值函数](~/ef6/modeling/designer/advanced/tvfs.md)（高级）

## <a name="other-resources"></a>其他资源
  - [异步查询和保存](~/ef6/fundamentals/async.md)
  - [使用 WinForms 进行数据绑定](~/ef6/fundamentals/databinding/winforms.md)
  - [使用 WPF 进行数据绑定](~/ef6/fundamentals/databinding/wpf.md)
  - [使用自跟踪实体的断开连接的方案](~/ef6/fundamentals/disconnected-entities/self-tracking-entities/walkthrough.md)（不再推荐）
