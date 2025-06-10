## 声明

ArkTS通过声明引入变量、常量、函数和类型。

### **变量声明**

以关键字let开头的声明引入变量，该变量在程序执行期间可以具有不同的值。

```typescript
let hi: string = 'hello';hi = 'hello, world';
```

### **常量声明**

以关键字const开头的声明引入只读常量，该常量只能被赋值一次。

```typescript
const hello: string = 'hello';
```

对常量重新赋值会造成编译时错误。

#### **自动类型推断**

由于ArkTS是一种静态类型语言，所有数据的类型都必须在编译时确定。

但是，如果一个变量或常量的声明包含了初始值，那么开发者就不需要显式指定其类型。ArkTS规范中列举了所有允许自动推断类型的场景。

以下示例中，两条声明语句都是有效的，两个变量都是string类型：

```typescript
let hi1: string = 'hello';
let hi2 = 'hello, world';
```

### type的使用

#####  一、`type` 是什么？

`type` 是 TypeScript 提供的一种 **类型别名定义方式**，用于为类型、对象结构、联合类型、函数类型等创建别名。

```typescript
type MyString = string;
```

------

#####  二、基础类型别名

```typescript
type ID = number;
type Name = string;
```

> 💡 用于简化代码、增强可读性。

------

#####  三、对象类型别名

```typescript
type User = {
  id: number;
  name: string;
  isActive: boolean;
};
```

------

#####  四、联合类型（Union Types）

```typescript
type Status = "success" | "error" | "loading";

type ID = number | string;
```

------

##### 五、交叉类型（Intersection Types）

```typescript
type A = { x: number };
type B = { y: number };
type C = A & B; // C 现在是 { x: number, y: number }
```

------

#####  六、函数类型别名

```typescript
type BinaryFunc = (a: number, b: number) => number;

const add: BinaryFunc = (x, y) => x + y;
```

------

#####  七、泛型类型别名

```typescript
type Result<T> = {
  data: T;
  error: string | null;
};

const response: Result<string> = {
  data: "Hello",
  error: null
};
```

------

#####  八、条件类型（高级用法）

```typescript
type IsString<T> = T extends string ? "yes" : "no";

type A = IsString<string>; // "yes"
type B = IsString<number>; // "no"
```

------

#####  九、映射类型（用于类型转换）

```typescript
type Keys = "name" | "age";
type Info = {
  [K in Keys]: string;
};
// 等价于：
// type Info = {
//   name: string;
//   age: string;
// }
```

------

##### 十、递归类型

```typescript
type JSONValue = 
  | string 
  | number 
  | boolean 
  | null 
  | JSONValue[] 
  | { [key: string]: JSONValue };
```

> 用于表示递归结构，如 JSON。



##### 十一、类型组合技巧

```typescript
type ID = number | string;
type User = {
  id: ID;
  name: string;
};

type Admin = User & {
  role: "admin";
};
```





## 函数

### 1.显式命名函数表达式

```typescript
const greet = function sayHello(name: string): void {
  console.log("Hello, " + name);
};
```

这时函数体内可以使用 `sayHello()` 来递归或引用自己：

```typescript
console.log(greet.name); // 👉 输出：'sayHello'
```

但注意：`sayHello` 只在函数体内可见，外部不能用它访问函数。

> 函数表达式中的命名（如 `sayHello`）**只在函数体内有效**，是**局部私有标识符**，外部必须通过赋值变量名（ `greet`）来访问。(sayHello为局部名称)

```typescript
const greet = function sayHello(name: string): void {
  if (name) {
    console.log("Hello, " + name);
  } else {
    sayHello("guest"); // ✅ 可在函数体内使用函数名
  }
};

greet();      // ✅ 调用函数
sayHello();   // ❌ Error: Cannot find name 'sayHello'

```



### 2. 匿名函数表达式（最常见）

```typescript
const greet = function(name: string): void {
  console.log("Hello, " + name);
};
```

这个函数本身是 **匿名的**，但因为赋值给了 `greet`，所以：

```typescript
console.log(greet.name); // 👉 输出：'greet'
```

引擎在赋值时自动把变量名赋给函数的 `.name` 属性。



