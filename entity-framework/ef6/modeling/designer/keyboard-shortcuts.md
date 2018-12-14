---
title: 实体框架设计器键盘快捷方式-EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 3c76cdd5-17c5-4c54-a6a5-cf21b974636b
ms.openlocfilehash: c75eafcca0863faa1ad64202e98b61832827377c
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45490241"
---
# <a name="entity-framework-designer-keyboard-shortcuts"></a>实体框架设计器键盘快捷方式
此页提供了一系列适用于 Visual Studio 的实体框架工具的各种屏幕中可用的键盘快捷方式。

## <a name="adonet-entity-data-model-wizard"></a>ADO.NET 实体数据模型向导

### <a name="step-one-choose-model-contents"></a>第一步： 选择模型内容

![向导的一个](~/ef6/media/wizardone.png)

| 快捷键  | 操作                                                     | 说明                                               |
|:----------|:-----------------------------------------------------------|:----------------------------------------------------|
| **Alt + n** | 转到下一个屏幕                                        | 为所有选择模型内容的不可用。 |
| **Alt + f** | 完成向导                                              | 为所有选择模型内容的不可用。 |
| **Alt + w** | 焦点切换到"应在模型包含的内容？" 窗格。 |                                                     |

### <a name="step-two-choose-your-connection"></a>第二步： 选择您的连接

![向导的两个](~/ef6/media/wizardtwo.png)

| 快捷键  | 操作                                                     | 说明                                                   |
|:----------|:-----------------------------------------------------------|:--------------------------------------------------------|
| **Alt + n** | 转到下一个屏幕                                        |                                                         |
| **Alt + p** | 将移动到上一屏幕                                    |                                                         |
| **Alt + w** | 焦点切换到"应在模型包含的内容？" 窗格。 |                                                         |
| **Alt + c** | 打开"连接属性"窗口                    | 允许的新的数据库连接的定义。 |
| **Alt + e** | 从连接字符串中排除敏感数据          |                                                         |
| **Alt + i** | 连接字符串中包含敏感数据            |                                                         |
| **Alt + s** | 切换"将连接设置保存在 App.Config 中"选项 |                                                         |

### <a name="step-three-choose-your-version"></a>第三步： 选择你的版本

![向导的三个](~/ef6/media/wizardthree.png)

| 快捷键  | 操作                                             | 说明                                                                                 |
|:----------|:---------------------------------------------------|:--------------------------------------------------------------------------------------|
| **Alt + n** | 转到下一个屏幕                                |                                                                                       |
| **Alt + p** | 将移动到上一屏幕                            |                                                                                       |
| **Alt + w** | 焦点切换到 Entity Framework 版本选择 | 允许在项目中指定使用实体框架的不同版本。 |

### <a name="step-four-choose-your-database-objects-and-settings"></a>第四步： 选择数据库对象和设置

![向导的四个](~/ef6/media/wizardfour.png)

| 快捷键  | 操作                                                                                    | 说明                                                               |
|:----------|:------------------------------------------------------------------------------------------|:--------------------------------------------------------------------|
| **Alt + f** | 完成向导                                                                             |                                                                     |
| **Alt + p** | 将移动到上一屏幕                                                                   |                                                                     |
| **Alt + w** | 焦点切换到数据库对象选择窗格                                            | 允许指定数据库对象进行反向工程处理。    |
| **Alt + s** | 切换"确定所生成的对象名称或"选项                       |                                                                     |
| **Alt + k** | 切换"在模型中包括外键列"选项                              | 为所有选择模型内容的不可用。                 |
| **Alt + i** | 切换"导入选定的存储的过程和函数到实体模型"选项 | 为所有选择模型内容的不可用。                 |
| **Alt + m** | 将焦点切换到"模型 Namespace"文本字段                                        | 为所有选择模型内容的不可用。                 |
| 空格键 | 在元素上切换所选内容                                                               | 如果元素没有子级，将同时切换所有子元素 |
| 左侧  | 折叠子树                                                                       |                                                                     |
| 右侧 | 展开子树                                                                         |                                                                     |
| **最多**    | 导航到上一个元素树中                                                      |                                                                     |
| **向下**  | 导航到树中的下一个元素                                                          |                                                                     |

## <a name="ef-designer-surface"></a>EF 设计器图面

![设计器图面](~/ef6/media/designersurface.png)

