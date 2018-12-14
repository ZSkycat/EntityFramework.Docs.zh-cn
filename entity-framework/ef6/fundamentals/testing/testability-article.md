---
title: 可测试性和 Entity Framework 4.0
author: divega
ms.date: 10/23/2016
ms.assetid: 9430e2ab-261c-4e8e-8545-2ebc52d7a247
ms.openlocfilehash: aec177438004fd255bef85a5e5047cf6b5a6f782
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46284039"
---
# <a name="testability-and-entity-framework-40"></a>可测试性和 Entity Framework 4.0
Scott Allen

发布日期： 2010 年 5 月

## <a name="introduction"></a>介绍

本白皮书主要介绍并演示如何编写使用 ADO.NET Entity Framework 4.0 和 Visual Studio 2010 的可测试代码。 本文不会尝试专注于特定的测试方法，如测试驱动设计 (TDD) 或行为驱动设计 (BDD)。 而是本白皮书将重点介绍如何编写使用 ADO.NET 实体框架，但就一直都易于找出并以自动方式测试的代码。 我们将介绍便于测试中的数据访问方案，请参阅如何使用框架时应用这些模式的常见设计模式。 我们还将介绍框架，以了解这些功能可以在可测试代码中的工作方式的特定功能。

## <a name="what-is-testable-code"></a>可测试代码是什么？

若要验证的组成部分软件使用自动化的单元测试的功能可使许多需要优势。 每个人都知道良好测试将减少在应用程序中，然后增加应用程序的质量-但将单元测试在位置远不止仅查找 bug 的软件缺陷的数量。

优良单元测试套件使开发团队可以节省时间并控制其创建的软件。 团队可以对现有代码、 重构、 重新设计，进行更改，并重组软件以满足新需求，并将新组件添加到应用程序中同时了解测试套件可以验证应用程序的行为。 单元测试是软件的快速反馈周期以推动变革和保留的复杂性增加时的可维护性的一部分。

单元测试但是造成一个弊端。 团队必须花些时间来创建和维护单元测试。 与直接相关的创建这些测试所需的工作量**可测试性**的底层的软件。 若要测试的软件是如何简单？ 使用可测试性记住设计软件的团队将比不可测试的软件的团队更快地创建有效的测试。

Microsoft 设计了可测试性记住 ADO.NET Entity Framework 4.0 (EF4)。 这并不意味着开发人员将编写单元测试框架代码本身。 相反，EF4 的可测试性目标更轻松创建可测试框架的基础上构建的代码。 我们查看具体示例之前，因此值得了解可测试代码的质量。

### <a name="the-qualities-of-testable-code"></a>可测试代码的质量

可以轻松测试的代码将始终具有至少两个对特性。 首先，可测试代码很容易**观察**。 给定输入某些组，它应该很容易观察代码的输出。 例如，测试下面的方法非常简单的因为该方法直接返回的计算结果。

``` csharp
    public int Add(int x, int y) {
        return x + y;
    }
```

测试一种方法是很困难，如果该方法将计算出的值写入到网络套接字、 数据库表或类似下面的代码文件。 测试必须执行额外的工作，检索的值。

``` csharp
    public void AddAndSaveToFile(int x, int y) {
         var results = string.Format("The answer is {0}", x + y);
         File.WriteAllText("results.txt", results);
    }
```

其次，可测试代码很容易**隔离**。 让我们使用下面的伪代码作为可测试代码的错误示例。

``` csharp
    public int ComputePolicyValue(InsurancePolicy policy) {
        using (var connection = new SqlConnection("dbConnection"))
        using (var command = new SqlCommand(query, connection)) {

            // business calculations omitted ...               

            if (totalValue > notificationThreshold) {
                var message = new MailMessage();
                message.Subject = "Warning!";
                var client = new SmtpClient();
                client.Send(message);
            }
        }
        return totalValue;
    }
```

该方法很容易注意 – 我们可以传入保险政策并验证返回值与预期的结果相匹配。 但是，若要测试的方法我们将需要使用正确的架构中，安装数据库和配置 SMTP 服务器，如果该方法尝试将发送一封电子邮件。

单元测试仅想要验证在方法中，计算逻辑，但测试可能会失败，因为电子邮件服务器处于脱机状态，或数据库服务器移动的。 这些故障的这两个测试要验证的行为与不相关。 行为是难以隔离。

软件开发人员努力编写可测试的代码通常致力于保持的它们编写代码中的关注点分离。 上述方法应专注于业务计算并委托给其他组件的数据库和电子邮件实现详细信息。 Robert C.Martin 调用此单一责任原则。 对象应封装单个、 有限的责任，如计算策略的值。 所有其他数据库和通知的工作应该是其他某个对象的责任。 以这种方式编写的代码很容易隔离因为它侧重于单个任务。

在.NET 中我们需要遵循单一责任原则，并实现隔离的抽象。 我们可以使用接口定义并强制代码以使用接口抽象，而不是具体的类型。 在本文中我们将看到如何像上面的错误示例可以使用的方法的接口*查找*像它们将与数据库进行通信。 在测试时，但是，我们可以替换虚拟实现，不会与数据库通信，但改为在内存中保存数据。 此虚拟实现将隔离不相关的问题中的数据访问代码或数据库配置中的代码。

还有其他优点到隔离。 中的最后一个方法的商业计算应只需几毫秒，若要执行，但测试本身可能会运行几秒钟作为代码周围的网络和访谈的跳转到各种服务器。 运行单元测试应快速以便于较小的更改。 单元测试应也是可重复和不失败，因为测试与无关的组件有问题。 编写易于观察和隔离的代码意味着，开发人员需要更轻松地编写代码的测试，减少测试执行，正在等待的时间和更多重要的是，花费更少时间来跟踪不存在的 bug。

希望你可以有哪些优势测试并了解可测试代码表现出的质量。 我们正准备解决如何编写代码，它使用 EF4 若要将数据保存到数据库，同时保持可观察和可以轻松隔离，但首先我们将缩小范围关注讨论数据访问的可测试的设计。

## <a name="design-patterns-for-data-persistence"></a>数据暂留的设计模式

两个错误示例前面提供了太多责任。 第一个错误示例必须执行一个计算*和*写入文件。 第二个错误示例不得不从数据库中读取数据*并*执行业务计算*和*发送电子邮件。 通过设计的分离关注点和责任委托给其他组件的较小方法将使向编写可测试代码的巨大进步。 目标是生成通过编写从小且集中的抽象操作的功能。

当它涉及数据持久性较小，我们正在寻找的已设定焦点的抽象是很少它们已记录了作为设计模式。 Martin Fowler 书模式的企业应用程序体系结构已介绍了这些模式中打印的第一个工作。 之前我们介绍这些 ADO.NET 实体框架如何实现和使用这些模式，我们将提供以下各节中的这些模式的简短说明。

### <a name="the-repository-pattern"></a>存储库模式

Fowler 说，一个存储库"之间进行调停的域和数据访问域对象使用类似于集合的接口映射层"。 存储库模式的目标是找出代码中的数据访问的繁琐细节上，正如我们看到的更早的隔离可测试性所需的特征。

隔离的关键是如何存储库中公开使用类似于集合的接口的对象。 为使用存储库并不知道如何编写的逻辑存储库中将具体化你请求的对象。 存储库可能会联系到数据库，或者它可能只是返回从内存中集合的对象。 你的代码需要知道的就是存储库中显示维护集合，并且可以检索、 添加和从集合中删除对象。

