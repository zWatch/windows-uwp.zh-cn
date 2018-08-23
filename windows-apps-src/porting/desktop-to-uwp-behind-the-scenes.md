---
author: normesta
Description: 本文深入研究桌面桥深层次的工作原理。
title: 在桌面桥幕后
ms.author: normesta
ms.date: 05/25/2017
ms.topic: article
ms.prod: windows
ms.technology: uwp
keywords: windows 10, uwp
ms.assetid: a399fae9-122c-46c4-a1dc-a1a241e5547a
ms.openlocfilehash: 050499baaf383fc135d833ae1e4733c95f2b5fa1
ms.sourcegitcommit: 7540962003b38811e6336451bb03d46538b35671
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/26/2017
ms.locfileid: "617352"
---
# <a name="behind-the-scenes-of-the-desktop-bridge"></a>在桌面桥幕后

本文深入研究桌面桥深层次的工作原理。

桌面桥的主要目标是尽量将应用程序状态与系统状态分离，同时保留与其他应用的兼容性。 桥实现此目的的方式是，将应用程序放置在通用 Windows 平台 (UWP) 软件包内，然后检测并重定向它在运行时对文件系统和注册表所作的一些更改。

你为桌面应用创建的软件包是仅限桌面的、完全信任的应用程序，并且不虚拟化或沙盒化。 这使它们可以使用与经典桌面应用程序相同的方式与其他应用交互。

## <a name="installation"></a>安装

应用包安装在 *C:\Program Files\WindowsApps\package_name* 下，并且可执行文件的标题为 *app_name.exe*。 每个软件包文件夹都包含一个清单（名为 AppxManifest.xml），其中包含已打包应用的特殊 XML 命名空间。 该清单文件内部是一个 ```<EntryPoint>``` 元素，该元素引用完全信任的应用。 当启动该应用时，它不会在应用容器内部运行，而是像往常一样以用户身份运行。

部署后，软件包文件由操作系统标记为只读并严格锁定。 如果这些文件遭到篡改，Windows 将阻止应用启动。

## <a name="file-system"></a>文件系统

为了包含应用状态，桥尝试捕获应用对 AppData 所作的更改。 对 AppData 文件夹（例如 *C:\Users\user_name\AppData*）的所有写入（包括创建、删除和更新）都在写入时复制到专用的每用户、每应用位置。 这创造出已打包的应用正在编辑真实 AppData 的错觉，但实际上它在修改专用副本。 通过以这种方式重定向写入，系统可以跟踪应用所作的所有文件修改。 这使系统可以在卸载应用时清理这些文件，从而减少系统“腐烂”并为用户提供更好的应用删除体验。

除了重定向 AppData，桥还将 Windows 的著名文件夹（System32、Program Files (x86) 等）与应用包中的对应目录动态合并。 每个软件包都在其根目录中包含一个名为“VFS”的文件夹。 VFS 目录中目录或文件的任何读取都在运行时与其各自的本机对应项合并。 例如，应用可能包含 *C:\Program Files\WindowsApps\package_name\VFS\SystemX86\vc10.dll* 作为其应用包的一部分，但文件看起来可能安装在 *C:\Windows\System32\vc10.dll* 中。  这保留了与可能预期文件处于非软件包位置的桌面应用程序的兼容性。

不允许写入应用包中的文件/文件夹。 桥忽略对不属于软件包的文件和文件夹的写入，并且只要用户具有权限，便允许此操作。

### <a name="common-operations"></a>常见操作

以下简短的参考表显示了常见文件系统操作以及桥如何处理它们。

操作 | 结果 | 示例
:--- | :--- | :---
读取或枚举已知的 Windows 文件或文件夹 | *C:\Program Files\package_name\VFS\well_known_folder* 与本地系统对应项的动态合并。 | 读取 *C:\Windows\System32* 会返回 *C:\Windows\System32* 的内容以及 *C:\Program Files\WindowsApps\package_name\VFS\SystemX86* 的内容。
在 AppData 下写入 | 在写入时复制到每用户、每应用位置。 | AppData 通常为 *C:\Users\user_name\AppData*。  
在软件包内写入 | 不允许。 软件包为只读。 | 不允许在 *C:\Program Files\WindowsApps\package_name* 下写入。
在软件包外写入 | 由桥忽略。 如果用户具有权限，则允许。 | 如果软件包不包含 *C:\Program Files\WindowsApps\package_name\VFS\SystemX86\foo.dll*，并且用户具有权限，则允许对 *C:\Windows\System32\foo.dll* 的写入。