| 快捷键                                                                                | 操作                      | 说明                                                                                                                                                                                                                               |
|:----------------------------------------------------------------------------------------|:----------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **空间或输入**                                                                         | 切换所选内容            | 上切换选择具有焦点的对象。                                                                                                                                                                                         |
| **Esc**                                                                                 | 取消选择            | 取消当前选择。                                                                                                                                                                                                      |
| **Ctrl + A**                                                                            | 全选                  | 选择设计图面上的所有形状。                                                                                                                                                                                       |
| **向上键**                                                                            | 上移                     | 将选定了一个网格增量的实体。 <br/> 如果在列表中，将移动到上一个同级子字段。                                                                                                                            |
| **向下键**                                                                          | 下移                   | 将选定的下一个网格增量的实体。 <br/> 如果在列表中，将移动到下一个同级子字段。                                                                                                                              |
| **向左键**                                                                          | 左移                   | 将所选实体左侧的一个网格增量。 <br/> 如果在列表中，将移动到上一个同级子字段。                                                                                                                          |
| **向右键**                                                                         | 向右移动                  | 将所选实体右移一个网格增量。 <br/> 如果在列表中，将移动到下一个同级子字段。                                                                                                                             |
| **Shift + 向的左键**                                                                  | 左大小形状             | 通过一个网格增量减少所选实体的宽度。                                                                                                                                                                     |
| **Shift + 向右键**                                                                 | 大小形状右侧            | 通过一个网格增量增加所选实体的宽度。                                                                                                                                                                   |
| **主文件夹**                                                                                | 第一个对等                  | 将焦点和选择内容转换为同一级别的对等方在设计图面上的第一个对象。                                                                                                                                         |
| **End**                                                                                 | 最后一个对等                   | 将焦点和选择内容转换为同一级别的对等方在设计图面上的最后一个对象。                                                                                                                                          |
| **Ctrl + Home**                                                                         | 第一个对等 （焦点）          | 与第一个对等方，但将焦点移动而不是移动焦点和选择内容相同。                                                                                                                                                          |
| **Ctrl + 结束**                                                                          | 最后一个对等 （焦点）           | 与上次相同对等互连，但移动焦点而不是移动焦点和选择内容。                                                                                                                                                           |
| Tab                                                                                 | 下一步的对等                   | 将焦点和选择内容转换为同一级别的对等方在设计图面上的下一个对象。                                                                                                                                          |
| **Shift+Tab**                                                                           | 上一个对等               | 将焦点和选择内容转换为同一级别的对等方在设计图面上的上一个对象。                                                                                                                                      |
| **Alt + Ctrl + Tab**                                                                        | 下一步的对等 （焦点）           | 下一步相同对等互连，但移动焦点而不是移动焦点和选择内容。                                                                                                                                                           |
| **Alt + Ctrl + Shift + Tab**                                                                  | 上一个对等 （焦点）       | 与上一个对等方，但将焦点移动而不是移动焦点和选择内容相同。                                                                                                                                                       |
| **&lt;**                                                                                | 中上升一个级别                      | 移动到层次结构中更高一级设计图面上的下一个对象。 如果没有层次结构中此形状的上方的任何形状 （即，该对象置于直接在设计图面上），选择该关系图。 |
| **&gt;**                                                                                | 降序                     | 移动到上一级设计图面的下面这个层次结构中的下一步包含对象。 如果不有任何包含的对象，这不是执行任何操作。                                                                              |
| **Ctrl + &lt;**                                                                         | 递增 （焦点）              | 与相同中上升一个级别命令，但没有选择情况下将焦点移。                                                                                                                                                                          |
| **Ctrl + &gt;**                                                                         | 降序 （焦点）             | 与相同降命令，但没有选择情况下将焦点移。                                                                                                                                                                         |
| **Shift + 结束**                                                                         | 按照已连接到操作         | 从实体中，将移动到此实体连接到的实体。                                                                                                                                                               |
| Delete                                                                                 | 删除                      | 从关系图中删除的对象或连接器。                                                                                                                                                                                     |
| **项**                                                                                 | Insert                      | 选择了"标量属性"隔离舱标头或属性本身时，将新属性添加到实体。                                                                                                           |
| **向上翻页**                                                                               | 向上滚动关系图           | 将设计图面上，滚动以等于当前可见的设计图面上的高度的 75%的增量。                                                                                                                    |
| **向下翻页**                                                                             | 向下滚动关系图         | 向下滚动设计图面。                                                                                                                                                                                                    |
| **Shift + 向下的翻页**                                                                     | 右侧的滚动关系图        | 滚动到右侧设计图面。                                                                                                                                                                                            |
| **Shift + 向上翻页**                                                                       | 剩余的滚动关系图         | 滚动左侧的设计图面。                                                                                                                                                                                             |
| F2                                                                                  | 进入编辑模式             | 文本控件进入编辑模式的标准键盘快捷方式。                                                                                                                                                               |
| **Shift + F10**                                                                         | 显示快捷菜单       | 显示选定的项的快捷菜单的标准键盘快捷方式。                                                                                                                                                          |
| **Control + Shift + 鼠标左键单击**  <br/> **控制 + Shift + 鼠标滚轮转发**  | 中的语义式缩放            | 放大关系图视图下的鼠标指针的区域。                                                                                                                                                                 |
| **Control + Shift + 鼠标右键单击** <br/> **Control + Shift + 鼠标滚轮向后** | 语义缩放           | 缩小关系图视图在鼠标指针下的区域。 太远缩小为当前关系图中心时，它重新中心关系图。                                                                          |
| **控制 + Shift + '+'** <br/> **Control + 鼠标滚轮转发**                        | 放大                     | 放大关系图视图的中心。                                                                                                                                                                                         |
| **控制 + Shift +-** <br/> **Control + 鼠标滚轮向后**                       | 缩小                    | 缩小关系图视图的被单击区域。 太远缩小为当前关系图中心时，它重新中心关系图。                                                                                            |
| **控制 + Shift + 向下绘制一个具有鼠标左键的矩形**                  | 缩放区域                   | 放大所选的区域上居中。 当您按住 Control + Shift 键时，您将看到光标更改为一个放大镜，允许你定义要缩放到的区域。                        |
| **上下文菜单键 + 是**                                                              | 打开映射详细信息窗口 | 打开映射详细信息窗口，编辑所选实体的映射                                                                                                                                                               |