在现有.NET 应用程序中的具体的存储库通常继承自泛型接口如下所示：

``` csharp
    public interface IRepository<T> {       
        IEnumerable<T> FindAll();
        IEnumerable<T> FindBy(Expression<Func\<T, bool>> predicate);
        T FindById(int id);
        void Add(T newEntity);
        void Remove(T entity);
    }
```

当我们为 ef4 仍提供实现，但基本概念保持不变时，我们将进行少量更改的接口定义。 代码可以使用实现此接口的具体存储库来通过其主键值，若要检索的实体基于谓词的计算结果集合中检索实体，或只需检索所有可用的实体。 该代码还可以添加和删除通过存储库接口的实体。

给定的员工 IRepository 对象，代码可以执行以下操作。

``` csharp
    var employeesNamedScott =
        repository
            .FindBy(e => e.Name == "Scott")
            .OrderBy(e => e.HireDate);
    var firstEmployee = repository.FindById(1);
    var newEmployee = new Employee() {/*... */};
    repository.Add(newEmployee);
```

因为代码使用接口 （IRepository 的员工），我们可以具有不同的接口的实现提供的代码。 一种实现可能是由 EF4 和保持对象到 Microsoft SQL Server 数据库实现。 （一个我们在测试期间使用） 的不同实现可能受内存中的员工列表对象。 该接口将有助于实现代码中的隔离。

请注意 IRepository&lt;T&gt;接口不会公开保存操作。 我们如何更新现有对象？ 您可能会遇到 IRepository 定义包含保存操作，这些存储库的实现需要立即将对象保存到数据库。 但是，许多应用程序中我们不希望单独保存对象。 相反，我们想要将对象，可能是从不同的存储库，修改一部分的业务活动，这些对象并稍后作为一个原子操作的一部分保存的所有对象。 幸运的是，是一种模式以允许这种行为。

### <a name="the-unit-of-work-pattern"></a>工作单元模式

Fowler 说的工作单元将"维护的对象列表受业务事务和协调弃用更改写入和并发问题的解决方法"。 它负责的工作要跟踪的对象，我们将从存储库并将我们对这些对象时的工作单元提交所做的更改，我们告诉任何更改持久保存的更改的单元。 它也是工作的要提取的新对象，我们已添加到所有存储库，并将对象插入到数据库，以及管理删除单元的责任。

如果你曾经使用 ADO.NET 数据集做过任何工作然后您将已经熟悉的工作模式单元。 ADO.NET 数据集必须能够跟踪我们的更新、 删除和插入的 DataRow 对象，可以 （使用 TableAdapter 的帮助） 协调所有我们更改到的数据库。 但是，数据集对象的模型已断开连接基础数据库的子集。 工作模式单元表现出相同的行为，但可处理业务对象，并独立于数据访问代码和不识别实例的数据库的域对象。

一种抽象模型中的.NET 代码的工作单元可能如下所示：

``` csharp
    public interface IUnitOfWork {
        IRepository<Employee> Employees { get; }
        IRepository<Order> Orders { get; }
        IRepository<Customer> Customers { get; }
        void Commit();
    }
```

通过公开来自我们可以确保单个工作单元的工作单元的存储库引用对象具有跟踪业务事务期间实例化的所有实体的功能。 实际单元提交方法的实现是工作的所有奇迹的地方来协调与数据库的内存中的更改。 

给定 IUnitOfWork 引用，代码可以检索到的业务对象从一个或多个存储库进行更改并保存使用原子提交操作的所有更改。

``` csharp
    var firstEmployee = unitofWork.Employees.FindById(1);
    var firstCustomer = unitofWork.Customers.FindById(1);
    firstEmployee.Name = "Alex";
    firstCustomer.Name = "Christopher";
    unitofWork.Commit();
```

### <a name="the-lazy-load-pattern"></a>延迟加载模式

Fowler 使用名称延迟加载来描述"一个对象不包含的所有数据，您需要但知道如何获取它"。 透明延迟加载是重要的功能时编写可测试性的业务代码和使用关系数据库。 作为示例，请考虑下面的代码。

``` csharp
    var employee = repository.FindById(id);
    // ... and later ...
    foreach(var timeCard in employee.TimeCards) {
        // .. manipulate the timeCard
    }
```

如何填充时间卡片集合？ 有两个可能的答案。 一个答案是员工存储库，当系统询问提取一名员工，发出一个查询来检索员工以及员工的相关联的考勤卡信息。 这通常需要具有对 JOIN 子句的查询，并可能会导致比应用程序的详细信息中检索的关系数据库中需要。 如果应用程序永远不需要接触时间卡片属性？

第二个答案是加载时间卡片属性"按需"。 此延迟加载是隐式和透明的业务逻辑，因为该代码不会调用特殊的 Api 来检索考勤卡信息。 该代码假定在需要时存在考勤卡信息时。 所涉及的通常涉及到运行时拦截的方法调用的延迟加载是一些奇妙的功能。 截取代码负责和数据库通信并检索考勤卡信息而使可用来为业务逻辑的业务逻辑。 这种延迟加载神奇允许从数据检索操作，并生成更多可测试代码中隔离本身业务代码。

延迟加载的缺点是，当应用程序*does*需要该代码将执行一个额外查询的考勤卡信息。 这不是问题的许多应用程序，但对于性能敏感型应用程序或应用程序循环遍历多个员工的对象并执行查询以在每次迭代循环 （问题通常称为 N + 1 的过程中检索时间卡片查询问题），延迟加载是拖动。 在这些情况下应用程序可能想要预先加载考勤卡信息可能最高效的方式。

幸运的是，我们将了解如何 EF4 支持这两个隐式延迟加载和高效的预先加载，因为我们将移动到下一节，并实现这些模式。

## <a name="implementing-patterns-with-the-entity-framework"></a>使用实体框架的实现模式

好消息是所有我们在上一部分中所述的设计模式是直接使用 EF4 时实现。 为了演示，我们将要使用一个简单的 ASP.NET MVC 应用程序来编辑和显示员工和其关联的考勤卡信息。 我们首先使用以下"plain old CLR objects"(Poco)。 

``` csharp
    public class Employee {
        public int Id { get; set; }
        public string Name { get; set; }
        public DateTime HireDate { get; set; }
        public ICollection<TimeCard> TimeCards { get; set; }
    }

    public class TimeCard {
        public int Id { get; set; }
        public int Hours { get; set; }
        public DateTime EffectiveDate { get; set; }
    }
```

我们将介绍不同的方法和功能的 EF4，但其目的是为持久性未知 (PI) 尽可能保留以下类，这些类定义将略有变化。 PI 对象并不知道*如何*，或甚至*如果*，它包含的状态存在于数据库内。 PI 和 Poco 来了解一下通过可测试软件。 使用 POCO 方法的对象是不太受约束、 更灵活和易于测试，因为这些服务器可以运行不含数据库存在。

使用就地 Poco 我们可以创建实体数据模型 (EDM) 在 Visual Studio 中 （请参阅图 1）。 我们不会使用 EDM 来为实体生成代码。 相反，我们想要使用我们 lovingly 手动创建的实体。 我们仅将使用 EDM 生成数据库架构，并提供 EF4 需要将对象映射到数据库的元数据。

![ef test_01](~/ef6/media/eftest-01.jpg)

**图 1**

