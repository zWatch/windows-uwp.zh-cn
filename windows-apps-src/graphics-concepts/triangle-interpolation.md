---
title: "三角形内插"
description: "在呈现期间，管道会在每个三角形中内插顶点数据。"
ms.assetid: 1A76DD78-CED7-42BE-BA81-B9050CD3AF9B
keywords: "三角形内插"
author: PeterTurcan
ms.author: pettur
ms.date: 02/08/2017
ms.topic: article
ms.prod: windows
ms.technology: uwp
ms.openlocfilehash: c440a499de801c3da62b2c2f9d4e436b44d5ce28
ms.sourcegitcommit: 909d859a0f11981a8d1beac0da35f779786a6889
translationtype: HT
---
# <a name="triangle-interpolation"></a>三角形内插


在呈现期间，管道会在每个三角形中内插顶点数据。 顶点数据可以是各种各样的数据，并且可以包括（但不限于）：漫射颜色、反射颜色、漫射 alpha（三角形不透明度）、反射 alpha 和雾系数。 对于可编程的顶点管道，雾系数取自雾注册。 对于固定函数顶点管道，雾系数取自反射 alpha。

对于某些顶点数据，内插取决于当前阴影模式，如下所示：

| 阴影模式 | 描述                                                                                                                                                                 |
|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 平面         | 仅在平面阴影模式下内插雾系数。 对于所有其他内插值，将在整个面中应用三角形中第一个顶点的颜色。 |
| 高氏      | 将在所有三个顶点之间执行线性内插。                                                                                                               |

 

漫射颜色和反射颜色的处理方式不同，具体取决于颜色模式。 在 RGB 颜色模式中，系统将在内插中使用红色、绿色和蓝色组件。

颜色的 alpha 组件将被视为单独的内插值，因为设备驱动程序可通过两种不同的方式实施透明：使用纹理混合或点彩。

## <a name="span-idrelated-topicsspanrelated-topics"></a><span id="related-topics"></span>相关主题


[坐标系和几何图形](coordinate-systems-and-geometry.md)

 

 



