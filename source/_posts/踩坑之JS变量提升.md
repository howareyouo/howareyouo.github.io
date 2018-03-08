---
layout: post
title: "踩坑之JS变量提升(Hoisting)"
date: 2018-03-07 20:48:46
tags: 踩坑
---

问题是在我这两天在重构我的[前端管理系统](https://gitee.com/backflow/framework-admin)的上传功能时候遇到的问题，大致要求是：
 - 用户选择文件后批量上传到七牛
 - 显示上传队列中各文件的文件名和上传进度
 - ...

代码如下:
``` javascript
// input[type=file]触发了change事件, 执行上传
function filechange (e) {
  for (var file of e.target.files) {
    var task = {
      complete: false
      percent: 0,
      loaded: 0,
      name: file.name,
      size: file.size
    }
    queue.unshift(task)
    qiniu.upload(file, key, token).subscribe({  // 开始上传
      next (res) => {   // 监听上传进度
        task.percent = res.total.percent
        task.loaded = res.total.loaded
      },
      error: (err) => console.log(err),
      complete (res) { 
        task.complete = true
      }
    })
  }
  e.target.value = '' // 清空value的值确保选择相同的文件可以正确触发change事件
}
```
咋一看代码很正常，相信有的同学根据需求第一时间会写出我这样的代码，那么问题出现了: 