注意： 如果你想要首先开发的 EDM 模型，就可以以干净的生成 POCO 代码从 EDM。 可以使用数据可编程性团队提供的 Visual Studio 2010 扩展来执行此操作。 若要下载该扩展，启动 Visual Studio 中的工具菜单中的扩展管理器，并搜索"POCO"（请参阅图 2） 的模板的联机库。 有几个 POCO 模板可用于 EF。 有关使用模板的详细信息，请参阅"[演练： POCO 实体框架的模板](https://blogs.msdn.com/adonet/pages/walkthrough-poco-template-for-the-entity-framework.aspx)"。

![ef test_02](~/ef6/media/eftest-02.png)

**图 2**

从起始点此 POCO 会探讨两个不同的方法可测试的代码。 第一种方法，我调用 EF 方法因为它充分利用从 Entity Framework API 来实现工作和存储库的单位的抽象。 在第二种方法中我们将创建我们自己自定义存储库的抽象，并查看每种方法的优缺点。 我们将开始通过探索 EF 方法。  

### <a name="an-ef-centric-implementation"></a>EF 为中心实现

请考虑以下控制器操作从 ASP.NET MVC 项目。 操作检索 Employee 对象并返回要显示的员工的详细的视图的结果。

``` csharp
    public ViewResult Details(int id) {
        var employee = _unitOfWork.Employees
                                  .Single(e => e.Id == id);
        return View(employee);
    }
```

是可测试的代码？ 有至少两个测试，我们需要验证操作的行为。 首先，我们想要验证操作会返回正确的视图中 – 轻松进行测试。 我们还想要编写一个测试，以验证操作中检索正确的员工，以及我们想要执行此操作无需执行代码以查询数据库。 请记住我们想要隔离受测代码。 隔离可确保测试不会因数据访问代码或数据库配置中的 bug 而失败。 如果测试失败，我们将知道我们有一个 bug，在控制器逻辑，而不是在一些较低级别系统组件。

若要实现的隔离，我们将存储库和工作单元的前面需要一些抽象，例如，我们演示了接口。 请记住存储库模式的设计域对象和数据映射层之间进行协调。 在此方案中 EF4*是*映射的数据层，并已经提供了一个名为 IObjectSet 的类似于存储库的抽象&lt;T&gt; （从 System.Data.Objects 命名空间）。 接口定义如下所示。

``` csharp
    public interface IObjectSet<TEntity> :
                     IQueryable<TEntity>,
                     IEnumerable<TEntity>,
                     IQueryable,
                     IEnumerable
                     where TEntity : class
    {
        void AddObject(TEntity entity);
        void Attach(TEntity entity);
        void DeleteObject(TEntity entity);
        void Detach(TEntity entity);
    }
```

IObjectSet&lt;T&gt;符合要求的存储库，因为它类似于对象的集合 (通过 IEnumerable&lt;T&gt;) 并提供方法来添加和删除对象从模拟的集合。 附加和分离方法公开 EF4 API 的其他的功能。 若要使用 IObjectSet&lt;T&gt;我们需要工作抽象单位以将存储库绑定在一起作为存储库的接口。

``` csharp
    public interface IUnitOfWork {
        IObjectSet<Employee> Employees { get; }
        IObjectSet<TimeCard> TimeCards { get; }
        void Commit();
    }
```

此接口的一个具体实现将与 SQL Server 进行通信，并可以轻松地创建使用从 EF4 的 ObjectContext 类。 ObjectContext 类是工作的真正的 EF4 API 中单元。

``` csharp
    public class SqlUnitOfWork : IUnitOfWork {
        public SqlUnitOfWork() {
            var connectionString =
                ConfigurationManager
                    .ConnectionStrings[ConnectionStringName]
                    .ConnectionString;
            _context = new ObjectContext(connectionString);
        }

        public IObjectSet<Employee> Employees {
            get { return _context.CreateObjectSet<Employee>(); }
        }

        public IObjectSet<TimeCard> TimeCards {
            get { return _context.CreateObjectSet<TimeCard>(); }
        }

        public void Commit() {
            _context.SaveChanges();
        }

        readonly ObjectContext _context;
        const string ConnectionStringName = "EmployeeDataModelContainer";
    }
```

将 IObjectSet&lt;T&gt;生动非常简单，只调用 CreateObjectSet ObjectContext 对象的方法。 在后台，框架将使用元数据，我们提供在 EDM 中生成具体 ObjectSet&lt;T&gt;。 我们只讨论与返回 IObjectSet&lt;T&gt;接口，因为它有助于保持客户端代码中的可测试性。

此具体实现可在生产中，但我们需要关注如何我们将使用我们 IUnitOfWork 抽象以便进行测试。

### <a name="the-test-doubles"></a>Test Double

若要隔离的控制器操作我们将需要在实际工作单元中 （由 ObjectContext） 和一个测试双精度或"假"工作单元 （执行内存中操作） 之间进行切换的功能。 若要执行此类型的切换的常用方法是工作的工作的不让 MVC 控制器实例单元，而是工作的工作的化插入控制器作为构造函数参数传递单元。

``` csharp
    class EmployeeController : Controller {
      publicEmployeeController(IUnitOfWork unitOfWork)  {
          _unitOfWork = unitOfWork;
      }
      ...
    }
```

上面的代码是依赖关系注入的示例。 我们不允许创建它的依赖项 （工作单元），但将依赖项注入到控制器的控制器。 在 MVC 项目中是通常使用自定义控制器工厂结合使用控制反转 (IoC) 容器来自动执行依赖关系注入。 这些主题不在本文中，范围，但可以在本文末尾的引用以下更通过读取。

我们可用于测试的工作实现一个虚设单元可能如下所示。

``` csharp
    public class InMemoryUnitOfWork : IUnitOfWork {
        public InMemoryUnitOfWork() {
            Committed = false;
        }
        public IObjectSet<Employee> Employees {
            get;
            set;
        }

        public IObjectSet<TimeCard> TimeCards {
            get;
            set;
        }

        public bool Committed { get; set; }
        public void Commit() {
            Committed = true;
        }
    }
```

请注意虚设的工作单元公开提交属性。 有时它可用于将功能添加到便于测试虚设类。 在这种情况下很容易注意是否代码通过检查已提交属性提交的工作单元。

我们还需要假 IObjectSet&lt;T&gt;在内存中保留对象的员工和时间卡片。 我们可以提供一个实现代码使用泛型。

``` csharp
    public class InMemoryObjectSet<T> : IObjectSet<T> where T : class
        public InMemoryObjectSet()
            : this(Enumerable.Empty<T>()) {
        }
        public InMemoryObjectSet(IEnumerable<T> entities) {
            _set = new HashSet<T>();
            foreach (var entity in entities) {
                _set.Add(entity);
            }
            _queryableSet = _set.AsQueryable();
        }
        public void AddObject(T entity) {
            _set.Add(entity);
        }
        public void Attach(T entity) {
            _set.Add(entity);
        }
        public void DeleteObject(T entity) {
            _set.Remove(entity);
        }
        public void Detach(T entity) {
            _set.Remove(entity);
        }
        public Type ElementType {
            get { return _queryableSet.ElementType; }
        }
        public Expression Expression {
            get { return _queryableSet.Expression; }
        }
        public IQueryProvider Provider {
            get { return _queryableSet.Provider; }
        }
        public IEnumerator<T> GetEnumerator() {
            return _set.GetEnumerator();
        }
        IEnumerator IEnumerable.GetEnumerator() {
            return GetEnumerator();
        }

        readonly HashSet<T> _set;
        readonly IQueryable<T> _queryableSet;
    }
```