### 3. 箭头函数 (Lambda函数)

```typescript
let sum = (x: number, y: number): number => {
  return x + y;
}
```

>  箭头函数的**返回类型**可以**省略**；省略时，返回类型通过函数体推断。
>
>  表达式可以指定为箭头函数，使表达更简短，因此以下两种表达方式是等价的：

```typescript
let sum1 = (x: number, y: number) => { return x + y; }
let sum2 = (x: number, y: number) => x + y;	//当且仅当函数体是单个表达式时可以使用,表达式的结果会被隐式作为返回值返回
```

在省略函数体括号的函数定义中，默认该语句为返回值；

> [!important] 
>
> 若需要实现无返回值，则需显示表明返回值为void，否则会根据类型推断自动推断返回类型并将语句结果返回
>
> ```typescript
> const g = () => 123;  // 返回类型自动推断为 number，不是 void
> const g = () => { 123; }; // 实际上返回 void，123 只是被执行但未返回
> const g = () => { console.log('hi'); }//推断为 void
> ```



### 可选参数

可选参数的格式可为name?: Type。

```typescript
function hello(name?: string) {
  if (name == undefined) {
    console.log('Hello!');
  } else {
    console.log(`Hello, ${name}!`);
  }
}
```

可选参数的另一种形式为设置的参数默认值。如果在函数调用中这个参数被省略了，则会使用此参数的默认值作为实参。

```typescript
function multiply(n: number, coeff: number = 2): number {
  return n * coeff;
}
multiply(2);  // 返回2*2
multiply(2, 3); // 返回2*3
```



### Rest参数

> 使用 Rest 参数时参数变量名前必须使用`...`，为语法规定

函数的最后一个参数可以是rest参数。rest参数的格式为...restArgs。rest参数允许函数接收一个由剩余实参组成的数组，用于处理不定数量的参数输入。

```typescript
function sum(...numbers: number[]): number {  
    let res = 0;  
    for (let n of numbers)    
        res += n;  
    return res;}
sum(); // 返回0
sum(1, 2, 3); // 返回6
```

### 返回类型

如果可以从函数体内推断出函数返回类型，则可在函数声明中省略标注返回类型。

```typescript
// 显式指定返回类型
function foo(): string { return 'foo'; }
// 推断返回类型为string
function goo() { return 'goo'; }
```

不需要返回值的函数的返回类型可以显式指定为void或省略标注。这类函数不需要返回语句。

以下示例中两种函数声明方式都是有效的：

```typescript
function hi1() { console.log('hi'); }
function hi2(): void { console.log('hi'); }
```



### 使用type进行函数类型定义

>  ==***函数类型定义***的作用为指定函数的参数列表（参数个数，每个参数的数据类型）与返回值类型==
>
>  仍为一种封装思想，封装函数**类型**的定义，若多次使用到该函数类型，封装后下次声明或定义函数时就无需再重复定义参数列表与返回类型



1. **定义函数类型：**

```typescript
type trigFunc = (x: number) => number;
```

这只是定义了“类型结构”，它告诉你函数应该是“接受一个 `number` 类型的参数，返回一个 `number` 类型的结果”。

1. **具体实现函数：**

```typescript
const sinFunc: trigFunc = (x) => Math.sin(x);	//详见lambda函数
const square: trigFunc = (x) => x * x;	//详见lambda函数
```

使用 `const` 来定义了函数 `sinFunc` 和 `square`，它们都是符合 `trigFunc` 类型结构的具体实现。



注意该处（使用type）并非为使用lambda表达式进行~~**函数定义**~~，而是**函数类型定义**

* 函数定义（使用lambda表达式）：`const/let 函数名 = (参数列表):返回类型 => {函数体}` 

* 函数类型定义：`type 函数类型名 = (参数列表) => 返回类型`

##### 1. `type` 用于定义**函数类型**（描述函数的结构）

```typescript
type trigFunc = (x: number) => number;
```

##### 2.`const` 用于定义**箭头函数的实现**

```typescript
const sinFunc = (x: number): number => Math.sin(x);
```



