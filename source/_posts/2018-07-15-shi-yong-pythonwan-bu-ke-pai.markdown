---
layout: post
title: "使用Python玩扑克牌"
date: 2018-07-15 22:09:20 +0800
comments: true
categories: Python
tags: Python
---

看完廖雪峰的《Python3 零起点》，也用`Flask`做了一个web application，也做了一些小爬虫来练习Python的使用。但是感觉对里面的一些用法和技巧了解还远远不够，于是Google了下Python的进阶书籍。然后，找到了`Fluent Python`，看完第一章立马就喜欢上了作者的行文。章一里作者介绍了使用Python来构建一副法国扑克牌，不得不赞叹Python的灵活与优雅。加上最近自己也在琢磨一些牌游戏的规则，故此一记。

{% img https://ws2.sinaimg.cn/large/006tKfTcgy1ftax5ypevej30cs0hxt9k.jpg 250 %}

其实法国扑克牌就是流传在国内的普通扑克牌，扑克牌的前身为`简化成52张卡牌的法国塔罗牌`，后来美国商人再加入了德国的Joker的2张`鬼牌`。

52张牌象征了全年52个星期。️黑体♠、红心♥、梅花♣️、方块♦与四季有关。每季13张，表示13个星期，其中13张牌的点数之和为91，所以每季也是91天。此外，全牌的红色代表了白天，黑色表示夜晚。

历史上，18世纪初的时候英国政府曾经征收过`扑克牌税`，黑桃A由政府印制，当时为了防伪，黑桃A做过各种复杂的设计，后来竟然成为了一种扑克牌传统。还记得小时候玩过：[24点](https://zh.wikipedia.org/wiki/24%E7%82%B9)、`吹牛`、[潜乌龟](https://zh.wikipedia.org/wiki/%E6%BD%9B%E7%83%8F%E9%BE%9C)...

首先，我们先用`namedtuple`定义`Card`，再定义`FrenchDeck`

```python
# python 构建一副法国牌
Card = collections.namedtuple('Card', ['rank', 'suit'])

class FrenchDeck:
  ranks = [str(i) for i in range(2, 11)] + list('AJQK')
  suits = 'spades diamonds clubs hearts'.split(' ')

  def __init__(self):
    self._cards = [Card(rank, suit) for suit in self.suits
                                    for rank in self.ranks]

  def __len__(self):
    return len(self._cards)

  def __getitem__(self, position):
    return self._cards[position]

  def __setitem__(self, position, card):
    self._cards[position] = card
```

<!--more-->

做一款牌游戏，最常见的操作有：洗牌，随机抽牌。这在Python中我们只需要一行代码就能搞定：

```python
deck = FrenchDeck()
rand_card = random.choice(deck)
shuffled_deck = random.shuffle(deck)
```

> 总之，强烈推荐`Fluent Python`这本书:D