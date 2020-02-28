---
title: "编写第一个Tampermonkey脚本"
date: 2020-02-26T16:41:13+08:00
draft: false
Toc: false
Categories:
 - "Technique"
tags:
 - "Javascript"
---

由于最近沉迷RSS服务，在使用[RSShub](https://github.com/DIYgod/RSSHub)订阅不提供RSS服务的社交网站的时候通常需要获取用户的ID等信息，再添加上部署rsshub的服务器地址，甚是麻烦于是于是学习了一下油猴脚本编写，实际上也顺便学习js编程，更准确地说是Tampermonkey脚本。

<!--more-->

（实际上[RSShub Radar](https://github.com/DIYgod/RSSHub-Radar)项目已经提供了强大的浏览器插件，只是部分网站还是没有覆盖完整）

## Hello World

```javascript
// ==UserScript==
// @name         New Userscript
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  try to take over the world!
// @author       You
// @match        http://*/*
// @grant        none
// ==/UserScript==
```

在Tampermonkey插件管理界面新建一个脚本之后会自动包含一部分元数据。

- **@name** 表示拥护脚本的名字，会显示在脚本在安装对话框中。
- **@namespace** 命名空间用来区分名称相同但是作者不同的多个脚本，可以换成自己的域名。
- **@version** 表示脚本的版本，用于用户更新。
- **@description** 关于脚本功能的描述，不应超过两句。
- **@author** 作者
- **@match** 定义脚本要生效的URL地址，类似的**@include**和**@exclude**的含义类似，其中地址可以使用正则匹配
- **@require** 用于引用外部脚本比如jQuery等库，会在脚本运行之前加载
- **@grant** 标签我觉得相当于函数声明，用于调用获取`GM_*`方法，就是插件提供的API，更多可以参考[API手册]()
- **@resource** 标签没有自动生成，也是用来引入外部资源，比如CSS等

```js
(function() {
    'use strict';

    // Your code here...
})();
```

在元数据之后，会包含一个函数模版。我们只需要把自己的代码添加到`// Your code here`处即可，默认情况下其中的代码会在页面加载完成后被执行，比如增加一行`alert("hello world")`。（脚本注入时间可以用`@run-at`标签控制）

## 在网页上做一些改动

根据我们的需求首先需要增加一个按钮以触发动作，可以创建一个新的node并添加到body上即可。

```js
var new_node = document.createElement ('div');
new_node.innerHTML = `
	<button id="myButton">
		Button
	</button>`
new_node.setAttribute ('id', 'myContainer');
document.body.appendChild (new_node);

document.getElementById ("myButton").addEventListener (
  "click", ButtonClickAction, false
);
```

当然直接一个按钮可太丑了，这里为了方便后续使用css等，先创建了一个`<div>`标签，并绑定相应id。CSS需要使用`GM_*`函数添加，同时也可以配合`@resource`引入外部CSS。

```js
// @resource     material_icons https://...
// @grant        GM_addStyle
// @grant        GM_getResourceText
var material_icons = GM_getResourceText ("material_icons"); 
GM_addStyle(material_icons);
GM_addStyle("my css")
```

另外还用到了`unsafeWindow`可以直接访问网站的变量和函数(之前居然不知道js变量名可以以`$`开头，还以为是某种特殊运算符...)

然后...根据不同网站获取URL，正则表达式分割一下再拼接上Rsshub的域名就完成了...