---
title: Vue之slot深度复制
date: 2017-03-08 19:41:02
tags:
---


在Vue中，`slot`是一个很有用的特性，可以用来向组件内部插入一些内容。`slot`就是“插槽”的意思，用大白话说就是：定义组件的时候留几个口子，由用户来决定插入的内容。

例如我们定义一个组件`MyComponent`，其包含一个`slot`:
```
Vue.component('MyComponent', {
  template: `
    <div>
      <slot></slot>
    </div>
  `
})
```

当调用`<MyComponent>123</MyComponent>`时，会渲染为如下DOM结构：
```
<div>
  123
</div>
```

现在又有新需求了，我们希望调用`<MyComponent>123</MyComponent>`时，渲染出这样的DOM结构：
```
<div>
  123
  123
</div>
```

看起来很容易实现，即再为`MyComponent`添加一个`slot`:
```
Vue.component('MyComponent', {
  template: `
    <div>
      <slot></slot>
      <slot></slot>
    </div>
  `
})
```

渲染出的结构也确实如你所愿，唯一美中不足的是控制台有一个小小的Warning：
```
Duplicate presence of slot "default" found in the same render tree
```

如果你不是强迫症患者，这时候你可以收工安心回家睡觉了。直到有一天你的同事向你抱怨，为什么向`MyComponent`插入一个自定义组件会渲染不出来？

例如有一自定义组件`MyComponent2`：
```
Vue.component('MyComponent2', {
  template: `
    <div>456</div>
  `
})
```

当调用`<MyComponent><MyComponent2></MyComponent2></MyComponent>`时，预期渲染为如下DOM结构：
```
<div>
  <div>456</div>
  <div>456</div>
</div>
```

为什么不能正常工作呢？估计是前面的那个Warning搞得鬼，通过查询发现在Vue 2.0中不允许有重名的`slot`:
> 重名的 Slots 移除
同一模板中的重名 <slot> 已经弃用。当一个 slot 已经被渲染过了，那么就不能在同一模板其它地方被再次渲染了。如果要在不同位置渲染同一内容，可一用 prop 来传递。


文档中提示可以用`props`来实现，然而在我的用例中显然是不合适的。经过搜索后，最靠谱的方法是手写render函数，将`slot`中的内容复制到其他的位置。

将之前的`MyComponent`改为render函数的方式定义：
```
Vue.component('MyComponent', {
  render (createElement) {
    return createElement('div', [
      ...this.$slots.default,
      ...this.$slots.default
    ])
  }
})
```

在上面的定义中我们插入了两个`this.$slots.default`，测试下能不能正常工作。然而并没有什么卵用，Vue文档在render函数这一章有以下说明：
> VNodes 必须唯一
所有组件树中的 VNodes 必须唯一

这意味着我们不能简单地在不同位置引用`this.$slots.default`，必须对`slot`进行深度复制。深度复制的函数如下：
```
function deepClone(vnodes, createElement) {

  function cloneVNode (vnode) {
    const clonedChildren = vnode.children && vnode.children.map(vnode => cloneVNode(vnode));
    const cloned = createElement(vnode.tag, vnode.data, clonedChildren);
    cloned.text = vnode.text;
    cloned.isComment = vnode.isComment;
    cloned.componentOptions = vnode.componentOptions;
    cloned.elm = vnode.elm;
    cloned.context = vnode.context;
    cloned.ns = vnode.ns;
    cloned.isStatic = vnode.isStatic;
    cloned.key = vnode.key;

    return cloned;
  }

  const clonedVNodes = vnodes.map(vnode => cloneVNode(vnode))
  return clonedVNodes;
}
```

上面的核心函数就是`cloneVNode()`，它递归地创建VNode，实现深度复制。VNode的属性很多，我并不了解哪些是关键属性，只是参照着Vue的源码一并地复制过来。

基于以上函数，我们更改`MyComponent`的定义：

```
Vue.component('MyComponent', {
  render (createElement) {
    return createElement('div', [
      ...this.$slots.default,
      ...deepClone(this.$slots.default, createElement)
    ])
  }
})
```

经测试，一切正常。

## 总结

在Vue 1.0中重名的slots并不会出现什么问题，不知道为什么在2.0中取消了这个功能。我听说React提供了复制Element的标准函数，希望Vue也能提供这个函数，免得大家踩坑。
