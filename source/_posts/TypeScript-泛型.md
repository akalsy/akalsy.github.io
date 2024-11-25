---
title: TypeScript 泛型
date: 2024-11-23 21:09:50
tags:
  - TypeScript
  - 泛型
---
在 TypeScript 中，**泛型**（Generics）是一种强大的工具，用于定义能够处理多种类型的函数、类和接口，而不失去类型安全性。它允许我们编写可复用的代码，避免类型重复，并且提高了代码的灵活性和可维护性。

泛型的核心思想是**抽象类型**，使得我们可以在编写代码时指定一个**类型占位符**，在使用时再指定具体的类型。

### 1. **基础概念：泛型的作用**

通过泛型，我们可以在定义函数、类或接口时，让它们适应多种数据类型，而不需要在每次使用时都重复写出类型。

例如，我们希望定义一个能够处理不同类型的函数，而不管数据类型是 `number`、`string` 还是其他类型，使用泛型可以让这个函数更具通用性。

#### 1.1 泛型函数

我们首先来看一个普通的函数，它的类型已经固定了：

```typescript
function identity(value: number): number {
  return value;
}

let num = identity(5);  // 合法
let str = identity("hello");  // Error: Argument of type 'string' is not assignable to parameter of type 'number'.
```

上面的函数只能接受 `number` 类型的值，并且返回 `number` 类型。这个函数无法用于处理其他类型的数据，若要复用此函数，我们需要为不同的类型编写多个版本，代码冗长且不易维护。

通过使用泛型，可以实现类型的灵活性：

```typescript
function identity<T>(value: T): T {
  return value;
}

let num = identity(5);       // 合法，类型推导为 number
let str = identity("hello"); // 合法，类型推导为 string
```

在这个例子中，`T` 是一个泛型参数，它代表了一个占位符，表示函数可以接受和返回任意类型。你可以在函数调用时自动推导出类型，也可以显式指定类型：

```typescript
let num = identity<number>(5); // 显式指定 T 为 number
let str = identity<string>("hello"); // 显式指定 T 为 string
```

#### 1.2 泛型类型约束

有时，你可能希望限制泛型类型的范围，比如要求泛型参数必须是某个特定类型的子类型。在这种情况下，可以使用 **类型约束** 来约束泛型类型。

例如，假设我们希望传入的 `value` 必须是 `length` 属性存在的对象（如 `string` 或 `array`），我们可以这样做：

```typescript
function logLength<T extends { length: number }>(value: T): number {
  return value.length;
}

console.log(logLength("Hello"));    // 5
console.log(logLength([1, 2, 3]));  // 3
console.log(logLength(5));          // Error: Argument of type '5' is not assignable to parameter of type '{ length: number; }'.
```

在这个例子中，`T extends { length: number }` 表示泛型 `T` 必须是一个拥有 `length` 属性的类型，`string`、`array` 等类型符合这一约束，但 `number` 类型不符合。

---

### 2. **泛型接口**

除了泛型函数外，TypeScript 还允许我们在接口中使用泛型来描述类型。

#### 2.1 泛型接口的基本用法

例如，我们可以定义一个泛型接口，它指定了一个方法，该方法的返回类型和输入类型都是泛型类型：

```typescript
interface IdentityFn<T> {
  (value: T): T;
}

const identity: IdentityFn<number> = (value) => value;
console.log(identity(5));  // 5
```

在这个例子中，`IdentityFn` 接口定义了一个泛型函数类型，它接受一个 `T` 类型的参数并返回 `T` 类型的值。我们为 `IdentityFn` 接口指定了具体类型 `number`。

#### 2.2 泛型接口的约束

你还可以为泛型接口指定约束。比如，假设我们希望泛型参数 `T` 必须是一个对象类型，且该对象必须有一个 `name` 属性，我们可以这样定义：

```typescript
interface Person {
  name: string;
  age: number;
}

interface Info<T extends Person> {
  value: T;
}

const personInfo: Info<Person> = { value: { name: "Alice", age: 30 } };
```

在这个例子中，`Info` 接口的泛型参数 `T` 必须是 `Person` 类型的子类型，因此 `personInfo` 必须包含一个 `name` 和 `age` 属性。

---

### 3. **泛型类**

类也可以使用泛型。泛型类的作用和泛型函数类似，可以使类能够处理多种类型。

#### 3.1 泛型类的基本用法

```typescript
class Box<T> {
  value: T;

  constructor(value: T) {
    this.value = value;
  }

  getValue(): T {
    return this.value;
  }
}

const numberBox = new Box(123);    // 泛型 T 被推导为 number
console.log(numberBox.getValue()); // 123

const stringBox = new Box("Hello"); // 泛型 T 被推导为 string
console.log(stringBox.getValue());  // "Hello"
```

在这个例子中，`Box` 是一个泛型类，它可以包装任何类型的值。通过 `T`，我们可以在实例化时指定类型。

#### 3.2 泛型类的构造函数

泛型类也可以有带泛型参数的构造函数：

```typescript
class Pair<T, U> {
  first: T;
  second: U;

  constructor(first: T, second: U) {
    this.first = first;
    this.second = second;
  }
}

const pair = new Pair(1, "one");
console.log(pair.first);   // 1
console.log(pair.second);  // "one"
```

在这个例子中，`Pair` 是一个带有两个泛型参数 `T` 和 `U` 的类，分别代表两个不同类型的值。

---

### 4. **泛型约束与默认类型**

#### 4.1 泛型约束

泛型参数可以通过 `extends` 进行约束，确保传入的类型满足特定的条件：

```typescript
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(value: T): number {
  return value.length;
}

logLength("Hello");    // 合法，"Hello" 有 length 属性
logLength([1, 2, 3]);  // 合法，[1, 2, 3] 有 length 属性
logLength(123);        // Error: Argument of type '123' is not assignable to parameter of type 'Lengthwise'.
```

这里，我们通过 `T extends Lengthwise` 限制了 `T` 必须是具有 `length` 属性的类型。

#### 4.2 泛型默认类型

你还可以为泛型指定默认类型。如果调用时没有显式指定类型，TypeScript 将使用默认类型。

```typescript
function wrap<T = string>(value: T): T {
  return value;
}

let wrapped = wrap(123);  // T 被推导为 number
let wrappedString = wrap(); // T 默认为 string
```

在这个例子中，`wrap` 函数的泛型参数 `T` 默认是 `string`，但如果调用时显式指定了类型（如 `number`），则会使用指定的类型。

---

### 5. **泛型的高级应用**

#### 5.1 使用多个泛型参数

你可以在一个函数、类或接口中使用多个泛型参数。常见的例子包括映射类型、双重泛型等。

```typescript
class Pair<T, U> {
  first: T;
  second: U;

  constructor(first: T, second: U) {
    this.first = first;
    this.second = second;
  }

  swap(): Pair<U, T> {
    return new Pair(this.second, this.first);
  }
}

const pair = new Pair(1, "one");
const swappedPair = pair.swap();
console.log(swappedPair.first);  // "one"
console.log(swappedPair.second); // 1
```

在这个例子中，`Pair` 类接受两个泛型参数 `T` 和 `U`，并通过 `swap` 方法返回一个交换了类型的 `Pair` 对象。

#### 5.2 泛型与函数重载

你还可以结合泛型和函数重载实现更加灵活的函数签名。

```typescript
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const merged = merge({ name: "Alice" }, { age: 30 });
console.log(merged); // { name: "Alice", age: 30 }
```

在这个例子中，`merge` 函数接受两个对象并返回它们的交集类型，