这个 test double 委托其大部分的工作为基础的 HashSet&lt;T&gt;对象。 请注意该 IObjectSet&lt;T&gt;需要作为类 （引用类型），强制 T 的泛型约束，并且还会强制实现 IQueryable&lt;T&gt;。 很容易地使内存中集合显示为 IQueryable&lt;T&gt;使用标准 LINQ 运算符 AsQueryable。

### <a name="the-tests"></a>测试

传统单元测试将使用单个测试类在一个 MVC 控制器中保存的所有测试的所有操作。 我们可以编写这些测试或任何类型的单元测试时，使用内存中 fakes 我们构建了。 但是，本文中我们将避免将单一式应用测试类的方式并改为组我们的测试可以专注于特定的一项功能。  例如，"创建新员工"可能是我们要测试，因此我们将使用单个测试类若要验证单个控制器操作负责创建新的员工的功能。

没有需要所有这些能够进行细化管理的测试类的一些常见设置代码。 例如，我们始终需要创建我们的内存中存储库和虚设工作单元。 我们还需要使用虚设的注入的工作单元的员工控制器的实例。 通过使用基类，我们将在测试类之间分享此常见设置代码。

``` csharp
    public class EmployeeControllerTestBase {
        public EmployeeControllerTestBase() {
            _employeeData = EmployeeObjectMother.CreateEmployees()
                                                .ToList();
            _repository = new InMemoryObjectSet<Employee>(_employeeData);
            _unitOfWork = new InMemoryUnitOfWork();
            _unitOfWork.Employees = _repository;
            _controller = new EmployeeController(_unitOfWork);
        }

        protected IList<Employee> _employeeData;
        protected EmployeeController _controller;
        protected InMemoryObjectSet<Employee> _repository;
        protected InMemoryUnitOfWork _unitOfWork;
    }
```

我们在基类中使用"对象母亲"是用于创建测试数据的一种常见模式。 对象的母亲包含实例使用的测试实体化跨多个测试装置的工厂方法。

``` csharp
    public static class EmployeeObjectMother {
        public static IEnumerable<Employee> CreateEmployees() {
            yield return new Employee() {
                Id = 1, Name = "Scott", HireDate=new DateTime(2002, 1, 1)
            };
            yield return new Employee() {
                Id = 2, Name = "Poonam", HireDate=new DateTime(2001, 1, 1)
            };
            yield return new Employee() {
                Id = 3, Name = "Simon", HireDate=new DateTime(2008, 1, 1)
            };
        }
        // ... more fake data for different scenarios
    }
```

