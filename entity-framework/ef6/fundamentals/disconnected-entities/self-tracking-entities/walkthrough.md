---
title: 自跟踪实体演练-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: b21207c9-1d95-4aa3-ae05-bc5fe300dab0
ms.openlocfilehash: d89c452410d34bea71e8220aae141c3bfca3e1ce
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490267"
---
# <a name="self-tracking-entities-walkthrough"></a>自跟踪实体演练
> [!IMPORTANT]
> 我们不再建议使用自跟踪实体模板。 它将仅继续用于支持现有应用程序。 如果应用程序需要使用断开连接的实体图，请考虑其他替代方案，例如[可跟踪实体](http://trackableentities.github.io/)，它与自跟踪实体类似，社区在更积极地开发这种技术，或使用低级别更改跟踪 API 编写自定义代码。

本演练演示 Windows Communication Foundation (WCF) 服务用于公开返回实体关系图的操作的方案。 接下来，客户端应用程序操作此关系图并提交到服务操作，用于验证并将更新保存到使用实体框架的数据库修改。

在完成本演练之前请确保您阅读[自我跟踪实体](index.md)页。

此演练完成以下操作：

-   创建一个数据库访问。
-   创建包含模型的类库。
-   交换到自跟踪实体生成器模板。
-   将实体类移到一个单独的项目。
-   创建进行操作来查询和保存实体公开的 WCF 服务。
-   创建客户端 （控制台和 WPF） 使用的应用程序服务。

我们将使用 Database First 在本演练中，但同样的技术同样适用于第一个模型。

## <a name="pre-requisites"></a>系统必备组件

若要完成本演练将需要最新版本的 Visual Studio。

## <a name="create-a-database"></a>创建数据库

随 Visual Studio 一起安装的数据库服务器是已安装的 Visual Studio 的版本而异：

-   如果使用 Visual Studio 2012，则您将创建 LocalDB 数据库。
-   如果您使用的 Visual Studio 2010 系统将创建一个 SQL Express 数据库。

让我们继续并生成的数据库。

-   打开 Visual Studio
-   **视图-&gt;服务器资源管理器**
-   右键单击**数据连接-&gt;添加连接...**
-   如果你尚未连接到数据库服务器资源管理器之前将需要选择**Microsoft SQL Server**作为数据源
-   连接到 LocalDB 或 SQL Express，具体取决于哪一个已安装
-   输入**STESample**作为数据库名称
-   选择**确定**并将你想要创建新数据库，请选择要求你**是**
-   新的数据库现在将出现在服务器资源管理器
-   如果你正在使用 Visual Studio 2012
    -   右键单击服务器资源管理器中的数据库，然后选择**新查询**
    -   将下面的 SQL 复制到新的查询，然后右键单击查询并选择**Execute**
-   如果您使用的 Visual Studio 2010
    -   选择**数据-&gt; Transact SQL 编辑器-&gt;新建查询连接...**
    -   输入 **。\\SQLEXPRESS**作为服务器名称，然后单击**确定**
    -   选择**STESample**数据库从下拉列表顶部的查询编辑器
    -   将下面的 SQL 复制到新的查询，然后右键单击查询并选择**执行 SQL**

``` SQL
    CREATE TABLE [dbo].[Blogs] (
        [BlogId] INT IDENTITY (1, 1) NOT NULL,
        [Name] NVARCHAR (200) NULL,
        [Url]  NVARCHAR (200) NULL,
        CONSTRAINT [PK_dbo.Blogs] PRIMARY KEY CLUSTERED ([BlogId] ASC)
    );

    CREATE TABLE [dbo].[Posts] (
        [PostId] INT IDENTITY (1, 1) NOT NULL,
        [Title] NVARCHAR (200) NULL,
        [Content] NTEXT NULL,
        [BlogId] INT NOT NULL,
        CONSTRAINT [PK_dbo.Posts] PRIMARY KEY CLUSTERED ([PostId] ASC),
        CONSTRAINT [FK_dbo.Posts_dbo.Blogs_BlogId] FOREIGN KEY ([BlogId]) REFERENCES [dbo].[Blogs] ([BlogId]) ON DELETE CASCADE
    );

    SET IDENTITY_INSERT [dbo].[Blogs] ON
    INSERT INTO [dbo].[Blogs] ([BlogId], [Name], [Url]) VALUES (1, N'ADO.NET Blog', N'blogs.msdn.com/adonet')
    SET IDENTITY_INSERT [dbo].[Blogs] OFF
    INSERT INTO [dbo].[Posts] ([Title], [Content], [BlogId]) VALUES (N'Intro to EF', N'Interesting stuff...', 1)
    INSERT INTO [dbo].[Posts] ([Title], [Content], [BlogId]) VALUES (N'What is New', N'More interesting stuff...', 1)
```

## <a name="create-the-model"></a>创建模型

首先，我们需要将该模型的项目。

-   **文件-&gt;新增-&gt;项目...**
-   选择**Visual C\#** 的左窗格中，然后**类库**
-   输入**STESample**作为名称，然后单击**确定**

现在我们将在 EF 设计器中访问我们的数据库创建一个简单的模型：

-   **项目-&gt;添加新项...**
-   选择**数据**的左窗格中，然后**ADO.NET 实体数据模型**
-   输入**BloggingModel**作为名称，然后单击**确定**
-   选择**从数据库生成**单击**下一步**
-   输入在上一部分中创建的数据库的连接信息
-   输入**bloggingcontext**作为连接字符串并单击名称**下一步**
-   旁边的复选框**表**单击**完成**

## <a name="swap-to-ste-code-generation"></a>对粘贴代码生成交换

现在，我们需要禁用默认代码生成和切换到自跟踪实体。

### <a name="if-you-are-using-visual-studio-2012"></a>如果你正在使用 Visual Studio 2012

-   展开**BloggingModel.edmx**中**解决方案资源管理器**并删除**BloggingModel.tt**并**BloggingModel.Context.tt** 
    *这将禁用默认代码生成*
-   右键单击空白区域在 EF 设计器图面上，然后选择**添加代码生成项...**
-   选择**联机**从左的窗格和搜索**STE 生成器**
-   选择**适用于 C STE 生成器\#** 模板，输入**STETemplate**作为名称，然后单击**添加**
-   **STETemplate.tt**并**STETemplate.Context.tt**嵌套 BloggingModel.edmx 文件下添加文件

### <a name="if-you-are-using-visual-studio-2010"></a>如果您使用的 Visual Studio 2010

-   右键单击空白区域在 EF 设计器图面上，然后选择**添加代码生成项...**
-   选择**代码**的左窗格中，然后**ADO.NET 自跟踪实体生成器**
-   输入**STETemplate**作为名称，然后单击**添加**
-   **STETemplate.tt**并**STETemplate.Context.tt**文件直接添加到你的项目

## <a name="move-entity-types-into-separate-project"></a>将实体类型移动到单独的项目

若要使用自跟踪实体我们的客户端应用程序需要有权从我们的模型生成的实体类。 因为我们不想要公开给客户端应用程序的整个模型，我们要将实体类移到一个单独的项目。

第一步是停止现有的项目中生成实体类：

-   右键单击**STETemplate.tt**中**解决方案资源管理器**，然后选择**属性**
-   在中**属性**窗口中清除**TextTemplatingFileGenerator**从**customtool**属性
-   展开**STETemplate.tt**中**解决方案资源管理器**删除嵌套在其下的所有文件

接下来，我们要添加新项目，并在其中生成实体类

-   **文件-&gt;添加-&gt;项目...**
-   选择**Visual C\#** 的左窗格中，然后**类库**
-   输入**STESample.Entities**作为名称，然后单击**确定**
-   **项目-&gt;添加现有项...**
-   导航到**STESample**项目文件夹
-   选择此选项可以查看**的所有文件 (\*。\*)**
-   选择**STETemplate.tt**文件
-   单击向下箭头旁边**外**按钮，然后选择**添加为链接**

    ![添加链接的模板](~/ef6/media/addlinkedtemplate.png)

我们还将确保在相同的命名空间上下文中生成实体类。 这只是减少使用我们需要添加整个我们的应用程序的语句。

-   右键单击链接**STETemplate.tt**中**解决方案资源管理器**，然后选择**属性**
-   在中**属性**窗口中设置**自定义工具 Namespace**到**STESample**

粘贴模板生成的代码将需要引用**System.Runtime.Serialization**为了编译。 此库所需的 WCF **DataContract**并**DataMember**可序列化的实体类型使用的属性。

-   右键单击**STESample.Entities**项目中**解决方案资源管理器**，然后选择**添加引用...**
    -   在 Visual Studio 2012-旁边的复选框**System.Runtime.Serialization**单击**确定**
    -   在 Visual Studio 2010-选择**System.Runtime.Serialization**单击**确定**

最后，使用我们的上下文中该项目将需要对实体类型的引用。

-   右键单击**STESample**项目中**解决方案资源管理器**，然后选择**添加引用...**
    -   在 Visual Studio 2012-选择**解决方案**从左窗格中，旁边的复选框**STESample.Entities**单击**确定**
    -   在 Visual Studio 2010-选择**项目**选项卡上，选择**STESample.Entities**单击**确定**

>[!NOTE]
> 将实体类型移动到一个单独的项目的另一个选项是将模板文件，而不是无需将其链接从其默认位置。 如果这样做，您需要更新**inputFile**变量中的模板来提供 edmx 文件的相对路径 (在此示例中，将为 **...\\BloggingModel.edmx**)。

## <a name="create-a-wcf-service"></a>创建 WCF 服务

现在就可以向其添加 WCF 服务公开我们的数据时，我们将首先创建项目。

-   **文件-&gt;添加-&gt;项目...**
-   选择**Visual C\#** 的左窗格中，然后**WCF 服务应用程序**
-   输入**STESample.Service**作为名称，然后单击**确定**
-   添加对的引用**System.Data.Entity**程序集
-   添加对的引用**STESample**并**STESample.Entities**项目

我们需要将 EF 连接字符串复制到此项目，以便在运行时找到该。

-   打开**App.Config**适用于文件 * * STESample * * 项目，然后复制**connectionStrings**元素
-   粘贴**connectionStrings**元素的子元素作为**配置**元素**Web.Config**文件中**STESample.Service**项目

现在就来实现实际的服务。

-   打开**IService1.cs**并使用以下代码替换内容

``` csharp
    using System.Collections.Generic;
    using System.ServiceModel;

    namespace STESample.Service
    {
        [ServiceContract]
        public interface IService1
        {
            [OperationContract]
            List<Blog> GetBlogs();

            [OperationContract]
            void UpdateBlog(Blog blog);
        }
    }
```

-   打开**Service1.svc**并使用以下代码替换内容

``` csharp
    using System;
    using System.Collections.Generic;
    using System.Data;
    using System.Linq;

    namespace STESample.Service
    {
        public class Service1 : IService1
        {
            /// <summary>
            /// Gets all the Blogs and related Posts.
            /// </summary>
            public List<Blog> GetBlogs()
            {
                using (BloggingContext context = new BloggingContext())
                {
                    return context.Blogs.Include("Posts").ToList();
                }
            }

            /// <summary>
            /// Updates Blog and its related Posts.
            /// </summary>
            public void UpdateBlog(Blog blog)
            {
                using (BloggingContext context = new BloggingContext())
                {
                    try
                    {
                        // TODO: Perform validation on the updated order before applying the changes.

                        // The ApplyChanges method examines the change tracking information
                        // contained in the graph of self-tracking entities to infer the set of operations
                        // that need to be performed to reflect the changes in the database.
                        context.Blogs.ApplyChanges(blog);
                        context.SaveChanges();

                    }
                    catch (UpdateException)
                    {
                        // To avoid propagating exception messages that contain sensitive data to the client tier
                        // calls to ApplyChanges and SaveChanges should be wrapped in exception handling code.
                        throw new InvalidOperationException("Failed to update. Try your request again.");
                    }
                }
            }        
        }
    }
```

## <a name="consume-the-service-from-a-console-application"></a>使用一个控制台应用程序中的服务

让我们创建的控制台应用程序使用我们的服务。

-   **文件-&gt;新增-&gt;项目...**
-   选择**Visual C\#** 的左窗格中，然后**控制台应用程序**
-   输入**STESample.ConsoleTest**作为名称，然后单击**确定**
-   添加对的引用**STESample.Entities**项目

我们需要对我们的 WCF 服务的服务引用

-   右键单击**STESample.ConsoleTest**项目中**解决方案资源管理器**，然后选择**添加服务引用...**
-   单击**发现**
-   输入**BloggingService**作为命名空间，然后单击**确定**

现在我们可以编写一些代码来使用该服务。

-   打开**Program.cs**和内容替换为以下代码。

``` csharp
    using STESample.ConsoleTest.BloggingService;
    using System;
    using System.Linq;

    namespace STESample.ConsoleTest
    {
        class Program
        {
            static void Main(string[] args)
            {
                // Print out the data before we change anything
                Console.WriteLine("Initial Data:");
                DisplayBlogsAndPosts();

                // Add a new Blog and some Posts
                AddBlogAndPost();
                Console.WriteLine("After Adding:");
                DisplayBlogsAndPosts();

                // Modify the Blog and one of its Posts
                UpdateBlogAndPost();
                Console.WriteLine("After Update:");
                DisplayBlogsAndPosts();

                // Delete the Blog and its Posts
                DeleteBlogAndPost();
                Console.WriteLine("After Delete:");
                DisplayBlogsAndPosts();

                Console.WriteLine("Press any key to exit...");
                Console.ReadKey();
            }

            static void DisplayBlogsAndPosts()
            {
                using (var service = new Service1Client())
                {
                    // Get all Blogs (and Posts) from the service
                    // and print them to the console
                    var blogs = service.GetBlogs();
                    foreach (var blog in blogs)
                    {
                        Console.WriteLine(blog.Name);
                        foreach (var post in blog.Posts)
                        {
                            Console.WriteLine(" - {0}", post.Title);
                        }
                    }
                }

                Console.WriteLine();
                Console.WriteLine();
            }

            static void AddBlogAndPost()
            {
                using (var service = new Service1Client())
                {
                    // Create a new Blog with a couple of Posts
                    var newBlog = new Blog
                    {
                        Name = "The New Blog",
                        Posts =
                        {
                            new Post { Title = "Welcome to the new blog"},
                            new Post { Title = "What's new on the new blog"}
                        }
                    };

                    // Save the changes using the service
                    service.UpdateBlog(newBlog);
                }
            }

            static void UpdateBlogAndPost()
            {
                using (var service = new Service1Client())
                {
                    // Get all the Blogs
                    var blogs = service.GetBlogs();

                    // Use LINQ to Objects to find The New Blog
                    var blog = blogs.First(b => b.Name == "The New Blog");

                    // Update the Blogs name
                    blog.Name = "The Not-So-New Blog";

                    // Update one of the related posts
                    blog.Posts.First().Content = "Some interesting content...";

                    // Save the changes using the service
                    service.UpdateBlog(blog);
                }
            }

            static void DeleteBlogAndPost()
            {
                using (var service = new Service1Client())
                {
                    // Get all the Blogs
                    var blogs = service.GetBlogs();

                    // Use LINQ to Objects to find The Not-So-New Blog
                    var blog = blogs.First(b => b.Name == "The Not-So-New Blog");

                    // Mark all related Posts for deletion
                    // We need to call ToList because each Post will be removed from the
                    // Posts collection when we call MarkAsDeleted
                    foreach (var post in blog.Posts.ToList())
                    {
                        post.MarkAsDeleted();
                    }

                    // Mark the Blog for deletion
                    blog.MarkAsDeleted();

                    // Save the changes using the service
                    service.UpdateBlog(blog);
                }
            }
        }
    }
```

现在可以运行应用程序来查看其实际运行情况。

-   右键单击**STESample.ConsoleTest**项目中**解决方案资源管理器**，然后选择**调试-&gt;启动新实例**

当应用程序执行时，您将看到以下输出。

```
Initial Data:
ADO.NET Blog
- Intro to EF
- What is New

After Adding:
ADO.NET Blog
- Intro to EF
- What is New
The New Blog
- Welcome to the new blog
- What's new on the new blog

After Update:
ADO.NET Blog
- Intro to EF
- What is New
The Not-So-New Blog
- Welcome to the new blog
- What's new on the new blog

After Delete:
ADO.NET Blog
- Intro to EF
- What is New

Press any key to exit...
```

## <a name="consume-the-service-from-a-wpf-application"></a>使用 WPF 应用程序中的服务

让我们创建使用我们的服务的 WPF 应用程序。

-   **文件-&gt;新增-&gt;项目...**
-   选择**Visual C\#** 的左窗格中，然后**WPF 应用程序**
-   输入**STESample.WPFTest**作为名称，然后单击**确定**
-   添加对的引用**STESample.Entities**项目

我们需要对我们的 WCF 服务的服务引用

-   右键单击**STESample.WPFTest**项目中**解决方案资源管理器**，然后选择**添加服务引用...**
-   单击**发现**
-   输入**BloggingService**作为命名空间，然后单击**确定**

现在我们可以编写一些代码来使用该服务。

-   打开**MainWindow.xaml**和内容替换为以下代码。

``` xaml
    <Window
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:STESample="clr-namespace:STESample;assembly=STESample.Entities"
        mc:Ignorable="d" x:Class="STESample.WPFTest.MainWindow"
        Title="MainWindow" Height="350" Width="525" Loaded="Window_Loaded">

        <Window.Resources>
            <CollectionViewSource
                x:Key="blogViewSource"
                d:DesignSource="{d:DesignInstance {x:Type STESample:Blog}, CreateList=True}"/>
            <CollectionViewSource
                x:Key="blogPostsViewSource"
                Source="{Binding Posts, Source={StaticResource blogViewSource}}"/>
        </Window.Resources>

        <Grid DataContext="{StaticResource blogViewSource}">
            <DataGrid AutoGenerateColumns="False" EnableRowVirtualization="True"
                      ItemsSource="{Binding}" Margin="10,10,10,179">
                <DataGrid.Columns>
                    <DataGridTextColumn Binding="{Binding BlogId}" Header="Id" Width="Auto" IsReadOnly="True" />
                    <DataGridTextColumn Binding="{Binding Name}" Header="Name" Width="Auto"/>
                    <DataGridTextColumn Binding="{Binding Url}" Header="Url" Width="Auto"/>
                </DataGrid.Columns>
            </DataGrid>
            <DataGrid AutoGenerateColumns="False" EnableRowVirtualization="True"
                      ItemsSource="{Binding Source={StaticResource blogPostsViewSource}}" Margin="10,145,10,38">
                <DataGrid.Columns>
                    <DataGridTextColumn Binding="{Binding PostId}" Header="Id" Width="Auto"  IsReadOnly="True"/>
                    <DataGridTextColumn Binding="{Binding Title}" Header="Title" Width="Auto"/>
                    <DataGridTextColumn Binding="{Binding Content}" Header="Content" Width="Auto"/>
                </DataGrid.Columns>
            </DataGrid>
            <Button Width="68" Height="23" HorizontalAlignment="Right" VerticalAlignment="Bottom"
                    Margin="0,0,10,10" Click="buttonSave_Click">Save</Button>
        </Grid>
    </Window>
```

-   打开主窗口代码隐藏 (**MainWindow.xaml.cs**) 并使用以下代码替换内容

``` csharp
    using STESample.WPFTest.BloggingService;
    using System.Collections.Generic;
    using System.Linq;
    using System.Windows;
    using System.Windows.Data;

    namespace STESample.WPFTest
    {
        public partial class MainWindow : Window
        {
            public MainWindow()
            {
                InitializeComponent();
            }

            private void Window_Loaded(object sender, RoutedEventArgs e)
            {
                using (var service = new Service1Client())
                {
                    // Find the view source for Blogs and populate it with all Blogs (and related Posts)
                    // from the Service. The default editing functionality of WPF will allow the objects
                    // to be manipulated on the screen.
                    var blogsViewSource = (CollectionViewSource)this.FindResource("blogViewSource");
                    blogsViewSource.Source = service.GetBlogs().ToList();
                }
            }

            private void buttonSave_Click(object sender, RoutedEventArgs e)
            {
                using (var service = new Service1Client())
                {
                    // Get the blogs that are bound to the screen
                    var blogsViewSource = (CollectionViewSource)this.FindResource("blogViewSource");
                    var blogs = (List<Blog>)blogsViewSource.Source;

                    // Save all Blogs and related Posts
                    foreach (var blog in blogs)
                    {
                        service.UpdateBlog(blog);
                    }

                    // Re-query for data to get database-generated keys etc.
                    blogsViewSource.Source = service.GetBlogs().ToList();
                }
            }
        }
    }
```

现在可以运行应用程序来查看其实际运行情况。

-   右键单击**STESample.WPFTest**项目中**解决方案资源管理器**，然后选择**调试-&gt;启动新实例**
-   您可以操作在显示的数据并将其保存通过服务使用**保存**按钮

![WPF 主窗口](~/ef6/media/wpf.png)
