---
title: "Typescript Tip"
date: 2021-08-11T20:41:52+08:00
lastmod: 2021-08-11T20:41:52+08:00
draft: false
toc: false
keywords: ["daily"]
description: "Typescript 的 keyof typeof keyin"
tags: ["TypeScript"]
author: "xuyou"
---

### `keyof typeof` 是什么意思

`keyof` 作用于 `object` 类型，并返回这个 `object` 的 `key` 的集合类型：

```ts
type Point = { x: number; y: number };
type P = keyof Point; // type '"x" | "y"'
```

`typeof` 操作符作用于 JavaScript 对象和作用于 TypeScript 类型的时候结果是不一样的：

- [JavaScript 的 `typeof` 操作符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof)会返回 `undefined` `object` `boolean` `number` `bigint` `string` `symbol` `function` 其中之一；
- [TypeScript 的 `typeof` 操作符](https://www.typescriptlang.org/docs/handbook/2/typeof-types.html)作用于 JavaScript 的对象时，返回的是更具体的对象类型。

```ts
const obj = {
  name: "aaa",
  key: "bbb",
};

type ObjType = typeof obj;
// type ObjType = { name: string; key: string; }

const newObj: ObjType = {
  name: 1111, // Type 'number' is not assignable to type 'string'.
};

console.log(typeof obj); // "object"
```

所以 `keyof typeof` 就是获取 JavaScript 对象的 `key` 的集合。

### 几个内置类型

{{% admonition info `Record<Keys, Type>` %}}

创建一个 `key` 类型是 `Keys`，`value` 类型是 `Type` 的对象类型。

```ts
interface CatInfo {
  age: number;
  breed: string;
}

type CatName = "miffy" | "boris" | "mordred";

const cats: Record<CatName, CatInfo> = {
  miffy: { age: 10, breed: "Persian" },
  boris: { age: 5, breed: "Maine Coon" },
  mordred: { age: 16, breed: "British Shorthair" },
};
```

{{% /admonition %}}

{{% admonition info `ReturnType<Type>` %}}

返回函数 `Type` 的返回类型。

```ts
type Fn = () => { a: number; b: number };

type Type1 = ReturnType<Fn>; // {a: number, b: number}

function fn() {
  return { a: 1, b: 2 };
}

type Type2 = ReturnType<fn>; // fn refers to a value, but is being used as a type here. Did you mean 'typeof f'?
type Type2 = ReturnType<typeof fn>; // {a: number, b: number}
```

{{% /admonition %}}

### 参考资料

- [What does “keyof typeof” mean in TypeScript?](https://stackoverflow.com/questions/55377365/what-does-keyof-typeof-mean-in-typescript)
- [Typeof Type Operator](https://www.typescriptlang.org/docs/handbook/2/typeof-types.html)
- [TypeScript 中限制对象键名的取值范围](https://zhuanlan.zhihu.com/p/373653890)
