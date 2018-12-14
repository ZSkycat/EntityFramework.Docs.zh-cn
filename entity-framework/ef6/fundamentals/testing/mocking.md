---
title: 使用模拟框架的 EF6 进行测试
author: divega
ms.date: 10/23/2016
ms.assetid: bd66a638-d245-44d4-8e71-b9c6cb335cc7
ms.openlocfilehash: 80fd97073744be40d66c09706d3513dba18e724d
ms.sourcegitcommit: 7a7da65404c9338e1e3df42576a13be536a6f95f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2018
ms.locfileid: "48834716"
---
# <a name="testing-with-a-mocking-framework"></a>使用模拟框架进行测试
> [!NOTE]
> **仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。 如果使用的是早期版本，则部分或全部信息不适用。  

为你的应用程序编写测试是，通常需要避免与数据库进行交互。  实体框架，可通过创建一个上下文 – 替换为你的测试 – 定义行为实现此目的使用内存中的数据。  

## <a name="options-for-creating-test-doubles"></a>用于创建 test double 的选项  

有两种可用于创建您的上下文的内存中版本的不同方法。  

- **创建你自己的 test double** – 这种方法涉及编写您自己的上下文和 Dbset 的内存中实现。 这使你可以控制如何类的行为，但可以写入和拥有合理的代码涉及很多。  
- **使用模拟框架来创建 test double** – 使用模拟框架 （如 Moq)，可以有您的内存中实现上下文和动态创建在运行时为你的设置。  

使用模拟框架将处理这篇文章。 有关创建自己的 test double 请参阅[测试与你自己的 Test Double](writing-test-doubles.md)。  