## <a name="mapping-details-window"></a>“映射详细信息”窗口

![映射详细信息的快捷方式](~/ef6/media/mappingdetailsshortcuts.png)

| 快捷键                  | 操作         | 说明                                                                                                                                 |
|:--------------------------|:---------------|:--------------------------------------------------------------------------------------------------------------------------------------|
| Tab                   | 上下文切换 | 主窗口区域与在左侧工具栏之间切换                                                                     |
| **箭头键**            | 导航     | 在主窗口区域中的列之间的行，或右侧和左侧中上下移动。 在左侧工具栏按钮之间移动。 |
| Enter <br/> 空格键 | 选择         | 在左侧工具栏中选择某个按钮。                                                                                          |
| **Alt + 向下键**      | 打开列表      | 下拉列表中，如果选择了单元格具有下拉列表。                                                                     |
| Enter                 | 列表中，选择    | 在下拉列表中选择元素。                                                                                               |
| **Esc**                   | 关闭列表     | 关闭下拉列表。                                                                                                              |

## <a name="visual-studio-navigation"></a>Visual Studio 导航

实体框架还提供了多个操作可以具有映射的自定义键盘快捷方式 （无快捷键映射默认情况下）。 若要创建这些自定义的快捷方式，请单击工具菜单，然后选项。  在环境中，选择键盘。  向下滚动直到您可以选择所需的命令，在"按快捷键"文本框中，输入该快捷方式，然后单击中间列表分配。 可能的快捷方式如下所示：

| 快捷键                                                                                       |
|:-----------------------------------------------------------------------------------------------|
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Add.ComplexProperty.ComplexTypes**        |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.AddCodeGenerationItem**                   |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.AddFunctionImport**                       |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.AddNew.AddEnumType**                      |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.AddNew.Association**                      |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.AddNew.ComplexProperty**                  |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.AddNew.ComplexType**                      |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.AddNew.Entity**                           |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.AddNew.FunctionImport**                   |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.AddNew.Inheritance**                      |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.AddNew.NavigationProperty**               |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.AddNew.ScalarProperty**                   |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.AddNewDiagram**                           |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.AddtoDiagram**                            |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Close**                                   |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Collapse**                                |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.ConverttoEnum**                           |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Diagram.CollapseAll**                     |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Diagram.ExpandAll**                       |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Diagram.ExportasImage**                   |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Diagram.LayoutDiagram**                   |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Edit**                                    |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.EntityKey**                               |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Expand**                                  |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.FunctionImportMapping**                   |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.GenerateDatabasefromModel**               |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.GoToDefinition**                          |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Grid.ShowGrid**                           |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Grid.SnaptoGrid**                         |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.IncludeRelated**                          |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.MappingDetails**                          |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.ModelBrowser**                            |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.MoveDiagramstoSeparateFile**              |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.MoveProperties.Down**                     |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.MoveProperties.Down5**                    |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.MoveProperties.ToBottom**                 |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.MoveProperties.ToTop**                    |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.MoveProperties.Up**                       |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.MoveProperties.Up5**                      |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.MovetonewDiagram**                        |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Open**                                    |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Refactor.MovetoNewComplexType**           |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Refactor.Rename**                         |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.RemovefromDiagram**                       |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Rename**                                  |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.ScalarPropertyFormat.DisplayName**        |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.ScalarPropertyFormat.DisplayNameandType** |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Select.BaseType**                         |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Select.Entity**                           |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Select.Property**                         |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Select.Subtype**                          |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.SelectAll**                               |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.SelectAssociation**                       |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.ShowinDiagram**                           |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.ShowinModelBrowser**                      |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.StoredProcedureMapping**                  |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.TableMapping**                            |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.UpdateModelfromDatabase**                 |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Validate**                                |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.10**                                 |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.100**                                |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.125**                                |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.150**                                |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.200**                                |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.25**                                 |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.300**                                |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.33**                                 |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.400**                                |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.50**                                 |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.66**                                 |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.75**                                 |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.Custom**                             |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.ZoomIn**                             |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.ZoomOut**                            |
| **OtherContextMenus.MicrosoftDataEntityDesignContext.Zoom.ZoomtoFit**                          |
| **View.EntityDataModelBrowser**                                                                |
| **View.EntityDataModelMappingDetails**                                                         |
