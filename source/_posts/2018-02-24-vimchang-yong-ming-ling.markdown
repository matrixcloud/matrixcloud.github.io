---
layout: post
title: "Vim常用命令"
date: 2018-02-24 21:56:11 +0800
comments: true
categories: Linux
---

本人非`Vim党`，亦不属于`Emacs圣教`，平常喜欢与`VSCode`玩耍。只是，在Linux服务器上配置时，避免不了使用Vi/Vim，记住一些命令，能提高编辑效率，故记之。

## 基本配置

```
set nu # 显示行号
syntax on # 启用语法高亮
# tab 4空格
set ts=4
set sw=4
set autoindent # 自动缩进
```

## 常用命令

- 剪切：`d`
- 剪切当前行: `dd`
- 复制当前行: `yy`
- 粘贴：`p`(lowercase 向下粘贴)、`P`(uppercase 向上粘贴)
- 多行注释
    1. 按`v`，进入Visual模式
    2. `j, k`选择要注释的行
    3. 按下大写`I`，进入插入模式
    4. 输入，注释符(//, #)
    5. ESC退出后，等一会才会出现多行注释
- 取消多行注释
    1. 按`v`，进入Visual模式
    2. `j, k`选择要取消注释的行
    3. 按下`x`或`d`
- 多行缩进
    1. 按`v`，进入Visual模式
    2. `j, k`选择要缩进的行
    3. `<`(左缩进)、`>`(右缩进)

> [Vim小游戏](https://vim-adventures.com/)