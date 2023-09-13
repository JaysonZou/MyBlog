---
layout: ../../layouts/PostLayout.astro
title: "在TypeScript中使用infer"
date: "2023-09-11"
categories:
  - typeScript
---

我们都曾遇到过这样的情况，即我们使用了一个很少键入的库。以下面的第三方函数为例：
```ts
function describePerson(person: {
  name: string;
  age: number;
  hobbies: [string, string]; // tuple
}) {
  return `${person.name} is ${person.age} years old and love ${person.hobbies.join(" and ")}.`；
}
```
如果函数库没有为 describePerson 的 person 参数提供独立类型，TypeScript 将无法正确推断出事先定义为 person 参数的变量。

```ts
const alex = {
  name: 'Alex',
  age: 20,
  hobbies: ['散步', '烹饪'] // type string[] != [string, string] 
}

describePerson(alex) /* 类型 string[] 不可赋值给类型 [string, string] */
```

TypeScript 将推断 alex 的类型为 `{ name: string; age: number; hobbies: string[] }`，不允许将其用作 describePerson 的参数。

而且，即使它允许，最好也能对 alex 对象本身进行类型检查，以实现正确的自动完成。借助 TypeScript 中的推断关键字，我们可以轻松做到这一点。
```ts
const alex: GetFirstArgumentOfAnyFunction<typeof describePerson> = {
  name: "Alex"、
  age: 20,
  hobbies: ["散步"、"烹饪"],
};

describePerson(alex); /* 没有 TypeScript 错误 */ 
```
通过 TypeScript 中的infer和条件类型，我们可以获取一个类型，并分离出其中的任何部分供以后使用。


# never 类型
在 TypeScript 中，never 被视为 "无值 "类型。你经常会看到它被用作一个死胡同类型。在 TypeScript 中，string | never 这样的联合类型会求值为 string，而舍弃 never。
要理解这一点，可以将 string 和 never 视为数学集合，其中 string 是一个包含所有字符串值的集合，而 never 是一个不包含任何值的集合。这两个集合的联合显然只有前者。
相比之下，union `string | any` 的值为 any。同样，你可以把它看作是字符串集合和包含所有集合的通用集合（U）的联合，而后者的求值结果也是它自己。
这就解释了为什么 never 被用作逃生舱口，因为与其他类型结合后，它就会消失。

# 在 TypeScript 中使用条件类型
条件类型根据类型是否满足特定约束来修改类型。它的工作原理类似于 JavaScript 中的三元运算符。

# extends 关键字
在 TypeScript 中，约束可以使用 extends 关键字来表达。T extends K 表示假设 T 类型的值也是 K 类型的值是安全的，例如 0 extends number，因为 var zero: number = 0 是类型安全的。

因此，我们可以使用泛型来检查是否满足约束，并返回不同的类型。

StringFromType 根据接收到的基元类型返回字面字符串：
```ts
type StringFromType<T> = T extends string ? string' : never

type lorem = StringFromType<'lorem ipsum'> // 'string' 类型
type ten = StringFromType<10> // never
```
为了涵盖 StringFromType 泛型的更多情况，我们可以像在 JavaScript 中嵌套三元运算符一样，链入更多条件。
```ts
type StringFromType<T> = T extends string
  ? '字符串
  : T extends boolean
  ? 布尔
  : T extends Error
  ? 错误
  never

type lorem = StringFromType<'lorem ipsum'> // '字符串' 类型
type isActive = StringFromType<false> // 'boolean' (布尔型)
type unassignable = StringFromType<TypeError> // 'error'（错误）。
```
# 条件类型和联合体
在扩展联合作为约束的情况下，TypeScript 将循环遍历联合的每个成员，并返回自己的联合：
```ts
type NullableString = string | null | undefined

type NonNullable<T> = T extends null | undefined ? never : T // 内置类型，仅供参考

type CondUnionType = NonNullable<NullableString> // 评估为`string`。
```
TypeScript 将测试 T extends null | undefined 约束，方法是循环遍历我们的联合（string | null | undefined），每次一个类型。

