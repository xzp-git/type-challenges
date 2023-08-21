# TS 类型体操

## easy

### 1. 实现 Pick

不使用 `Pick<T, K>` ，实现 TS 内置的 `Pick<T, K>` 的功能。

**从类型 `T` 中选出符合 `K` 的属性，构造一个新的类型**。

例如：

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

type TodoPreview = MyPick<Todo, 'title' | 'completed'>

const todo: TodoPreview = {
  title: 'Clean room',
  completed: false,
}
```

```ts
type MyPick<T, K extends keyof T> = {
  [key in K]: T[key]
}
```

> 相关知识点

1. `keyof` : 取 `interface` 的键后保存为联合类型

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

type KeyofValue = keyof Todo // KeyofValue = "title" | "description" | "completed"
```

2. `in` : 取联合类型每一个的值，主要用于数组和对象的构建, 切记不要用于 `interface`, 否则会报错

```ts
type Name = 'title' | 'description' | 'completed'

type TName = {
  [key in Name]: string
}
/*
type TName = {
  title: string
  description: string
  completed: boolean
}

interface TName = {
  [key in Name]: string
} 
用于 interface 会报错
 */
```

### 2. 对象属性只读

不要使用内置的`Readonly<T>`，自己实现一个。

泛型 `Readonly<T>` 会接收一个 _泛型参数_，并返回一个完全一样的类型，只是所有属性都会是只读 (readonly) 的。

也就是不可以再对该对象的属性赋值。

例如：

```ts
interface Todo {
  title: string
  description: string
}

const todo: MyReadonly<Todo> = {
  title: 'Hey',
  description: 'foobar',
}

todo.title = 'Hello' // Error: cannot reassign a readonly property
todo.description = 'barFoo' // Error: cannot reassign a readonly property
```

```ts
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P]
}
```

> 解析： 用 `keyof` 拿到 T 的键组成的联合类型，然后再用 `in` 拿到每一个联合类型的值用 `P` 保存下来，T[P] 就是原来的类型了

### 3. 元组转换为对象

将一个元组类型转换为对象类型，这个对象类型的键/值和元组中的元素对应。

例如：

```ts
const tuple = ['tesla', 'model 3', 'model X', 'model Y'] as const

type result = TupleToObject<typeof tuple> // expected { tesla: 'tesla', 'model 3': 'model 3', 'model X': 'model X', 'model Y': 'model Y'}
```

```ts
type TupleToObject<T extends readonly PropertyKey[]> = {
  [P in T[number]]: P
}
```

> 相关知识点

1. `as const`

```ts
// 不加 as const
const tuple1 = ['tesla', 'model 3', 'model X', 'model Y'] // 此时 tuple的类型 会被TypeScript推断为 string[]

// 加 as const
const tuple2 = ['tesla', 'model 3', 'model X', 'model Y'] as const // 此时 tuple的类型 会被TypeScript推断为 元组类型 readonly ["tesla", "model 3", "model X", "model Y"]
```

2. `T[number]`

```ts
const tuple1 = ['tesla', 'model 3', 'model X', 'model Y']

type A1 = typeof tuple1 // A -> string[]

type B1 = A[number] // B ->  string

const tuple2 = ['tesla', 'model 3', 'model X', 'model Y'] as const

type A2 = typeof tuple2 // A2 -> 元组类型 readonly ["tesla", "model 3", "model X", "model Y"]

type B2 = A[number] // B2 ->  "tesla" | "model 3" | "model X" | "model Y" 元组组成的联合类型
```

3. `PropertyKey 与 keyof any`

   `PropertyKey` 是内置类型 与 `keyof any`一样， 都代表 `string | number | symbol`, 普通对象中的任何键都是`string | number | symbol`。

### 4. 第一个元素

实现一个`First<T>`泛型，它接受一个数组`T`并返回它的第一个元素的类型。

例如：