> 函数类型通常用于定义回调：
>
> ```typescript
> type trigFunc = (x: number) => number // 这是一个函数类型
> 
> function do_action(f: trigFunc) {
> f(3.141592653589); // 调用函数
> }
> 
> do_action(Math.sin); // 将函数作为参数传入
> ```



### 箭头函数（Lambda函数）

函数可以定义为箭头函数，例如：

```typescript
let sum = (x: number, y: number): number => {  return x + y;}
```

箭头函数的返回类型可以省略；省略时，返回类型通过函数体推断。

表达式可以指定为箭头函数，使表达更简短，因此以下两种表达方式是等价的：

```typescript
let sum1 = (x: number, y: number) => { return x + y; }
let sum2 = (x: number, y: number) => x + y;
```



### 闭包

> **“闭包”指的是内层函数（也就是那个被返回或传递的函数），以及它“捕获”的外层作用域环境。**
>
> 闭包 = 内层函数( `g()` ) + 外层函数的变量环境( `count` )
>
> ***通俗解释：***
>
> * 外层函数 **创建作用域**（为`count`分配内存）
>
> * 内层函数 **捕获这个作用域**（能够使为`count`所分配的内存不被销毁：在内层函数仍被引用的情况下）
>
> * 当外层函数（`f`）执行完毕后，**这个作用域本应销毁**
>
> * 但只要内层函数（`g`）还存在(尚被其他变量所引用：被`z`所引用)，它就<span style="background:white;color:green">“闭住”了这块作用域(`count`)的内存空间（因为`g()`中使用到`count`，并且`g()`仍在被引用，无法直接销毁其内存），于是这个“闭住的作用域”就被称作**闭包（Closure）**</span>



==闭包就是函数“记住了”它定义时的作用域，即使在外层函数已经返回之后，仍然可以访问这个作用域中的变量。==

==可以把返回的函数理解为一个**已经封装了“特定上下文”的函数实例，调用时只需要提供它尚未封装的参数部分**==

```typescript
function f(): () => number {	//f的函数定义是传统的函数定义方法
    							//而其返回值类型定义使用的是箭头表达式，最后{}中的便为f的函数体
  let count = 0;
  let g = (): number => { count++; return count; };
  return g;
}

let z = f();
z(); // 返回：1
z(); // 返回：2
```

> [!tip]
>
> `function f(): () => number { ... }`
>
> `f`为一个函数
>
> 第一个`()`：`f`不接受任何入参
>
> `:() => number`：`f`的返回值为一个函数
>
> `() => number`：返回函数同样不接受入参，并且返回值类型为number
>
> `{}`：`f`的函数体
>
> **`f` 的返回值为函数类型定义的写法**



**关于为什么`count`内存不会被释放：**

> ```typescript
> function outer() {
> let count = 0; // count 只在 outer 的作用域中
> 
> return function inner() {
> count++;
> return count;
> };
> }
> ```
>
> - `inner()` 是在 `outer()` 的作用域中定义的，所以它**能访问 count**
> - 当 `outer()` 执行完后，它本应该销毁所有局部变量（如 count）
> - **但是！** `inner()` 仍然被外部变量持有（如 `let z = outer();`），这个函数还在使用 `count`
>
> > 👉 所以 JS 引擎不会释放 `count`，因为它还被 `inner()` **引用**着！

当这两个条件都满足，闭包变量才会被释放：

1. **闭包函数本身不再被引用**（比如变量指向了 null）
2. **闭包函数之外没有任何地方再引用它访问的局部变量**



> 更多示例：
>
> ```typescript
> function makeCounter() {		//不显示指出返回类型，由TS推断
> let count = 0;
> return () => ++count;
> }
> 
> const counterA = makeCounter();  // 创建一个闭包
> const counterB = makeCounter();  // 又创建一个新的闭包
> 
> console.log(counterA()); // 1
> console.log(counterA()); // 2
> console.log(counterB()); // 1   <-- 是新的 count！
> ```
>
> 返回函数带参：
>
> ```typescript
> function outer(): (x: number) => number {
> let multiplier = 2;
> return function inner(x: number): number {
>  return x * multiplier;
> };
> }
> 
> let double = outer(); // double 是一个函数，接受 x: number
> console.log(double(5)); // 输出：10
> console.log(double(7)); // 输出：14
> ```
>
> 自主声明闭包所引用其父函数变量(base)的值
>
> ```typescript
> function createAdder(base: number): (x: number) => number {
> return (x: number) => base + x;
> }
> 
> const addFive = createAdder(5);  // 返回一个函数，记住 base = 5
> console.log(addFive(10)); // 输出 15
> console.log(addFive(3));  // 输出 8
> 
> ```
>
> 

