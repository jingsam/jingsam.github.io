---
title: 两个学习flexbox的小游戏
date: 2016-03-03 14:55:39
tags:
---

网页布局的传统方法是使用`dispaly`、`position`、`float`三个属性，实现起来相当麻烦。2009年，W3C提出flexbox布局方案，大大简化了布局的难度。如今，各大浏览器都支持flexbox布局方案，我们可以放心使用。

虽然flexbox降低了网页布局的难度，但是其也有上十种属性，想要完全掌握也不是轻而易举的。

相比于读枯燥的说明文档，通过游戏学习flexbox布局将更有趣、更易理解。本文将介绍两款学习flexbox布局的小游戏。


## Flexbox defense

[Flexbox defense][1]是一款塔防游戏，通过flexbox相关属性调整炮塔的位置，消灭行进中的怪物。游戏总共有10关，完成所有的关卡可以学习`justify-content`、`align-items`、`flex-direction`、`order`、`align-self`这5个属性。游戏比较简单，初学者10分钟左右就能通关。

![flexbox defense](/assets/flexbox-defense.png)


## Flexbox froggy
[Flexbox froggy][2]要求你通过设置flex属性，将青蛙移到相应颜色的荷叶上。游戏总共有24关，难度较前一个游戏高，不仅包括前面已经学习的5个属性，还增加了`flex-wrap`、`flex-flow`、`align-content`3个属性。但是，玩通前一个游戏后再玩这个游戏，也不算难。

![flexbox froggy](/assets/flexbox-froggy.png)

第21关的`align-content`属性容易与`align-items`相混淆，不易理解。`align-items`所调整的是元素整体的位置，而`align-content`调整的是元素整体中所包含的多行子元素间的布局关系。因此，如果元素整体只有1行，`align-content`属性是无效的。


## 更多资料
除了以上提到的8个flexbox属性外，还有`flex-grow`、`flex-shrink`、`flex-basis`、`flex`4个属性，详情请参考阮一峰先生的博文：
[Flex 布局教程：语法篇][3]
[Flex 布局教程：实例篇][4]


[1]: http://www.flexboxdefense.com/
[2]: http://flexboxfroggy.com/#zh-cn
[3]: http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html
[4]: http://www.ruanyifeng.com/blog/2015/07/flex-examples.html
