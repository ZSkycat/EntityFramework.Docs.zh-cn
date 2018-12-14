---
title: 使用你自己的 test double 的 EF6 进行测试
author: divega
ms.date: 10/23/2016
ms.assetid: 16a8b7c0-2d23-47f4-9cc0-e2eb2e738ca3
ms.openlocfilehash: 2158dc73585c2720e7293096b0478c73edf522d9
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490904"
---
# <a name="testing-with-your-own-test-doubles"></a>使用你自己的 test double 进行测试
> [!NOTE]
> **仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。 如果使用的是早期版本，则部分或全部信息不适用。  

为你的应用程序编写测试是，通常需要避免与数据库进行交互。  实体框架，可通过创建一个上下文 – 替换为你的测试 – 定义行为实现此目的使用内存中的数据。  

## <a name="options-for-creating-test-doubles"></a>用于创建 test double 的选项  

有两种可用于创建您的上下文的内存中版本的不同方法。  

- **创建你自己的 test double** – 这种方法涉及编写您自己的上下文和 Dbset 的内存中实现。 这使你可以控制如何类的行为，但可以写入和拥有合理的代码涉及很多。  
- **使用模拟框架来创建 test double** – 使用模拟框架 （如 Moq)，可以有您的内存中实现上下文和动态创建在运行时为你的设置。  

本文将处理创建你自己的测试双精度。 使用模拟框架的信息，请参阅[测试提供了模拟框架](mocking.md)。  

## <a name="testing-with-pre-ef6-versions"></a>使用预 EF6 版本进行测试  

