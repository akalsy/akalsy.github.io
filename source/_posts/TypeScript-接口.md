---
title: TypeScript 接口
date: 2024-11-22 15:39:16
tags: 
  - TypeScript
  - interface
---
在 TypeScript 中 `interface`（接口） 是一种用于定义对象结构的方式。它允许你定义对象的属性、方法、函数签名、类行为等的形状。接口提供了一个强类型的检查机制，帮助开发者在编写代码时保证对象的结构符合预期，并减少潜在的错误。

接口是 TypeScript 类型系统的核心之一，它的作用是描述对象的结构（包括属性和方法）和类的行为。接口可以通过继承、合并、混合等方式，使得 TypeScript 代码更加灵活、可复用和安全。

---

### 1. **基本概念：接口的定义和使用**

#### 1.1 定义接口

使用 `interface` 关键字可以定义接口。接口描述对象的形状，也就是说，它指定了对象中应该包含哪些属性以及它们的类型。

```typescript
interface Person {
  name: string;
  age: number;
}
```

在上面的例子中，`Person` 是一个接口，包含两个属性：`name`（类型是 `string`）和 `age`（类型是 `number`）。

#### 1.2 使用接口

定义了接口后，我们可以使用该接口来声明对象，确保对象符合接口定义的结构。

```typescript
const person: Person = {
  name: 'Alice',
  age: 30
};
```

如果对象的结构不符合接口定义，TypeScript 会报错。

```typescript
const invalidPerson: Person = {
  name: 'Bob'
  // Error: Property 'age' is missing in type '{ name: string; }' but required in type 'Person'.
};
```

---

### 2. **接口的可选属性**

接口中的属性可以通过在属性名后加上 `?` 来标记为可选属性。这意味着该属性可以存在，也可以缺失。

```typescript
interface Person {
  name: string;
  age?: number;  // age 是可选的
}

const person1: Person = { name: 'Alice' };   // 合法
const person2: Person = { name: 'Bob', age: 25 };  // 合法
```

在这个例子中，`age` 属性是可选的，`person1` 可以没有 `age` 属性，而 `person2` 可以有。

---

### 3. **接口的只读属性**

通过在接口的属性前加上 `readonly`，可以指定该属性是只读的，意味着一旦赋值后，不能再修改该属性的值。

```typescript
interface Person {
  readonly name: string;
  age: number;
}

const person: Person = { name: 'Alice', age: 30 };

person.name = 'Bob';  // Error: Cannot assign to 'name' because it is a read-only property.
person.age = 35;      // This is allowed
```

在上面的例子中，`name` 是只读属性，不能再修改其值。

---

### 4. **接口的方法**

接口不仅可以定义对象的属性，还可以定义方法。接口中的方法也可以有参数类型和返回值类型。

```typescript
interface Person {
  name: string;
  greet(message: string): void;
}

const person: Person = {
  name: 'Alice',
  greet(message: string) {
    console.log(`${this.name} says: ${message}`);
  }
};

person.greet('Hello!');  // Alice says: Hello!
```

在这个例子中，接口 `Person` 定义了一个 `greet` 方法，方法接受一个 `string` 类型的 `message` 参数，并返回 `void`。

---

### 5. **接口的继承**

接口支持继承，允许一个接口继承其他接口。这种方式可以扩展现有的接口，增加更多的属性和方法。

#### 5.1 单一继承

```typescript
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

const dog: Dog = {
  name: 'Buddy',
  breed: 'Golden Retriever'
};
```

在这个例子中，`Dog` 接口继承了 `Animal` 接口，`Dog` 除了 `name` 属性外，还添加了 `breed` 属性。

#### 5.2 多重继承

TypeScript 允许接口继承多个接口（多重继承）。这样，你可以创建一个组合多个接口的接口。

```typescript
interface Animal {
  name: string;
}

interface Vehicle {
  wheels: number;
}

interface Car extends Animal, Vehicle {
  brand: string;
}

const car: Car = {
  name: 'Tesla',
  wheels: 4,
  brand: 'Model S'
};
```

在这个例子中，`Car` 接口继承了 `Animal` 和 `Vehicle` 两个接口，合并了它们的属性。

---

### 6. **接口的类实现**

接口可以用于约束类的结构。类通过 `implements` 关键字来实现接口，类必须包含接口中定义的所有属性和方法。

```typescript
interface Greetable {
  greet(): void;
}

class Person implements Greetable {
  constructor(public name: string) {}

  greet() {
    console.log(`Hello, ${this.name}`);
  }
}

const person = new Person('Alice');
person.greet();  // Hello, Alice
```

在这个例子中，`Person` 类实现了 `Greetable` 接口，必须实现 `greet` 方法。

#### 6.1 类实现多个接口

类也可以实现多个接口。

```typescript
interface Animal {
  name: string;
}

interface Flyable {
  fly(): void;
}

class Bird implements Animal, Flyable {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  fly() {
    console.log(`${this.name} is flying`);
  }
}

const bird = new Bird('Eagle');
bird.fly();  // Eagle is flying
```

在这个例子中，`Bird` 类实现了 `Animal` 和 `Flyable` 两个接口。

---

### 7. **接口的函数类型**

接口不仅可以描述对象的结构，还可以描述函数的签名。接口可以指定一个函数的参数类型和返回值类型。

```typescript
interface Add {
  (a: number, b: number): number;
}

const add: Add = (x, y) => x + y;

console.log(add(2, 3));  // 5
```

在这个例子中，`Add` 接口描述了一个接受两个 `number` 类型的参数并返回一个 `number` 类型的函数签名。

---

### 8. **索引签名**

索引签名允许你为接口添加任意数量的属性，而不需要事先定义每个属性。这对处理具有动态属性的对象非常有用。

```typescript
interface Dictionary {
  [key: string]: string;  // 允许字符串类型的键和对应的字符串值
}

const dict: Dictionary = {
  hello: 'world',
  foo: 'bar'
};
```

在这个例子中，`Dictionary` 接口允许对象拥有任意数量的 `string` 类型的键，并且值也必须是 `string` 类型。

---

### 9. **混合类型的接口**

有时候，你可能需要描述同时具有多个不同类型的对象，比如一个对象既有属性又有方法。可以通过接口来描述这种**混合类型**。

```typescript
interface Counter {
  (start: number): string;
  count: number;
}

const counter: Counter = (start: number) => {
  counter.count = start;
  return `Counter started at ${start}`;
};

counter.count = 0;
console.log(counter(5));  // Counter started at 5
console.log(counter.count);  // 5
```

在这个例子中，`Counter` 接口同时描述了一个函数类型和一个属性 `count`。

---

### 10. **接口的声明合并**

接口有一个有趣的特性，就是 **声明合并**。当多个接口声明同名时，TypeScript 会自动将它们合并成一个接口。这对于扩展已有接口非常有用。

```typescript
interface Window {
  title: string;
}

interface Window {
  width: number;
}

const myWindow: Window = {
  title: 'Main Window',
  width: 800
};
```

在这个例子中，`Window` 接口被声明了两次，TypeScript 会自动将两个声明合并为一个接口。因此 `Window` 既有 `title` 属性，也有 `width` 属性。

---

### 11. **总结**

- **接口** 是 TypeScript 中用于描述对象、类、函数等类型的结构的一种方式。
- 接口可以定义**属性**、**方法**、**可选属性**、**只读属性**等。
- 接口支持**继承**，可以通过 `extends` 关键字继承多个接口，也可以被类实现。
- **函数类型**和**混合类型**可以通过接口来描述。
- 接口还支持**声明合并**，多个相同名字的接口会自动合并成一个接口。
