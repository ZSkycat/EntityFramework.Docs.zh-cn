---
title: 每个模型的 EF6 的多个关系图
author: divega
ms.date: 10/23/2016
ms.assetid: b95db5c8-de8d-43bd-9ccc-5df6a5e25e1b
ms.openlocfilehash: e78b927a147143629ac5b73e23edf439ba6d0264
ms.sourcegitcommit: 269c8a1a457a9ad27b4026c22c4b1a76991fb360
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2018
ms.locfileid: "46283610"
---
# <a name="multiple-diagrams-per-model"></a>每个模型的多个关系图
> [!NOTE]
> **EF5 及更高版本仅**的功能，Api，Entity Framework 5 中引入了此页所述的等。 如果使用的是早期版本，则部分或全部信息不适用。

此视频和页显示了如何将模型分割成多个关系图使用实体框架设计器 （EF 设计器）。 您可能想要您的模型变得太大，无法查看或编辑时使用此功能。

在早期版本的 EF 设计器可以只让每个 EDMX 文件一个关系图。 从 Visual Studio 2012 开始，你可以使用 EF 设计器拆分为多个关系图的 EDMX 文件。

## <a name="watch-the-video"></a>观看视频
此视频介绍如何将模型分割成多个关系图使用实体框架设计器 （EF 设计器）。 您可能想要您的模型变得太大，无法查看或编辑时使用此功能。

**主讲人**： 以下 Julia Kornich

**视频**: [WMV](https://download.microsoft.com/download/5/C/2/5C2B52AB-5532-426F-B078-1E253341B5FA/HDI-ITPro-MSDN-winvideo-multiplediagrams.wmv) | [MP4](https://download.microsoft.com/download/5/C/2/5C2B52AB-5532-426F-B078-1E253341B5FA/HDI-ITPro-MSDN-mp4video-multiplediagrams.m4v) | [WMV (ZIP)](https://download.microsoft.com/download/5/C/2/5C2B52AB-5532-426F-B078-1E253341B5FA/HDI-ITPro-MSDN-winvideo-multiplediagrams.zip)

## <a name="ef-designer-overview"></a>EF 设计器概述

当您创建使用 EF 设计器的实体数据模型向导的模型时，.edmx 文件被创建并添加到你的解决方案。 此文件定义的实体以及它们如何映射到数据库的形状。

在 EF 设计器包括以下组件：

-   用于编辑的模型的可视化设计图面。 您可以创建、修改或删除实体和关联。
-   一个**模型浏览器**提供模型的树视图的窗口。  实体和其关联都位于下*\[ModelName\]* 文件夹。 数据库表和约束都位于下 *\[ModelName\]*。将文件夹存储。
-   一个**映射详细信息**窗口用于查看和编辑映射。 您可以将实体类型或关联映射到数据库表、列和存储过程。 

实体数据模型向导完成后，将自动打开可视化设计图面窗口。 如果模型浏览器不可见，请右键单击主设计图面，然后选择**模型浏览器**。

以下屏幕截图显示了在 EF 设计器中打开.edmx 文件。 屏幕截图显示了可视化设计图面 （左侧） 和**模型浏览器**窗口 （右侧）。

![EF 设计器 2](~/ef6/media/efdesigner2.png)

若要撤消在 EF 设计器中完成的操作，请单击 Ctrl Z。

## <a name="working-with-diagrams"></a>使用的关系图

默认情况下 EF 设计器创建一个名为 Diagram1 的关系图。 如果您有包含大量实体和关联的关系图，您将最喜欢想要以逻辑方式拆分。 从 Visual Studio 2012 开始，可以查看多个关系图中的概念模型。   

当您添加新关系图，它们显示在模型浏览器窗口的关系图文件夹下。 若要重命名关系图： 模型浏览器窗口中选择关系图，一次单击名称，然后键入新名称。  此外可以右键单击关系图名称，然后选择**重命名**。

在 Visual Studio 编辑器中的.edmx 文件名称旁边显示关系图名称。 例如 model1.edmx\[Diagram1\]。

![DiagramName](~/ef6/media/diagramname.png)

关系图内容 （形状和颜色的实体和关联） 存储中。 edmx.diagram 文件。 若要查看此文件，请选择解决方案资源管理器并展开.edmx 文件。 

![DiagramFiles](~/ef6/media/diagramfiles.png)

不应编辑。 edmx.diagram 手动，文件可能被 EF 设计器覆盖此文件的内容。
 
## <a name="splitting-entities-and-associations-into-a-new-diagram"></a>拆分实体和关联到新关系图

您可以选择现有的关系图 （按住 shift 键来选择多个实体） 上的实体。 单击鼠标右键，然后选择**移动到新关系图**。 创建新关系图和所选的实体和其关联移动到关系图。

或者，可以右键单击模型浏览器中的关系图文件夹并选择**添加新关系图。** 你可以然后拖放实体从模型浏览器中拖动到设计图面上的实体类型文件夹下。

此外可以剪切或将实体 （使用 Ctrl-X 或 Ctrl + C 键） 从一个关系图复制并粘贴其他 （使用 Ctrl + V 键）。 如果关系图在其中粘贴实体已包含具有相同名称的实体，将创建一个新实体，并将其添加到模型中。  例如： 图 2 包含 Department 实体。 然后，将另一部门粘贴在上图 2。 Department1 实体被创建并添加到概念模型。   

若要在关系图中包括相关的实体，右击实体并选择**包括相关**。 在指定的关系图中，这将使相关的实体和关联的副本。

## <a name="changing-the-color-of-entities"></a>更改实体的颜色

除了将模型拆分成多个关系图，还可以更改你的实体的颜色。

若要更改颜色，请选择设计图面上的实体 （或多个实体）。 然后，单击鼠标右键并选择**属性**。 在属性窗口中，选择**填充颜色**属性。 指定使用有效的颜色名称 （例如，红色） 或有效的 RGB （例如，255，128，128） 的颜色。 

![颜色](~/ef6/media/color.png)

## <a name="summary"></a>总结

本主题中介绍了如何将模型拆分为多个关系图以及如何指定不同的颜色使用实体框架设计器的实体。 
