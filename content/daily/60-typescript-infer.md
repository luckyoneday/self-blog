---

title: "Typescript 条件类型和 infer"
date: 2023-07-12T15:12:20+08:00
lastmod: 2023-07-12T15:12:20+08:00
draft: false
toc: false
keywords: ["daily"]
description: "Typescript 条件类型和 infer 推断"
tags: ["Typescript"]
author: "xuyou"

---

> 推荐一个学习 typescript 的网站： [https://www.typescript-training.com/](https://www.typescript-training.com/)

> 再推荐一个 [ts-challenges](https://github.com/type-challenges/type-challenges)


## 条件类型

举个🌰：

```typescript
type Diff<T, U> = T extends U ? never : T; // 移除 T 中可以赋值给 U 的类型
type Filter<T, U> = T extends U ? T : never; // 移除 T 中不可以赋值给 U 的类型
type NonNullable<T> = Diff<T, null | undefined>; // 移除 T 中为 null 或 undefined 的类型

type T30 = Diff<"a" | "b" | "c" | "d", "a" | "c" | "f">; // "b" | "d"
type T31 = Filter<"a" | "b" | "c" | "d", "a" | "c" | "f">; // "a" | "c"

type T34 = NonNullable<string | number | undefined>; // string | number
type T35 = NonNullable<string | string[] | null | undefined>; // string | string[]
```

条件类型通常会结合 [映射类型](/daily/54-typescript-mapped-types) 一起使用：

```typescript
type FunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T];
type FunctionProperties<T> = Pick<T, FunctionPropertyNames<T>>;

type NonFunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? never : K;
}[keyof T];
type NonFunctionProperties<T> = Pick<T, NonFunctionPropertyNames<T>>;

interface Part {
  id: number;
  name: string;
  subparts: Part[];
  updatePart(newName: string): void;
}
type T40 = FunctionPropertyNames<Part>; // "updatePart"
type T41 = NonFunctionPropertyNames<Part>; // "id" | "name" | "subparts"
type T42 = FunctionProperties<Part>; // { updatePart(newName: string): void }
type T43 = NonFunctionProperties<Part>; // { id: number, name: string, subparts: Part[] }
```

## `infer` 关键字

可以使用 `infer` 来定义要推断的类型变量。例如获取函数的返回类型：

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

type T0 = ReturnType<() => string>; // string
```

获取属性类型：

```typescript
type Foo<T> = T extends { a: infer U; b: infer U } ? U : never;

type T1 = Foo<{ a: string; b: number }>; // string | number
```

获取函数参数类型：

```typescript
type Bar<T> = T extends { a: (x: infer U) => void; b: (x: infer U) => void }
  ? U
  : never;

type T20 = Bar<{ a: (x: string) => void; b: (x: string) => void }>; // string
type T21 = Bar<{ a: (x: string) => void; b: (x: number) => void }>; // string & number 其实也就是 never
```

获取构造函数中参数类型：

```typescript
type ConstructorParameters<T> = T extends new (...args: infer R) => any
  ? R
  : never;
```

获取实例类型：

```typescript
type InstanceType<T> = T extends new (...args: any[]) => infer R ? R : any;
```

## 使用示例

{{% admonition info "数组示例" %}}
```typescript
type Pop<T extends unknown[]> = T extends [...infer U, unknown] ? U : T
type Push<T extends unknown[], U> = [...T, U]
type Shift<T extends unknown[]> = T extends [unknown, ...infer U] ? U : T
type Unshift<T extends unknown[], U> = [U, ...T]
```
{{% /admonition %}}


{{% admonition info "字符串示例" %}}
```typescript
type Blank = '\t'| '\n' | ' '
type TrimLeft<S extends string> = S extends `${Blank}${infer R}` ? TrimLeft<R> : S;
type TrimRight<S extends string> = S extends `${infer R}${Blank}` ? TrimRight<R> : S;
type Trim<S extends string> =  TrimLeft<TrimRight<S>>;
```
{{% /admonition %}}