若要演示如何使用模拟框架使用 EF 我们将使用 Moq。 若要获取 Moq 的最简单方法是安装[来自 NuGet 的 Moq 包](http://nuget.org/packages/Moq/)。  

## <a name="testing-with-pre-ef6-versions"></a>使用预 EF6 版本进行测试  

在本文中所示的方案是依赖于我们对 DbSet 在 EF6 中进行一些更改。 要使用 EF5 和早期版本进行测试，请参阅[测试使用虚设上下文](http://romiller.com/2012/02/14/testing-with-a-fake-dbcontext/)。  

## <a name="limitations-of-ef-in-memory-test-doubles"></a>EF 内存中测试双精度数的限制  

内存中测试双精度型值可以是提供单元级别的测试覆盖率的应用程序使用 EF 的位的好方法。 但是，执行此操作时将使用 LINQ to Objects 对内存中数据执行查询。 这可能导致比使用 EF 的 LINQ 提供程序 (LINQ to Entities) 为针对数据库运行的 SQL 查询转换为不同的行为。  

此类差异的一个示例加载相关的数据。 如果创建一系列的博客，每个具有相关的文章，则在使用内存中数据相关的帖子将始终加载每个博客。 但是，针对数据库运行时，如果您使用 Include 方法只被加载数据。  

出于此原因，建议始终包括一定程度的端到端测试 （除了单元测试），确保正确地对数据库应用程序正常工作。  

## <a name="following-along-with-this-article"></a>根据这篇文章  

本文提供了您可以复制到 Visual Studio，若要跟着介绍一起操作，如果您想的完整代码列表。 它是最简单的方法创建**单元测试项目**，你需要为目标 **.NET Framework 4.5**来完成使用异步的部分。  

## <a name="the-ef-model"></a>EF 模型  

我们要测试的服务利用了 EF 模型 bloggingcontext 和在博客和文章类组成。 此代码可能由 EF 设计器生成也是 Code First 模型。  

``` csharp
using System.Collections.Generic;
using System.Data.Entity;

namespace TestingDemo
{
    public class BloggingContext : DbContext
    {
        public virtual DbSet<Blog> Blogs { get; set; }
        public virtual DbSet<Post> Posts { get; set; }
    }

    public class Blog
    {
        public int BlogId { get; set; }
        public string Name { get; set; }
        public string Url { get; set; }

        public virtual List<Post> Posts { get; set; }
    }

    public class Post
    {
        public int PostId { get; set; }
        public string Title { get; set; }
        public string Content { get; set; }

        public int BlogId { get; set; }
        public virtual Blog Blog { get; set; }
    }
}
```  

### <a name="virtual-dbset-properties-with-ef-designer"></a>使用 EF 设计器的虚拟 DbSet 属性  

请注意在上下文的 DbSet 属性标记为虚拟的。 这将允许模拟框架，用于从我们的上下文和重写的模拟实现这些属性派生。  

如果使用 Code First 然后可以直接编辑您的类。 如果您使用的 EF 设计器将需要编辑 T4 模板生成您的上下文。 打开\<model_name\>。Context.tt 文件下您 edmx 文件，嵌套的查找以下代码段，并添加 virtual 关键字中所示。  

``` csharp
public string DbSet(EntitySet entitySet)
{
    return string.Format(
        CultureInfo.InvariantCulture,
        "{0} virtual DbSet\<{1}> {2} {{ get; set; }}",
        Accessibility.ForReadOnlyProperty(entitySet),
        _typeMapper.GetTypeName(entitySet.ElementType),
        _code.Escape(entitySet));
}
```  

## <a name="service-to-be-tested"></a>要测试的服务  

若要演示如何使用内存中 test double 测试我们将会为 BlogService 编写一些测试。 该服务是能够创建新的博客 (AddBlog)，返回所有博客按名称 (GetAllBlogs) 排序。 除了 GetAllBlogs，我们还提供了将以异步方式获取按名称 (GetAllBlogsAsync) 排序的所有博客的方法。  

``` csharp
using System.Collections.Generic;
using System.Data.Entity;
using System.Linq;
using System.Threading.Tasks;

namespace TestingDemo
{
    public class BlogService
    {
        private BloggingContext _context;

        public BlogService(BloggingContext context)
        {
            _context = context;
        }

        public Blog AddBlog(string name, string url)
        {
            var blog = _context.Blogs.Add(new Blog { Name = name, Url = url });
            _context.SaveChanges();

            return blog;
        }

        public List<Blog> GetAllBlogs()
        {
            var query = from b in _context.Blogs
                        orderby b.Name
                        select b;

            return query.ToList();
        }

        public async Task<List<Blog>> GetAllBlogsAsync()
        {
            var query = from b in _context.Blogs
                        orderby b.Name
                        select b;

            return await query.ToListAsync();
        }
    }
}
```  

## <a name="testing-non-query-scenarios"></a>测试非查询方案  

这是我们需要执行的操作开始测试非查询方法。 以下测试使用 Moq 创建上下文。 然后，它创建 DbSet\<博客\>，并向上连接要从上下文的博客属性中返回。 接下来，上下文用于创建新 BlogService 然后用于创建新的博客-使用 AddBlog 方法。 最后，此测试将验证该服务添加新的博客，并在上下文中调用 SaveChanges。  

``` csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;
using Moq;
using System.Data.Entity;

namespace TestingDemo
{
    [TestClass]
    public class NonQueryTests
    {
        [TestMethod]
        public void CreateBlog_saves_a_blog_via_context()
        {
            var mockSet = new Mock<DbSet<Blog>>();

            var mockContext = new Mock<BloggingContext>();
            mockContext.Setup(m => m.Blogs).Returns(mockSet.Object);

            var service = new BlogService(mockContext.Object);
            service.AddBlog("ADO.NET Blog", "http://blogs.msdn.com/adonet");

            mockSet.Verify(m => m.Add(It.IsAny<Blog>()), Times.Once());
            mockContext.Verify(m => m.SaveChanges(), Times.Once());
        }
    }
}
```  

## <a name="testing-query-scenarios"></a>测试查询方案  

若要执行针对我们的 DbSet 测试双精度的查询将我们需要设置的 IQueryable 实现。 第一步是创建一些内存中数据 – 我们将使用列表\<博客\>。 接下来，我们创建一个上下文和 DBSet\<博客\>然后接通 DbSet – IQueryable 实现它们只需将委派给 LINQ to Objects 提供程序的列表的合作\<T\>。  

然后，我们可以创建基于我们的 test double BlogService 并确保我们得到从 GetAllBlogs 的数据按名称排序。  

``` csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;
using Moq;
using System.Collections.Generic;
using System.Data.Entity;
using System.Linq;

namespace TestingDemo
{
    [TestClass]
    public class QueryTests
    {
        [TestMethod]
        public void GetAllBlogs_orders_by_name()
        {
            var data = new List<Blog>
            {
                new Blog { Name = "BBB" },
                new Blog { Name = "ZZZ" },
                new Blog { Name = "AAA" },
            }.AsQueryable();

            var mockSet = new Mock<DbSet<Blog>>();
            mockSet.As<IQueryable<Blog>>().Setup(m => m.Provider).Returns(data.Provider);
            mockSet.As<IQueryable<Blog>>().Setup(m => m.Expression).Returns(data.Expression);
            mockSet.As<IQueryable<Blog>>().Setup(m => m.ElementType).Returns(data.ElementType);
            mockSet.As<IQueryable<Blog>>().Setup(m => m.GetEnumerator()).Returns(data.GetEnumerator());

            var mockContext = new Mock<BloggingContext>();
            mockContext.Setup(c => c.Blogs).Returns(mockSet.Object);

            var service = new BlogService(mockContext.Object);
            var blogs = service.GetAllBlogs();

            Assert.AreEqual(3, blogs.Count);
            Assert.AreEqual("AAA", blogs[0].Name);
            Assert.AreEqual("BBB", blogs[1].Name);
            Assert.AreEqual("ZZZ", blogs[2].Name);
        }
    }
}
```  

### <a name="testing-with-async-queries"></a>使用异步查询测试

Entity Framework 6 引入了一组可用于以异步方式执行查询的扩展方法。 这些方法的示例包括 ToListAsync、 FirstAsync、 ForEachAsync 等。  

因为实体框架查询使使用 LINQ，扩展方法定义 IQueryable 和 IEnumerable。 但是，因为它们仅旨在与实体框架一起使用可能会收到以下错误，如果你尝试将其用于不是实体框架查询的 LINQ 查询：

> 源 IQueryable 没有实现 IDbAsyncEnumerable{0}。 实现 IDbAsyncEnumerable 的源可以用于实体框架的异步操作。 有关更多详细信息，请参阅[ http://go.microsoft.com/fwlink/?LinkId=287068 ](https://go.microsoft.com/fwlink/?LinkId=287068)。  

同时对 EF 查询运行时，才支持异步方法，可能想要针对内存中运行测试的 DbSet 的双时，在单元测试中使用它们。  

若要使用异步方法，我们需要创建内存中 DbAsyncQueryProvider 来处理异步查询。 很可能设置使用 Moq 向查询提供程序，同时是更易于在代码中创建测试 double 实现。 此实现的代码如下所示：  

``` csharp
using System.Collections.Generic;
using System.Data.Entity.Infrastructure;
using System.Linq;
using System.Linq.Expressions;
using System.Threading;
using System.Threading.Tasks;

namespace TestingDemo
{
    internal class TestDbAsyncQueryProvider<TEntity> : IDbAsyncQueryProvider
    {
        private readonly IQueryProvider _inner;

        internal TestDbAsyncQueryProvider(IQueryProvider inner)
        {
            _inner = inner;
        }

        public IQueryable CreateQuery(Expression expression)
        {
            return new TestDbAsyncEnumerable<TEntity>(expression);
        }

        public IQueryable<TElement> CreateQuery<TElement>(Expression expression)
        {
            return new TestDbAsyncEnumerable<TElement>(expression);
        }

        public object Execute(Expression expression)
        {
            return _inner.Execute(expression);
        }

        public TResult Execute<TResult>(Expression expression)
        {
            return _inner.Execute<TResult>(expression);
        }

        public Task<object> ExecuteAsync(Expression expression, CancellationToken cancellationToken)
        {
            return Task.FromResult(Execute(expression));
        }

        public Task<TResult> ExecuteAsync<TResult>(Expression expression, CancellationToken cancellationToken)
        {
            return Task.FromResult(Execute<TResult>(expression));
        }
    }

    internal class TestDbAsyncEnumerable<T> : EnumerableQuery<T>, IDbAsyncEnumerable<T>, IQueryable<T>
    {
        public TestDbAsyncEnumerable(IEnumerable<T> enumerable)
            : base(enumerable)
        { }

        public TestDbAsyncEnumerable(Expression expression)
            : base(expression)
        { }

        public IDbAsyncEnumerator<T> GetAsyncEnumerator()
        {
            return new TestDbAsyncEnumerator<T>(this.AsEnumerable().GetEnumerator());
        }

        IDbAsyncEnumerator IDbAsyncEnumerable.GetAsyncEnumerator()
        {
            return GetAsyncEnumerator();
        }

        IQueryProvider IQueryable.Provider
        {
            get { return new TestDbAsyncQueryProvider<T>(this); }
        }
    }

    internal class TestDbAsyncEnumerator<T> : IDbAsyncEnumerator<T>
    {
        private readonly IEnumerator<T> _inner;

        public TestDbAsyncEnumerator(IEnumerator<T> inner)
        {
            _inner = inner;
        }

        public void Dispose()
        {
            _inner.Dispose();
        }

        public Task<bool> MoveNextAsync(CancellationToken cancellationToken)
        {
            return Task.FromResult(_inner.MoveNext());
        }

        public T Current
        {
            get { return _inner.Current; }
        }

        object IDbAsyncEnumerator.Current
        {
            get { return Current; }
        }
    }
}
```  

现在，我们有一个异步查询提供商我们可以为我们新 GetAllBlogsAsync 方法编写单元测试。  

``` csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;
using Moq;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.Infrastructure;
using System.Linq;
using System.Threading.Tasks;

namespace TestingDemo
{
    [TestClass]
    public class AsyncQueryTests
    {
        [TestMethod]
        public async Task GetAllBlogsAsync_orders_by_name()
        {

            var data = new List<Blog>
            {
                new Blog { Name = "BBB" },
                new Blog { Name = "ZZZ" },
                new Blog { Name = "AAA" },
            }.AsQueryable();

            var mockSet = new Mock<DbSet<Blog>>();
            mockSet.As<IDbAsyncEnumerable<Blog>>()
                .Setup(m => m.GetAsyncEnumerator())
                .Returns(new TestDbAsyncEnumerator<Blog>(data.GetEnumerator()));

            mockSet.As<IQueryable<Blog>>()
                .Setup(m => m.Provider)
                .Returns(new TestDbAsyncQueryProvider<Blog>(data.Provider));

            mockSet.As<IQueryable<Blog>>().Setup(m => m.Expression).Returns(data.Expression);
            mockSet.As<IQueryable<Blog>>().Setup(m => m.ElementType).Returns(data.ElementType);
            mockSet.As<IQueryable<Blog>>().Setup(m => m.GetEnumerator()).Returns(data.GetEnumerator());

            var mockContext = new Mock<BloggingContext>();
            mockContext.Setup(c => c.Blogs).Returns(mockSet.Object);

            var service = new BlogService(mockContext.Object);
            var blogs = await service.GetAllBlogsAsync();

            Assert.AreEqual(3, blogs.Count);
            Assert.AreEqual("AAA", blogs[0].Name);
            Assert.AreEqual("BBB", blogs[1].Name);
            Assert.AreEqual("ZZZ", blogs[2].Name);
        }
    }
}
```  
