---
Description: 使用指针动画向用户提供用户点击某个项目时的视觉反馈。
title: 指针单击动画
ms.assetid: EEB10A2C-629A-4705-8468-4D019D74DDFF
ms.date: 09/24/2020
ms.topic: article
keywords: windows 10, uwp
ms.localizationpriority: medium
ms.openlocfilehash: d429f55b6c8004ea0b5e16f842f5c7ee492d754d
ms.sourcegitcommit: eda7bbe9caa9d61126e11f0f1a98b12183df794d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91218844"
---
# <a name="pointer-click-animations"></a>指针单击动画



使用指针动画向用户提供用户点击某个项目时的视觉反馈。 当第一次点击某个项目时，指针向下动画会略微缩小和倾斜所按的项目，并且会进行播放。 当用户释放指针时，会播放指针向上动画，这会将该项目还原到其原始位置。


> **重要 API**：[**PointerUpThemeAnimation 类**](/uwp/api/Windows.UI.Xaml.Media.Animation.PointerUpThemeAnimation)、[**PointerDownThemeAnimation 类**](/uwp/api/Windows.UI.Xaml.Media.Animation.PointerDownThemeAnimation)


## <a name="dos-and-donts"></a>应做事项和禁止事项

-   在使用指针向上动画时，如果用户释放指针，将立即触发动画。 这样，即使点击触发的操作（如导航到新页面）的响应速度较慢，也可以为用户提供其操作已被识别的即时反馈。

## <a name="related-articles"></a>相关文章

* [动画概述](./xaml-animation.md)
* [创建指针单击动画](/previous-versions/windows/apps/jj649432(v=win.10))
* [快速入门：使用库动画创建 UI 动画](/previous-versions/windows/apps/hh452703(v=win.10))
* [**PointerUpThemeAnimation 类**](/uwp/api/Windows.UI.Xaml.Media.Animation.PointerUpThemeAnimation)
* [**PointerDownThemeAnimation 类**](/uwp/api/Windows.UI.Xaml.Media.Animation.PointerDownThemeAnimation)

 

 
