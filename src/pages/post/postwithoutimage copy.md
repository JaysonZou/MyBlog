---
layout: ../../layouts/PostLayout.astro
title: "TS关键字 infer 的用法"
date: "2023-08-31 17:31"
categories:
  - TypeScript
---

# TS关键字 `infer` 的用法
在 TypeScript 中，`infer` 关键字通常与泛型条件类型一起使用，用于从条件类型中提取并推断类型信息。`infer` 关键字允许你在泛型条件类型中提取某个条件分支的类型，然后将其用作变量的类型。这在编写通用代码时非常有用，因为它可以帮助你在编译时获得更精确的类型信息。

以下是一个使用 `infer` 关键字的示例：

```typescript
type ExtractReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function add(a: number, b: number): number {
    return a + b;
}

type AddFunctionReturnType = ExtractReturnType<typeof add>; // 此时 AddFunctionReturnType 是 number 类型
```

在上面的示例中，我们定义了一个泛型类型 `ExtractReturnType<T>`，它接受一个类型 `T`，并使用条件类型来检查 `T` 是否是一个函数类型（`(...args: any[]) => infer R`）。如果 `T` 是函数类型，那么 `infer R` 就会提取函数的返回类型，并将其赋值给 `R`。如果 `T` 不是函数类型，那么返回 `never` 类型。

然后，我们定义了一个名为 `add` 的函数，并使用 `typeof add` 来获取函数的类型。接着，我们使用 `ExtractReturnType<typeof add>` 来提取 `add` 函数的返回类型，并将其赋值给 `AddFunctionReturnType`。因为 `add` 函数返回一个 `number`，所以 `AddFunctionReturnType` 最终被推断为 `number` 类型。

这个示例展示了如何使用 `infer` 关键字从条件类型中提取类型信息，以便在泛型中做更精确的类型推断。