2. 



## 类

#### 1.类的定义

```typescript
class Person {		//定义类
  name: string = '';
  surname: string = '';
  constructor (n: string, sn: string) {
    this.name = n;
    this.surname = sn;
  }
  fullName(): string {
    return this.name + ' ' + this.surname;
  }
}
```

**可选变量：**

> [!NOTE]
>
> 如果你定义了构造函数的可选参数，例如 `color?: string`，**在调用构造函数时没有传入 `color` 参数，那么 `this.color = color` 仍然会执行**，只不过它会赋值为 `undefined`



```typescript
class Car {
  brand: string;
  color?: string;  // 可选属性

  constructor(brand: string, color?: string) {
    this.brand = brand;
    this.color = color;  // 👈 这句仍然会执行！
  }
}

let car1 = new Car("Toyota");
console.log(car1.color); // 输出：undefined
```



#### 2. **实例化**

* 使用new进行实例化

```typescript
let p = new Person('John', 'Smith');
console.log(p.fullName());
```

* 使用对象字面量进行实例化

```typescript
let p:person = {name: "JarBow", surname: "Fun"}
```



#### **3. 字段初始化**

为了减少运行时的错误和获得更好的执行性能，

> [!CAUTION]
>
> ArkTS要求**所有字段在声明时或者构造函数中显式初始化**。这和标准TS中的 strictPropertyInitialization 模式一样。
>
> 以下代码是在ArkTS中<span style="color:red">不合法的代码</span>。

> 字段：直接在类中声明的某种类型的变量

```typescript
class Person {
  name: string; // undefined
  
  setName(n:string): void {
    this.name = n;
  }
  
  getName(): string {
    // 开发者使用"string"作为返回类型，这隐藏了name可能为"undefined"的事实。
    // 更合适的做法是将返回类型标注为"string | undefined"，以告诉开发者这个API所有可能的返回值。
    return this.name;
  }
}

let jack = new Person();
// 假设代码中没有对name赋值，即未调用"jack.setName('Jack')"
jack.getName().length; // 运行时异常：name is undefined
```

`ArkTS` 中的正确写法

```typescript
class Person {
  name: string = '';
  
  setName(n:string): void {
    this.name = n;
  }
  
  // 类型为'string'，不可能为"null"或者"undefined"
  getName(): string {
    return this.name;
  }
}
  

let jack = new Person();
// 假设代码中没有对name赋值，例如调用"jack.setName('Jack')"
jack.getName().length; // 0, 没有运行时异常
```

接下来的代码展示了如果name的值可以是undefined，那么应该如何写代码。

```typescript
class Person {
  name?: string; // 可能为`undefined`

  setName(n:string): void {
    this.name = n;
  }

  // 编译时错误：name可以是"undefined"，所以这个API的返回值类型不能仅定义为string类型
  getNameWrong(): string {
    return this.name;
  }

  getName(): string | undefined { // 返回类型匹配name的类型
    return this.name;
  }
}

let jack = new Person();
// 假设代码中没有对name赋值，例如调用"jack.setName('Jack')"

// 编译时错误：编译器认为下一行代码有可能会访问undefined的属性，报错
jack.getName().length;  // 编译失败

jack.getName()?.length; // 编译成功，没有运行时错误
```



#### **4. getter和setter**

setter和getter可用于提供对对象属性的受控访问。

在以下示例中，setter用于禁止将_age属性设置为无效值：

```typescript
class Person {
  name: string = '';
  private _age: number = 0;

  // getter（读取 _age）
  get age(): number {		//get:关键字
    return this._age;
  }

  // setter（设置 _age）
  set age(x: number) {		//set：关键字
    if (x < 0) {
      throw Error('Invalid age argument'); // 防止负数
    }
    this._age = x;
  }
}
```