```ts
type arr1 = ['a', 'b', 'c']
type arr2 = [3, 2, 1]

type head1 = First<arr1> // 应推导出 'a'
type head2 = First<arr2> // 应推导出 3
```

```ts
type First1<T extends unknown[]> = T extends [] ? never : T[0]
type First2<T extends unknown[]> = T['length'] extends 0 ? never : T[0]
type First3<T extends unknown[]> = T extends [infer A, ...unknown[]] ? A : never
type First$<T extends unknown[]> = T[number] extends never ?
```

### 5.获取元组长度

创建一个`Length`泛型，这个泛型接受一个只读的元组，返回这个元组的长度。

例如：

```ts
type tesla = ['tesla', 'model 3', 'model X', 'model Y']
type spaceX = ['FALCON 9', 'FALCON HEAVY', 'DRAGON', 'STARSHIP', 'HUMAN SPACEFLIGHT']

type teslaLength = Length<tesla> // expected 4
type spaceXLength = Length<spaceX> // expected 5
```

```ts
type Length<T extends readonly unknown[]> = T['length']
```

### 6. 实现 Exclude

实现内置的 `Exclude<T, U>` 类型，但不能直接使用它本身。

> 从联合类型 `T` 中排除 `U` 中的类型，来构造一个新的类型。

例如：

```ts
type Result = MyExclude<'a' | 'b' | 'c', 'a'> // 'b' | 'c'
```

```ts
type MyExclude<T, U> = T extends U ? never : T
```

> 知识点

#### 分配条件类型

- 当条件类型作用于泛型类型时，如果给定联合类型，它们就会变得具有分布式性。例如，采用以下内容：

```ts
type ToArray<Type> = Type extends any ? Type[] : never
```

- 如果我们将联合类型插入 ToArray，则条件类型将应用于该联合的每个成员。

```ts
type ToArray<Type> = Type extends any ? Type[] : never

type StrArrOrNumArr = ToArray<string | number>

type StrArrOrNumArr = string[] | number[]
```

- 这里发生的情况是 ToArray 分布在：

  `string | number`

- 并将联盟的每个成员类型映射到有效的：

  `ToArray<string> | ToArray<number>`

  这给我们留下了：

  `string[] | number[];`

- 通常，分布性是期望的行为。extends 为了避免这种行为，您可以用方括号将关键字的每一侧括起来。

```ts
type ToArrayNonDist<Type> = [Type] extends [any] ? Type[] : never

// 'StrArrOrNumArr' is no longer a union.
type StrArrOrNumArr = ToArrayNonDist<string | number>

type StrArrOrNumArr = (string | number)[]
```

### 7. Awaited

假如我们有一个 Promise 对象，这个 Promise 对象会返回一个类型。在 TS 中，我们用 Promise<T> 中的 T 来描述这个 Promise 返回的类型。请你实现一个类型，可以获取这个类型。

例如：`Promise<ExampleType>`，请你返回 ExampleType 类型。

```ts
type ExampleType = Promise<string>

type Result = MyAwaited<ExampleType> // string
```

```ts
type MyAwaited<T extends PromiseLike<any | PromiseLike<any>>> = T extends PromiseLike<infer U>
  ? U extends PromiseLike<any>
    ? MyAwaited<U>
    : U
  : never
```

> 知识点：注意 `promise` 嵌套

### 8.If

实现一个 `IF` 类型，它接收一个条件类型 `C` ，一个判断为真时的返回类型 `T` ，以及一个判断为假时的返回类型 `F`。 `C` 只能是 `true` 或者 `false`， `T` 和 `F` 可以是任意类型。

例如：

```ts
type A = If<true, 'a', 'b'> // expected to be 'a'
type B = If<false, 'a', 'b'> // expected to be 'b'
```

```ts
type If<C extends boolean, T, F> = C extends true ? T : F
```

### 9. Concat

在类型系统里实现 JavaScript 内置的 `Array.concat` 方法，这个类型接受两个参数，返回的新数组类型应该按照输入参数从左到右的顺序合并为一个新的数组。