我们可以使用 EmployeeControllerTestBase 作为基类的几个测试装置 （见图 3）。 每个测试固定例程将测试的特定控制器操作。 例如，一个测试固定例程将重点介绍测试期间 HTTP GET 请求 （若要显示用于创建员工视图），使用创建操作和不同装置将重点介绍使用 HTTP POST 请求中的创建操作 (才能提交的信息要为用户创建员工）。 每个派生的类只是负责在其特定上下文中，并提供验证其特定的测试上下文的结果所需的断言所需的设置。

![ef test_03](~/ef6/media/eftest-03.png)

**图 3**

此处提供的命名约定和测试样式不是必需的可测试代码，它是一种方法。 图 4 显示了在 Jet 大脑 Resharper 中运行的测试用于 Visual Studio 2010 的测试运行程序插件。

![ef test_04](~/ef6/media/eftest-04.png)

**图 4**

有了基类来处理共享的设置代码，针对每个控制器操作的单元测试是尽可能小并且易于编写。 测试将会执行快速 （因为我们在执行内存中操作），并且不应因不相关的基础结构或环境问题而失败，（因为我们已独立测试的单元）。

``` csharp
    [TestClass]
    public class EmployeeControllerCreateActionPostTests
               : EmployeeControllerTestBase {
        [TestMethod]
        public void ShouldAddNewEmployeeToRepository() {
            _controller.Create(_newEmployee);
            Assert.IsTrue(_repository.Contains(_newEmployee));
        }
        [TestMethod]
        public void ShouldCommitUnitOfWork() {
            _controller.Create(_newEmployee);
            Assert.IsTrue(_unitOfWork.Committed);
        }
        // ... more tests

        Employee _newEmployee = new Employee() {
            Name = "NEW EMPLOYEE",
            HireDate = new System.DateTime(2010, 1, 1)
        };
    }
```

在这些测试中的基类执行大部分设置工作。 请记住基类构造函数创建内存中存储库、 虚设的工作，单元和 EmployeeController 类的实例。 测试类从此基类派生，而侧重于测试的 Create 方法的详细信息。 在这种情况下细节上看，这会看到任何单元测试过程中的"排列，操作和断言"步骤：

-   创建 newEmployee 对象，用于模拟传入的数据。
-   调用 EmployeeController 的 Create 操作，并传入 newEmployee。
-   验证创建操作产生预期的结果 （存储库中显示员工）。

我们已构建，我们可以测试任何 EmployeeController 操作。 例如，当我们编写测试员工控制器的 Index 操作时我们可以继承测试基类来建立我们的测试中的相同基本设置。 再次类的基类将创建内存中存储库、 虚设的工作单元，以及 EmployeeController 的一个实例。 索引操作的测试只需专注于调用 Index 操作并测试操作的模型的质量返回。

``` csharp
    [TestClass]
    public class EmployeeControllerIndexActionTests
               : EmployeeControllerTestBase {
        [TestMethod]
        public void ShouldBuildModelWithAllEmployees() {
            var result = _controller.Index();
            var model = result.ViewData.Model
                          as IEnumerable<Employee>;
            Assert.IsTrue(model.Count() == _employeeData.Count);
        }
        [TestMethod]
        public void ShouldOrderModelByHiredateAscending() {
            var result = _controller.Index();
            var model = result.ViewData.Model
                         as IEnumerable<Employee>;
            Assert.IsTrue(model.SequenceEqual(
                           _employeeData.OrderBy(e => e.HireDate)));
        }
        // ...
    }
```

我们将使用内存中 fakes 创建的测试都是面向测试*状态*的软件。 例如，测试我们想要检查的存储库的状态后，将执行创建操作 – 创建操作时没有存储库中保存新员工？

``` csharp
    [TestMethod]
    public void ShouldAddNewEmployeeToRepository() {
        _controller.Create(_newEmployee);
        Assert.IsTrue(_repository.Contains(_newEmployee));
    }
```

稍后我们将介绍基于交互测试。 如果待测试的代码调用我们的对象上的适当方法并传递正确的参数，将要求交互基于测试。 现在我们来看看在封面上另一种设计模式 – 延迟加载。

## <a name="eager-loading-and-lazy-loading"></a>预先加载和延迟加载

在某个时间点在 ASP.NET MVC web 应用程序，我们可能想要显示员工的信息，包括员工的关联时间卡片。 例如，我们可能会有考勤卡摘要显示在系统中显示该员工的姓名和总时间卡片。 有几种方法，我们可以实现此功能。

### <a name="projection"></a>投影

一个简单的方法来创建摘要是以构造模型专用于我们想要在视图中显示的信息。 在此方案中模型可能如下所示。

``` csharp
    public class EmployeeSummaryViewModel {
        public string Name { get; set; }
        public int TotalTimeCards { get; set; }
    }
```

请注意，EmployeeSummaryViewModel 不是一个实体即它不是我们想要保留在数据库中的内容。 我们只会使用此类数据进行随机排列到视图中以强类型化方式。 视图模型是将数据传输对象 (DTO)，因为它不包含的任何行为 （无方法） – 仅属性等。 属性将保存，需要移动数据。 很容易地实例化此视图模型，使用 LINQ 的标准投影运算符 – 选择运算符。

``` csharp
    public ViewResult Summary(int id) {
        var model = _unitOfWork.Employees
                               .Where(e => e.Id == id)
                               .Select(e => new EmployeeSummaryViewModel
                                  {
                                    Name = e.Name,
                                    TotalTimeCards = e.TimeCards.Count()
                                  })
                               .Single();
        return View(model);
    }
```

有两个重要功能上面的代码。 第一次 – 代码很容易地测试，因为仍然很容易观察和隔离。 Select 运算符对我们的内存中 fakes 有效也很好用，针对实际的工作单元一样。

``` csharp
    [TestClass]
    public class EmployeeControllerSummaryActionTests
               : EmployeeControllerTestBase {
        [TestMethod]
        public void ShouldBuildModelWithCorrectEmployeeSummary() {
            var id = 1;
            var result = _controller.Summary(id);
            var model = result.ViewData.Model as EmployeeSummaryViewModel;
            Assert.IsTrue(model.TotalTimeCards == 3);
        }
        // ...
    }
```

第二个值得注意的功能是代码如何允许 EF4 生成单一、 高效的查询来一起组合员工和考勤卡信息。 我们已加载员工信息和考勤卡信息到同一个对象而无需使用任何特殊的 Api。 该代码只被表示它需要使用标准 LINQ 运算符的工作内存中数据源以及为远程数据源的信息。 EF4 无法转换表达式树由 LINQ 查询和 C 生成\#编译器转换为单个和高效的 T-SQL 查询。

``` SQL
    SELECT
    [Limit1].[Id] AS [Id],
    [Limit1].[Name] AS [Name],
    [Limit1].[C1] AS [C1]
    FROM (SELECT TOP (2)
      [Project1].[Id] AS [Id],
      [Project1].[Name] AS [Name],
      [Project1].[C1] AS [C1]
      FROM (SELECT
        [Extent1].[Id] AS [Id],
        [Extent1].[Name] AS [Name],
        (SELECT COUNT(1) AS [A1]
         FROM [dbo].[TimeCards] AS [Extent2]
         WHERE [Extent1].[Id] =
               [Extent2].[EmployeeTimeCard_TimeCard_Id]) AS [C1]
              FROM [dbo].[Employees] AS [Extent1]
               WHERE [Extent1].[Id] = @p__linq__0
         )  AS [Project1]
    )  AS [Limit1]
```

有些的时候时我们不想要使用的视图模型或 DTO 对象，但真正的实体。 当我们知道我们需要某位员工*和*员工的时间卡片，我们可以自愿加载相关的数据中有非介入式且高效的方式。

### <a name="explicit-eager-loading"></a>显式预先加载

如果我们想要预先加载相关的实体信息的业务逻辑 （或在此方案中，控制器操作逻辑），我们需要某种途径来表达其需要向存储库。 EF4 ObjectQuery&lt;T&gt;类定义包含方法以指定要在查询过程中检索的相关的对象。 请记住 EF4 ObjectContext 公开实体通过具体的 ObjectSet&lt;T&gt;类继承自 ObjectQuery&lt;T&gt;。  如果我们使用 ObjectSet&lt;T&gt;中我们可以编写以下代码到我们控制器操作的引用指定每个雇员的考勤卡信息的预先加载。

``` csharp
    _employees.Include("TimeCards")
              .Where(e => e.HireDate.Year > 2009);
```

但是，由于我们正在尝试保持我们的代码可测试性我们不公开 ObjectSet&lt;T&gt;从外部工作类的实际单元。 相反，我们依赖 IObjectSet&lt;T&gt;接口的虚设，更容易，但 IObjectSet&lt;T&gt;不会定义 Include 方法。 LINQ 的优点是，我们可以创建我们自己 Include 运算符。

``` csharp
    public static class QueryableExtensions {
        public static IQueryable<T> Include<T>
                (this IQueryable<T> sequence, string path) {
            var objectQuery = sequence as ObjectQuery<T>;
            if(objectQuery != null)
            {
                return objectQuery.Include(path);
            }
            return sequence;
        }
    }
```

请注意，此 Include 运算符定义为扩展方法的 IQueryable&lt;T&gt;而不是 IObjectSet&lt;T&gt;。 这使我们能够使用方法和广泛的可能的类型，包括 IQueryable&lt;T&gt;，IObjectSet&lt;T&gt;，ObjectQuery&lt;T&gt;，和 ObjectSet&lt;T&gt;。 如果基础序列不是正版 EF4 ObjectQuery&lt;T&gt;，则没有完成任何损害，Include 运算符不执行任何操作。 如果基础序列*是*ObjectQuery&lt;T&gt; (或派生自 ObjectQuery&lt;T&gt;)，EF4 会请参阅我们的其他数据的要求并制定正确的 SQL查询。

此位置中的新运算符与我们可以显式请求考勤卡信息的预先加载从存储库。

``` csharp
    public ViewResult Index() {
        var model = _unitOfWork.Employees
                               .Include("TimeCards")
                               .OrderBy(e => e.HireDate);
        return View(model);
    }
```

针对实际 ObjectContext 运行，则代码将产生以下单个查询。 该查询从数据库中一次是将 employee 对象具体化和完全填充其时间卡片属性收集足够的信息。

``` SQL
    SELECT
    [Project1].[Id] AS [Id],
    [Project1].[Name] AS [Name],
    [Project1].[HireDate] AS [HireDate],
    [Project1].[C1] AS [C1],
    [Project1].[Id1] AS [Id1],
    [Project1].[Hours] AS [Hours],
    [Project1].[EffectiveDate] AS [EffectiveDate],
    [Project1].[EmployeeTimeCard_TimeCard_Id] AS [EmployeeTimeCard_TimeCard_Id]
    FROM ( SELECT
         [Extent1].[Id] AS [Id],
         [Extent1].[Name] AS [Name],
         [Extent1].[HireDate] AS [HireDate],
         [Extent2].[Id] AS [Id1],
         [Extent2].[Hours] AS [Hours],
         [Extent2].[EffectiveDate] AS [EffectiveDate],
         [Extent2].[EmployeeTimeCard_TimeCard_Id] AS
                    [EmployeeTimeCard_TimeCard_Id],
         CASE WHEN ([Extent2].[Id] IS NULL) THEN CAST(NULL AS int)
         ELSE 1 END AS [C1]
         FROM  [dbo].[Employees] AS [Extent1]
         LEFT OUTER JOIN [dbo].[TimeCards] AS [Extent2] ON [Extent1].[Id] = [Extent2].[EmployeeTimeCard_TimeCard_Id]
    )  AS [Project1]
    ORDER BY [Project1].[HireDate] ASC,
             [Project1].[Id] ASC, [Project1].[C1] ASC
```

好消息是在操作方法中的代码仍可完全测试。 我们不需要提供我们 fakes 以支持，Include 运算符的任何其他功能。 不幸的是我们需要使用我们想要保留持久性未知的代码内，Include 运算符。 这是类型的你将需要评估构建可测试代码时的权衡取舍的一个典型示例。 有些的时候您需要让外部存储库抽象以满足性能目标的持久性问题泄漏。

预先加载的替代方法是延迟加载。 延迟加载意味着我们*不*需要我们业务的代码以显式宣布关联的数据的要求。 相反，我们在应用程序中使用我们的实体和其他数据是否需要实体框架会按需将数据加载。

### <a name="lazy-loading"></a>延迟加载

它很容易想象我们不知道数据的业务逻辑将需要一个场景。 我们可能会知道逻辑需要 employee 对象，但我们可能会转换为不同的执行路径，其中某些这些路径需要从该员工的考勤卡信息而有些则不进行分支。 类似上述的情形是隐式延迟加载，因为在按需奇迹般地显示数据的最佳选择。

在我们的实体对象上，延迟加载，也称为延迟加载，does 放置一些要求。 使用 true 持久性无感知的 Poco 不将会面临暂留层中，从任何要求，但 true 持久性无感知几乎不可能实现。  而是我们会度量持久化透明的相对度。 如果我们需要从持久性面向基类继承，或使用专用的集合来实现延迟加载在 Poco，它会令人遗憾。 幸运的是，EF4 具有较少受侵入的解决方案。

### <a name="virtually-undetectable"></a>几乎不可检测

在使用 POCO 对象时，EF4 可以动态生成实体的运行时的代理。 这些代理不可见的方式包装实例化的 Poco 和提供通过截获每个属性的其他服务获取和设置操作以执行额外工作。 一个此类服务就是我们所期望的延迟加载功能。 另一个服务是有效的更改跟踪机制可以记录程序更改实体的属性值时。 更改的列表使用将 objectcontext 在 SaveChanges 方法期间保留任何已修改的实体使用 UPDATE 命令。

这些代理才能起作用，但是，他们需要一种方法要挂钩到属性 get 和实体，并在代理上的设置操作通过重写虚拟成员实现此目标。 因此，如果我们想要具有隐式延迟加载和有效的更改跟踪我们需要返回到我们的 POCO 类定义并将属性标记为虚拟。

``` csharp
    public class Employee {
        public virtual int Id { get; set; }
        public virtual string Name { get; set; }
        public virtual DateTime HireDate { get; set; }
        public virtual ICollection<TimeCard> TimeCards { get; set; }
    }
```

我们仍可以说雇员实体是主要是持久性未知。 唯一要求是使用虚拟成员，这不会影响代码的可测试性。 我们不需要从任何特殊的基类派生，或甚至使用专用于延迟加载的特殊集合。 如代码所示，任何实现 ICollection 的类&lt;T&gt;可用来保存相关的实体。

此外，还有一个细微的改动，我们需要使我们的工作单元内。 延迟加载是*关闭*默认情况下，当直接使用 ObjectContext 对象。 没有一个我们可以设置的属性 ContextOptions 属性上以启用延迟加载，并且我们可以在我们真正的工作单元内设置此属性，如果我们想要启用延迟加载所有位置。

``` csharp
    public class SqlUnitOfWork : IUnitOfWork {
         public SqlUnitOfWork() {
             // ...
             _context = new ObjectContext(connectionString);
             _context.ContextOptions.LazyLoadingEnabled = true;
         }    
         // ...
     }
```

使用隐式启用延迟加载后，应用程序代码可以使用员工和员工的关联时间卡片，剩余根本不用考虑 EF 将额外的数据加载所需的工作时。

``` csharp
    var employee = _unitOfWork.Employees
                              .Single(e => e.Id == id);
    foreach (var card in employee.TimeCards) {
        // ...
    }
```

延迟加载可以使应用程序代码容易编写，并与代理神奇代码保持完全可测试。 内存中的工作单元的 fakes 只可以预加载关联的数据时需要在测试期间具有伪实体。

此时我们将我们把注意力从构建存储库使用 IObjectSet&lt;T&gt;并查看的抽象，隐藏的持久性框架的所有符号。

## <a name="custom-repositories"></a>自定义存储库

当我们首先会看到工作设计模式的单元在本文中我们提供一些示例代码的工作单元可能如下所示。 让我们重新显示使用中的雇员和员工考勤卡方案，我们一直使用此最初的思路。

``` csharp
    public interface IUnitOfWork {
        IRepository<Employee> Employees { get; }
        IRepository<TimeCard> TimeCards { get;  }
        void Commit();
    }
```

此工作单位的值和我们在上一部分中创建的工作单位的主要区别是如何工作此单元不使用 EF4 框架中的任何抽象概念 (没有任何 IObjectSet&lt;T&gt;)。 IObjectSet&lt;T&gt;适用于的存储库接口，但它公开的 API 可能不与我们的应用程序的需求而完全对齐。 在此方法中即将推出，我们将表示存储库使用自定义 IRepository&lt;T&gt;抽象。

采用测试驱动的设计、 行为驱动设计和域驱动的方法设计的许多开发人员更喜欢 IRepository&lt;T&gt;方法的几个原因。 首先，IRepository&lt;T&gt;接口表示"反损坏"层。 中所述由 Eric Evans 著作域驱动设计防损层可保持基础结构 Api，例如持久性 API 离开域代码。 其次，开发人员可以构建到满足应用程序的确切需求 （如发现编写测试时） 的存储库的方法。 例如，我们可能经常需要查找单个实体使用的 ID 值，因此我们可以将一个 FindById 方法添加到存储库接口。  我们 IRepository&lt;T&gt;定义将如下所示。

``` csharp
    public interface IRepository<T>
                    where T : class, IEntity {
        IQueryable<T> FindAll();
        IQueryable<T> FindWhere(Expression<Func\<T, bool>> predicate);
        T FindById(int id);       
        void Add(T newEntity);
        void Remove(T entity);
    }
```

请注意，我们将使用 IQueryable 重新放置&lt;T&gt;接口，以公开实体集合。 IQueryable&lt;T&gt;允许 LINQ 表达式树排入 EF4 提供程序，并为该提供程序提供查询的整体视图。 第二个选项是返回 IEnumerable&lt;T&gt;，这意味着 EF4 LINQ 提供程序将只能看到在存储库内生成的表达式。 不到 SQL 命令发送到数据库，可能会损害性能将撰写任何分组、 排序和外部存储库完成的投影。 另一方面，存储库返回仅 IEnumerable&lt;T&gt;结果将永远不会让人感到惊讶与新的 SQL 命令。 这两种方法将起作用，且这两种方法仍可测试性。

直接提供的单一实现 IRepository&lt;T&gt;接口使用泛型和 EF4 ObjectContext API。

``` csharp
    public class SqlRepository<T> : IRepository<T>
                                    where T : class, IEntity {
        public SqlRepository(ObjectContext context) {
            _objectSet = context.CreateObjectSet<T>();
        }
        public IQueryable<T> FindAll() {
            return _objectSet;
        }
        public IQueryable<T> FindWhere(
                               Expression<Func\<T, bool>> predicate) {
            return _objectSet.Where(predicate);
        }
        public T FindById(int id) {
            return _objectSet.Single(o => o.Id == id);
        }
        public void Add(T newEntity) {
            _objectSet.AddObject(newEntity);
        }
        public void Remove(T entity) {
            _objectSet.DeleteObject(entity);
        }
        protected ObjectSet<T> _objectSet;
    }
```

IRepository&lt;T&gt;方法为我们提供一些附加我们的查询控制因为客户端必须调用一个方法来获取到的实体。 在方法内，我们可以提供其他检查和强制实施应用程序约束的 LINQ 运算符。 请注意，此接口具有两个约束的泛型类型参数。 第一个约束是不所需的 ObjectSet 类缺点&lt;T&gt;，并在第二个约束强制实施我们实体来实现 IEntity – 创建应用程序的抽象。 IEntity 接口强制实体具有可读的 Id 属性，以及如何我们 FindById 方法中可以将此属性。 IEntity 是使用以下代码定义。

``` csharp
    public interface IEntity {
        int Id { get; }
    }
```

IEntity 无法被视为小型违规持久性无感知的因为我们实体都必须实现此接口。 请记住持久性无感知非常权衡，以及许多 FindById 功能将大于所施加的接口约束。 该接口上可测试性没有任何影响。

实例化实时 IRepository&lt;T&gt;需要 EF4 ObjectContext，因此工作实现的具体单元应管理实例化。

``` csharp
    public class SqlUnitOfWork : IUnitOfWork {
        public SqlUnitOfWork() {
            var connectionString =
                ConfigurationManager
                    .ConnectionStrings[ConnectionStringName]
                    .ConnectionString;

            _context = new ObjectContext(connectionString);
            _context.ContextOptions.LazyLoadingEnabled = true;
        }

        public IRepository<Employee> Employees {
            get {
                if (_employees == null) {
                    _employees = new SqlRepository<Employee>(_context);
                }
                return _employees;
            }
        }

        public IRepository<TimeCard> TimeCards {
            get {
                if (_timeCards == null) {
                    _timeCards = new SqlRepository<TimeCard>(_context);
                }
                return _timeCards;
            }
        }

        public void Commit() {
            _context.SaveChanges();
        }

        SqlRepository<Employee> _employees = null;
        SqlRepository<TimeCard> _timeCards = null;
        readonly ObjectContext _context;
        const string ConnectionStringName = "EmployeeDataModelContainer";
    }
```

### <a name="using-the-custom-repository"></a>使用自定义存储库

使用我们自定义的存储库并不明显不同于使用存储库基于 IObjectSet&lt;T&gt;。 而不是直接向属性应用 LINQ 运算符，我们将首先需要调用其中一个存储库的方法来获取 IQueryable&lt;T&gt;引用。

``` csharp
    public ViewResult Index() {
        var model = _repository.FindAll()
                               .Include("TimeCards")
                               .OrderBy(e => e.HireDate);
        return View(model);
    }
```

请注意，我们实现了以前的自定义 Include 运算符需要进行任何修改。 存储库的 FindById 方法从操作尝试检索单个实体中删除重复的逻辑。

``` csharp
    public ViewResult Details(int id) {
        var model = _repository.FindById(id);
        return View(model);
    }
```

可测试性的我们探讨了这两种方法没有显著差异。 我们无法提供假实现 IRepository&lt;T&gt;通过构建由 HashSet 提供支持的具体类&lt;员工&gt;-就像我们在上一部分中所做。 但是，一些开发人员更愿意使用模拟对象和模拟对象而不是生成 fakes 框架。 我们将介绍使用模拟来测试我们的实现，并讨论下一节中的模拟和 fakes 之间的差异。

### <a name="testing-with-mocks"></a>使用模拟测试

有不同的方法生成"test double"的哪些 Martin Fowler 的调用。 Test double （如电影 stunt double) 是一个对象生成的用于"突出显示"为实际生产测试期间的对象。 我们创建内存中存储库是与 SQL Server 的存储库的 test double。 我们已了解如何在单元测试过程中使用这些 test double 来隔离代码并使快速运行的测试。

