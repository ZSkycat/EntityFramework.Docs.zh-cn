---
title: 处理事务提交失败的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 5b1f7a7d-1b24-4645-95ec-5608a31ef577
ms.openlocfilehash: 27e75e6a1919ee2300fe76cfcdf67cceaad887b3
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283649"
---
# <a name="handling-transaction-commit-failures"></a>处理事务提交失败
> [!NOTE]
> **EF6.1 及更高版本仅**的功能，Api，Entity Framework 6.1 中引入了此页所述的等。 如果使用的是早期版本，则部分或全部信息不适用。  

6.1 的一部分，我们引入新的连接复原功能 ef： 检测和暂时性连接故障影响的确认的事务提交时自动恢复的能力。 该方案的完整详细信息是最适合的博客文章中所述[SQL 数据库连接和幂等性问题](https://blogs.msdn.com/b/adonet/archive/2013/03/11/sql-database-connectivity-and-the-idempotency-issue.aspx)。  总之，该方案是在事务提交过程中引发异常时有两个可能的原因：  

1. 在服务器上的事务提交失败
2. 在服务器上的事务提交成功，但连接问题导致无法从到达客户端的成功通知  

第一种情况发生在应用程序或用户可以重试该操作，但第二种情况发生时应避免使用重试次数和应用程序无法自动恢复。 面临的挑战是操作的，如果不能检测异常报告，在提交期间，应用程序不能选择正确过程的实际原因是操作的什么。 EF 6.1 中的新功能使得 EF 可仔细检查与数据库，如果事务成功，并以透明方式采取适当的操作过程。  

## <a name="using-the-feature"></a>使用的功能  

若要启用该功能需要包括对[SetTransactionHandler](https://msdn.microsoft.com/library/system.data.entity.dbconfiguration.setdefaulttransactionhandler.aspx)的构造函数中你**DbConfiguration**。 如果您不熟悉**DbConfiguration**，请参阅[代码基于配置](~/ef6/fundamentals/configuring/code-based.md)。 可以结合使用自动重试机制在 EF6 中，我们引入了可帮助在中，事务实际上无法提交对服务器造成暂时性故障的情况下使用此功能：  

``` csharp
using System.Data.Entity;
using System.Data.Entity.Infrastructure;
using System.Data.Entity.SqlServer;

public class MyConfiguration : DbConfiguration  
{
  public MyConfiguration()  
  {  
    SetTransactionHandler(SqlProviderServices.ProviderInvariantName, () => new CommitFailureHandler());  
    SetExecutionStrategy(SqlProviderServices.ProviderInvariantName, () => new SqlAzureExecutionStrategy());  
  }  
}
```  

## <a name="how-transactions-are-tracked"></a>如何跟踪事务  

启用该功能后，EF 将自动将新表添加到名为数据库 **__Transactions**。 每次事务创建的 EF，如果在提交期间发生事务失败，检查该行存在时，此表中插入新行。  

尽管 EF 将执行这一最大努力来修剪表中的行，当不再需要时，应用程序在退出过早，因此可能需要手动在某些情况下表中清除表可能增大。  

## <a name="how-to-handle-commit-failures-with-previous-versions"></a>如何处理与早期版本的提交失败

在 EF 6.1 之前没有机制来处理 EF 产品中的提交失败。 有几种方法来处理这种情况可与以前版本的 EF6 应用：  

* 选项 1-不执行任何操作  

  因此，可能会使应用程序直接失败，如果实际上发生这种情况可接受的事务提交过程中的连接故障的可能性较低。  

* 选项 2-使用数据库来重置状态  

  1. 放弃当前的 DbContext  
  2. 创建新的 DbContext 和从数据库中还原应用程序的状态  
  3. 通知用户，最后一次操作可能不具有已成功完成  

* 选项 3-手动跟踪事务  

  1. 将非跟踪的表添加到用于跟踪事务的状态的数据库。  
  2. 插入到表中每个事务的开始处的行。  
  3. 如果连接失败在提交期间，检查存在的数据库中的相应行。  
     - 如果存在该行，则继续正常运行，因为该事务已提交成功  
     - 如果该行不存在，使用执行策略以重试当前操作。  
  4. 如果提交成功，删除对应的行，以避免对表的增长。  

[这篇博客文章](https://blogs.msdn.com/b/adonet/archive/2013/03/11/sql-database-connectivity-and-the-idempotency-issue.aspx)包含在 SQL Azure 上实现此目的的示例代码。  
