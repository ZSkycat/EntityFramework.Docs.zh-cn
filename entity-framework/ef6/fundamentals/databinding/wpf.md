---
title: 与 WPF-EF6 的数据绑定
author: divega
ms.date: 10/23/2016
ms.assetid: e90d48e6-bea5-47ef-b756-7b89cce4daf0
ms.openlocfilehash: 1933988277d3be8fecc02fced3293f2b7f80c901
ms.sourcegitcommit: ae399f9f3d1bae2c446b552247bd3af3ca5a2cf9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/04/2018
ms.locfileid: "48575660"
---
# <a name="databinding-with-wpf"></a>使用 WPF 数据绑定
此分步演练说明如何将 POCO 类型绑定到"母版-详细信息"窗体中的 WPF 控件。 应用程序使用 Entity Framework Api 填充数据库中的数据对象、 跟踪更改，然后将数据保存到数据库。

模型定义一个对多关系中参与的两种类型：**类别**(主体\\master) 和**产品**(相关\\详细信息)。 然后，Visual Studio 工具用于绑定到 WPF 控件模型中定义的类型。 WPF 数据绑定框架，相关对象之间导航： 在母版视图中选择行导致要使用相应的子数据更新的详细信息视图。

从 Visual Studio 2013 创建屏幕快照和在本演练中的代码列表，但你可以完成此演练使用 Visual Studio 2012 或 Visual Studio 2010。

## <a name="use-the-object-option-for-creating-wpf-data-sources"></a>使用 Object 选项来创建的 WPF 数据源

我们使用建议使用与以前版本的实体框架**数据库**选项时创建新的数据源基于使用 EF 设计器创建的模型。 这是因为设计器将生成派生自 ObjectContext 的上下文和从 entityobject 继承派生的实体类。 使用数据库选项可帮助你编写与此 API 图面进行交互的最佳代码。

用于 Visual Studio 2012 和 Visual Studio 2013 EF 设计器生成派生自 DbContext 和简单的 POCO 实体类的上下文。 使用 Visual Studio 2010，我们建议交换到使用 DbContext，稍后在本演练中所述的代码生成模板。

使用 DbContext API 外围应用时应使用**对象**选项时创建新的数据源，如本演练中所示。

如果需要可以[还原到基于 ObjectContext 代码生成](~/ef6/modeling/designer/codegen/legacy-objectcontext.md)使用 EF 设计器创建的模型。

## <a name="pre-requisites"></a>系统必备组件

需要具有 Visual Studio 2013，Visual Studio 2012 或 Visual Studio 2010 安装来完成本演练。