我们构建了 test double 都具有实际的工作实现。 在后台每个存储的对象，具体集合，并且它们将添加并从此集合中移除对象，如我们在测试期间操作在存储库。 一些开发人员要构建其 test double 这种方式 – 的实际代码和工作实现。  这些测试双精度型值是我们所说*fakes*。 它们的工作实现，但它们不是实际生产环境中使用。 虚设储存库实际上不会写入数据库。 虚设的 SMTP 服务器实际上不会通过网络发送一封电子邮件。

### <a name="mocks-versus-fakes"></a>Fakes 与模拟

还有另一种类型的双精度称为测试*模拟*。 Fakes 有工作实现，而模拟就派上没有实现。 Mock 对象框架帮助我们在运行时构造这些模拟对象，并将它们用作 test double。 在本部分中我们将使用模拟框架 Moq 开放源代码。 下面是使用 Moq 动态创建一个测试双员工存储库的简单示例。

``` csharp
    Mock<IRepository<Employee>> mock =
        new Mock<IRepository<Employee>>();
    IRepository<Employee> repository = mock.Object;
    repository.Add(new Employee());
    var employee = repository.FindById(1);
```

我们请求 Moq 是 IRepository&lt;员工&gt;实现和它生成一个动态。 我们可以处理该对象实现 IRepository&lt;员工&gt;通过访问对象属性的模拟&lt;T&gt;对象。 我们可以将它们传递给我们的控制器，此内部对象，并且它们不会知道如果此参数的 test double 或实际的存储库。 就像我们将调用具有真正的实现的对象的方法一样，我们可以调用对象上的方法。

