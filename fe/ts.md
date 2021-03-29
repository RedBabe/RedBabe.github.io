### ts不是面向对象

ts是面向接口编程，它是鸭子类型语言。就是说，只要这个实参满足这个样子（如{name: string, id: number}），就是匹配的；并不是像面向对象语言那样，必须是指定类的实例才可以。

#### 联合类型

```typescript
// 联合类型
let a: string | number;

// 类型别名
type BType = string | number;
let b: BType;
```

联合类型和类型别名都不能用interface来代替。

TS中的`typeof`是用来读取一个函数的参数类型。如：

`Parameters<typeof myFun>`返回函数的参数类型。

#### Utility Type

> 给它传一个范型，utility type会对这个类型进行操作

```typescript
// 只使用部分类型
type Person = {
    name: string
    age: number
}

// 可以只传name或age，或者都传或都不传
const a: Partial<Person> = { age: 2 }
// 可以去掉指定的类型
const b: Omit<Person, 'name'> = { age: 3 }
```

#### 看一眼源码

Partial的源码：

```typescript
type Partial<T> = {
    [P in keyof T]?: T[p];
}
```

取出所有的键，都变成optional。

还有很多utility types，包括`Pick`、`Exclude`、`Parameters`等。

#### object类型

如果想表示一个纯对象，就是有任意键值对的JSON对象，不能使用`let a: object;`，因为`object`不仅不能表示任意键值，还不能表示纯对象（比如可以是函数）。

答案：

```typescript
let a: { [key: string]: unknown; } = {
  name: 'h',
  age: 23
};
```