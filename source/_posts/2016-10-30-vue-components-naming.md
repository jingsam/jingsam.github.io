---
title: 聊聊 Vue 组件命名那些事
date: 2016-10-30 15:56:01
tags:
---

> There are only two things in Computer Sciences: cache invalidation and naming things.
—— Phil Karlton

诚如上述所言，编程中变量命名确实令人很头疼。我们模糊地知道，Vue 组件的名称最好不要和原生 HTML 标签相同。为了避免重名，通常会在组件名称前面加上一个前缀，如 `el-button`、`el-input`、`el-date-picker`。这通常不会有什么问题，但有时候你的模板中混杂了原生 HTML 标签和组件标签，要想区分它们并不是很容易。

当我看到 Ant.design 的 React 组件是下面这样的时候，我感觉到一种自由的味道。首先，组件名可以使用原生 HTML 标签名，意味着再也不用较劲脑汁去规避原生 HTML 标签了。另外，这些组件都使用了首字母大写标签名，使它们很容易地与原生小写的 HTML 标签区分。
```js
ReactDOM.render(
  <div>
    <Button type="primary">Primary</Button>
    <Input placeholder="Basic usage" />
    <Select defaultValue=".com" style={{ width: 70 }}>
      <Option value=".com">.com</Option>
      <Option value=".jp">.jp</Option>
      <Option value=".cn">.cn</Option>
      <Option value=".org">.org</Option>
    </Select>
  </div>,
  mountNode
);
```

受 Ant.design 的启发，我思考 Vue 组件命名能不能达到同样的效果呢？要找到答案，必须摸清楚 Vue 组件命名到底有什么限制。下面将分别从 Vue 1.0 和 Vue 2.0 来谈谈组件命名的机制：

## Vue 1.0 组件命名机制

### 组件注册

我们以一个最简单的例子来研究 Vue 组件的注册过程：
```js
Vue.component('MyComponent', {
  template: '<div>hello, world</div>'
})
```

通过跟踪代码的执行过程，发现对组件的名称有两处检查。

1. 检查名称是否与 HTML 元素或者 Vue 保留标签重名，不区分大小写。可以发现，只检查了常用的 HTML 元素，还有很多元素没有检查，例如 `button`、`main`。
```js
if (type === 'component' && (commonTagRE.test(id) || reservedTagRE.test(id))) {
  warn('Do not use built-in or reserved HTML elements as component ' + 'id: ' + id);
}

// var commonTagRE = /^(div|p|span|img|a|b|i|br|ul|ol|li|h1|h2|h3|h4|h5|h6|code|pre|table|th|td|tr|form|label|input|select|option|nav|article|section|header|footer)$/i;
// var reservedTagRE = /^(slot|partial|component)$/i;
```

2. 检查组件名称是否以字母开头，后面跟字母、数值或下划线。
```js
if (!/^[a-zA-Z][\w-]*$/.test(name)) {
  warn('Invalid component name: "' + name + '". Component names ' + 'can only contain alphanumeric characaters and the hyphen.');
}
```

基于以上两点，可以总结出组件的命名规则为：组件名以字母开头，后面跟字母、数值或下划线，并且不与 HTML 元素或 Vue 保留标签重名。

然而我们注意到，在上面的检查中，不符合规则的组件名称是 warn 而不是 error，意味着检查并不是强制的。实际上，**Vue 组件注册的名称是没有限制的**。你可以用任何 JavaScript 能够表示的字符串，不管是数字、特殊符号、甚至汉字，都可以成功注册。

### 模板解析

虽然 Vue 组件没有命名限制，但是我们终究是要在模板中引用的，不合理的组件名可能会导致我们无法引用它。

为了弄清楚 Vue 是如何将模板中的标签对应到自定义组件的，我们以一段简单的代码说明：
```js
new Vue({
  el: '#app',
  template: '<my-component></my-component>'
})
```

总体来说，模板解析分为两个过程：

首先，Vue 会将 `template` 中的内容插到 DOM 中，以方便解析标签。由于 HTML 标签不区分大小写，所以在生成的标签名都会转换为小写。例如，当你的 `template` 为 `<MyComponent></MyComponent>` 时，插入 DOM 后会被转换为 `<mycomponent></mycomponent>`。

然后，通过标签名寻找对应的自定义组件。**匹配的优先顺序从高到低为：原标签名、camelCase化的标签名、PascalCase化的标签名。**例如 `<my-component>` 会依次匹配 my-component、myComponent、MyComponent。camelCase 和 PascalCase 的代码如下：
```js
var camelizeRE = /-(\w)/g;

function camelize(str) {
  return str.replace(camelizeRE, toUpper);
}

function toUpper(_, c) {
  return c ? c.toUpperCase() : '';
}

function pascalize(str) {
  var camelCase = camelize(str);
  return camelCase.charAt(0).toUpperCase() + camelCase.slice(1)
}
```

对于一个 Vue 新手，经常对以下示例代码不能正常运行感到非常疑惑：
```js
Vue.component('MyComponent', {
  template: '<div>hello, world</div>'
})

new Vue({
  el: '#app',
  template: '<MyComponent></MyComponent>'
})
```

如果我们按照模板解析的过程推理，就很好解释了。模板 `<MyComponent></MyComponent>` 插入到 DOM 后会变成 `<mycomponent></mycomponent>`。标签 mycomponent 匹配的组件依次为 mycomponent（原标签名）、mycomponent（camelCase形式）、Mycomponent（PascalCase形式），并没有匹配到注册的组件名 MyComponent，所以会报找不到组件 <mycomponent> 的警告。

