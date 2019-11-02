---
title: Guava 中好用的集合类型
date: 2019-07-06 07:06:04
tags: Java
---

## Multiset

让我们先思考一下，给出一个包含单词的集合 `words = ["guava", "apple", "banana", "guava", "apple", "watermelon"]`，设计一个词频统计程序，要求输出：`(guava, 2); (apple, 2); (banana, 1); (watermelon, 1)`。通常的写法如下，

```java
String[] words = {"guava", "apple", "banana", "guava", "apple", "watermelon"};
Map<String, Integer> counts = new HashMap<String, Integer>();

for (String word : words) {
  Integer count = counts.get(word);
  if (count == null) {
    counts.put(word, 1);
  } else {
    counts.put(word, count + 1);
  }
}
```

现在我们用 `Multiset` 去重构我们的代码。

```java
String[] words = {"guava", "apple", "banana", "guava", "apple", "watermelon"};
List<String> wordList = Arrays.asList(words);
Multiset<String> multiset = HashMultiset.create();
multiset.addAll(wordList);

String r = Joiner.on(", ")
        .join(multiset.elementSet().stream()
                .map(word -> String.format("(%s, %d)", word, multiset.count(word)))
                .collect(Collectors.toList()));
System.out.println(r);
```

在数学上 `set` 是无序但不可重复元素的集合，而 `multiset` 则是无序但可重复元素的集合。比如 `{a, a, b}` 



## Multimap

现在你需要把以下苹果按颜色分好类。

| Color | Weight |
|-------|------|
| CYAN | William |
| CLS_2 | 

```java
public static void performClassification(List<Apple> apples) {
    Multimap<Apple.Color, Apple> multimap = ArrayListMultimap.create();
    apples.forEach(apple -> multimap.put(apple.color, apple));

    multimap.keys().forEach(color -> System.out.println(multimap.get(color)));
}
```

## BiMap



## Table

## ClassToInstanceMap

## RangeSet

## RangeMap