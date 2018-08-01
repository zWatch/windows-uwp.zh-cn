---
author: jwmsft
Description: Learn how Fluent motion uses timing and easing functions.
title: 计时和缓动 - UWP 应用中的动画
label: Timing and easing
template: detail.hbs
ms.author: jimwalk
ms.date: 05/19/2017
ms.topic: article
ms.prod: windows
ms.technology: uwp
keywords: windows 10, uwp
pm-contact: stmoy
design-contact: jeffarn
doc-status: Draft
ms.localizationpriority: medium
ms.openlocfilehash: 412ba7e36c2bb36562ceee13bb1e204ff402a882
ms.sourcegitcommit: 517c83baffd344d4c705bc644d7c6d2b1a4c7e1a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2018
ms.locfileid: "1843739"
---
# <a name="timing-and-easing"></a>计时和缓动

虽然运动以现实世界为基础，但我们同时也是一个数字媒体，提供你期待的速度和效果。 

## <a name="how-fluent-motion-uses-time"></a>Fluent 运动如何使用时间

计时是让对象进入、退出或在 UI 内移动的运动感觉很自然的重要元素。

1. 进入视图的对象或场景很快，但是一种欢迎形式。 这些动画的持续时间通常比退出更长，以允许分层构建场景。
1. 退出视图的对象或场景非常快。 用户应该能够了解 UI 的去向。 但是，UI 消除后，它应该退出。
1. 跨场景转换的对象的持续时间应与它们移动的距离相对应。

## <a name="timing-in-fluent-motion"></a>Fluent 运动的计时

Fluent 的运动计时以 500 毫秒（或二分之一秒）作为基准，因为这是用户即时感知的最长时间。

![主图](images/time.gif)

### <a name="150ms-exit"></a>**150 毫秒**（退出）

:::行::: :::列::: 用于正在退出场景或关闭的对象或页面。
允许计时不会妨碍帧速率实现流畅动画的退出 UI 呈现非常快速的方向反馈。
:::列末::: :::列::: ![150 毫秒运动](images/150msAlt.gif) :::列末::: :::行末:::

### <a name="300ms-enter"></a>**300 毫秒**（进入）

:::行::: :::列::: 用于正在进入场景或打开的对象或页面。
允许内容进入场景时留有欢迎内容的合理时间量。
:::列末::: :::列::: ![300 毫秒运动](images/300ms.gif) :::列末::: :::行末:::

### <a name="500ms-move"></a>**≤500 毫秒**（移动）

:::行::: :::列::: 用于正在跨单个或多个场景转换的对象。 :::列末::: :::列::: ![500 毫秒运动](images/500ms.gif) :::列末::: :::行末:::

## <a name="easing-in-fluent-motion"></a>Fluent 运动的缓动

缓动是对对象的移动速度进行操作的方法。 它是将所有 Fluent 运动体验绑定在一起的纽带。 虽然比较极端，但系统中使用的缓动有助于统一对象在整个系统中移动的物理感觉。 这是模拟现实世界的一种方式，让运动中的对象感觉好像与其所在的环境融为一体。

![主图](images/easing.gif)

## <a name="apply-easing-to-motion"></a>对运动应用缓动

这些缓动将帮助你实现更自然的感觉，而且是我们用于 Fluent 运动的基准。

