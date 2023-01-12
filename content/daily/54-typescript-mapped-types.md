---

title: "TypeScript映射类型"
date: 2022-11-09T12:36:44+08:00
lastmod: 2022-11-09T12:36:44+08:00
draft: false
toc: false
keywords: ["daily"]
description: "TypeScript映射类型"
author: "xuyou"
tags: ["TypeScript"]

---

映射类型( `Mapped types` )允许我们从现有类型中创建新的类型。例如结合索引类型，将一个类型的所有属性设置为 `boolean`：

```typescript
type BooleanFlags<Type> = {
  [Property in keyof Type]: boolean;
};

type FeatureFlags = {
  darkMode: () => void;
  newUserProfile: () => void;
};
 
type FeatureOptions = BooleanFlags<FeatureFlags>;
// type FeatureOptions = {
//    darkMode: boolean;
//    newUserProfile: boolean;
// }
```

或者另一个常见的使用场景：将一个对象的所有属性变为*只读 (read-only)*。

```typescript
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};
```

`Readonly` 类型的作用是将传入的属性变为只读。例如 TypeScript 中是这样定义 [`Object.freeze`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)的：

```typescript
interface ObjectConstructor {
  /**
   * 防止修改现有的属性和值
   * 也防止新增属性
   * @param o 需要被锁的对象
   */
  freeze<T>(o: T): Readonly<T>;
}
```

如我们所见，`Object.freeze` 函数返回使用 Readonly 修饰符映射的对象。

还有另一个修饰符 `?`，用于设置属性可选：

```typescript
// 删除属性中的可选属性
type Optional<Type> = {
  [Property in keyof Type]?: Type[Property];
};
 
type User = {
  id: string;
  name: string;
  age: number;
};
 
type MaybeUser = Optional<User>;

// type MaybeUser = {
//   id?: string;
//   name?: string;
//   age?: number;
// };
```

也可以通过前缀 `-` 或者 `+` 删除或者添加这些修饰符，如果没有写前缀，相当于使用了 `+` 前缀。

```typescript
// 删除属性中的只读属性
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};
 
type LockedAccount = {
  readonly id: string;
  readonly name: string;
};
 
type UnlockedAccount = CreateMutable<LockedAccount>;

// type UnlockedAccount = {
//    id: string;
//    name: string;
// }

// 删除属性中的可选属性
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};
 
type MaybeUser = {
  id: string;
  name?: string;
  age?: number;
};
 
type User = Concrete<MaybeUser>;
// type User = {
//    id: string;
//    name: string;
//    age: number;
// }
```

