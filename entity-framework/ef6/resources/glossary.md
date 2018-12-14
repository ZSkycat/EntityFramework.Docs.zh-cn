---
title: 实体框架术语表-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 3f05ffdd-49bc-499c-9732-4a368bf5d2d7
ms.openlocfilehash: 9ed7a2d841c625de35de57edb4e57e69b89a3db9
ms.sourcegitcommit: 5d74ac575f813110db6d870720f50dd7606446bc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2018
ms.locfileid: "48881262"
---
# <a name="entity-framework-glossary"></a>实体框架术语表
## <a name="code-first"></a>Code First
创建使用代码的实体框架模型。 该模型可以指定的目标和现有数据库或新数据库。

## <a name="context"></a>上下文
一个类，表示与数据库中，您可以查询和保存数据的会话。 上下文派生自 DbContext 或 ObjectContext 类。

## <a name="convention-code-first"></a>约定 （代码优先）
实体框架使用推断的模型从您的类形状的规则。

## <a name="database-first"></a>数据库优先
创建实体框架模型，使用 EF 设计器中，面向现有数据库。

## <a name="eager-loading"></a>预先加载
加载相关的数据的一种类型的实体的查询在查询的一部分，并加载相关的实体的模式。

## <a name="ef-designer"></a>EF 设计器
可以创建使用框和行的实体框架模型的 Visual Studio 中可视化设计器。

## <a name="entity"></a>实体
表示客户、产品和订单等应用程序数据的类或对象。

## <a name="entity-data-model"></a>实体数据模型
描述实体和它们之间的关系模型。 EF 使用 EDM 来描述概念模型对其开发人员计划。 引入的灾难恢复的实体关系模型上构建 EDM。Peter Chen。 EDM 的最初开发的一系列来自 Microsoft 的开发人员和服务器技术成为通用数据模型的主要目标。 EDM 也用作 OData 协议的一部分。

## <a name="explicit-loading"></a>显式加载
加载相关的对象通过调用 API 的加载位置的相关的数据的模式。

## <a name="fluent-api"></a>Fluent API
可用于配置 Code First 模型 API。

## <a name="foreign-key-association"></a>外键关联
其中该属性表示外键包含依赖的实体的类中的实体之间的关联。 例如，产品包含 CategoryId 属性。

## <a name="identifying-relationship"></a>标识关系
一种关系，其中主体实体的主键是依赖实体的主键的一部分。 在这种关系中，没有主体实体，依赖实体就不能存在。

## <a name="independent-association"></a>独立关联
实体之间的关联没有表示依赖实体的类中的外键没有属性。 例如，Product 类包含一个到类别，但没有 CategoryId 属性关系。 实体框架跟踪独立于位于两个关联端的实体的状态关联的状态。

## <a name="lazy-loading"></a>延迟加载
加载相关的数据的相关的对象时，自动加载访问导航属性的模式。

## <a name="model-first"></a>模型优先
创建实体框架模型，使用 EF 设计器，然后用于创建新的数据库。

## <a name="navigation-property"></a>导航属性
实体引用另一个实体的属性。 例如，产品包含的类别导航属性和类别包含产品导航属性。

## <a name="poco"></a>POCO
纯旧式 CLR 对象的首字母缩写。 一个简单的用户类，并不依赖的任何框架。 在 EF 中，不是派生从 entityobject 继承、 实现任何接口或执行在 EF 中定义的任何特性的实体类的上下文。 此类从持久性框架分离的实体类也被称为是"持久性未知"。  

## <a name="relationship-inverse"></a>关系反转
一种关系，例如，产品中相对的一端。类别和类别。产品。

## <a name="self-tracking-entity"></a>自跟踪实体
从代码生成模板，使用 N 层开发可帮助生成实体。

## <a name="table-per-concrete-type-tpc"></a>表每个具体类型 (TPC)
一种映射继承层次结构中的每个非抽象类型映射到单独数据库中的表的其中一个方法。

## <a name="table-per-hierarchy-tph"></a>每个层次结构一张表 (TPH)
映射位置层次结构中的所有类型都映射到数据库中的同一个表继承的方法。 鉴别器列用于标识哪种类型的每一行与之关联。

## <a name="table-per-type-tpt"></a>每种类型一个表 (TPT)
映射的继承层次结构中的所有类型的通用属性映射到在数据库中，同一个表，但唯一每种类型的属性映射到一个单独的表的方法。

## <a name="type-discovery"></a>类型发现
标识应该是实体框架模型的一部分的类型的过程。
