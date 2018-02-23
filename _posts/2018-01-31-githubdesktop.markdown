---
layout:     post
title:      "mac上GitHub Desktop客户端的使用"
subtitle:   " \"试试使用界面友好的git桌面工具\""
date:       2018-01-30 09:29:00
author:     "Leezw"
header-img: "img/post-bg-2015.jpg"
tags:
    - mac工具
---

通常想分享我们自己写一些酷炫的动画，或者优美的自定义控件时，大多都会想到将代码代码上传到github上，github推出客户端后，为我们提供了对github更加便捷的使用，同时还能实现versions版本控制的功能（虽然在mac上我们大多数情况下依然还是直接用git命令行的形式比较多），这里介绍下github的mac客户端上如何管理我们的代码

## **GitHub客户端下载**

1. mac客户端下载地址：[desktop.github.com](https://desktop.github.com)
2. 安装完后启动客户端，在程序的preference(偏好设置)中选择Account并登陆你的github账号便可成功关联。

## **GitHub Desktop的使用**

1. **将代码先导入到github desktop中管理**
左上角“+”号 ->选择“add”选项->点击“choose..”->选择上传文件,点击Open->点击Create&Add Reposiroty(创建仓库)
![](http://upload-images.jianshu.io/upload_images/1640181-2612569329d99729.gif?imageMogr2/auto-orient/strip)创建完后能在客户端的左侧的仓库列表中看到Other的目录下有我们刚上传的项目，并且前面是一个电脑的图标，表示当前项目只是存在本地电脑上。

2. **将仓库同步到github服务器上**
在左侧的列表中选择我们刚创建的仓库，点击页面右上角的"Publish"(发布)，输入代码的名称以及文字描述，确认下面选择的是你要上传的账号。
![](http://upload-images.jianshu.io/upload_images/1640181-0dd01fed38f55ae4.gif?imageMogr2/auto-orient/strip)在等待服务器上传完之后发现左侧的列表中项目从Other栏下进入了Github栏下，现在我们在github的网站上便可找到我们新建的仓库，我们可以右键我们的项目，选择第一个View On GitHub。（这一步只是提交了仓库，但是代码还没有提交。）

3. **提交代码**
选中我们的项目后，在中间下方输入一下提交描述，（xcode中的git也是，每次提交都要必须要写一下，不写不让上传）
![](http://upload-images.jianshu.io/upload_images/1640181-fff3ae09b258ec88.gif?imageMogr2/auto-orient/strip)提交完成后，原本提示的14Changes消失了，中间的列表变成了提交记录。在github上便能看到代码 ![](http://upload-images.jianshu.io/upload_images/1640181-cceb4da6fa9b3708.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 其他说明：
>1. 客户端右上角的圆圈表示提交记录时间点，其中最右边分裂成4个弧的大圆表示当前状态
 2. 圆之间的线段的长短与提交的时间差成正比
 3. 可以点击某个圆后，在中间的列表中点开数字，可以查看具体提交或文件修改的内容
 4. 每次有新的内容想要提交时，可以点击右上角最右边的大圆，或者中上方两个大按钮中“history”左侧的按钮。之后填写描述后点commit即可。
 5. 如果提交完后到网页上没有发现上传的代码，可以在客户端上查看右上角的是Public还是Sync，如果还是Public可以多点几次，直到变成Sync才表示代码已经上传。

4. **代码下载**
GitHub Desktop客户端除了可以上传外，也可以下载你账号下创建的仓库和你fork的一些开源项目。通过这一点可以实现办公室和家里代码同步，实现versions的功能。具体操作如下：
![](http://upload-images.jianshu.io/upload_images/1640181-7acc8a6ff87d5666.gif?imageMogr2/auto-orient/strip)Clone下的选项中为当前关联的账号中还没有还没有导入到本地电脑的项目。

5. **删除仓库**
想要把某个仓库删除，仅仅在客户端上通过邮件项目remove是不够的，这需要到网页上执行。来到项目首页，默认是code界面，我们选择最后一个Settings，然后把页面滑到最下面，选择红色框框中最后一条“Delete this repository”，点击按钮后，输入一遍这个要删除的仓库的名字（不区分大小写），然后点击最下面的按钮“I understand the consequences，delete the repository”后就删除成功了

##### **相关文章**

- [Getting Started with GitHub Desktop](https://help.github.com/desktop/guides/getting-started/)
- [Contributing to Projects with GitHub Desktop](https://help.github.com/desktop/guides/contributing/)
- [知乎上关于github的详细学习总结](http://www.zhihu.com/question/20070065)

---