例如：

```ts
type Result = Concat<[1], [2]> // expected to be [1, 2]
```

```ts
type Concat<T extends readonly unknown[], U extends readonly unknown[]> = [...T, ...U]
```

### 10. Includes

在类型系统里实现 JavaScript 的 `Array.includes` 方法，这个类型接受两个参数，返回的类型要么是 `true` 要么是 `false`。

例如：

```ts
type isPillarMen = Includes<['Kars', 'Esidisi', 'Wamuu', 'Santana'], 'Dio'> // expected to be `false`
```

```ts
type Includes<T extends readonly unknown[], U> = T extends [infer First, ...infer Rest]
  ? Equal<First, U> extends true
    ? true
    : Includes<Rest, U>
  : false
```

### 11. Push

在类型系统里实现通用的 `Array.push` 。

例如：

```ts
type Result = Push<[1, 2], '3'> // [1, 2, '3']
```

```ts
type Push<T extends unknown[], U> = [...T, U]
```

### 12.Unshift

实现类型版本的 `Array.unshift`。

例如：

```typescript
type Result = Unshift<[1, 2], 0> // [0, 1, 2,]
```

```ts
type Unshift<T extends unknown[], U> = [U, ...T]
```

### 13. Parameters

实现内置的 Parameters<T> 类型，而不是直接使用它，可参考[TypeScript 官方文档](https://www.typescriptlang.org/docs/handbook/utility-types.html#parameterstype)。

例如：

```ts
const foo = (arg1: string, arg2: number): void => {}

type FunctionParamsType = MyParameters<typeof foo> // [arg1: string, arg2: number]
```

```ts
type MyParameters<T extends (...args: any[]) => any> = T extends (...args: infer P) => any ? P : never
```

### 14. 获取函数返回类型

不使用 `ReturnType` 实现 TypeScript 的 `ReturnType<T>` 泛型。

例如：

```ts
const fn = (v: boolean) => {
  if (v) return 1
  else return 2
}

type a = MyReturnType<typeof fn> // 应推导出 "1 | 2"
```

```ts
type MyReturnType<T extends (...args: any[]) => any> = T extends (...args: any[]) => infer R ? R : never
```

### 15. 实现 Omit

不使用 `Omit` 实现 TypeScript 的 `Omit<T, K>` 泛型。

`Omit` 会创建一个省略 `K` 中字段的 `T` 对象。

例如：

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

type TodoPreview = MyOmit<Todo, 'description' | 'title'>

const todo: TodoPreview = {
  completed: false,
}
```

```ts
// type UnPick<T, K> = T extends K ? never : T

// type MyOmit<T, K extends keyof T> = {
//   [P in UnPick<keyof T, K>]: T[P]
// }

type MyOmit<T, K extends keyof T> = {
  [P in keyof T as P extends K ? never : P]: T[P]
}
```

> 知识点

- 在 TypeScript 4.1 及更高版本中，您可以使用映射类型中的 as 子句重新映射映射类型中的键。[通过 as 进行键重新映射](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#key-remapping-via-as)

### 16. 对象部分属性只读

实现一个泛型`MyReadonly2<T, K>`，它带有两种类型的参数`T`和`K`。

类型 `K` 指定 `T` 中要被设置为只读 (readonly) 的属性。如果未提供`K`，则应使所有属性都变为只读，就像普通的`Readonly<T>`一样。

例如

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

const todo: MyReadonly2<Todo, 'title' | 'description'> = {
  title: 'Hey',
  description: 'foobar',
  completed: false,
}

todo.title = 'Hello' // Error: cannot reassign a readonly property
todo.description = 'barFoo' // Error: cannot reassign a readonly property
todo.completed = true // OK
```

```ts
type MyReadonly2<T, K extends keyof T = keyof T> = {
  [P in keyof T as P extends K ? never : P]: T[P]
} & {
  readonly [R in K]: T[R]
}
```

> 知识点

- 因为第二个泛型可能为空，所以需要通过=来赋默认值
- 因为第二个交叉类型使用了 T extends K ? never : T 判断，所以默认值应该是 keyof T

### 17. 对象属性只读（递归）

实现一个泛型 `DeepReadonly<T>`，它将对象的每个参数及其子对象递归地设为只读。

您可以假设在此挑战中我们仅处理对象。不考虑数组、函数、类等。但是，您仍然可以通过覆盖尽可能多的不同案例来挑战自己。

例如

```ts
type X = {
  x: {
    a: 1
    b: 'hi'
  }
  y: 'hey'
}

type Expected = {
  readonly x: {
    readonly a: 1
    readonly b: 'hi'
  }
  readonly y: 'hey'
}

type Todo = DeepReadonly<X> // should be same as `Expected`
```

```ts
type DeepReadonly<T> = {
  readonly [key in keyof T]: keyof T[key] extends never ? T[key] : DeepReadonly<T[key]>
}
```

> 知识点： 可以通过 `keyof T[key] extends never` 来判断该属性是不是一个普通类型

### 18. 元组转合集

实现泛型`TupleToUnion<T>`，它返回元组所有值的合集。

例如

```ts
type Arr = ['1', '2', '3']

type Test = TupleToUnion<Arr> // expected to be '1' | '2' | '3'
```

```ts
type TupleToUnion<T extends readonly unknown[]> = T[number]
```

### 19.可串联构造器

在 JavaScript 中我们经常会使用可串联（Chainable/Pipeline）的函数构造一个对象，但在 TypeScript 中，你能合理的给它赋上类型吗？

在这个挑战中，你可以使用任意你喜欢的方式实现这个类型 - Interface, Type 或 Class 都行。你需要提供两个函数 `option(key, value)` 和 `get()`。在 `option` 中你需要使用提供的 key 和 value 扩展当前的对象类型，通过 `get` 获取最终结果。

例如

```ts
declare const config: Chainable

const result = config.option('foo', 123).option('name', 'type-challenges').option('bar', { value: 'Hello World' }).get()

// 期望 result 的类型是：
interface Result {
  foo: number
  name: string
  bar: {
    value: string
  }
}
```

你只需要在类型层面实现这个功能 - 不需要实现任何 TS/JS 的实际逻辑。

你可以假设 `key` 只接受字符串而 `value` 接受任何类型，你只需要暴露它传递的类型而不需要进行任何处理。同样的 `key` 只会被使用一次。

- 第一步
  可以使用 T = {} 来作为默认值，甚至默认参数与默认返回值，再通过递归传递 T 即可实现递归全局记录
  option 是一个函数接收两个值：K 和 V，为了约束 key 不可重复必须范型传入，value 是任意类型范型不做约束直接透传

```ts
type Chainable<T = {}> = {
  option<K extends string, V>(key: K, value: V): Chainable<T & Record<K, V>>
  get(): T
}
```

- 第二步
  验证重复密钥，实现相同的密钥报错

```ts
type Chainable<T = {}> = {
  option<K extends string, V>(key: K extends keyof T ? never : K, value: V): Chainable<T & Record<K, V>>
  get(): T
}
```

- 第三步
  最后直接&联合并不能将相同 key 的类型覆盖，因此需 Omit 去掉前一个类型中相同的 key

```ts
type Chainable<T = {}> = {
  option: <K extends string, V>(key: K extends keyof T ? never : K, value: V) => Chainable<Omit<T, K> & Record<K, V>>
  get(): T
}
```

### 20. 最后一个元素

实现一个`Last<T>`泛型，它接受一个数组`T`并返回其最后一个元素的类型。

例如

```ts
type arr1 = ['a', 'b', 'c']
type arr2 = [3, 2, 1]

type tail1 = Last<arr1> // 应推导出 'c'
type tail2 = Last<arr2> // 应推导出 1
```

```ts
type Last<T extends unknown[]> = T extends [...unknown[], infer Last] ? Last : never
```

### 21. 排除最后一项

实现一个泛型`Pop<T>`，它接受一个数组`T`，并返回一个由数组`T`的前 N-1 项（N 为数组`T`的长度）以相同的顺序组成的数组。

例如

```ts
type arr1 = ['a', 'b', 'c', 'd']
type arr2 = [3, 2, 1]

type re1 = Pop<arr1> // expected to be ['a', 'b', 'c']
type re2 = Pop<arr2> // expected to be [3, 2]
```

**额外**：同样，您也可以实现`Shift`，`Push`和`Unshift`吗？

```ts
type Pop<T extends unknown[]> = T extends [...infer R, unknown] ? R : T
```

### 22. Promise.all

给函数`PromiseAll`指定类型，它接受元素为 Promise 或者类似 Promise 的对象的数组，返回值应为`Promise<T>`，其中`T`是这些 Promise 的结果组成的数组。

```ts
const promise1 = Promise.resolve(3)
const promise2 = 42
const promise3 = new Promise<string>((resolve, reject) => {
  setTimeout(resolve, 100, 'foo')
})

// 应推导出 `Promise<[number, 42, string]>`
const p = PromiseAll([promise1, promise2, promise3] as const)
```

```ts
// declare function PromiseAll<T extends unknown[]>(
//   values: readonly [...T],
// ): Promise<{
//   [I in keyof T]: Awaited<T[I]>
// }>

type MyAwaited<T> = T extends PromiseLike<infer R> ? MyAwaited<R> : T

type A = MyAwaited<Promise<number>> // number
type B = MyAwaited<Promise<Promise<string>>> // string

declare function PromiseAll<T extends unknown[]>(
  values: readonly [...T],
): Promise<{
  [I in keyof T]: MyAwaited<T[I]>
}>
```

### 23. 查找类型

有时，您可能希望根据某个属性在联合类型中查找类型。

在此挑战中，我们想通过在联合类型`Cat | Dog`中通过指定公共属性`type`的值来获取相应的类型。换句话说，在以下示例中，`LookUp<Dog | Cat, 'dog'>`的结果应该是`Dog`，`LookUp<Dog | Cat, 'cat'>`的结果应该是`Cat`。

```ts
interface Cat {
  type: 'cat'
  breeds: 'Abyssinian' | 'Shorthair' | 'Curl' | 'Bengal'
}

interface Dog {
  type: 'dog'
  breeds: 'Hound' | 'Brittany' | 'Bulldog' | 'Boxer'
  color: 'brown' | 'white' | 'black'
}

type MyDog = LookUp<Cat | Dog, 'dog'> // expected to be `Dog`
```

```ts
type LookUp<U extends { type: string }, T extends string> = U extends { type: T } ? U : never
```

### 24. 去除左侧空白

实现 `TrimLeft<T>` ，它接收确定的字符串类型并返回一个新的字符串，其中新返回的字符串删除了原字符串开头的空白字符串。

例如

```ts
type trimed = TrimLeft<'  Hello World  '> // 应推导出 'Hello World  '
```

```ts
type Space = ' ' | '\n' | '\t'

type TrimLeft<S extends string> = S extends `${Space}${infer R}` ? TrimLeft<R> : S
```

### 25. 去除两端空白字符

实现`Trim<T>`，它接受一个明确的字符串类型，并返回一个新字符串，其中两端的空白符都已被删除。
例如

```ts
type trimed = Trim<'  Hello World  '> // expected to be 'Hello World'
```

```ts
type Space = ' ' | '\n' | '\t'

// type TrimLeft<S extends string> = S extends `${Space}${infer R}` ? TrimLeft<R> : S
// type TrimRight<S extends string> = S extends `${infer R}${Space}` ? TrimRight<R> : S

// type Trim<S extends string> = TrimLeft<TrimRight<S>>

type Trim<S extends string> = S extends `${Space}${infer R}` | `${infer R}${Space}` ? Trim<R> : S
```