你可以把它想象成下面的示例代码：

stringLoop = string extends null | undefined ? never : string // string

nullLoop = null extends null | undefined ? never : null // never

undefinedLoop = undefined extends null | undefined ? never : undefined // never

类型 ReturnUnion = stringLoop | nullLoop | undefinedLoop // string
因为 ReturnUnion 是 string | never | never 的联合，所以它的值为字符串（见上文解释）。

您可以看到将扩展的 union 抽象到我们的泛型中，是如何让我们在 TypeScript 中创建内置的 Extract 和 Exclude 实用程序类型的：

type Extract<T, U> = T extends U ? T : 从不
type Exclude<T, U> = T extends U ? never : T
条件类型和函数
要检查类型是否扩展了某个函数形状，不能使用函数类型。相反，可以使用下面的签名来扩展所有可能的函数：

type AllFunctions = (...args: any[]) => any
...args: any[] 将涵盖零或更多参数，而 => any 将涵盖任何返回类型。

# 在 TypeScript 中使用infer
推断关键字是对条件类型的赞美，不能在 extends 子句之外使用。推断允许我们在约束中定义一个变量，以便引用或返回。

以 TypeScript 内置的 ReturnType 工具为例。它接收函数类型并给出其返回类型：
```ts
type a = ReturnType<() => void> // void
type b = ReturnType<() => string | number> // string | number
类型 c = ReturnType<() => any> // any
```

它首先检查你的类型参数（T）是否是一个函数，在检查过程中，返回类型被做成一个变量，推断为 R，如果检查成功则返回：

`type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any；`
如前所述，这主要用于访问和使用我们无法使用的类型。

React 的道具类型
在 React 中，我们经常需要访问道具类型。为此，React 提供了一种由推断关键字驱动的用于访问道具类型的实用程序类型，称为 ComponentProps。

类型 `ts
ComponentProps<
  T extends keyof JSX.IntrinsicElements | JSXElementConstructor<any>
> = T extends JSXElementConstructor<infer P> ?
  ? P
  : T extends keyof JSX.IntrinsicElements
  ? JSX.IntrinsicElements[T] ?
  : {}
`
在检查我们的类型参数是否为 React 组件后，它会推导出其道具并返回。如果失败，它会检查类型参数是否为 IntrinsicElements（div、按钮等），并返回其道具。如果全部失败，则返回 {}，在 TypeScript 中，这意味着 "任何非空值"。

# infer 的常见用法
使用推断关键字通常被描述为解包类型。以下是推断关键字的一些常见用法。

Function’s first argument:：
这是第一个示例中的解决方案：
```ts
type GetFirstArgumentOfAnyFunction<T> = T extends (
  first: infer FirstArgument,
  ...args: any[]
) => any
  ? FirstArgument
  : never

type t = GetFirstArgumentOfAnyFunction<(name: string, age: number) => void> // string
```
Function’s second argument:
```ts
type GetSecondArgumentOfAnyFunction<T> = T extends (
  first: any,
  second: infer SecondArgument,
  ...args: any[]
) => any
  ? SecondArgument
  : never

type t = GetSecondArgumentOfAnyFunction<(name: string, age: number) => void> // number
```
Promise ReturnType
```ts
type PromiseReturnType<T> = T extends Promise<infer Return> ? Return：T

type t = PromiseReturnType<Promise<string>> // string
```

## 数组类型
```ts
type ArrayType<T> = T extends (infer Item)[] ? Item ： T

type t = ArrayType<[string, number]> // string | number
```
# 结论
infer 是一个功能强大的工具，它允许我们在使用第三方 TypeScript 代码时解包和存储类型。在本文中，我们讲解了使用 never 关键字、extends 关键字、联合体和函数签名编写健壮的条件类型的各个方面。

