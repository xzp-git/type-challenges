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