| 成员           | 说明                                           |
| -------------- | ---------------------------------------------- |
| `private _age` | 实际存储数据的字段，只能在类内部访问           |
| `get age()`    | 读取属性时会自动调用此方法，返回 `_age` 的值   |
| `set age(x)`   | 设置属性时会自动调用此方法，可以加入验证逻辑等 |

```typescript
let p = new Person();

console.log(p.age);   // 自动调用 get age()，输出：0

p.age = 25;           // 自动调用 set age(25)，合法 ✔️

p.age = -42;          // 自动调用 set age(-42)，抛出异常 ❌

```

| 外部写法     | 实际行为               |
| ------------ | ---------------------- |
| `p.age`      | 自动调用 `get age()`   |
| `p.age = 30` | 自动调用 `set age(30)` |
| `_age`       | 只是内部保存数据的变量 |



> 实质上为一种 TypeScript 的语法糖，提供了更优雅的访问方式。

看起来像在读写属性：

```typescript
p.age = 20;
console.log(p.age);
```

但实际上是在调用函数：

```typescript
p.setAge(20); // 传统方式（Java、C++）
p.getAge();
```

例如：

```typescript
class Person {
  private _age = 0;

  get age() {
    console.log("调用了getter");
    return this._age;
  }

  set age(value: number) {
    console.log("调用了setter");
    this._age = value;
  }
}

let p = new Person();

p.age = 18; // 控制台输出：调用了setter
console.log(p.age); // 控制台输出：调用了getter，然后输出 18

```



##### 为什么`age`前需要加 `_` ?

 🔹 `_age` 是**内部变量（私有）**

- 是真正存储数据的地方
- 不想让外部直接访问它

 🔹 `age` 是**对外暴露的接口(本质上为get/set方法)**

- 提供一个“控制访问”的通道
- 外部访问的是 `age`，但背后操作的是 `_age`

如果直接写成：

```typescript
set age(value: number) {
  this.age = value; // ❌ 无限递归调用自己，栈溢出
}
```

所以**引入 `_age`** 作为“真正的数据容器”，避免命名冲突、逻辑混乱。



##### ✍️ 命名规范：

约定俗成的命名方式，并非语法强制要求：

| 命名      | 说明                         |
| --------- | ---------------------------- |
| `_变量名` | 表示“内部用，不建议直接访问” |
| `变量名`  | 表示“对外暴露”的属性名       |



> [!tip]
>
> ##### **最佳实践（Maybe）**
>
> 若不遵循 TS 规范，沿用 Java 习惯，可在定义变量时不加`_`，而是在定义`getter/setter`时将方法名定义为`getAttribute()`的形式，此时在访问属性时即可使用 `object.getAttribute`;

##### 拓展

只定义一个 `getter` 而不定义 `setter`，这样属性就是**只读的**：

```typescript
class Circle {
  constructor(private radius: number) {}
  get area() {
    return Math.PI * this.radius ** 2;
  }
}

const c = new Circle(5);
console.log(c.area); // ✅ 可以读取
c.area = 100;        // ❌ 报错：无法分配只读属性
```

`private radius: number` 是 构造函数参数属性的简写形式，相当于：

```typescript
private radius: number 是 构造函数参数属性（Parameter Properties） 的简写形式，相当于：
```



#### 5.继承

一个类可以继承另一个类（称为基类），并使用以下语法实现多个接口：

```typescript
class [extends BaseClassName] [implements listOfInterfaces] {
  // ...
}
```

继承类继承基类的字段和方法，但不继承构造函数。继承类可以新增定义字段和方法，也可以覆盖其基类定义的方法。

基类也称为“父类”或“超类”。继承类也称为“派生类”或“子类”。

示例：

```typescript
class Person {
  name: string = '';
  private _age = 0;
  get age(): number {
    return this._age;
  }
}
class Employee extends Person {
  salary: number = 0;
  calculateTaxes(): number {
    return this.salary * 0.42;
  }
}
```

包含implements子句的类必须实现列出的接口中定义的所有方法，但使用默认实现定义的方法除外。

