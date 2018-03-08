---
layout: post
title: "踩坑之JS变量提升(Hoisting)"
date: 2018-03-07 20:48:46
tags: 踩坑
---

问题是在我这两天在重构我的[前端管理系统](https://gitee.com/backflow/framework-admin)时候遇到的问题:
循环中声明的变量会变提升到
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

### Quick Start

``` javascript
for (var file of target.files) {
  var task = qiniu.upload(file, key, token)
  Object.assign(task, {
    percent: 0,
    loaded: 0,
    name: file.name,
    size: file.size
  })
  queue.unshift(task)
}
```

