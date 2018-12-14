---
title: SQLite 数据库提供程序的限制的 EF Core
author: rowanmiller
ms.date: 04/09/2017
ms.assetid: 94ab4800-c460-4caa-a5e8-acdfee6e6ce2
uid: core/providers/sqlite/limitations
ms.openlocfilehash: 69c40fcd8b7ddb925728b1bad9992ad2a81e7540
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42994659"
---
# <a name="sqlite-ef-core-database-provider-limitations"></a>SQLite EF Core 数据库提供程序限制

SQLite 提供程序具有许多迁移限制。 其中大多数限制是基础的 SQLite 数据库引擎中的限制的结果并不特定于 EF。

## <a name="modeling-limitations"></a>建模限制

（由实体框架关系数据库提供程序共享） 的公共关系库定义建模概念所共有的大多数关系数据库引擎的 Api。 SQLite 提供程序不支持几个这些概念。

* 架构
* 序列

## <a name="migrations-limitations"></a>迁移限制

SQLite 数据库引擎不支持多个架构操作支持的其他关系数据库的大多数。 如果你尝试将其中一个不受支持的操作应用到 SQLite 数据库则`NotSupportedException`将引发。

| 操作            | 支持？ | 需要版本 |
|:---------------------|:-----------|:-----------------|
| AddColumn            | ✔          | 1.0              |
| AddForeignKey        | ✗          |                  |
| AddPrimaryKey        | ✗          |                  |
| AddUniqueConstraint  | ✗          |                  |
| AlterColumn          | ✗          |                  |
| CreateIndex          | ✔          | 1.0              |
| CreateTable          | ✔          | 1.0              |
| DropColumn           | ✗          |                  |
| DropForeignKey       | ✗          |                  |
| DropIndex            | ✔          | 1.0              |
| DropPrimaryKey       | ✗          |                  |
| DropTable            | ✔          | 1.0              |
| DropUniqueConstraint | ✗          |                  |
| RenameColumn         | ✗          |                  |
| RenameIndex          | ✔          | 2.1              |
| RenameTable          | ✔          | 1.0              |
| EnsureSchema         | ✔ （无操作）  | 2.0              |
| DropSchema           | ✔ （无操作）  | 2.0              |
| Insert               | ✔          | 2.0              |
| 更新               | ✔          | 2.0              |
| 删除               | ✔          | 2.0              |

## <a name="migrations-limitations-workaround"></a>迁移限制的解决方法

可以解决一些通过手动编写代码在你迁移执行表中的这些限制的重新生成。 表重新生成涉及重命名现有表、 创建新表、 将数据复制到新的表和删除旧表。 将需要使用`Sql(string)`方法来执行这些步骤中的一部分。

请参阅[进行其他类型的表架构更改](http://sqlite.org/lang_altertable.html#otheralter)SQLite 文档了解详细信息中。

将来，EF 可能支持某些操作使用在后台的表重新生成方法。 你可以[跟踪此功能在我们的 GitHub 项目](https://github.com/aspnet/EntityFrameworkCore/issues/329)。
