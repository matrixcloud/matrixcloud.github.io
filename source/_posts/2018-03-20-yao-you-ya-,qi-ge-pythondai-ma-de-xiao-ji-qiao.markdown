---
layout: post
title: "要优雅，七个Python代码的小技巧"
date: 2018-03-20 23:45:00 +0800
comments: true
categories: Python
tags: Python
---

**enumerate**

```python
cites = ['Shanghai', 'London', 'New York', 'Beijing']
# bag way
i = 0
for city in cites:
    print(i, city)
    i += 1
# good way
for i, city in enumerate(cites):
    print(i, city)
```

**zip**

```python
x_list = [1, 3, 5]
y_list = [2, 4, 8]
# bad way
for i in len(x_list):
    print(x_list[i], y_list[i])
# good way
zipped = zip(x_list, y_list)
for x, y in zipped:
    print(x, y)
```

**swap**

```python
# bad way
x = 10
y = -10
tmp = x
x = y
y = tmp
print('x = %s, y = %s' % (x, y))

# good way
x = 10
y = -10
x, y = y, x
print('x = %s, y = %s' % (x, y))
```

<!--more-->

**dict default value**

```python
ages = {
    'nick': 22,
    'mary': 20
}
# bad way
if 'dick' in ages:
    age = ages['dick']
else:
    age = 'unknown'
print('dick age is %s' % age)
# good way
print('dick age is %s' % ages.get('dick', 'unkown'))
```

**if no break occurred**

```python
needle = 'd'
haystack = ['a', 'b', 'c']

# bad way
found = False
for ch in haystack:
    if ch == needle:
        print('Found')
        found = True
        break
if not found:
    print('Not found')
# good way
for ch in haystack:
    if ch == needle:
        print('Found')
        break
else: # if no break occurred
    print('Not found')
```

**read line**

```python
# bad way
f = open('story.txt')
text = f.read()
for line in text.split('\n'):
    print(line)
f.close()
# good way
with open('story.txt') as f:
    for line in f:
        print(line)
```

**try except else**

```python
try:
    int('d')
except:
    print('conversion failed')
else: # if no except
    print('conversion success')
finally:
    print('done')
```

> 参考：[视频](https://www.youtube.com/watch?v=VBokjWj_cEA)