您必须想知道 mock 存储库时我们调用 Add 方法将执行的操作。 由于没有实现 mock 对象后的，添加将任何影响。 正如我们已经有了使用 fakes，我们编写了，因此员工将被丢弃，没有在后台具体集合。 FindById 的返回值呢？ 在这种情况下该模拟对象的功能的唯一可以执行，这是返回默认值。 由于我们要返回引用类型 （员工），则返回值是 null 值。

模拟可能听起来毫无价值;但是，还有两个模拟我们没有讨论过的功能。 首先，Moq framework 记录所做的是模拟对象上的所有调用。 以后在代码中我们可以要求 Moq，如果任何人都调用 Add 方法，或者任何人都调用 FindById 方法。 我们将看到更高版本如何在测试中使用此"黑盒"录制功能。

第二个重要功能是我们可以如何使用 Moq 进行编程的 mock 对象*期望*。 期望告知 mock 对象如何对任何给定交互作出响应。 例如，我们可以到我们模拟程序期望并告诉它要有人调用 FindById 时返回 employee 对象。 Moq 框架将使用安装程序 API 和 lambda 表达式进行编程这些预期提高。

``` csharp
    [TestMethod]
    public void MockSample() {
        Mock<IRepository<Employee>> mock =
            new Mock<IRepository<Employee>>();
        mock.Setup(m => m.FindById(5))
            .Returns(new Employee {Id = 5});
        IRepository<Employee> repository = mock.Object;
        var employee = repository.FindById(5);
        Assert.IsTrue(employee.Id == 5);
    }
```

我们要求 Moq 来动态创建一个存储库，在此示例中，然后我们编程的假定条件下的存储库。 预期结果指示要返回具有 Id 值为 5 的新员工对象，当有人调用 FindById 方法传递的值为 5 时的模拟对象。 此测试通过，并且我们不需要生成完整的实现为假 IRepository&lt;T&gt;。

让我们回顾我们在前面编写的测试，并对它们而不是 fakes 使用模拟进行返工。 就像之前，我们将使用基类来设置我们需要为所有控制器的测试的基础结构的常见组成部分。

``` csharp
    public class EmployeeControllerTestBase {
        public EmployeeControllerTestBase() {
            _employeeData = EmployeeObjectMother.CreateEmployees()
                                                .AsQueryable();
            _repository = new Mock<IRepository<Employee>>();
            _unitOfWork = new Mock<IUnitOfWork>();
            _unitOfWork.Setup(u => u.Employees)
                       .Returns(_repository.Object);
            _controller = new EmployeeController(_unitOfWork.Object);
        }

        protected IQueryable<Employee> _employeeData;
        protected Mock<IUnitOfWork> _unitOfWork;
        protected EmployeeController _controller;
        protected Mock<IRepository<Employee>> _repository;
    }
```

