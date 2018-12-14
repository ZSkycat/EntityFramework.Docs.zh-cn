---
title: 使用 WinForms-EF6 进行数据绑定
author: divega
ms.date: 10/23/2016
ms.assetid: 80fc5062-2f1c-4dbd-ab6e-b99496784b36
ms.openlocfilehash: 8da5bf653221b7919abb89d6d33bc8ed172828a4
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490146"
---
# <a name="databinding-with-winforms"></a>使用 WinForms 进行数据绑定
此分步演练说明如何将 POCO 类型绑定到"母版-详细信息"窗体中的 Window 窗体 (WinForms) 控件。 应用程序使用实体框架来填充数据库中的数据对象、 跟踪更改，然后将数据保存到数据库。

模型定义一个对多关系中参与的两种类型： 类别 (主体\\master) 和产品 (依赖\\详细信息)。 然后，Visual Studio 工具用于绑定到 WinForms 控件模型中定义的类型。 WinForms 数据绑定框架，相关对象之间导航： 在母版视图中选择行导致要使用相应的子数据更新的详细信息视图。

从 Visual Studio 2013 创建屏幕快照和在本演练中的代码列表，但你可以完成此演练使用 Visual Studio 2012 或 Visual Studio 2010。

## <a name="pre-requisites"></a>系统必备组件

需要具有 Visual Studio 2013，Visual Studio 2012 或 Visual Studio 2010 安装来完成本演练。