这些代码示例显示了如何向 Storyboard 动画 (XAML) 或合成动画 (C#) 应用推荐的缓动值。

### <a name="accelerate-exit"></a>**加快**（退出）

:::行::: :::列::: 用于正在退出场景的 UI 或对象。

        Objects become powered and gain momentum until they reach escape velocity.
        The resulting feel is that the object is trying its hardest to get out of the user's way and make room for new content to come in.
    :::column-end:::
    :::column:::
        ![accelerate easing](images/accelEase.gif)
    :::column-end:::
:::行末:::

```
cubic-bezier(0.7 , 0 , 1 , 0.5)
```

```xaml
<!-- Use for XAML Storyboard animations. -->
<Storyboard x:Name="Storyboard">
    <DoubleAnimation Storyboard.TargetName="Translation" Storyboard.TargetProperty="X" From="0" To="200" Duration="0:0:0.15">
        <DoubleAnimation.EasingFunction>
            <ExponentialEase Exponent="4.5" EasingMode="EaseIn" />
        </DoubleAnimation.EasingFunction>
    </DoubleAnimation>
</Storyboard>
```

```csharp
// Use for Composition animations.
CubicBezierEasingFunction accelerate =
    _compositor.CreateCubicBezierEasingFunction(new Vector2(0.7f, 0.0f), new Vector2(1.0f, 0.5f));
_exitAnimation = _compositor.CreateScalarKeyFrameAnimation();
_exitAnimation.InsertKeyFrame(0.0f, _startValue);
_exitAnimation.InsertKeyFrame(1.0f, _endValue, accelerate);
_exitAnimation.Duration = TimeSpan.FromMilliseconds(150);
```

### <a name="decelerate-enter"></a>**减速**（进入）

:::行::: :::列::: 用于正在进入场景（导航或生成）的对象或 UI。

        Once on-scene, the object is met with extreme friction, which slows the object to rest.
        The resulting feel is that the object traveled from a long distance away and entered at an extreme velocity, or is quickly returning to a rest state.

        Even if it's preceded by a moment of unresponsiveness, the velocity of the incoming object has the effect of feeling fast and responsive.
    :::column-end:::
    :::column:::
        ![decelerate easing](images/decelEase.gif)
    :::column-end:::
:::行末:::

```
cubic-bezier(0.1 , 0.9 , 0.2 , 1)
```

```xaml
<!-- Use for XAML Storyboard animations. -->
<Storyboard x:Name="Storyboard">
    <DoubleAnimation Storyboard.TargetName="Translation" Storyboard.TargetProperty="X" From="0" To="200" Duration="0:0:0.3">
        <DoubleAnimation.EasingFunction>
            <ExponentialEase Exponent="7" EasingMode="EaseOut" />
        </DoubleAnimation.EasingFunction>
    </DoubleAnimation>
</Storyboard>
```

```csharp
// Use for Composition animations.
CubicBezierEasingFunction decelerate =
    _compositor.CreateCubicBezierEasingFunction(new Vector2(0.1f, 0.9f), new Vector2(0.2f, 1.0f));
_enterAnimation = _compositor.CreateScalarKeyFrameAnimation();
_enterAnimation.InsertKeyFrame(0.0f, _startValue);
_enterAnimation.InsertKeyFrame(1.0f, _endValue, decelerate);
_enterAnimation.Duration = TimeSpan.FromMilliseconds(300);
```

### <a name="standard-easing-move"></a>**标准缓动**（移动）

:::行::: :::列::: 这是用于系统内所有动画参数更改的基准缓动。
为屏幕上发生状态改变（如简单的位置改变）的对象使用标准缓动。 此外，为在场景内变形（如生长的对象）的对象使用此缓动。

        The resulting feel is that objects changing state from A to B are overcoming, and taken over by, natural forces.
    :::column-end:::
    :::column:::
        ![standard easing](images/standardEase.gif)
    :::column-end:::
:::行末:::

```
cubic-bezier(0.8 , 0 , 0.2 , 1)
```

```xaml
<!-- Use for XAML Storyboard animations. -->
<Storyboard x:Name="Storyboard">
    <DoubleAnimation Storyboard.TargetName="Translation" Storyboard.TargetProperty="X" From="0" To="200" Duration="0:0:0.5">
        <DoubleAnimation.EasingFunction>
            <CircleEase EasingMode="EaseInOut" />
        </DoubleAnimation.EasingFunction>
    </DoubleAnimation>
</Storyboard>
```

```csharp
// Use for Composition animations.
CubicBezierEasingFunction standard =
    _compositor.CreateCubicBezierEasingFunction(new Vector2(0.8f, 0.0f), new Vector2(0.2f, 1.0f));
 _moveAnimation = _compositor.CreateScalarKeyFrameAnimation();
 _moveAnimation.InsertKeyFrame(0.0f, _startValue);
 _moveAnimation.InsertKeyFrame(1.0f, _endValue, standard);
 _moveAnimation.Duration = TimeSpan.FromMilliseconds(500);
```

## <a name="related-articles"></a>相关文章

- [运动概述](index.md)
- [方向性和引力](directionality-and-gravity.md)