```typescript
interface DateInterface {
  now(): string;
}
class MyDate implements DateInterface {
  now(): string {
    // 在此实现
    return 'now';
  }
}
```

关键字super可用于访问父类的实例字段、实例方法和构造函数。在实现子类功能时，可以通过该关键字从父类中获取所需接口：

```typescript
class RectangleSize {
  protected height: number = 0;
  protected width: number = 0;

  constructor (h: number, w: number) {
    this.height = h;
    this.width = w;
  }

  draw() {
    /* 绘制边界 */
  }
}
class FilledRectangle extends RectangleSize {
  color = ''
  constructor (h: number, w: number, c: string) {
    super(h, w); // 父类构造函数的调用
    this.color = c;
  }

  draw() {
    super.draw(); // 父类方法的调用
    // super.height -可在此处使用
    /* 填充矩形 */
  }
}
```



#### **6. 方法重载**

通过重载签名，指定方法的不同调用。具体方法为，为同一个方法写入多个同名但签名不同的方法头，方法实现紧随其后。

> **签名（Signature）** 指的是函数的**名称 + 参数列表 + 返回类型**的组合。也可以理解为函数对“你可以怎么调用我”的一种声明。

```typescript
class C {
  foo(x: number): void;            /* 第一个签名 */
  foo(x: string): void;            /* 第二个签名 */
  foo(x: number | string): void {  /* 实现签名 */
  }
}
let c = new C();
c.foo(123);     // OK，使用第一个签名
c.foo('aa'); // OK，使用第二个签名
```

如果两个重载签名的名称和参数列表均相同，则为错误。



>  [!important]
>
>  :exclamation:  指明一处易错点：
>
>  为什么不能像 Java 与 cpp 一样直接实现函数重载（只要保证函数名相同参数列表不同即可）？

```typescript
// ❌ 错误，TS 不支持这样写两个函数体
foo(x: number): void {
  ...
}
foo(x: string): void {
  ...
}

```



从TypeScript 和 JavaScript 的**语言本质差异**说起👇

> [!note]
>
> **核心原因：JavaScript 本身不支持函数重载**
>
> **TypeScript 是基于 JavaScript 构建的，最终代码也要编译成 JavaScript 去运行**。



但 **JavaScript 的函数是“弱类型+动态的”，不会区分多个同名函数**：

```typescript
function foo(x) { console.log("第一个"); }
function foo(x, y) { console.log("第二个"); }

foo(1); // 调用的是第二个函数，前一个被覆盖了
```

在 JavaScript 里，**后定义的函数会覆盖前面的同名函数**。所以 JS 根本就不能真正支持多个同名函数 —— 它只保留最后一个。

因此`ArkTS`利用 **多重声明 + 单一实现** 实现函数的重载



#### 7. 构造函数

类声明可以包含用于初始化对象状态的构造函数。

构造函数定义如下：

```typescript
constructor ([parameters]) {
  // ...
}
```

如果未定义构造函数，则会自动创建具有空参数列表的默认构造函数，例如：

```typescript
class Point {
  x: number = 0;
  y: number = 0;
}
let p = new Point();
```

在这种情况下，默认构造函数使用字段类型的默认值来初始化实例中的字段。



**构造函数重载签名**

我们可以通过编写重载签名，指定构造函数的不同调用方式。

具体方法为，为同一个构造函数写入多个同名但签名不同的构造函数头，构造函数实现紧随其后。

```typescript
class C {
  constructor(x: number)             /* 第一个签名 */
  constructor(x: string)             /* 第二个签名 */
  constructor(x: number | string) {  /* 实现签名 */
  }
}
let c1 = new C(123);      // OK，使用第一个签名
let c2 = new C('abc');    // OK，使用第二个签名
```

如果两个重载签名的名称和参数列表均相同，则为错误。

#### 8. 可见性修饰符

类的方法和属性都可以使用可见性修饰符。

可见性修饰符包括：private、protected和public。默认可见性为public。

**Public（公有）**

public修饰的类成员（字段、方法、构造函数）在程序的任何可访问该类的地方都是可见的。

**Private（私有）**

private修饰的成员不能在声明该成员的类之外访问，例如：