如果使用 Visual Studio 2010，您还必须安装 NuGet。 有关详细信息，请参阅[安装 NuGet](http://docs.nuget.org/docs/start-here/installing-nuget)。

## <a name="create-the-application"></a>创建应用程序

-   打开 Visual Studio
-   **文件-&gt;新增-&gt;项目...**
-   选择**Windows**的左窗格中并**Windows FormsApplication**右窗格中
-   输入**WinFormswithEFSample**作为名称
-   选择“确定”

## <a name="install-the-entity-framework-nuget-package"></a>安装 Entity Framework NuGet 包

-   在解决方案资源管理器，右键单击**WinFormswithEFSample**项目
-   选择**管理 NuGet 包...**
-   在管理 NuGet 包对话框中，选择**联机**选项卡，选择**EntityFramework**包
-   单击**安装**  
    > [!NOTE]
    > 除了 EntityFramework 程序集也添加到 System.ComponentModel.DataAnnotations 的引用。 如果项目具有对 System.Data.Entity 的引用，然后它将删除在安装 EntityFramework 包。 System.data.entity 的引用程序集不能再用于 Entity Framework 6 应用程序。

## <a name="implementing-ilistsource-for-collections"></a>为集合实现 IListSource

集合属性必须实现 IListSource 接口以启用与排序时使用 Windows 窗体的双向数据绑定。 若要执行此操作，我们将要扩展 ObservableCollection 添加 IListSource 功能。

-   添加**ObservableListSource**到项目的类：
    -   右键单击项目名称
    -   选择**添加-&gt;新项**
    -   选择**类**并输入**ObservableListSource**的类名
-   默认情况下，使用以下代码生成的代码替换为：

*此类支持双向数据绑定，以及排序。类派生自 ObservableCollection&lt;T&gt;并添加 IListSource 的显式实现。实现 IListSource getlist （） 方法以返回与 ObservableCollection 保持同步的 IBindingList 实现。生成的 ToBindingList IBindingList 实现支持排序。ToBindingList 扩展方法定义中的 EntityFramework 程序集。*

``` csharp
    using System.Collections;
    using System.Collections.Generic;
    using System.Collections.ObjectModel;
    using System.ComponentModel;
    using System.Diagnostics.CodeAnalysis;
    using System.Data.Entity;

    namespace WinFormswithEFSample
    {
        public class ObservableListSource<T> : ObservableCollection<T>, IListSource
            where T : class
        {
            private IBindingList _bindingList;

            bool IListSource.ContainsListCollection { get { return false; } }

            IList IListSource.GetList()
            {
                return _bindingList ?? (_bindingList = this.ToBindingList());
            }
        }
    }
```

## <a name="define-a-model"></a>定义模型

在本演练中你可以选择实现使用 Code First 或 EF 设计器的模型。 完成以下两个部分之一。

### <a name="option-1-define-a-model-using-code-first"></a>选项 1： 定义使用 Code First 模型

本部分演示如何创建模型和其关联的数据库使用 Code First。 请跳到下一节 (**选项 2： 定义模型使用 Database First)** 如果您更愿意使用 Database First 进行反向工程使用 EF 设计器从数据库模型

使用 Code First 开发时通常首先编写.NET Framework 类定义概念 （域） 模型。

-   添加一个新**产品**到项目的类
-   默认情况下，使用以下代码生成的代码替换为：

``` csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;

    namespace WinFormswithEFSample
    {
        public class Product
        {
            public int ProductId { get; set; }
            public string Name { get; set; }

            public int CategoryId { get; set; }
            public virtual Category Category { get; set; }
        }
    }
```

-   添加**类别**到项目的类。
-   默认情况下，使用以下代码生成的代码替换为：

``` csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;

    namespace WinFormswithEFSample
    {
        public class Category
        {
            private readonly ObservableListSource<Product> _products =
                    new ObservableListSource<Product>();

            public int CategoryId { get; set; }
            public string Name { get; set; }
            public virtual ObservableListSource<Product> Products { get { return _products; } }
        }
    }
```

除了定义实体，还需要定义一个类，派生自**DbContext** ，并公开**DbSet&lt;TEntity&gt;** 属性。 **DbSet**属性使知道你想要在模型中包含哪些的类型的上下文。 **DbContext**并**DbSet**类型定义中的 EntityFramework 程序集。

在运行时，其中包括填充数据库中的数据的对象期间将 DbContext 派生类型的实例管理的实体对象，到数据库更改跟踪，并保留数据。

-   添加一个新**ProductContext**到项目的类。
-   默认情况下，使用以下代码生成的代码替换为：

``` csharp
    using System;
    using System.Collections.Generic;
    using System.Data.Entity;
    using System.Linq;
    using System.Text;

    namespace WinFormswithEFSample
    {
        public class ProductContext : DbContext
        {
            public DbSet<Category> Categories { get; set; }
            public DbSet<Product> Products { get; set; }
        }
    }
```

编译该项目。

### <a name="option-2-define-a-model-using-database-first"></a>选项 2： 定义模型使用 Database First

本部分演示如何使用第一个数据库进行反向工程，使用 EF 设计器从数据库模型。 如果你在完成上一节 (**选项 1： 使用 Code First 定义模型)**，然后跳过此部分并直接转到**延迟加载**部分。

#### <a name="create-an-existing-database"></a>创建现有数据库

通常当你面向的现有数据库，则将已创建，但我们需要在本演练中创建的数据库访问。

随 Visual Studio 一起安装的数据库服务器是已安装的 Visual Studio 的版本而异：

-   如果您使用的 Visual Studio 2010 系统将创建一个 SQL Express 数据库。
-   如果你正在使用 Visual Studio 2012，则您将创建[LocalDB](https://msdn.microsoft.com/library/hh510202.aspx)数据库。

让我们继续并生成的数据库。

-   **视图-&gt;服务器资源管理器**
-   右键单击**数据连接-&gt;添加连接...**
-   如果你尚未连接到数据库服务器资源管理器之前将需要选择 Microsoft SQL Server 作为数据源

    ![更改数据源](~/ef6/media/changedatasource.png)

-   连接到 LocalDB 或 SQL Express，具体取决于哪一个已安装，并输入**产品**作为数据库名称

    ![添加连接 LocalDB](~/ef6/media/addconnectionlocaldb.png)

    ![添加连接 Express](~/ef6/media/addconnectionexpress.png)

-   选择**确定**并将你想要创建新数据库，请选择要求你**是**

    ![创建数据库](~/ef6/media/createdatabase.png)

-   新数据库现在将出现在服务器资源管理器，右键单击它并选择**新查询**
-   将下面的 SQL 复制到新的查询，然后右键单击查询并选择**Execute**

``` SQL
    CREATE TABLE [dbo].[Categories] (
        [CategoryId] [int] NOT NULL IDENTITY,
        [Name] [nvarchar](max),
        CONSTRAINT [PK_dbo.Categories] PRIMARY KEY ([CategoryId])
    )

    CREATE TABLE [dbo].[Products] (
        [ProductId] [int] NOT NULL IDENTITY,
        [Name] [nvarchar](max),
        [CategoryId] [int] NOT NULL,
        CONSTRAINT [PK_dbo.Products] PRIMARY KEY ([ProductId])
    )

    CREATE INDEX [IX_CategoryId] ON [dbo].[Products]([CategoryId])

    ALTER TABLE [dbo].[Products] ADD CONSTRAINT [FK_dbo.Products_dbo.Categories_CategoryId] FOREIGN KEY ([CategoryId]) REFERENCES [dbo].[Categories] ([CategoryId]) ON DELETE CASCADE
```

#### <a name="reverse-engineer-model"></a>反向工程模型

我们将使用实体框架设计器，它是作为 Visual Studio 的一部分，若要创建我们的模型。

-   **项目-&gt;添加新项...**
-   选择**数据**左侧的菜单，然后**ADO.NET 实体数据模型**
-   输入**ProductModel**作为名称，然后单击**确定**
-   这将启动**实体数据模型向导**
-   选择**从数据库生成**单击**下一步**

    ![ChooseModelContents](~/ef6/media/choosemodelcontents.png)

-   选择连接到第一个部分中创建的数据库中，输入**ProductContext**作为名称的连接字符串和单击**下一步**

    ![选择您的连接](~/ef6/media/chooseyourconnection.png)

-   单击表导入的所有表并单击完成旁边的复选框

    ![选择对象](~/ef6/media/chooseyourobjects.png)

反向工程过程完成后的新模型添加到你的项目，并打开，以便在实体框架设计器中查看。 此外具有已 App.config 文件添加到你的项目数据库的连接详细信息。

#### <a name="additional-steps-in-visual-studio-2010"></a>Visual Studio 2010 中的其他步骤

如果你正在 Visual Studio 2010 中将需要更新 EF 设计器使用 EF6 代码生成。

-   在您的模型在 EF 设计器中的某一空白点上右键单击并选择**添加代码生成项...**
-   选择**联机模板**从左侧的菜单并搜索**DbContext**
-   选择**EF 6.x DbContext Generator 于 c 语言\#，** 输入**ProductsModel**作为名称并单击添加

#### <a name="updating-code-generation-for-data-binding"></a>更新数据绑定的代码生成

EF 从您的模型使用 T4 模板生成代码。 随 Visual Studio 或从 Visual Studio 库中下载的模板适用于常规用途使用。 这意味着从这些模板生成的实体具有简单 ICollection&lt;T&gt;属性。 但是，在执行数据绑定时，需要具有实现 IListSource 的集合属性。 这是为什么我们创建了上面的 ObservableListSource 类，而且我们现在将要修改的模板，以使此类的使用。

-   打开**解决方案资源管理器**并找到**ProductModel.edmx**文件
-   查找**ProductModel.tt**将嵌套在 ProductModel.edmx 文件下的文件

    ![产品模型模板](~/ef6/media/productmodeltemplate.png)

-   双击 ProductModel.tt 文件以在 Visual Studio 编辑器中打开它
-   查找和替换的两个实例"**ICollection**"with"**ObservableListSource**"。 这些是位于大约行 296 和 484。
-   查找和替换第一个匹配项"**HashSet**"with"**ObservableListSource**"。 此匹配项位于大约第 50 行。 **不这样做**替换 HashSet 更高版本的代码中找到的第二个匹配项。
-   保存 ProductModel.tt 文件。 这应该会导致重新生成的实体的代码。 如果不自动重新生成代码，然后 ProductModel.tt 右键单击并选择"运行自定义工具"。

如果现在打开 Category.cs 文件 （其中嵌套在 ProductModel.tt 下），则你应看到产品集合具有类型**ObservableListSource&lt;产品&gt;**。

编译该项目。

## <a name="lazy-loading"></a>延迟加载

**产品**上的属性**类别**类并**类别**属性**产品**类是导航属性。 在实体框架中，导航属性提供一种导航两个实体类型之间的关系方法。

EF 提供了一个选项的相关的实体从数据库中加载自动首次访问导航属性。 使用此类型的加载 （称为延迟加载），注意，访问每个导航属性的第一次一个单独的查询时将执行针对数据库内容尚不在上下文中。

使用 POCO 实体类型时，EF 可以通过在运行时期间创建派生的代理类型的实例，然后重写虚拟属性添加加载挂钩在类中的实现延迟加载。 若要获取相关对象的延迟加载，您必须声明导航属性 getter 作为**公共**和**虚拟**(**Overridable**在 Visual Basic 中)，而且不能为您的类**密封**(**NotOverridable**在 Visual Basic 中)。 使用数据库时将自动进行虚拟若要启用延迟加载第一个导航属性。 在代码优先部分中我们所做的导航属性虚拟出于同样的原因

## <a name="bind-object-to-controls"></a>将对象绑定到控件

添加此 WinForms 应用程序的数据源作为模型中定义的类。

-   从主菜单中，选择**项目-&gt;添加新数据源...**
    (需要在 Visual Studio 2010 中，选择要**数据-&gt;添加新数据源...**)
-   在选择数据源类型窗口中，选择**对象**单击**下一步**
-   在选择数据对象对话框中，展开**WinFormswithEFSample**两次，然后选择**类别**没有不需要选择产品数据源，因为我们将通过该产品的获取到它类别数据源的属性。

    ![“数据源”](~/ef6/media/datasource.png)

-   单击**完成。**
    *如果未显示数据源窗口，选择 * * * 视图-&gt;其他 Windows-&gt;数据源**
-   按固定图标，以便数据源窗口不会不会自动隐藏。 您可能需要按刷新按钮，如果窗口已可见。

    ![数据源 2](~/ef6/media/datasource2.png)

-   在解决方案资源管理器中双击**Form1.cs**文件以在设计器中打开主窗体。
-   选择**类别**数据源，并将其拖动窗体上。 默认情况下，新 DataGridView (**categoryDataGridView**) 和导航工具栏控件添加到设计器。 这些控件绑定到 BindingSource (**categoryBindingSource**) 和绑定的导航器 (**categoryBindingNavigator**) 也创建的组件。
-   在编辑的列**categoryDataGridView**。 我们想要设置**CategoryId**列添加到只读的。 值**CategoryId**我们保存数据后，由数据库生成的属性。
    -   右键单击在 DataGridView 控件，并选择编辑列...
    -   选择类别 id 列，然后将 ReadOnly 设置为 True
    -   按确定
-   从类别的数据源下选择产品，并将其拖动窗体上。 则 productDataGridView 和 productBindingSource 都会添加到窗体。
-   编辑 productDataGridView 上的列。 我们想要隐藏的类别 Id 和类别列并将产品 id 设置为只读的。 我们将保存数据后，将由数据库生成 ProductId 属性的值。
    -   右键单击在 DataGridView 控件，然后选择**编辑列...**.
    -   选择**ProductId**列并设置**ReadOnly**到**True**。
    -   选择**CategoryId**列，然后按**删除**按钮。 执行具有相同的操作**类别**列。
    -   按**确定**。

    到目前为止，我们可以我们 DataGridView 控件与 BindingSource 组件在设计器中。 在下一部分中我们会将代码添加到代码隐藏将 categoryBindingSource.DataSource 设置为当前跟踪通过 DbContext 的实体的集合。 当从 WinForms 类别下拖动和删除产品谨慎 productsBindingSource.DataSource 属性，并为产品 categoryBindingSource 和 productsBindingSource.DataMember 属性设置。 由于此绑定将中 productDataGridView 显示仅属于当前所选类别的产品。
-   启用**保存**通过单击鼠标右键并选择导航工具栏上的按钮**已启用**。

    ![窗体 1 设计器](~/ef6/media/form1-designer.png)

-   添加保存的事件处理程序通过双击该按钮的按钮。 这将添加事件处理程序，并转到代码隐藏窗体。 代码**categoryBindingNavigatorSaveItem\_单击**将在下一节中添加事件处理程序。

## <a name="add-the-code-that-handles-data-interaction"></a>添加用于处理数据交互的代码

我们现在将添加代码以使用 ProductContext 执行数据访问。 更新主窗体窗口的代码，如下所示。

该代码声明了 ProductContext 的长时间运行实例。 ProductContext 对象用于查询并将数据保存到数据库。 从重写的 OnClosing 方法然后调用 ProductContext 实例上的 dispose （） 方法。 代码注释提供了有关代码的作用的详细信息。

``` csharp
    using System;
    using System.Collections.Generic;
    using System.ComponentModel;
    using System.Data;
    using System.Drawing;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;
    using System.Windows.Forms;
    using System.Data.Entity;

    namespace WinFormswithEFSample
    {
        public partial class Form1 : Form
        {
            ProductContext _context;
            public Form1()
            {
                InitializeComponent();
            }

            protected override void OnLoad(EventArgs e)
            {
                base.OnLoad(e);
                _context = new ProductContext();

                // Call the Load method to get the data for the given DbSet
                // from the database.
                // The data is materialized as entities. The entities are managed by
                // the DbContext instance.
                _context.Categories.Load();

                // Bind the categoryBindingSource.DataSource to
                // all the Unchanged, Modified and Added Category objects that
                // are currently tracked by the DbContext.
                // Note that we need to call ToBindingList() on the
                // ObservableCollection<TEntity> returned by
                // the DbSet.Local property to get the BindingList<T>
                // in order to facilitate two-way binding in WinForms.
                this.categoryBindingSource.DataSource =
                    _context.Categories.Local.ToBindingList();
            }

            private void categoryBindingNavigatorSaveItem_Click(object sender, EventArgs e)
            {
                this.Validate();

                // Currently, the Entity Framework doesn’t mark the entities
                // that are removed from a navigation property (in our example the Products)
                // as deleted in the context.
                // The following code uses LINQ to Objects against the Local collection
                // to find all products and marks any that do not have
                // a Category reference as deleted.
                // The ToList call is required because otherwise
                // the collection will be modified
                // by the Remove call while it is being enumerated.
                // In most other situations you can do LINQ to Objects directly
                // against the Local property without using ToList first.
                foreach (var product in _context.Products.Local.ToList())
                {
                    if (product.Category == null)
                    {
                        _context.Products.Remove(product);
                    }
                }

                // Save the changes to the database.
                this._context.SaveChanges();

                // Refresh the controls to show the values         
                // that were generated by the database.
                this.categoryDataGridView.Refresh();
                this.productsDataGridView.Refresh();
            }

            protected override void OnClosing(CancelEventArgs e)
            {
                base.OnClosing(e);
                this._context.Dispose();
            }
        }
    }
```

## <a name="test-the-windows-forms-application"></a>测试 Windows 窗体应用程序

-   编译和运行应用程序，而且可以测试功能。

    ![窗体 1 保存之前](~/ef6/media/form1beforesave.png)

-   保存后生成的存储密钥显示在屏幕上。

    ![窗体 1 保存后](~/ef6/media/form1aftersave.png)

-   如果使用 Code First，那么您还会看到**WinFormswithEFSample.ProductContext**为您创建数据库。

    ![Server 对象资源管理器](~/ef6/media/serverobjexplorer.png)
