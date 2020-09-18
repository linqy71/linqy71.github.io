---
title: 把bookmarks等用户数据重新导入chrome
date: 2019-02-16 19:24:44
tags: chrome 书签 浏览记录 导入
categories: 随笔
copyright: true
---

> 近日把电脑的机械硬盘换成了固态硬盘，因此需要把旧硬盘中的数据搬过来。重新下载一个chrome其实更方便，但是里面的书签以及浏览记录都会不见，如果有账号倒是不用担心这些问题。

1. 废了好大力气找到旧硬盘中的Chrome所在位置：`旧C盘:\Users\用户名\AppData\Local\Google\Chrome`，喜出望外地将整个文件夹复制到新硬盘中，启动chrome.exe，出现的却是一个全新的Chrome，不知道怎么回事。

2. 于是我删掉复制的文件夹，重新下载一个Chrome，它自动安装在了`C:\Program Files (x86)\Google\Chrome`，神奇的是在Chrome的安装位置下竟没有User Data这个文件夹，经过一番查找发现Chrome的用户数据存放在另外的位置：`C:\Users\用户名\AppData\Local\Google\Chrome`

3. 将旧硬盘中Chrome的User Data文件夹复制到`C:\Users\用户名\AppData\Local\Google\Chrome`，覆盖刚刚安装的Chrome的数据文件，(如果出现一些提示说无法移动 直接跳过就可以了)，然后重新启动Chrome

4. 这时候就会发现所有的书签以及浏览记录等数据全都回来了，你的Chrome又是从前爱的那个Chrome了。 