在本文中所示的代码适用于 EF6。 要使用 EF5 和早期版本进行测试，请参阅[测试使用虚设上下文](http://romiller.com/2012/02/14/testing-with-a-fake-dbcontext/)。  

## <a name="limitations-of-ef-in-memory-test-doubles"></a>EF 内存中测试双精度数的限制  

内存中测试双精度型值可以是提供单元级别的测试覆盖率的应用程序使用 EF 的位的好方法。 但是，执行此操作时将使用 LINQ to Objects 对内存中数据执行查询。 这可能导致比使用 EF 的 LINQ 提供程序 (LINQ to Entities) 为针对数据库运行的 SQL 查询转换为不同的行为。  

此类差异的一个示例加载相关的数据。 如果创建一系列的博客，每个具有相关的文章，则在使用内存中数据相关的帖子将始终加载每个博客。 但是，针对数据库运行时，如果您使用 Include 方法只被加载数据。  

出于此原因，建议始终包括一定程度的端到端测试 （除了单元测试），确保正确地对数据库应用程序正常工作。  

## <a name="following-along-with-this-article"></a>根据这篇文章  

本文提供了您可以复制到 Visual Studio，若要跟着介绍一起操作，如果您想的完整代码列表。 它是最简单的方法创建**单元测试项目**，你需要为目标 **.NET Framework 4.5**来完成使用异步的部分。  

## <a name="creating-a-context-interface"></a>创建上下文接口  

我们要看一下测试利用了 EF 的服务模型。 为了能够使用内存中版本进行测试替换我们的 EF 上下文，我们将定义一个接口，我们的 EF 上下文 （和内存中双） 将 imeplement。  

我们要测试该服务将查询和修改使用我们的上下文的 DbSet 属性的数据，调用 SaveChanges 以将更改推送到数据库。 因此我们要包括在接口上的这些成员。  

``` csharp
using System.Data.Entity;

namespace TestingDemo
{
    public interface IBloggingContext
    {
        DbSet<Blog> Blogs { get; }
        DbSet<Post> Posts { get; }
        int SaveChanges();
    }
}
```  

## <a name="the-ef-model"></a>EF 模型  

我们要测试的服务利用了 EF 模型 bloggingcontext 和在博客和文章类组成。 此代码可能由 EF 设计器生成也是 Code First 模型。  

``` csharp
using System.Collections.Generic;
using System.Data.Entity;

namespace TestingDemo
{
    public class BloggingContext : DbContext, IBloggingContext
    {
        public DbSet<Blog> Blogs { get; set; }
        public DbSet<Post> Posts { get; set; }
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

### <a name="implementing-the-context-interface-with-the-ef-designer"></a>实现与 EF 设计器上下文接口  

请注意，我们的上下文实现 IBloggingContext 接口。  

如果使用 Code First 然后可以编辑您的上下文直接以实现该接口。 如果您使用的 EF 设计器将需要编辑 T4 模板生成您的上下文。 打开\<model_name\>。Context.tt 文件下您 edmx 文件，嵌套的查找以下代码段，并添加在界面中所示。  

``` csharp  
<#=Accessibility.ForType(container)#> partial class <#=code.Escape(container)#> : DbContext, IBloggingContext
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
        private IBloggingContext _context;

        public BlogService(IBloggingContext context)
        {
            _context = context;
        }

        public Blog AddBlog(string name, string url)
        {
            var blog = new Blog { Name = name, Url = url };
            _context.Blogs.Add(blog);
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

<a name="creating-the-in-memory-test-doubles"/> # # 创建内存中测试两倍，  

现在，我们已实际 EF 模型和服务可使用它，就可以创建内存中测试 double，我们可将用于测试。 我们已经为我们的上下文来创建 TestContext 测试双精度。 在 test double 选择我们想要以支持测试的行为，我们将我们将运行。 在此示例中我们只捕获的次数调用 SaveChanges 时，但可以包含验证正在测试的方案所需的任何逻辑。  

我们还创建了提供内存中实现的 DbSet TestDbSet。 我们提供完整实现的所有方法，以在 DbSet （除外找到），但只需实现你的测试方案将使用的成员。  

TestDbSet 使用我们提供了以确保可以处理异步查询的一些其他基础结构类。  

``` csharp
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Data.Entity;
using System.Data.Entity.Infrastructure;
using System.Linq;
using System.Linq.Expressions;
using System.Threading;
using System.Threading.Tasks;

namespace TestingDemo
{
    public class TestContext : IBloggingContext
    {
        public TestContext()
        {
            this.Blogs = new TestDbSet<Blog>();
            this.Posts = new TestDbSet<Post>();
        }

        public DbSet<Blog> Blogs { get; set; }
        public DbSet<Post> Posts { get; set; }
        public int SaveChangesCount { get; private set; }
        public int SaveChanges()
        {
            this.SaveChangesCount++;
            return 1;
        }
    }

    public class TestDbSet<TEntity> : DbSet<TEntity>, IQueryable, IEnumerable<TEntity>, IDbAsyncEnumerable<TEntity>
        where TEntity : class
    {
        ObservableCollection<TEntity> _data;
        IQueryable _query;

        public TestDbSet()
        {
            _data = new ObservableCollection<TEntity>();
            _query = _data.AsQueryable();
        }

        public override TEntity Add(TEntity item)
        {
            _data.Add(item);
            return item;
        }

        public override TEntity Remove(TEntity item)
        {
            _data.Remove(item);
            return item;
        }

        public override TEntity Attach(TEntity item)
        {
            _data.Add(item);
            return item;
        }

        public override TEntity Create()
        {
            return Activator.CreateInstance<TEntity>();
        }

        public override TDerivedEntity Create<TDerivedEntity>()
        {
            return Activator.CreateInstance<TDerivedEntity>();
        }

        public override ObservableCollection<TEntity> Local
        {
            get { return _data; }
        }

        Type IQueryable.ElementType
        {
            get { return _query.ElementType; }
        }

        Expression IQueryable.Expression
        {
            get { return _query.Expression; }
        }

        IQueryProvider IQueryable.Provider
        {
            get { return new TestDbAsyncQueryProvider<TEntity>(_query.Provider); }
        }

        System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator()
        {
            return _data.GetEnumerator();
        }

        IEnumerator<TEntity> IEnumerable<TEntity>.GetEnumerator()
        {
            return _data.GetEnumerator();
        }

        IDbAsyncEnumerator<TEntity> IDbAsyncEnumerable<TEntity>.GetAsyncEnumerator()
        {
            return new TestDbAsyncEnumerator<TEntity>(_data.GetEnumerator());
        }
    }

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

### <a name="implementing-find"></a>实现查找  

Find 方法很难采用一般形式实现。 如果您需要测试进行的代码使用 Find 方法是最简单的方法创建的每种需要支持实体类型的 DbSet 查找的测试。 然后可以编写逻辑来查找实体，该特定类型，如下所示。  

``` csharp
using System.Linq;

namespace TestingDemo
{
    class TestBlogDbSet : TestDbSet<Blog>
    {
        public override Blog Find(params object[] keyValues)
        {
            var id = (int)keyValues.Single();
            return this.SingleOrDefault(b => b.BlogId == id);
        }
    }
}
```  

## <a name="writing-some-tests"></a>编写测试  

这是我们需要执行的操作开始测试。 以下测试创建 TestContext，然后基于此上下文的服务。 然后使用该服务以创建新的博客-使用 AddBlog 方法。 最后，此测试将验证该服务添加到上下文的博客属性的新博客，并在上下文中调用 SaveChanges。  

这是只是一个可以测试与双精度的内存中测试的内容类型的示例，您可以调整的 test double 和以满足您的要求的验证逻辑。  

``` csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System.Linq;

namespace TestingDemo
{
    [TestClass]
    public class NonQueryTests
    {
        [TestMethod]
        public void CreateBlog_saves_a_blog_via_context()
        {
            var context = new TestContext();

            var service = new BlogService(context);
            service.AddBlog("ADO.NET Blog", "http://blogs.msdn.com/adonet");

            Assert.AreEqual(1, context.Blogs.Count());
            Assert.AreEqual("ADO.NET Blog", context.Blogs.Single().Name);
            Assert.AreEqual("http://blogs.msdn.com/adonet", context.Blogs.Single().Url);
            Assert.AreEqual(1, context.SaveChangesCount);
        }
    }
}
```  

下面是测试的这一次一个的方式执行查询的另一个示例。 测试开始通过使用其网络日志属性-请注意，数据不是按字母顺序中的某些数据创建测试上下文。 我们然后可以创建基于我们的测试上下文 BlogService 并确保我们得到从 GetAllBlogs 的数据按名称排序。  

``` csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace TestingDemo
{
    [TestClass]
    public class QueryTests
    {
        [TestMethod]
        public void GetAllBlogs_orders_by_name()
        {
            var context = new TestContext();
            context.Blogs.Add(new Blog { Name = "BBB" });
            context.Blogs.Add(new Blog { Name = "ZZZ" });
            context.Blogs.Add(new Blog { Name = "AAA" });

            var service = new BlogService(context);
            var blogs = service.GetAllBlogs();

            Assert.AreEqual(3, blogs.Count);
            Assert.AreEqual("AAA", blogs[0].Name);
            Assert.AreEqual("BBB", blogs[1].Name);
            Assert.AreEqual("ZZZ", blogs[2].Name);
        }
    }
}
```  

最后，我们将编写一个测试，以使用我们的异步方法以确保 async 基础结构我们纳入[TestDbSet](#creating-the-in-memory-test-doubles)是否正常工作。  

``` csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System.Collections.Generic;
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
            var context = new TestContext();
            context.Blogs.Add(new Blog { Name = "BBB" });
            context.Blogs.Add(new Blog { Name = "ZZZ" });
            context.Blogs.Add(new Blog { Name = "AAA" });

            var service = new BlogService(context);
            var blogs = await service.GetAllBlogsAsync();

            Assert.AreEqual(3, blogs.Count);
            Assert.AreEqual("AAA", blogs[0].Name);
            Assert.AreEqual("BBB", blogs[1].Name);
            Assert.AreEqual("ZZZ", blogs[2].Name);
        }
    }
}
```  
