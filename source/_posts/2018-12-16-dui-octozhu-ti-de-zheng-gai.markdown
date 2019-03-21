---
layout: post
title: "对Octo主题的整改"
date: 2018-12-16 14:34:56 +0800
comments: true
categories: 前端
tags: css
---

找了很久的GitHub Pages博客主题，最终还是`Octopress`符合我的口味。
用了一段时间，推荐给朋友用，然后由于朋友不是很懂前端的知识，问我是怎么调整文章缩进以及怎么给图片加弹出效果的。所以写在这里分享给有需要的朋友:D

## 自定义CSS

`Octopress`写Markdown文档的列表时，列表无法缩进，table不好用，行内代码块难看，quote域难看，有点难以忍受，只需在/sass/custom/_styels.css添加以下代码就能解决。

```
article {
    ul {
        margin-left: 2em;
    }
     blockquote {
        padding-left: 0.5em;
        font-size: 1em;
        font-style: normal;
    }
     code {
        color: #c7254e;
        background-color: #f9f2f4;
        border: none;
        border-radius: 4px;
    }
     table {
        th {
            border: 1px solid #ddd;
            font-weight: 700;
            padding: 0.1em 0.5em;
        }
         td {
            border: 1px solid #ddd;
            padding: 0.1em 0.5em;
            font-size: 0.9em;
        }
         tr:nth-of-type(even) {
            background-color: rgba(102,128,153,.05);
        }
    }
}
```

## 弹出层图片

做这个，我是借助Fancybox来做的。首先需要先把fancybox的js和css文件分别加入到` .themes/classic/source/javascripts/libs/`和`source/stylesheets/`目录下，
再在head部分引入css，到footer引入js。然后就能用了，使用示例如下：

```bash
{% img fancybox https://ws4.sinaimg.cn/large/006tKfTcgy1fph3cqu4ghj31kw0mwtbg.jpg %}
```

具体修改可以看下：[这里](https://github.com/matrixcloud/matrixcloud.github.io/commit/e63fb097a864b661ef8fc1b2fe36a7af7d63d6e3)

## 集成Travis CI

先去`Travis`[官网](https://travis-ci.org)开通下你的账号，然后根据以下步骤配置就好了。

<!--more-->

**1. 在`source`分支上新建`.travis.yml`**


```yaml
language: ruby

script: "bundle exec rake generate"

deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GITHUB_TOKEN
  keep-history: true
  local-dir: ./public
  on:
    all_branches: true
  target-branch: master
```

**2. 去Github官网里生成`token`，并确认以下权限**

![](/images/oct/github_token.png)

**3. 在Travis上配置好的`$GITHUB_TOKEN`**

![](/images/oct/travis_token.png)

## 其他

**首页文章显示more的分割指令**

```bash
<!--more-->
```

EOF