```typescript
class C {
  public x: string = '';
  private y: string = '';
  set_y (new_y: string) {
    this.y = new_y; // OK，因为y在类本身中可以访问
  }
}
let c = new C();
c.x = 'a'; // OK，该字段是公有的
c.y = 'b'; // 编译时错误：'y'不可见
```

**Protected（受保护）**

protected修饰符的作用与private修饰符非常相似，不同点是protected修饰的成员允许在派生类中访问，例如：

```typescript
class Base {
  protected x: string = '';
  private y: string = '';
}
class Derived extends Base {
  foo() {
    this.x = 'a'; // OK，访问受保护成员
    this.y = 'b'; // 编译时错误，'y'不可见，因为它是私有的
  }
}
```



#### 9.对象字面量

> 可用于代替`new`创建实例

> [!IMPORTANT]
>
> 如果类中包含private属性，无法通过对象字面量初始化该类。



对象字面量是一个表达式，可用于创建类实例并提供一些初始值。它在某些情况下更方便，可以用来代替new表达式。

对象字面量的表示方式是：封闭在花括号对({})中的'属性名：值'的列表。

```typescript
class C {
  n: number = 0;
  s: string = '';
}

let c: C = {n: 42, s: 'foo'};
```

ArkTS是静态类型语言，如上述示例所示，对象字面量只能在可以推导出该字面量类型的上下文中使用。其他正确的例子：

```typescript
class C {
  n: number = 0;
  s: string = '';
}

function foo(c: C) {}

let c: C

c = {n: 42, s: 'foo'};  // 使用变量的类型
foo({n: 42, s: 'foo'}); // 使用参数的类型

function bar(): C {
  return {n: 42, s: 'foo'}; // 使用返回类型
}
```

也可以在数组元素类型或类字段类型中使用：

```typescript
class C {
  n: number = 0;
  s: string = '';
}
let cc: C[] = [{n: 1, s: 'a'}, {n: 2, s: 'b'}];
```



**【反例】**

```kotlin
class C {  count: number = 0
  getCount(): number {    return this.count  }}
```

**【正例】**

```kotlin
class C {  private count: number = 0
  public getCount(): number {    return this.count  }}
```



#### **10. Record类型的对象字面量**

> 同 cpp java 中的mao，只是对 **键** 的数据类型有限定：只能为 `string` 或 `number`

泛型Record<K, V>用于将类型（键类型）的属性映射到另一个类型（值类型）。常用对象字面量来初始化该类型的值：

```typescript
let map: Record<string, number> = {
  'John': 25,
  'Mary': 21,
}

map['John']; // 25
```

类型`Key`可以是字符串类型或数值类型，而`Value`可以是任何类型。

```typescript
interface PersonInfo {
  age: number;
  salary: number;
}
let map: Record<string, PersonInfo> = {
  'John': { age: 25, salary: 10},
  'Mary': { age: 21, salary: 20}
}
```



#### 11. 泛型

##### 泛型类

类和接口可以定义为泛型，将参数添加到类型定义中，如以下示例中的类型参数Element：

```typescript
class CustomStack<Element> {		//Element:泛型名称，自定义，可以起任何名字
  public push(e: Element):void {
    // ...
  }
}
```

要使用类型CustomStack，必须为每个类型参数指定类型实参：

```typescript
let s = new CustomStack<string>();
s.push('hello');
```

编译器在使用泛型类型和函数时会确保类型安全。参见以下示例：

```typescript
let s = new CustomStack<string>();
s.push(55); // 将会产生编译时错误
```

##### 泛型函数

使用泛型函数可编写更通用的代码。比如返回数组最后一个元素的函数：

```typescript
function last(x: number[]): number {
  return x[x.length - 1];
}
last([1, 2, 3]); // 3
```

如果需要为任何数组定义相同的函数，使用类型参数将该函数定义为泛型：

```typescript
function last<T>(x: T[]): T {
  return x[x.length - 1];
}
```

在函数调用中，类型实参可以显式或隐式设置：

```typescript
// 显式设置的类型实参
last<string>(['aa', 'bb']);
last<number>([1, 2, 3]);

// 隐式设置的类型实参
// 编译器根据调用参数的类型来确定类型实参
last([1, 2, 3]);
```