### 命名限制

通过分析组件注册和模板解析的过程，发现 Vue 组件命名限制并没有我们想象得多。大家可以尝试一下各种命名，我试过 `<a_=-*%按钮></a_=-*%按钮>` 都可正常运行。

但是，并不意味着完全没有限制。由于在模板需要插入到 DOM 中，所以模板中的标签名必须能够被 DOM 正确地解析。主要有三种情况：一是完全不合法的标签名，例如 </>；二是与 HTML 元素重名会产生不确定的行为，例如使用 input 做组件名不会解析到自定义组件，使用 button 在 Chrome 上正常但在 IE 上不正常；三是与 Vue 保留的 slot、partial、component 重名，因为会优先以本身的意义解析，从而产生非预期的结果。

上述命名限制存在的根本原因，在于模板解析的过程依赖了 DOM。能不能对模板解析过程改进一下，使其不依赖于 DOM 呢？实际上，这正是 Vue 2.0 的主要改进，将模板解析过程使用 Virtual DOM 实现，使得组件命名更加灵活。


## Vue 2.0 组件命名机制

### 组件注册

Vue 2.0 的组件注册过程与 Vue 1.0 基本相同，只是 HTML 标签和 Vue 保留标签范围有些不同：
```js
// 区分大小写
var isHTMLTag = makeMap(
  'html,body,base,head,link,meta,style,title,' +
  'address,article,aside,footer,header,h1,h2,h3,h4,h5,h6,hgroup,nav,section,' +
  'div,dd,dl,dt,figcaption,figure,hr,img,li,main,ol,p,pre,ul,' +
  'a,b,abbr,bdi,bdo,br,cite,code,data,dfn,em,i,kbd,mark,q,rp,rt,rtc,ruby,' +
  's,samp,small,span,strong,sub,sup,time,u,var,wbr,area,audio,map,track,video,' +
  'embed,object,param,source,canvas,script,noscript,del,ins,' +
  'caption,col,colgroup,table,thead,tbody,td,th,tr,' +
  'button,datalist,fieldset,form,input,label,legend,meter,optgroup,option,' +
  'output,progress,select,textarea,' +
  'details,dialog,menu,menuitem,summary,' +
  'content,element,shadow,template'
);

// 不区分大小写
var isSVG = makeMap(
  'svg,animate,circle,clippath,cursor,defs,desc,ellipse,filter,font,' +
  'font-face,g,glyph,image,line,marker,mask,missing-glyph,path,pattern,' +
  'polygon,polyline,rect,switch,symbol,text,textpath,tspan,use,view',
  true
);

var isReservedTag = function (tag) {
  return isHTMLTag(tag) || isSVG(tag)
};

// 区分大小写
var isBuiltInTag = makeMap('slot,component', true);
```

虽然 HTML 元素重名警告的标签数大大增加了，但重要的是重名区分大小写，所以我们可以愉快地使用 Input、Select、Option 等而不用担心重名。这个功劳属于 Vue 2.0 引入的 Virtual DOM。

### 模板解析

前面提到，Vue 2.0 相对于 1.0 的最大改进就是引入了 Virtual DOM，使模板的解析不依赖于 DOM。

使用 Virtual DOM 解析模板时，不必像 DOM 方式那样将模板中的标签名转成小写，而是原汁原味地保留原始标签名。然后，使用原始的标签名进行匹配组件。例如，`<MyComponent></MyComponent>` 不会转为为小写形式，直接以 MyComponent 为基础开始匹配。当然，匹配的规则与 1.0 是一样的，即依次匹配：原标签名、camelCase化的标签名、PascalCase化的标签名。

之前在 1.0 不能正常运行的示例代码，在 2.0 中可以正常运行了：
```js
Vue.component('MyComponent', {
  template: '<div>hello, world</div>'
})

new Vue({
  el: '#app',
  template: '<MyComponent></MyComponent>'
})
```

在 Vue 1.0 和 2.0 中还有一种定义组件模板的方式，即使用 DOM 元素。在这种情况下，解析模板时仍然会将标签转为小写形式。所以下面的代码，在 1.0 和 2.0 均不能正常运行。
```js
// index.html
<div id="app">
  <MyComponent></MyComponent>
</div>

// main.js
Vue.component('MyComponent', {
  template: '<div>hello, world</div>'
})

new Vue({
  el: '#app'
})
```

### 命名限制

Vue 2.0 中组件的命名限制与 1.0 的最大区别在于区分了大小写。总结一下就是：一是不使用非法的标签字符；二是不与 HTML 元素（区分大小写）或 SVG 元素（不区分大小写）重名；三是不使用 Vue 保留的 slot 和 component（区分大小写）。

除了以上三条，由于 Vue 2.0 内置了 KeepAlive、Transition、TransitionGroup 三个组件，所以尽量避免与这三个组件重名。但从另一方面讲，你也可以故意重名来实现一些特殊的功能。例如，keep-alive 的匹配顺序为 keep-alive、keepAlive、KeepAlive，所以我们可以注册一个 keep-alive 组件来拦截 KeepAlive 匹配。


## 总结

到这里，我们可以知道 Vue 2.0 完全可以像 React 那样使用 PascalCase 形式的组件标签。对于 Vue 1.0，想以 PascalCase 形态写模板，尽量以全小写或者仅首字母大写形式注册组件，例如 `<InputNumber>` 组件，可以注册为 inputnumber 或者 Inputnumber。但是，如果你想在 1.0 中使用 Input、Select 这类与 HTML 元素重名的标签名，基本上是无解的，所以是时候尝试下 Vue 2.0 了。
