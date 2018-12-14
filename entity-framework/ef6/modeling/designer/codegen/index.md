---
title: 设计器代码生成模板 - EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 56e00fa2-f9f0-48b3-8006-f8266ca7e74b
ms.openlocfilehash: e4e99a86e7c273682c85eba06042af9a2a837d12
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283857"
---
# <a name="designer-code-generation-templates"></a>设计器代码生成模板
使用 Entity Framework 设计器创建模型时，会自动生成类和派生上下文。 除默认代码生成之外，我们还提供了许多模板，可用于自定义生成的代码。 这些模板以 T4 文本模板的形式提供，可按需自定义模板。

默认生成的代码取决于创建模型的 Visual Studio 版本：

-   在 Visual Studio 2012 和 2013 中创建的模型将生成简单的 POCO 实体类和派生的简化 DbContext 的上下文。
-   在 Visual Studio 2010 中创建的模型将生成派生自 EntityObject 的实体类和派生自 ObjectContext 的上下文。
  > [!NOTE]    
  > 建议在添加模型后切换到 DbContext 生成器模板。

此页介绍了可用的模板，并说明了如何将模板添加到模型。

## <a name="available-templates"></a>可用模板

以下模板由实体框架团队提供：

### <a name="dbcontext-generator"></a>DbContext 生成器

此模板将生成简单的 POCO 实体类和派生自使用 EF6 的 DbContext 的上下文。
除有需要使用以下列出的其他模板外，推荐使用此模板。
如果使用的是最新版本的 Visual Studio（Visual Studio 2013 以上版本），则也默认使用代码生成模板：创建新模型时，默认使用此模板，且 T4 文件 (.tt) 会嵌套在 .edmx 文件下。

#### <a name="older-versions-of-visual-studio"></a>较旧版本的 Visual Studio
- **Visual Studio 2012：** 若要获取 EF 6.x DbContextGenerator 模板，则需安装适用于 Visual Studio 的最新 Entity Framework Tools - 请参阅[获取实体框架](~/ef6/fundamentals/install.md)页以了解详细信息。
- **Visual Studio 2010：** EF 6.x DbContextGenerator 模板不适用于 Visual Studio 2010。

#### <a name="dbcontext-generator-for-ef-5x"></a>EF 5.x 的 DbContext 生成器

如果使用的是较旧版本的 EntityFramework NuGet 包（具有主要版本 5），则需使用 EF 5.x DbContext 生成器模板。

如果使用的是 Visual Studio 2013 或 2012，则其中已安装该模板。

如果使用的是 Visual Studio 2010，添加模板时，则需选择“联机”选项卡，从 Visual Studio 库中下载模板。 或者可直接从 Visual Studio 库提前安装模板。 因为模板包含在较新版本的 Visual Studio 中，因此只能将库上的版本安装在 Visual Studio 2010 中。

