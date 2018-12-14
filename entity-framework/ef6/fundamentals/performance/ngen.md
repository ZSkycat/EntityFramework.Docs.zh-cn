---
title: 使用 NGen-EF6 以提高启动性能
author: divega
ms.date: 10/23/2016
ms.assetid: dc6110a0-80a0-4370-8190-cea942841cee
ms.openlocfilehash: 2e9a93c7741046da25b890fed084a6b280bf5643
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45489955"
---
# <a name="improving-startup-performance-with-ngen"></a>使用 NGen 以提高启动性能
> [!NOTE]
> **仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。 如果使用的是早期版本，则部分或全部信息不适用。  

.NET Framework 的托管应用程序支持的本机映像生成和库帮助应用程序启动速度更快的途径，而且还在某些情况下使用更少的内存。 将转换到之前执行该应用程序时，包含本机指令的文件的托管的代码程序集从而减轻无需生成的本机说明.NET JIT （实时） 编译器创建本机映像应用程序运行时。  

之前的版本 6，EF 运行时的核心库是.NET Framework 的一部分，并为其自动生成本机映像。 从版本 6 开始整个 EF 运行时已合并到 EntityFramework NuGet 包。 生成本机映像已从现在将使用 NGen.exe 命令行工具来获得类似结果。  

经验的观测值显示的 EF 运行时程序集的本机映像可以剪切的应用程序启动时间的 1 到 3 秒之间。  

## <a name="how-to-use-ngenexe"></a>如何使用 NGen.exe  

NGen.exe 工具的最基本功能是"安装"(即，创建并保存到磁盘) 的程序集和其直接依赖项的本机映像。 下面是如何实现此目的：  

1. 以管理员身份打开命令提示符窗口  
2. 将当前工作目录更改为你想要生成本机映像程序集的位置：  

  ``` console
    cd <*Assemblies location*>  
  ```
3. 具体取决于你的操作系统和应用程序的配置可能需要生成本机映像用于 32 位体系结构，64 位体系结构或同时用于两者。  

    适用于运行的 32 位：  
  ``` console
    %WINDIR%\Microsoft.NET\Framework\v4.0.30319\ngen install <Assembly name>  
  ```
    适用于运行 64 位：
  ``` console
    %WINDIR%\Microsoft.NET\Framework64\v4.0.30319\ngen install <Assembly name>  
  ```

> [!TIP]
> 正在生成本机映像的体系结构错误是一个非常常见的错误。 可以只需生成本机映像应用到计算机中安装操作系统的所有体系结构的有疑问。  

NGen.exe 还支持其他功能，如卸载和显示的已安装的本机映像，队列生成的多个图像，等等。有关使用情况的更多详细信息请阅读[NGen.exe 文档](https://msdn.microsoft.com/library/6t9t5wcf.aspx)。  

## <a name="when-to-use-ngenexe"></a>何时使用 NGen.exe  

就决定要在基于 EF 6 或更高版本的应用程序中生成本机映像的程序集，应考虑以下选项：  

- **主要 EF 运行时程序集，EntityFramework.dll**： 典型的基于 EF 应用程序到数据库从在启动时或在其首次访问此程序集执行大量的代码。 因此，创建此程序集的本机映像将生成的最大提高启动性能。  
- **应用程序使用的任何 EF 提供程序程序集**： 启动时间也可稍有受益于正在生成的这些本机映像。 例如，如果应用程序使用 SQL Server EF 提供程序将想要为 EntityFramework.SqlServer.dll 生成本机映像。  
- **应用程序的程序集和其他依赖项**: [NGen.exe 文档](https://msdn.microsoft.com/library/6t9t5wcf.aspx)介绍了有关选择要生成本机映像和安全性，本机映像的影响的程序集的一般准则高级选项，例如"硬绑定"，例如，使用本机映像中调试和分析方案等方案。  

> [!TIP]
> 请确保仔细测量使用本机映像上的启动性能和应用程序的整体性能的影响，并针对实际要求对它们进行比较。 本机映像通常会有帮助提高启动性能并在某些情况下降低内存使用情况，而不是所有情况下将同样受益。 例如，在稳定状态执行 （也就是说，一旦调用至少一次的应用程序正在使用的所有方法） 由 JIT 编译器生成的代码可以实际产生略有更好的性能比本机映像。  

## <a name="using-ngenexe-in-a-development-machine"></a>在开发计算机中使用 NGen.exe  

在开发.NET JIT 编译器将提供频繁更改的代码的最佳整体代价。 正在生成的已编译依赖项，如 EF 运行时程序集的本机映像可帮助加速开发和测试通过削减几秒钟，在每次执行的开始处。  

查找 EF 运行时程序集的好时机是解决方案的 NuGet 包位置。 例如，SQL Server 中使用 EF 6.0.2 和面向.NET 4.5 或更高版本的应用程序您可以键入以下命令提示符窗口中 （请记住将其打开以管理员身份）：  

``` console
cd <Solution directory>\packages\EntityFramework.6.0.2\lib\net45
%WINDIR%\Microsoft.NET\Framework\v4.0.30319\ngen install EntityFramework.SqlServer.dll
%WINDIR%\Microsoft.NET\Framework64\v4.0.30319\ngen install EntityFramework.SqlServer.dll
```  

> [!NOTE]
> 这会利用这一事实，安装 SQL Server 的 EF 提供程序的本机映像还会默认情况下安装的本机映像的主要 EF 运行时程序集。 这样做的原因 NGen.exe 可以检测到 EntityFramework.dll 是 EntityFramework.SqlServer.dll 程序集位于同一目录中的直接依赖项。  

## <a name="creating-native-images-during-setup"></a>在安装过程中创建本机映像  

WiX 工具包支持队列的本机映像生成托管程序集安装过程中，在此所述[操作方法指南](http://wixtoolset.org/documentation/manual/v3/howtos/files_and_registry/ngen_managed_assemblies.html)。 另一种方法是创建一个自定义安装程序任务，执行 NGen.exe 命令。  

## <a name="verifying-that-native-images-are-being-used-for-ef"></a>正在验证的 EF 使用本机映像  

你可以验证特定的应用程序通过查找具有扩展的加载程序集来使用本机程序集"。 ni.dll"。 ni.exe"。 例如，EF 的主运行时程序集的本机映像，将调用 EntityFramework.ni.dll。 若要检查的进程加载的.NET 程序集的简单方法是使用[Process Explorer](https://technet.microsoft.com/sysinternals/bb896653)。  

## <a name="other-things-to-be-aware-of"></a>需要注意的其他事项  

**创建本机映像的程序集不应混淆注册中的程序集[GAC （全局程序集缓存）](https://msdn.microsoft.com/library/yf1d93sz.aspx)**。 NGen.exe 允许创建不在 GAC 中的程序集的映像和事实上，使用 EF 的特定版本的多个应用程序可以共享相同的本机映像。 虽然 Windows 8 可以自动创建的程序集放置在 GAC 中本机映像，EF 运行时优化以与你的应用程序一起部署，建议不注册它在 GAC 中，因为这对程序集解析有负面影响和为您在其他方面之间的应用程序提供服务。  