如果使用 Visual Studio 2010，您还必须安装 NuGet。 有关详细信息，请参阅[安装 NuGet](https://docs.microsoft.com/nuget/install-nuget-client-tools)。  

## <a name="create-the-application"></a>创建应用程序

-   打开 Visual Studio
-   **文件-&gt;新增-&gt;项目...**
-   选择**Windows**的左窗格中并**WPFApplication**右窗格中
-   输入**WPFwithEFSample**作为名称
-   选择“确定”

## <a name="install-the-entity-framework-nuget-package"></a>安装 Entity Framework NuGet 包

-   在解决方案资源管理器，右键单击**WinFormswithEFSample**项目
-   选择**管理 NuGet 包...**
-   在管理 NuGet 包对话框中，选择**联机**选项卡，选择**EntityFramework**包
-   单击**安装**  
    >[!NOTE]
    > 除了 EntityFramework 程序集也添加到 System.ComponentModel.DataAnnotations 的引用。 如果项目具有对 System.Data.Entity 的引用，然后它将删除在安装 EntityFramework 包。 System.data.entity 的引用程序集不能再用于 Entity Framework 6 应用程序。

## <a name="define-a-model"></a>定义模型

在本演练中你可以选择实现使用 Code First 或 EF 设计器的模型。 完成以下两个部分之一。

### <a name="option-1-define-a-model-using-code-first"></a>选项 1： 定义使用 Code First 模型

本部分演示如何创建模型和其关联的数据库使用 Code First。 请跳到下一节 (**选项 2： 定义模型使用 Database First)** 如果您更愿意使用 Database First 进行反向工程使用 EF 设计器从数据库模型

使用 Code First 开发时通常首先编写.NET Framework 类定义概念 （域） 模型。

-   添加到一个新类**WPFwithEFSample:**
    -   右键单击项目名称
    -   选择**添加**，然后**新项**
    -   选择**类**并输入**产品**的类名
-   替换**产品**类定义替换以下代码：

``` csharp
    namespace WPFwithEFSample
    {
        public class Product
        {
            public int ProductId { get; set; }
            public string Name { get; set; }

            public int CategoryId { get; set; }
            public virtual Category Category { get; set; }
        }
    }

-   Add a **Category** class with the following definition:

    using System.Collections.ObjectModel;

    namespace WPFwithEFSample
    {
        public class Category
        {
            public Category()
            {
                this.Products = new ObservableCollection<Product>();
            }

            public int CategoryId { get; set; }
            public string Name { get; set; }

            public virtual ObservableCollection<Product> Products { get; private set; }
        }
    }
```

**产品**上的属性**类别**类并**类别**属性**产品**类是导航属性。 在实体框架中，导航属性提供一种导航两个实体类型之间的关系方法。

除了定义实体，还需要定义一个类，派生自 DbContext，并公开 DbSet&lt;TEntity&gt;属性。 DbSet&lt;TEntity&gt;属性使知道你想要在模型中包含哪些的类型的上下文。

在运行时，其中包括填充数据库中的数据的对象期间将 DbContext 派生类型的实例管理的实体对象，到数据库更改跟踪，并保留数据。

-   添加一个新**ProductContext**类具有以下定义的项目：

``` csharp
    using System.Data.Entity;

    namespace WPFwithEFSample
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

    ![选择模型内容](~/ef6/media/choosemodelcontents.png)

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

EF 从您的模型使用 T4 模板生成代码。 随 Visual Studio 或从 Visual Studio 库中下载的模板适用于常规用途使用。 这意味着从这些模板生成的实体具有简单 ICollection&lt;T&gt;属性。 但是，执行数据绑定使用 WPF 时，需使用**ObservableCollection**集合属性，以便该 WPF 可以跟踪对集合所做的更改。 为此，我们将修改为使用 ObservableCollection 的模板。

-   打开**解决方案资源管理器**并找到**ProductModel.edmx**文件
-   查找**ProductModel.tt**将嵌套在 ProductModel.edmx 文件下的文件

    ![WPF 产品模型模板](~/ef6/media/wpfproductmodeltemplate.png)

-   双击 ProductModel.tt 文件以在 Visual Studio 编辑器中打开它
-   查找和替换的两个实例"**ICollection**"with"**ObservableCollection**"。 这些文件位于大约有 296 和 484 的行。
-   查找和替换第一个匹配项"**HashSet**"with"**ObservableCollection**"。 这种情况非常大约位于第 50 行。 **不这样做**替换 HashSet 更高版本的代码中找到的第二个匹配项。
-   查找和替换的唯一匹配项"**System.Collections.Generic**"with"**System.Collections.ObjectModel**"。 这是大约位于行 424。
-   保存 ProductModel.tt 文件。 这应该会导致重新生成的实体的代码。 如果不自动重新生成代码，然后 ProductModel.tt 右键单击并选择"运行自定义工具"。

如果现在打开 Category.cs 文件 （其中嵌套在 ProductModel.tt 下），则你应看到产品集合具有类型**ObservableCollection&lt;产品&gt;**。

编译该项目。

## <a name="lazy-loading"></a>延迟加载

**产品**上的属性**类别**类并**类别**属性**产品**类是导航属性。 在实体框架中，导航属性提供一种导航两个实体类型之间的关系方法。

EF 提供了一个选项的相关的实体从数据库中加载自动首次访问导航属性。 使用此类型的加载 （称为延迟加载），注意，访问每个导航属性的第一次一个单独的查询时将执行针对数据库内容尚不在上下文中。

使用 POCO 实体类型时，EF 可以通过在运行时期间创建派生的代理类型的实例，然后重写虚拟属性添加加载挂钩在类中的实现延迟加载。 若要获取相关对象的延迟加载，您必须声明导航属性 getter 作为**公共**和**虚拟**(**Overridable**在 Visual Basic 中)，而且不能为您的类**密封**(**NotOverridable**在 Visual Basic 中)。 使用数据库时将自动进行虚拟若要启用延迟加载第一个导航属性。 在代码优先部分中我们所做的导航属性虚拟出于同样的原因

## <a name="bind-object-to-controls"></a>将对象绑定到控件

添加此 WPF 应用程序的数据源作为模型中定义的类。

-   双击**MainWindow.xaml**在解决方案资源管理器打开主窗体
-   从主菜单中，选择**项目-&gt;添加新数据源...**
    (需要在 Visual Studio 2010 中，选择要**数据-&gt;添加新数据源...**)
-   在选择数据源 Typewindow 中，选择**对象**单击**下一步**
-   在选择数据对象对话框中，展开**WPFwithEFSample**两次，然后选择**类别**  
    *若要选择无需**产品**数据源，因为会对通过此**产品**的属性**类别**数据源*  

    ![选择数据对象](~/ef6/media/selectdataobjects.png)

-   单击**完成。**
-   数据源窗口打开 MainWindow.xaml 窗口的旁边*如果未显示数据源窗口，选择**视图-&gt;其他 Windows-&gt;数据源***
-   按固定图标，以便数据源窗口不会不会自动隐藏。 您可能需要按刷新按钮，如果窗口已可见。

    ![Data Sources](~/ef6/media/datasources.png)

-   选择**类别**数据源，并将其拖动窗体上。

我们拖动到此源时，将发生以下的情况：

-   **CategoryViewSource**资源并**categoryDataGrid**控件添加到 XAML 
-   父网格元素的 DataContext 属性设置为"{StaticResource **categoryViewSource** }"。 **CategoryViewSource**资源用作绑定源的外部\\父网格元素。 内部的网格元素然后从父网格 （categoryDataGrid 的 ItemsSource 属性设置为"{Binding}"） 继承的 DataContext 值

``` xml
    <Window.Resources>
        <CollectionViewSource x:Key="categoryViewSource"
                                d:DesignSource="{d:DesignInstance {x:Type local:Category}, CreateList=True}"/>
    </Window.Resources>
    <Grid DataContext="{StaticResource categoryViewSource}">
        <DataGrid x:Name="categoryDataGrid" AutoGenerateColumns="False" EnableRowVirtualization="True"
                    ItemsSource="{Binding}" Margin="13,13,43,191"
                    RowDetailsVisibilityMode="VisibleWhenSelected">
            <DataGrid.Columns>
                <DataGridTextColumn x:Name="categoryIdColumn" Binding="{Binding CategoryId}"
                                    Header="Category Id" Width="SizeToHeader"/>
                <DataGridTextColumn x:Name="nameColumn" Binding="{Binding Name}"
                                    Header="Name" Width="SizeToHeader"/>
            </DataGrid.Columns>
        </DataGrid>
    </Grid>
```

## <a name="adding-a-details-grid"></a>添加详细信息网格

现在，我们已有一个网格，可以让我们来显示类别添加详细信息网格以显示相关联的产品。

-   选择**产品**属性从下的**类别**数据源，并将其拖动窗体上。
    -   **CategoryProductsViewSource**资源并**productDataGrid**网格添加到 XAML
    -   此资源的绑定路径设置为产品
    -   WPF 数据绑定框架可确保，与所选类别相关的唯一产品显示在**productDataGrid**
-   从工具箱拖动**按钮**到窗体。 设置**名称**属性设置为**buttonSave**并**内容**属性设置为**保存**。

在窗体应类似于以下所示：

![Designer](~/ef6/media/designer.png) 

## <a name="add-code-that-handles-data-interaction"></a>添加代码来处理数据交互

就可以将一些事件处理程序添加到主窗口。

-   在 XAML 窗口中，单击**&lt;窗口**元素，这将选择主窗口
-   在中**属性**窗口中选择**事件**右上角，然后双击右侧的文本框**Loaded**标签

    ![主窗口属性](~/ef6/media/mainwindowproperties.png)

-   此外将添加**单击**事件**保存**通过双击设计器中的保存按钮的按钮。 

转到代码隐藏窗体中，我们现在将编辑代码以使用 ProductContext 执行数据访问。 更新代码的主窗口，如下所示。

该代码声明了的长时间运行的实例**ProductContext**。 **ProductContext**对象用于查询并将数据保存到数据库。 **Dispose （)** 上**ProductContext**实例然后称为从重写**OnClosing**方法。 代码注释提供了有关代码的作用的详细信息。

``` csharp
    using System.Data.Entity;
    using System.Linq;
    using System.Windows;

    namespace WPFwithEFSample
    {
        public partial class MainWindow : Window
        {
            private ProductContext _context = new ProductContext();
            public MainWindow()
            {
                InitializeComponent();
            }

            private void Window_Loaded(object sender, RoutedEventArgs e)
            {
                System.Windows.Data.CollectionViewSource categoryViewSource =
                    ((System.Windows.Data.CollectionViewSource)(this.FindResource("categoryViewSource")));

                // Load is an extension method on IQueryable,
                // defined in the System.Data.Entity namespace.
                // This method enumerates the results of the query,
                // similar to ToList but without creating a list.
                // When used with Linq to Entities this method
                // creates entity objects and adds them to the context.
                _context.Categories.Load();

                // After the data is loaded call the DbSet<T>.Local property
                // to use the DbSet<T> as a binding source.
                categoryViewSource.Source = _context.Categories.Local;
            }

            private void buttonSave_Click(object sender, RoutedEventArgs e)
            {
                // When you delete an object from the related entities collection
                // (in this case Products), the Entity Framework doesn’t mark
                // these child entities as deleted.
                // Instead, it removes the relationship between the parent and the child
                // by setting the parent reference to null.
                // So we manually have to delete the products
                // that have a Category reference set to null.

                // The following code uses LINQ to Objects
                // against the Local collection of Products.
                // The ToList call is required because otherwise the collection will be modified
                // by the Remove call while it is being enumerated.
                // In most other situations you can use LINQ to Objects directly
                // against the Local property without using ToList first.
                foreach (var product in _context.Products.Local.ToList())
                {
                    if (product.Category == null)
                    {
                        _context.Products.Remove(product);
                    }
                }

                _context.SaveChanges();
                // Refresh the grids so the database generated values show up.
                this.categoryDataGrid.Items.Refresh();
                this.productsDataGrid.Items.Refresh();
            }

            protected override void OnClosing(System.ComponentModel.CancelEventArgs e)
            {
                base.OnClosing(e);
                this._context.Dispose();
            }
        }

    }
```

## <a name="test-the-wpf-application"></a>测试的 WPF 应用程序

-   编译并运行该应用程序。 如果使用 Code First，那么，你将看到**WPFwithEFSample.ProductContext**为您创建数据库。
-   在底部网格中的顶级网格和产品名称中输入类别名称*不要输入任何内容 ID 在列中，因为由数据库生成的主键*

    ![使用新类别和产品的主窗口](~/ef6/media/screen1.png)

-   按**保存**按钮以将数据保存到数据库

调用的 DbContext 的后面**savechanges （)**，Id 将填入数据库生成值。 因为我们调用**Refresh()** 后**savechanges （)** **DataGrid**控件更新使用新值。

![使用 Id 填充主窗口](~/ef6/media/screen2.png)

## <a name="additional-resources"></a>其他资源

若要了解有关数据绑定到集合使用 WPF 的详细信息，请参阅[本主题](https://docs.microsoft.com/dotnet/framework/wpf/data/data-binding-overview#binding-to-collections)WPF 文档中。  