### <a name="packaged-vfs-locations"></a>打包的 VFS 位置

下表显示了为应用在系统上的哪个位置覆盖作为程序包一部分交付的文件。 你的应用将发现这些文件位于所列的系统位置中，但这些文件实际上位于 *C:\Program Files\WindowsApps\package_name\VFS* 内的重定向位置中。 根据 [**KNOWNFOLDERID**](https://msdn.microsoft.com/library/windows/desktop/dd378457.aspx) 常量确定 FOLDERID 位置。

系统位置 | 重定向位置（在 [PackageRoot]\VFS\ 下） | 支持的体系结构
 :--- | :--- | :---
FOLDERID_SystemX86 | SystemX86 | x86, amd64
FOLDERID_System | SystemX64 | amd64
FOLDERID_ProgramFilesX86 | ProgramFilesX86 | x86, amd6
FOLDERID_ProgramFilesX64 | ProgramFilesX64 | amd64
FOLDERID_ProgramFilesCommonX86 | ProgramFilesCommonX86 | x86, amd64
FOLDERID_ProgramFilesCommonX64 | ProgramFilesCommonX64 | amd64
FOLDERID_Windows | Windows | x86, amd64
FOLDERID_ProgramData | Common AppData | x86, amd64
FOLDERID_System\catroot | AppVSystem32Catroot | x86, amd64
FOLDERID_System\catroot2 | AppVSystem32Catroot2 | x86, amd64
FOLDERID_System\drivers\etc | AppVSystem32DriversEtc | x86, amd64
FOLDERID_System\driverstore | AppVSystem32Driverstore | x86, amd64
FOLDERID_System\logfiles | AppVSystem32Logfiles | x86, amd64
FOLDERID_System\spool | AppVSystem32Spool | x86, amd64

## <a name="registry"></a>注册表

桥处理类似于文件系统的注册表。 应用包含有一个 registry.dat 文件，该文件充当真实注册表中的 *HKLM\Software* 的逻辑等效项。 在运行时，此虚拟注册表将此配置单元的内容合并到原生系统配置单元，以提供两者的单一视图。 例如，如果 registry.dat 包含单个项“Foo”，则运行时 *HKLM\Software* 的读取也将显示为包含“Foo”（除了所有原生系统项）。

仅 *HKLM\Software* 下的项是软件包的一部分；*HKCU* 或注册表其他部分下的项不是。 不允许写入软件包中的项或值。 桥忽略对不属于软件包的项或值的写入，并且只要用户具有权限，便允许此操作。

HKCU 下的所有写入都在写入时复制到每用户、每应用位置。 在卸载清理方面，这提供了与桥处理文件系统相同的好处。 在传统上，卸载程序无法清除 *HKEY_CURRENT_USER*，因为注销用户的注册表数据已卸载并且不可用。

所有写入都在软件包升级期间保留，并且仅在完全删除应用时删除。

### <a name="common-operations"></a>常见操作

以下简短的参考表显示常见注册表操作以及桥如何处理它们。

操作 | 结果 | 示例
:--- | :--- | :---
读取或枚举 *HKLM\Software* | 软件包配置单元与本地系统对应项的动态合并。 | 如果 registry.dat 包含单个项“Foo”，则在运行时，*HKLM\Software* 的读取将显示 *HKLM\Software* 和 *HKLM\Software\Foo* 的内容。
在 HKCU 下写入 | 在写入时复制到每用户、每应用专用位置。 | 与文件的 AppData 相同。
在软件包内写入。 | 不允许。 软件包为只读。 | 如果软件包配置单元中存在对应的项/值，则不允许在 *HKLM\Software* 下写入。
在软件包外写入 | 由桥忽略。 如果用户具有权限，则允许。 | 只要软件包配置单元中不存在对应的项/值并且用户具有正确的访问权限，则允许在 *HKLM\Software* 下写入。

## <a name="uninstallation"></a>卸载

当用户卸载程序包时，删除位于 *C:\Program Files\WindowsApps\package_name* 下的所有文件和文件夹，并删除对 AppData 或桥捕获的注册表的任何重定向写入。

## <a name="next-steps"></a>后续步骤

**查找特定问题的答案**

我们的团队监视这些 [StackOverflow 标记](http://stackoverflow.com/questions/tagged/project-centennial+or+desktop-bridge)。

**提供关于本文的反馈**

请使用下面的评论区。