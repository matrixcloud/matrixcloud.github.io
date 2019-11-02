---
title: TypeScript 简明指南(上)
date: 2019-10-13 20:39:28
tags: TypeScript
---

## 为什么使用 TypeScript

TypeScript 是 JavaScript 的超集，它提供了一套完整的类型系统方便我们更好地编写 JavaScript 代码，即支持代码的编写时的静态检查、编译时的错误检查；由于类型系统的支持，也方便我们将以前的代码重构。当我们写完 TypeScript 需要将其编译成 JavaScript，这个翻译过程由 `TSC` (TypeScript Compiler) 完成，`TSC`主要包含以下几个核心组件：

- Scanner
- Parser
- Binder
- Checker
- Emitter

其处理流程大致如下。

![](/images/ts/tsc_processing.png)

## 类型系统

TypeScript 包含了以下基本类型，其中箭头方向的含义是指，头节点类型可以向下兼容尾节点类型。例如，`string` 可以向上转换成 `any` 类型，这个转变过程称为**型变 (Variance)**。

![](/images/ts/types.png)

### object

**型变 (Variance)** 我们之后来详解，我们先来看看 TypeScript 中变化较多的类型 `object`。
当我们声明如下的一个 `object` ，并来获取它的 `name` 属性。

```ts
let foo: object = {
  name: 'bar'
}
foo.name // Error TS2339: Property 'name' does not exist on type 'object'
```

这个错误产生的原因是因为 `object` 仅仅告诉你 `foo` 这个值是一个 `object`，而并没有描述任何关于这个值的结构。所以正确的做法应该如下，声明好 `foo` 的结构，这样的语法被称为 —— **object的字面量声明语法**。

```ts
let foo: {name: string} = {
  name: 'bar'
}
foo.name
```

TypeScript 作为使用**鸭子类型 (Ducking Typing)** 的语言以对于以下两种 `object` 的实例化都是等价的。

```ts
let c: {
  firstName: string
  lastName: string
} = {
  firstName: 'john',
  lastName: 'barrowman'
}

class Person {
  constructor(
    public firstName: string,  // 这里的声明等价于 this.firstName = firstName
    public lastName: string
  ) {}
}
c = new Person('matt', 'smith') // OK
```

> 当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。

我们定义对象 `c` 并让它的结构如下，来看看 `object` 声明的奇妙之处。

```ts
let c: {
   firtName: string,
   lastName?: string,
   [key: number]: boolean
}
```

- `lastName?`：表示这是个可选属性。
- `[key: number]`: 表示该对象可以拥有任意属性名为 `number` 类型的，其值为 `boolean` 类型的属性。

基于以上两点，我们可以初始化具有以下属性的 `c`。

```ts
c = {
  firtName: 'Matt',
  0: false,
  1: true
}
```

上面使用 `[key: number]: boolean` 这种语法被称为 **索引签名 (Index Signature)**，它告诉对于形如`c`这样的一个 `object`，所以 `number` 类型的 `key` 其值必须为 `boolean`。

> 索引签名的语法为：`[key: T]: U`，但需要注意的是 —— `key` 必须为 `number` 类型和 `string` 类型，索引签名让我们更加安全和方便的添加某一类型的属性。

思考下面这种写法会获得怎样的结果？

```ts
class Client {
  //...
}

class ClientInfo {
  //...
}

class ClientManager {
  private [key: Client]: ClientInfo

  public bind(client: Client, info: ClientInfo): void {
    this[client] = info
  }
}
```

### 类型别名

在 TypeScript 中你可以用 `type` 声明类型别名。

```ts
type Age = number
type Person = {
  name: string
  age: Age
}
```

同样，和变量一样，每个代码块和函数具有它自己作用域。如下所示，代码块内部的类型别名会覆盖外面的定义。

```ts
type Color = 'red'

let x = Math.random() < .5

if (x) {
  type Color = 'blue'  // 此处会覆盖上面的 Color 声明
  let b: Color = 'blue'
} else {
  let c: Color = 'red'
}
```

> TypeScript 支持字面量类型。

在 TypeScript 中，支持对类型使用 `|` 和 `&` 进行并交操作。

```ts
type Cat = {name: string, purrs: boolean}
type Dog = {name: string, barks: boolean, wags: boolean}
type CatOrDogOrBoth = Cat | Dog
type CatAndDog = Cat & Dog
```

### 型变（Variance）

在计算机科学及类型推断的语境中，型变（Variance）这个词表示的是，两种类型之间的关系是如何影响他们派生出的复杂类型之间的关系的，主要包含以下。

- 不变 (Invariance):
- 协变 (Covariance):
- 逆变（Contravariance): 

## 函数

### Generator

## 类与接口