安装程序代码主要是保持相同。 而不是使用 fakes，我们将使用 Moq 构造模拟对象。 要返回模拟存储库，当代码调用员工属性时的模拟工作单元安排的基类。 模拟的安装程序的其余部分会在专用于每个特定的方案测试装置内进行。 例如，索引操作测试固定例程将设置模拟存储库时要返回的员工列表操作都调用 FindAll 方法的模拟存储库。

``` csharp
    [TestClass]
    public class EmployeeControllerIndexActionTests
               : EmployeeControllerTestBase {
        public EmployeeControllerIndexActionTests() {
            _repository.Setup(r => r.FindAll())
                        .Returns(_employeeData);
        }
        // .. tests
        [TestMethod]
        public void ShouldBuildModelWithAllEmployees() {
            var result = _controller.Index();
            var model = result.ViewData.Model
                          as IEnumerable<Employee>;
            Assert.IsTrue(model.Count() == _employeeData.Count());
        }
        // .. and more tests
    }
```

除预期值，我们的测试看起来类似于一些前面用过的测试。 但是，一个模拟框架的录制功能我们可以着手测试从不同角度。 我们来看看下一节中此新的透视。

### <a name="state-versus-interaction-testing"></a>与测试交互状态

有不同的技术可用于测试和模拟对象的软件。 一种方法是使用状态基于测试，这是我们做了什么本白皮书中到目前为止。 基于状态的有关软件的状态的测试进行断言。 中的最后一个测试我们调用操作方法的控制器上做，并且它应生成有关模型的断言。 下面是一些其他示例的测试状态：

-   验证存储库创建在执行后包含新的 employee 对象。
-   验证索引执行后，模型包含的所有员工的列表。
-   验证删除执行后，存储库不包含给定的员工。

你将看到使用模拟对象的另一种方法是验证*交互*。 虽然基于状态的有关对象的状态的测试可以断言，交互将基于对象之间如何交互的测试进行断言。 例如：

-   验证控制器创建执行时调用该存储库的 Add 方法。
-   验证控制器索引执行时调用该存储库的 FindAll 方法。
-   验证控制器调用要保存更改时编辑执行工作的 Commit 方法的单元。

因为我们不在集合内四处闲逛和验证计数，交互测试通常需要较少的测试数据。 例如，如果我们知道的详细信息操作调用使用正确的值的存储库的 FindById 方法然后操作可能运行正常。 我们可以验证此行为，而无需设置 FindById 从返回的任何测试数据。

``` csharp
    [TestClass]
    public class EmployeeControllerDetailsActionTests
               : EmployeeControllerTestBase {
         // ...
        [TestMethod]
        public void ShouldInvokeRepositoryToFindEmployee() {
            var result = _controller.Details(_detailsId);
            _repository.Verify(r => r.FindById(_detailsId));
        }
        int _detailsId = 1;
    }
```

在上面测试固定例程所需的唯一设置是由基类提供的安装程序。 当我们调用控制器操作时，Moq 将记录与模拟存储库之间的交互。 使用验证 Moq API，我们可以要求 Moq 是否控制器调用 FindById 具有正确的 ID 值。 如果控制器未调用方法，或调用带有意外的参数值的方法，验证方法将引发异常，则测试将失败。

下面是工作的另一个示例，验证创建操作调用上当前单元提交。

``` csharp
    [TestMethod]
    public void ShouldCommitUnitOfWork() {
        _controller.Create(_newEmployee);
        _unitOfWork.Verify(u => u.Commit());
    }
```

使用交互测试的一个危险是往往会通过指定的交互。 要记录并验证使用模拟对象的每个交互并不意味着该测试的模拟对象的功能应尝试验证的每个交互。 一些交互是实现详细信息，你应仅验证交互*需*以满足当前的测试。

选择模拟或虚设之间很大程度上取决于要测试的系统和你的个人 （或团队） 首选项。 模拟对象可以显著减少您需要实现 test double 的代码量，但不是每个人都能轻松地编程的期望和验证交互。

## <a name="conclusions"></a>结论

在本文中我们演示了几种方法使用 ADO.NET 实体框架实现数据持久性的同时创建可测试代码。 我们可以利用内置的抽象，例如 IObjectSet&lt;T&gt;，或创建我们自己抽象，例如 IRepository&lt;T&gt;。  在这两种情况下，ADO.NET Entity Framework 4.0 中的 POCO 支持允许这些抽象的使用者保持持久性未知和高度可测试性。 其他 EF4 功能，例如隐式延迟加载允许业务和应用程序的服务代码工作而无需担心关系数据存储的详细信息。 最后，我们创建的抽象可以方便地进行模拟或虚设内部单元测试，并且我们可以使用这些 test double，以实现快速运行，高度隔离和可靠的测试。

### <a name="additional-resources"></a>其他资源

-   Robert C.Martin"[单一责任原则](http://www.objectmentor.com/resources/articles/srp.pdf)"
-   Martin Fowler[模式目录](http://www.martinfowler.com/eaaCatalog/index.html)从*的企业应用程序体系结构模式*
-   Griffin Caprio"[依赖关系注入](https://msdn.microsoft.com/magazine/cc163739.aspx)"
-   数据可编程性博客"[演练： 测试驱动开发使用 Entity Framework 4.0](https://blogs.msdn.com/adonet/pages/walkthrough-test-driven-development-with-the-entity-framework-4-0.aspx)"。
-   数据可编程性博客"[结合使用 Entity Framework 4.0 使用存储库和工作单元模式](https://blogs.msdn.com/adonet/archive/2009/06/16/using-repository-and-unit-of-work-patterns-with-entity-framework-4-0.aspx)"
-   Dave Astels" [BDD 简介](http://blog.daveastels.com/files/BDD_Intro.pdf)"
-   Aaron Jensen"[简介机规范](http://codebetter.com/blogs/aaron.jensen/archive/2008/05/08/introducing-machine-specifications-or-mspec-for-short.aspx)"
-   Eric Lee"[使用 MSTest BDD](https://blogs.msdn.com/elee/archive/2009/01/20/bdd-with-mstest.aspx)"
-   Eric Evans"[域驱动设计](http://books.google.com/books?id=7dlaMs0SECsC&printsec=frontcover&dq=evans%20domain%20driven%20design&hl=en&ei=cHztS6C8KIaglAfA_dS1CA&sa=X&oi=book_result&ct=result&resnum=1&ved=0CCoQ6AEwAA)"
-   Martin Fowler" [Mock 不是存根](http://martinfowler.com/articles/mocksArentStubs.html)"
-   Martin Fowler"[测试双](http://martinfowler.com/bliki/TestDouble.html)"
-   Jeremy Miller"[状态和交互测试](http://codebetter.com/blogs/jeremy.miller/articles/129544.aspx)"
-   [Moq](http://code.google.com/p/moq/)

### <a name="biography"></a>作者简介

Scott Allen 是 Pluralsight 技术团队的成员和 OdeToCode.com 的创始人。 在 15 年的商业软件开发中，Scott 曾解决方案从 8 位到高度可扩展的 ASP.NET web 应用程序的嵌入式设备的一切事项。 您可以在他的博客上是 OdeToCode，或在 Twitter 上通过 Scott [ http://twitter.com/OdeToCode ](http://twitter.com/OdeToCode)。
