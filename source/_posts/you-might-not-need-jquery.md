---
title: You might not need jquery (不用jquery话代码怎么写)
date: 2017-12-24 11:20:03
tags: [javascript, sites]
categories: sites
---
自从用了`vuejs`之后，对`jquery`越来越抵触了，项目里遇到什么好用的插件都想去掉`jquery`依赖重写一遍...

今天机缘巧合之下发现了[http://youmightnotneedjquery.com](http://youmightnotneedjquery.com/)这个网站，假如你也恰好想用原生`javascript`实现`jquery`某个函数的功能而又不知道怎么做的时候，或者只为了`jquery`某个函数而不想引入整个类库的时候，与其去翻`jquery`源码，不如来试试这个网站。

例如我要记得要获取某元素相对于文档根部(`document`)的位置信息，可以用`jquery`的`offset()`方法：
```javascript
$('#id').offset() // {left: 66, top: 666}
```
我想要知道纯js如何实现这个方法，在[http://youmightnotneedjquery.com](http://youmightnotneedjquery.com)里搜索一下，它就会告诉你可以这样实现`offset`：
```javascript
var rect = el.getBoundingClientRect();
{
  top: rect.top + document.body.scrollTop,
  left: rect.left + document.body.scrollLeft
}
```
原来也没怎么复杂嘛，它还会显示支持浏览的最低版本是IE8+，是不是很方便？