---
title: SVN安装与配置 (openSUSE-12.3)
date: 2016-03-19 10:05:31
tags: linux
---
### 方案一： 使用图形化设置工具
openSUSE集成有图形界面操作环境，安装软件包可以通过管理员设置工具 **YaST** (Yet another Setup Tool，SUSE特有的强大设置工具) ：

开始菜单 -> 系统 -> YaST -> Software -> Software Management -> 输入搜索关键字，如 `subversion`，选择结果安装即可。

![yast][1]

YaST几乎可完成所有的系统任务，比如：

* 安装和卸载软件（下章介绍）
* 配置防火墙
* 开启和禁用系统服务
* 软件安装源（添加、修改或禁用软件源）
* 用户和组管理（设置用户权限、添加删除用户）
* ...

> 如果您不想用YaST, 也可以用命令行手动编辑配置文件来完成相同的任务。

<!-- more -->

### 方案二： 使用终端中的 YaST
在终端中同样也可以使用 YaST，如用SSH远程登录或您的图形界面 环境坏掉了或压根没装图形界面。

在终端以 **root** 用户运行 yast 命令即可：

![yast][2]

通过方向键移动，Enter 键选择，Alt+[高亮字母] 为快捷键 (比如 Alt+Q 退出)。

> 安装软件包的方式与方案一一样，除了不能鼠标操作。

### 方案三： 使用命令行
openSUSE命令行中的包管理器入口为： `zypper`， 这里以安装SVN为例：

* 搜索软件包: `zypper se subversion`
* 安装软件包: `zypper in subversion`
* 卸载软件包: `zypper rm subversion`
* ...
更多命令可以直接输入 `zypper` 进行查看。

---
### 附上SVN的配置：
要创建一个 Subversion 版本库：

* 创建文件夹:
    `$ mkdir -p /srv/svn/repos`
* 运行创建版本库的命令:
    `$ svnadmin create /srv/svn/repos`
* 查看生成版本库的结构:
    `$ ls /srv/svn/repos`

如生成成功应该有:
```
/conf /format /hooks /locks README.txt
```
这样的文件结构， 表示你已经成功安装配置好SVN并可以使用它了。

最后， 让我们启动SVN服务：
`svnserve -d -r /srv/svn/repos`

---

### 最后的最后， 请不要忘记打开防火墙的 **3690** 端口，这是SVN默认的服务端口。


  [1]: https://lug.ustc.edu.cn/sites/opensuse-guide/images/screenshots/yast-controlcenter.png
  [2]: https://lug.ustc.edu.cn/sites/opensuse-guide/images/screenshots/yast-ncurses.png