- [适用于 C# 的 EF 5.x DbContext 生成器](https://visualstudiogallery.msdn.microsoft.com/da740968-02f9-42a9-9ee4-1a9a06d896a2)
- [适用于 C# 网站的 EF 5.x DbContext 生成器](https://visualstudiogallery.msdn.microsoft.com/5d01a981-91b8-492c-b42c-c771c3f31e03)
- [适用于 VB.NET 的 EF 5.x DbContext 生成器](https://visualstudiogallery.msdn.microsoft.com/875c882d-333e-455a-8dae-5353510527dd?src=featured)
- [适用于 VB.NET 网站的 EF 5.x DbContext 生成器](https://visualstudiogallery.msdn.microsoft.com/d4d7d4cd-c2d0-43e6-8944-12f6ff8f2614)

#### <a name="dbcontext-generator-for-ef-4x"></a>EF 4.x 的 DbContext 生成器

如果使用的是较旧版本的 EntityFramework NuGet 包（具有主要版本 4），则需使用 EF 4.x DbContext 生成器模板。 添加模板时，可在“联机”选项卡中找到此模板，或可直接从 Visual Studio 库提前安装模板。

- [适用于 C# 的 EF 4.x DbContext 生成器](https://visualstudiogallery.msdn.microsoft.com/7812b04c-db36-4817-8a84-e73c452410a2)
- [适用于 C# 网站的 EF 4.x DbContext 生成器](https://visualstudiogallery.msdn.microsoft.com/de0e9bc6-e86a-4448-8a2e-a1260a53203e)
- [适用于 VB.NET 的 EF 4.x DbContext 生成器](https://visualstudiogallery.msdn.microsoft.com/73679ae5-e358-4e76-a538-c7b5e04ac073)
- [适用于 VB.NET 网站的 EF 4.x DbContext 生成器](https://visualstudiogallery.msdn.microsoft.com/86f5a660-306e-4831-840c-2e4ee7474a92)

### <a name="entityobject-generator"></a>EntityObject 生成器

此模板将生成从 EntityObject 派生的实体类和从 ObjectContext 派生的上下文。

> [!NOTE]
> 请考虑使用 DbContext 生成器

现推荐新应用程序使用 DbContext 生成器模板。 DbContext 生成器利用了更简单的 DbContext API。 EntityObject 生成器仍支持用于现有应用程序。

**Visual Studio 2010、2012 和 2013**

添加模板时，需选择“联机”选项卡，从 Visual Studio 库中下载模板。 或可直接从 Visual Studio 库提前安装模板。

- [适用于 C# 的 EF 6.x EntityObject 生成器](https://visualstudiogallery.msdn.microsoft.com/66612113-549c-4a9e-a14a-f629ceb3f89a)
- [适用于 C# 网站的 EF 6.x EntityObject 生成器](https://visualstudiogallery.msdn.microsoft.com/076140f3-6dbe-451f-a0e0-16b6d2bd8996)
- [适用于 VB.NET 的 EF 6.x EntityObject 生成器](https://visualstudiogallery.msdn.microsoft.com/ff479d55-2c85-43c5-a4d6-21cd659435ea)
- [适用于 VB.NET 网站的 EF 6.x EntityObject 生成器](https://visualstudiogallery.msdn.microsoft.com/668e2b92-c142-4da2-8e60-866c6346fc6a)

**EF 5.x 的 EntityObject 生成器**


如果使用的是 Visual Studio 2012 或 2013，添加模板时，则需选择“联机”选项卡，从 Visual Studio 库中下载模板。 或可直接从 Visual Studio 库提前安装模板。 因为模板包含在 Visual Studio 2010 中，因此只能将库上的模板安装在 Visual Studio 2012 和 2013 中。

- [适用于 C# 的 EF 5.x EntityObject 生成器](https://visualstudiogallery.msdn.microsoft.com/1da40393-b5ec-404a-a000-6a7e6e911339)
- [适用于 C# 网站的 EF 5.x EntityObject 生成器](https://visualstudiogallery.msdn.microsoft.com/94b48556-fcf0-4b9b-8615-20f9066ae9ac)
- [适用于 VB.NET 的 EF 5.x EntityObject 生成器](https://visualstudiogallery.msdn.microsoft.com/92c0129e-40dc-488c-a836-7e30846dfb30)
- [适用于 VB.NET 网站的 EF 5.x EntityObject 生成器](https://visualstudiogallery.msdn.microsoft.com/5dd7f75c-8c98-4eb7-b4bc-06f0d0b03b41)

如果只需要生成 ObjectContext 代码，无需编辑模板，则可[还原为 EntityObject 代码生成](~/ef6/modeling/designer/codegen/legacy-objectcontext.md)。

如果使用的是 Visual Studio 2010，则其中已安装该模板。 如果在 Visual Studio 2010 中创建新模型，则默认使用此模板，但项目中不包含 .tt 文件。 若想自定义模板，则需将其添加到项目中。

### <a name="self-tracking-entities-ste-generator"></a>自跟踪实体 (STE) 生成器

此模板将生成自跟踪实体类和从 ObjectContext 派生的上下文。 在 EF 应用程序中，上下文负责跟踪实体中的更改。 但是，在 N 层方案中，上下文可能无法用于修改实体的层。 自跟踪实体有助于跟踪任意层中的更改。 有关详细信息，请参阅[自跟踪实体](~/ef6/fundamentals/disconnected-entities/self-tracking-entities/index.md)。

> [!NOTE]
> 不推荐使用 STE 模板

不再建议在新应用程序中使用 STE 模板，虽然该模板仍支持用于现有应用程序。 请查看[断开连接的实体文章](~/ef6/fundamentals/disconnected-entities/index.md)了解推荐用于 N 层方案的其他选项。

> [!NOTE]
> 没有 EF 6.x 版的 STE 模板。


> [!NOTE]
> 没有 Visual Studio 2013 版的 STE 模板。

### <a name="visual-studio-2012"></a>Visual Studio 2012

如果使用的是 Visual Studio 2012，添加模板时，则需选择“联机”选项卡，从 Visual Studio 库中下载模板。 或可直接从 Visual Studio 库提前安装模板。 因为模板包含在 Visual Studio 2010 中，因此只能将库上的模板安装在 Visual Studio 2012 中。

- [适用于 C# 的 EF 5.x STE 生成器](https://visualstudiogallery.msdn.microsoft.com/a3ac10a5-9365-4096-bb58-d9a1ba71db8f)
- [适用于 C# 网站的 EF 5.x STE 生成器](https://visualstudiogallery.msdn.microsoft.com/1b55ab82-eeb4-47ba-8d35-3c7c8b5f5a8c)
- [适用于 VB.NET 的 EF 5.x STE 生成器](https://visualstudiogallery.msdn.microsoft.com/1ba8c6a3-44e9-4e1f-b21e-596f3168474b)
- [适用于 VB.NET 网站的 EF 5.x STE 生成器](https://visualstudiogallery.msdn.microsoft.com/a9fd5f0a-9af4-4e32-9c09-0e057072152e)

#### <a name="visual-studio-2010"></a>Visual Studio 2010**

如果使用的是 Visual Studio 2010，则其中已安装该模板。

### <a name="poco-entity-generator"></a>POCO 实体生成器

此模板将生成 POCO 实体类和从 ObjectContext 派生的上下文

> [!NOTE]
> 请考虑使用 DbContext 生成器

现推荐使用 DbContext 生成器模板在新应用程序中生成 POCO 类。 DbContext 生成器利用了新的 DbContext API，且可生成更简单的 POCO 类。 POCO 实体生成器仍支持用于现有应用程序。

> [!NOTE]
> 没有EF 5.x 或 EF 6.x 版的 STE 模板。

> [!NOTE]
> 没有 Visual Studio 2013 版的 POCO 模板。

#### <a name="visual-studio-2012-amp-visual-studio-2010"></a>Visual Studio 2012 和 Visual Studio 2010

添加模板时，需选择“联机”选项卡，从 Visual Studio 库中下载模板。 或可直接从 Visual Studio 库提前安装模板。

- [适用于 C# 的 EF 4.x POCO 生成器](https://visualstudiogallery.msdn.microsoft.com/23df0450-5677-4926-96cc-173d02752313)
- [适用于 C# 网站的 EF 4.x POCO 生成器](https://visualstudiogallery.msdn.microsoft.com/fe568da5-aa1a-4178-a2a5-48813c707a7f)
- [适用于 VB.NET 的 EF 4.x POCO 生成器](https://visualstudiogallery.msdn.microsoft.com/53ecbded-8936-4299-ab04-1e44e5489752)
- [适用于 VB.NET 网站的 EF 4.x POCO 生成器](https://visualstudiogallery.msdn.microsoft.com/463c5aca-05ad-4cdb-910b-2e4f83269e34)

### <a name="what-are-the-web-sites-templates"></a>“网站”模板是什么

“网站”模板（例如适用于 C\# 网站的 EF 5.x DbContext 生成器）可用于通过“文件”-&gt;“新建”-&gt;“网站...”创建的网站项目。这不同于通过“文件”-&gt;“新建”-&gt;项目...”创建的 Web 应用程序，后者使用的是标准模板。 我们单独提供了这些模板，因为 Visual Studio 中的项模板系统需要这些模板。

## <a name="using-a-template"></a>使用模板

若要开始使用代码生成模板，请右键单击 EF 设计器中设计界面上的空白区域，然后选择“添加代码生成项...”。

![添加代码生成项](~/ef6/media/add-code-gen-item.png)

如果已安装要使用的模板（或模板包含在 Visual Studio 中），则可从左侧菜单的“代码”或“数据”部分使用此模板。

![已安装](~/ef6/media/installed.png)

如果尚未安装模板，请选择左侧菜单中的“联机”并搜索所需模板。

![搜索](~/ef6/media/search.png) 

如果使用的是 Visual Studio 2012，则新的 .tt 文件将嵌套在 .edmx 文件下。*

> [!NOTE]
> 对于在 Visual Studio 2012 中创建的模型，需删除用于默认代码生成的模板，否则会生成重复的类和上下文。 默认文件是 &lt;model name&gt;.tt 和 &lt;model name&gt;.context.tt。 

![VS2012 模板](~/ef6/media/vs2012-templates.png)

如果使用的是 Visual Studio 2010，则 tt 文件会直接添加到项目中。  

![VS2010 模板](~/ef6/media/vs2010-templates.png)
