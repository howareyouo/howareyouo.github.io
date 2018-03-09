---
title: "踩坑之JS变量提升(Hoisting)"
date: 2018-2-21 20:48:46
tags: javascript, 踩坑
---
问题是在我这两天在重构我的[前端管理系统](https://gitee.com/backflow/framework-admin)的上传功能时候遇到的问题，大致要求是：
 - 用户选择文件后批量上传到七牛
 - 显示上传队列中各文件的文件名和上传进度
 - ...

代码像这样：
```javascript
/* input[type=file]触发了change事件, 执行上传 */
function filechange (e) {
  for (var file of e.target.files) {
    var task = {            // 建立上传任务对象
      complete: false,
      percent: 0,
      loaded: 0,
      name: file.name,
      size: file.size
    }
    queue.unshift(task)     // 插入队伍前端
    qiniu.upload(file, key, token).subscribe({  // 开始ajax异步上传
      next (res) {                              // 上传进度更新回调
        task.percent = res.total.percent        // 更新每个task的上传进度信息
        task.loaded = res.total.loaded
      },
      error: (err) => console.log(err),         // 上传错误回调
      complete (res) {                          // 上传完成回调
        task.complete = true                    // 标记每个task上传完成
        // post and save to db ...
      }
    })
  }
  e.target.value = ''   // 清空value的值确保选择相同的文件可以正确触发change事件
}
```
咋一看代码很正常，相信有的同学根据需求第一时间会写出我这样的代码。如果用相同逻辑的java代码去运行的话是没问题的，但js这么写的话就有问题了:
**回调程序永远只更新最后一个`unshift()`到队列的`task`的上传进度!**

<!-- more -->

造成这样的原因有二：
  - `javascript`没有`块级作用域`，只有`函数作用域`。（`for`循环后的花括号里并没有自己的作用域）
  - 变量提升(Hoisting)。(`for`循环里声明的变量会被提升到函数顶部)

实际上就像这样：
```javascript
function filechange (e) {
  var task  // 变量提升至函数体内顶部，作用于整个函数
  for (...) {
    task = {...}  // task指向新的对象
    queue.unshift(task)  // 插入队列前端
    qiniu.upload({...}).subscribe({  // 执行上传
      next (res) { // 上传为异步操作，这里的代码执行的时候循环已经结束
        // 这里对task的操作永远是队列里最前面那一个
        task.percent = res.total.percent
        task.loaded = res.total.loaded
      }
    })
  }
}
```
WOW...

既然找到问题了，那么怎么解决它呢？
如果允许使用ES6语法，用`let`或`const`在循环中声明`task`即可，不需要改动程序逻辑：
 - `let`声明的变量只在当前花括号的`块级作用域`内可见，它使得每次循环声明的`task`都是全新的对象。
 - `const`就更好了，它不但全新，还使声明的变量不可修改。

如果不允许使用ES6，需要避免`task`对象从`for`循环中提升，使其拥有自己单独的作用域，联想到前面提到过的js只有`函数作用域`这一特性，因此要为循环中的每个`task`创建自己的`函数作用域`。
做法就是单独创建一个`upload(file)`函数，在新的函数中创建`task`，并在循环中调用这个函数，最终的代码看起来就是这样：
```javascript
/* input[type=file]触发了change事件, 执行上传 */
function filechange (e) {
  for (var file of e.target.files) {
    upload(file)          // 传入file对象，开始上传
  }
  e.target.value = ''     // 清空value的值确保选择相同的文件可以正确触发change事件
}

/* 为task建立单独的函数作用域 */
function upload(file) {
    var task = {          // 声明新的task, 用不着提升了，我已经在顶部了，整个函数操作的task全都指向我
      complete: false,
      percent: 0,
      loaded: 0,
      name: file.name,
      size: file.size
    }
    queue.unshift(task)   // 插入队列前端
    qiniu.upload(file, key, token).subscribe({  // 开始上传
      next (res) {                              // 监听上传进度
        task.percent = res.total.percent
        task.loaded = res.total.loaded
      },
      error: (err) => console.log(err),
      complete (res) {
        task.complete = true
        // post and save to db ...
      }
    })
}
```
问题解决！

总结一下，这个问题其实曾经也遇到过，时间一久就忘记了，这次定位并解决这个BUG花了不少时间和精力，趁着这个机会写下这篇日记，总结一下血的教训， 那就是：

**永远不要写javascript代码的同时去写java代码呀 (╯‵□′)╯︵┻━┻ ！！**