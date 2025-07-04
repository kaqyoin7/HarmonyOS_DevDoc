[TOC]



### 🧱 HarmonyOS 中的开发架构：

```kot
ArkTS语言层
 └── struct / interface / class / enum / logic
ArkUI框架层
 └── @Component / @Entry / build() / 状态管理（@State/@Link）/ 生命周期钩子
系统运行时（Stage模型）
 └── 应用生命周期 / Ability管理 / 页面路由 / 进程通信
```



## 自定义组件的基本结构

### @Component （=>ArkUI范畴）

`@Component`装饰器仅能装饰`struct`关键字声明的数据结构。`@Component`可以接受一个可选的bool类型参数。

> `struct`被`@Component`装饰后具备**组件化**的能力，**需要实现`build`方法**描述 UI，一个`struct`只能被一个`@Component`装饰。

```typescript
@Component
struct MyComponent {}
```



### struct（=>ArkTS范畴）

​	`struct`可定义结构体，可结合`@Component`定义自定义组件，**@Component + struct + 自定义组件名 + {...}**的组合构成**自定义组件**，不能有继承关系。对于struct的实例化，可以省略new。

> 自定义组件名、类名、函数名不能和系统组件名相同。
>
> 系统组件名：Text、Button、Column、Row 等

> [!tip]
>
> **<span style="font-size:25px">Q & A:</span>**  
>
> ArkTS中使用struct关键字定义一个自定义组件，为什么还需要使用@Component修饰   ***？***
>
> #### ✅ `@Component` 的作用是明确结构体的**组件语义**
>
> 虽然用 `struct` 定义了一个结构体，但在 ArkTS 中，并不是所有 `struct` 都代表 **UI组件**。使用 `@Component` 装饰器是为了告诉 ArkTS 编译器和框架：
>
> > “这个 struct 是一个自定义组件（Component），应该参与页面渲染生命周期。”
>
> #### 📌不加 `@Component` :
>
> 如果省略 `@Component`，该 `struct` 就只是一个**<span style="color:green">普通数据结构</span>**，ArkTS 不会把它当作 UI 组件去解析和渲染。因此：
>
> - `build()` 方法不会被执行；
> - 无法在页面中像组件一样使用（例如 `<MyComponent />`）；
> - 生命周期钩子（如 `aboutToAppear`、`aboutToDisappear`）不会触发；
> - 无法响应式绑定或使用 `@State` 等组件状态特性。
>
> 
>
> 是否可以理解为 `struct` 是类似 C++ 中的数据结构？
>
> > 可以这样理解，但要加上**ArkTS 的特点**来完整把握：
> >
> > * 在 **ArkTS 中，`struct` 既可以用于定义普通结构体数据类型（如 DTO、Model），也可以作为组件的承载体**；（由于HarmonyOS前后端可使用ArkTS统一开发）
> >
> > * 当一个 `struct` 被 `@Component` 修饰时，它就被 ArkUI 识别为具有生命周期、可渲染的 UI 组件；
> >
> > * 如果没有 `@Component`，它就是普通的 ArkTS 数据结构，**不会参与 UI 渲染系统**

`struct` 是 ArkTS 的<span style="color:green">结构定义关键字</span>，而 `@Component` 是告诉系统这个结构是一个**<span style="color:green">UI组件</span>**，需要框架管理它的生命周期、状态和渲染。

| Item         | 属于范畴              | 作用说明                                                     |
| ------------ | --------------------- | ------------------------------------------------------------ |
| `struct`     | ✅ **ArkTS**           | ArkTS 中定义结构体的语法关键字，类似于 C/C++/Rust 中的 `struct` |
| `@Component` | ✅ **ArkUI**（UI框架） | 一个 **装饰器**，用来标记 `struct` 是一个可以被框架识别为 **UI 组件** 的结构体 |



### @Entry

`@Entry`装饰的自定义组件将作为UI页面的入口。在单个UI页面中，最多可以使用`@Entry`装饰一个自定义组件。`@Entry`可以接受一个可选的[LocalStorage](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-localstorage-V5)的参数。(表明该组件为一个入口页面，可以通过UIAbility加载，也可以通过路由来访问)

```typescript
@Entry
@Component
struct MyComponent {
}

@Entry({ routeName : 'myPage' })
@Component
struct MyComponent {
}
```

| 名称                | 类型                                                         | 必填 |                             说明                             |
| :------------------ | :----------------------------------------------------------- | :--- | :----------------------------------------------------------: |
| routeName           | string                                                       | 否   |                 表示作为命名路由页面的名字。                 |
| storage             | [LocalStorage](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-localstorage-V5) | 否   |                     页面级的UI状态存储。                     |
| useSharedStorage12+ | boolean                                                      | 否   | 是否使用LocalStorage.getShared()接口返回的[LocalStorage](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-localstorage-V5)实例对象，默认值false。 |



### @State

`@State` 用于定义**组件内部的本地状态变量**。当它的值发生变化时，**会自动触发 UI 重新构建（build）更新界面**。

🧪 示例代码

```ts
@Component
struct Counter {
  @State count: number = 0;

  build() {
    Column() {
      Text(`当前计数：${this.count}`)
      Button("增加")
        .onClick(() => {
          this.count++;  // 修改状态，UI 自动更新
        })
    }
  }
}
```

在这个例子中：

- `count` 是组件的内部状态；
- 每点击一次按钮，`count++`；
- **页面会自动刷新**，重新渲染最新的 `count`。



> **Q&A**
>
> **Q1**：不使用 `@State`，改用普通变量行不行？
>
>  =》**if/else渲染**部分：if和else if后的条件语句可以使用状态变量或常规变量（状态变量的值改变时会实时渲染UI，而常规变量的值改变则不会）[从简单的页面开始-华为开发者学堂](https://developer.huawei.com/consumer/cn/training/course/slightMooc/C101717497398588123?pathId=101667550095504391)
>
> - ✅ 语法允许，但 ❌ 不推荐。
> - 普通变量的变化 **不会触发 UI 更新**，因此在声明式 UI 中 **失去了响应性**。
>
> ```ts
> // 这样写变量不会触发 UI 变化
> private count: number = 0;
> ```
>
> 
>
> **Q2：**为什么说不适用@State声明就无法触发UI更新？上述案例中不是直接在Text中显示count的值吗？
>
> **ArkUI 是声明式 UI 框架**，**视图更新的触发机制依赖于状态变更的监听**。这和传统命令式 UI（如 Android Java 或原生 HTML/JS）不同。
>
> `Text(${this.count})`<span style="color:green">**会根据当时的值渲染一次**</span>，但如果不使用@State，<span style="color:green">**后续 count 改变了，UI 不会更新**</span>，因为系统不会“察觉”这个变化，ArkUI <span style="color:green">**不会认为页面需要重新 build**</span>，所以 UI 上 `Text(this.count)` 并不会自动更新。
>
> 换句话说：
>
> - 初次渲染： ✅ 正常
> - 值变后 UI 更新：❌ 不会（除非用 @State）
>
> 
>
> 在 ArkUI 中：
>
> ```ts
> private count: number = 0;
> ```
>
> 是一个**普通变量**，**ArkUI 不会自动追踪它的变化**。如果你修改它：
>
> ```ts
> this.count++;
> ```
>
> ArkUI <span style="color:green">**不会认为页面需要重新 build**</span>，所以 UI 上 `Text(this.count)` 并不会自动更新。
>
> 
>
> 使用 `@State`：
>
> ```ts
> @State count: number = 0;
> ```
>
> 系统会在内部将 `count` 转化为**响应式代理对象**，一旦值变化，ArkUI 会**自动触发组件重新构建**：
>
> ```ts
> this.count++; // UI 自动刷新！
> ```
>
> 
>
> **Q3：**是否能使用if、else来代替状态变量对UI更新的触发？
>
> * 🚫 `if` / `else` 无法代替状态更新触发机制
>
> `if/else` 只是一种**控制视图结构的条件语句**，本身**不会触发视图重构**。你可以写：
>
> ```ts
> build() {
>   Column() {
>     if (this.flag) {
>       Text('显示 A')
>     } else {
>       Text('显示 B')
>     }
>   }
> }
> ```
>
> 但这段逻辑依赖于 `this.flag` 的值。**如果 `flag` 不是用 `@State` 或其他响应式变量修饰，它变化了，页面也不会自动更新**。





### 声明式编程与传统命令式编程的差别

**Q：**传统的js与html在无@State机制下仍能实现点击后立即渲染的效果，所以@State有什么意义？

> ## 🧠 传统 JS + HTML：**命令式编程**
>
> 在传统 Web 中，我们写的代码是这样的：
>
> ```
> html复制编辑<p id="counter">0</p>
> <button onclick="increment()">Click</button>
> 
> <script>
>   let count = 0;
> 
>   function increment() {
>     count++;
>     document.getElementById("counter").innerText = count; // 手动更新 DOM
>   }
> </script>
> ```
>
> ### ✅ 特点：
>
> - 我们自己负责 DOM 的“查找”和“修改”。
> - 数据变了，我们**手动更新视图**。
> - 控制权在开发者手里（但易出错，维护成本高）。
>
> ------
>
> ## 🌱 ArkUI：**声明式编程**
>
> ArkUI 是参考 React/Vue 的现代框架，其理念是：
>
> > “告诉框架我想**展示什么状态**（状态改变驱动的UI改变），而不是告诉它**怎么做**。”
>
> 例如：
>
> ```
> ts复制编辑@State count: number = 0;
> 
> build() {
>   Column() {
>     Text(`${this.count}`)
>     Button("Click")
>       .onClick(() => {
>         this.count++;
>       })
>   }
> }
> ```
>
> 你不再手动更新 `Text`，而是修改状态，ArkUI 自动更新界面。
>
> 🧭 总结对比
>
> | 对比项                  | 命令式传统开发（JS+HTML） | 声明式 ArkUI（@State）       |
> | ----------------------- | ------------------------- | ---------------------------- |
> | UI 是否自动响应状态变化 | ❌ 否，需手动 DOM 更新     | ✅ 是，自动视图重建           |
> | UI 状态管理方式         | 零散、易出错              | 响应式、集中、统一           |
> | 适合场景                | 简单页面，直接操作        | 复杂交互、多状态、组件化应用 |



### build()

所有声明在`build()`函数的语句，我们统称为**UI描述**，UI描述需要遵循以下规则：

- `@Entry`装饰的自定义组件，其`build()`函数下的**根节点唯一且必要**，且**<span style="color:red">必须为容器组件</span>**，其中ForEach禁止作为根节点。
- `@Component`装饰的自定义组件，其`build()`函数下的**根节点唯一且必要**，**可以为非容器组件**，其中ForEach禁止作为根节点。

```typescript
@Entry
@Component
struct MyComponent {
  build() {
    // 根节点唯一且必要，必须为容器组件
    Row() {
      ChildComponent() 
    }
  }
}

@Component
struct ChildComponent {
  build() {
    // 根节点唯一且必要，可为非容器组件
    Image('test.jpg')
  }
}
```

* 不允许声明本地变量，反例如下:

```typescript
build() {
  // 反例：不允许声明本地变量
  let num: number = 1;
}
```

**Q1：**ArkUI中若无法 在build中声明变量，那应该在何处声明？对于函数又是否能在build中声明呢？

> 在 ArkUI（基于 ArkTS）中，组件是类似“类结构”的 `@Component struct`，而 `build()` 方法是组件的渲染函数（类似 React 的 render），它应**仅关注 UI 构建**，**不推荐也不允许在其中定义状态性变量**。
>
> ArkUI 中的状态、属性等变量要在组件体内定义（`build()` 外）：
>
> ### ✅ 正确示例：
>
> ```ts
> @Component
> struct Example {
>   @State count: number = 0  // ✅ 在组件体内声明状态变量
> 
>   build() {
>     Column() {
>       Text(`${this.count}`)
>       Button("Add").onClick(() => {
>         this.count++   // 正确访问
>       })
>     }
>   }
> }
> ```
>
> ## 函数能否在 `build()` 中声明？
>
> 可以，但有条件。
>
> ### ✅ 合理示例（小函数 + 不依赖响应式）：
>
> ```ts
> build() {
>   const greet = () => {
>     console.log('Hello')
>   }
> 
>   Column() {
>     Button("Greet").onClick(() => greet())
>   }
> }
> ```
>
> 这类函数通常用于**事件回调封装**或**轻量逻辑处理**，但也存在如下问题：
>
> - 每次 `build()` 调用都会重新创建函数 → 性能负担增大。
> - 无法访问组件内响应式变量（需用 `this`）。
>
> ### ✅ 更推荐做法：函数作为组件方法成员
>
> ```ts
> @Component
> struct Demo {
>   @State count: number = 0
> 
>   increment() {
>     this.count++
>   }
> 
>   build() {
>     Column() {
>       Text(`${this.count}`)
>       Button("Add").onClick(() => this.increment()) // 更清晰，避免重复声明函数
>     }
>   }
> }
> ```

- 不允许在UI描述里直接使用console.info，但允许在方法或者函数里使用，反例如下。

```typescript
build() {
  // 反例：不允许本地作用域
  {
    // ...
  }
}
```

* 不允许调用没有用 @Builder 装饰的方法（在将该方法**作为UI组件使用的情况下不允许**；若是作为事件监听函数，则可以通过箭头函数的形式不经 @Builder 修饰直接在 build() 中出现，详见后文build()中三种函数调用的对比），允许系统组件的参数是TS方法的返回值。

```typescript
@Component
struct ParentComponent {
  doSomeCalculations() {
  }

  calcTextValue(): string {
    return 'Hello World';
  }

  @Builder doSomeRender() {
    Text(`Hello World`)
  }

  build() {
    Column() {
      // 反例：不能调用没有用@Builder装饰的方法
      this.doSomeCalculations();
      // 正例：可以调用
      this.doSomeRender();
      // 正例：参数可以为调用TS方法的返回值
      Text(this.calcTextValue())
    }
  }
}
```

* 不允许使用表达式，请使用if组件，示例如下。

```typescript
build() {
  Column() {
    // 反例：不允许使用表达式
    (this.aVar > 10) ? Text('...') : Image('...')


    // 正例：使用if判断
    if(this.aVar > 10) {
      Text('...')
    } else {
      Image('...')
    }
  }
}
```



#### ArkUI中的三种函数调用对比

| 函数类型                | 示例语法           | 作用场景                    | 是否自动绑定 `this`              | 是否能在 `build()` 中直接调用                             |
| ----------------------- | ------------------ | --------------------------- | -------------------------------- | --------------------------------------------------------- |
| ✅ 普通成员方法          | `myFn()`           | 逻辑处理/业务计算           | ❌ 不自动绑定，需要 `.bind(this)` | ❌ 不允许直接在 `build()` 中调用（除非用作参数）           |
| ✅ 箭头函数属性          | `fn = () => {}`    | 事件处理（如 `.onClick()`） | ✅ 自动绑定 `this`                | ✅ 可作为表达式调用（不能构造组件/不能作为UI组件返回使用） |
| ✅ `@Builder` 装饰器函数 | `@Builder fn() {}` | 用来返回 UI 元素            | ✅ 自动绑定（由 ArkUI 框架处理）  | ✅ 允许在 `build()` 中直接调用，等价于嵌套组件             |

**普通方法 + `bind(this)`（事件中使用）**

```typescript
myClickHandler(): void {
  this.counter += 2;
}

Button('add counter')
  .onClick(this.myClickHandler.bind(this))

```

* 普通成员方法 **不会自动绑定 `this`**；（在 JS 中，函数的调用者是谁，就决定了 `this` 指向谁）

* 所以需要手动 `.bind(this)`，否则 `this` 在事件回调中将指向 `undefined` 或其他作用域；

* 不推荐ArkTS 中使用这种方式，原因是：繁琐、容易出错、不易维护。



**箭头函数属性（推荐事件处理方式）**

```typescript
fn = () => {
  this.counter++;
}

Button('add counter')
  .onClick(this.fn)
```

* 箭头函数在定义时绑定了当前作用域，因此 `this` 自动绑定；

* 简洁、可靠，是 ArkTS 推荐的事件处理方式；

* 可用于 `onClick`、`onAppear` 等组件事件绑定。



**`@Builder` 装饰器函数（UI渲染用途） => @Builder不支持箭头函数**

```typescript
@Builder doSomeRender() {
  Text(`Hello World`)
}

build() {
  Column() {
    this.doSomeRender()  // ✅ 允许
      
  	@Builder add = () => { // ❌ 错误：Builder 不支持箭头函数
    	this.count++
  	}

  	renderInfo = () => {   // ❌ 错误：不能用箭头函数构建 UI
    	Text(`Wrong`)
  	}

  }
}
```

* `@Builder` 是 ArkUI 中 **专门用于构建 UI 片段的方法装饰器**；

* 被 `@Builder` 装饰的方法可以在 `build()` 方法中像组件一样调用；

* 它的返回值必须是 **可渲染的 UI 组件结构**。



> [!tip]
>
> 可将每个用`@Component + struct`申明的自定义组件看作一个**UI模块**或函数，而带`@Entry`的自定义组件：`@Entry + @component + struct`视作**拼装这些UI模块的主界面**/入口函数` main()`

```typescript
@Component
struct MyComponent {
  private countDownFrom: number = 0;
  private color: Color = Color.Blue;

  build() {
  }
}

@Entry
@Component
struct ParentComponent {
  private someColor: Color = Color.Pink;

  build() {
    Column() {
      // 创建MyComponent实例，并将创建MyComponent成员变量countDownFrom初始化为10，将成员变量color初始化为this.someColor
      MyComponent({ countDownFrom: 10, color: this.someColor })
    }
  }
}
```



### 自定义组件与页面的声明周期

[从简单的页面开始-华为开发者学堂](https://developer.huawei.com/consumer/cn/training/course/slightMooc/C101717497398588123?pathId=101667550095504391)



## 自定义组件的定义与使用

### 成员函数/变量

自定义组件除了必须要实现build()函数外，还可以实现其他成员函数

**成员函数**具有以下约束：

- 自定义组件的成员函数为私有的，且不建议声明成静态函数。

自定义组件可以包含成员变量

**成员变量**具有以下约束：

- 自定义组件的成员变量为私有，且不建议声明成静态变量。
- 自定义组件的成员变量本地初始化有些是可选的，有些是必选的。具体是否需要本地初始化，是否需要从父组件通过参数传递初始化子组件的成员变量，请参考[状态管理](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-state-management-overview-V5)。



### 自定义组件的传参

在build方法里创建自定义组件，在创建自定义组件的过程中，根据装饰器的规则来初始化自定义组件的参数。

```typescript
@Component
struct MyComponent {
  private countDownFrom: number = 0;
  private color: Color = Color.Blue;
    
  build() {
  }
}

@Entry
@Component
struct ParentComponent {
  private someColor: Color = Color.Pink;

  build() {
    Column() {
      // 创建MyComponent实例，并将创建MyComponent成员变量countDownFrom初始化为10，将成员变量color初始化为this.someColor
      MyComponent({ countDownFrom: 10, color: this.someColor })
    }
  }
}
```

> <span style="color:green">**实用技巧**</span>：如果担心写起来繁琐，可以这样写:
>
> ```tsx
> let initProps = {
>   countDownFrom: 10,
>   color: this.someColor
> }
> MyComponent(initProps)
> ```
>
> 这在组件参数比较多时有利于**清晰整理和复用**。

下面的示例代码将父组件中的函数传递给子组件，并在子组件中调用

```typescript
@Entry
@Component
struct Parent {
  @State cnt: number = 0
	//此处submit: () => void的写法是为将成员类型显示声明为函数类型
    //同样可以依赖类型的自动推断而省略声明，直接 submit = () => {}
    submit: () => void = () => {
    this.cnt++;
  }

  build() {
    Column() {
      Text(`${this.cnt}`)
      Son({ submitArrow: this.submit })
    }
  }
}


@Component
struct Son {
  submitArrow?: () => void
//只是定义类型，没有赋初值；常用于组件的入参属性（用于接收父组件传入的回调函数）；
//若没有赋值，运行时默认值为 undefined；加 ? 是为了在调用时先判断是否存在。
  build() {
    Row() {
      Button('add')
        .width(80)
        .onClick(() => {
          if (this.submitArrow) {
            this.submitArrow()
          }
        })
    }
    .height(56)
  }
}
```



### 自定义组件示例

```typescript
//自定义组件
@Component
struct HelloComponent {
  @State message: string = 'Hello, World!';

  build() {
    // HelloComponent自定义组件组合系统组件Row和Text
    Row() {
      Text(this.message)
        .onClick(() => {
          // 状态变量message的改变驱动UI刷新，UI从'Hello, World!'刷新为'Hello, ArkUI!'
          this.message = 'Hello, ArkUI!';
        })
    }
  }
}
```



> [!important]
>
> 如果在另外的文件中引用该自定义组件，需要使用export关键字导出，并在使用的页面import该自定义组件。
>
> HelloComponent 可以在其他自定义组件中的build()函数中多次创建，实现自定义组件的重用。

```typescript
//在声明了@Entry的主页面中使用其他自定义组件
@Entry
@Component
struct ParentComponent {
  build() {
    Column() {
      Text('ArkUI message')
      HelloComponent({ message: 'Hello World!' });
      Divider()
      HelloComponent({ message: '你好，世界!' });
    }
  }
}
```



### 路由声明Q&A

> [!note]
>
> **Q & A**
>
> * 假设创建了一个页面LoginPage.ets，在其中声明了与文件名不同的入口组件Login。在main_pages.json中声明页面路由时，需声明的是页面文件名还是入口组件名称？
>
> > 在 ArkUI 中配置页面路由时，**你需要声明的是**：
> >
> > ✅ **组件名**，即 `@Component` 修饰的组件的名字（当前仅为特殊情况，因为页面入口被@Entry修饰的同时必定被@Component所修饰），而不是文件名
>
> 
>
> * 假设我在某个页面（HomePage）中仅声明了组件Home（@Component修饰），而未声明入口组件（@Entry），此时应当如何声明路由？
>
> > 在跳转的页面（如 `HomePage.ets`）中，只要定义了 `@Component`，并在 `main_pages.json` 中被正确引用，就可以作为跳转目标，无需加 `@Entry`。
>
> 
>
> * 如果一个页面中含多个组件，应当如何声明路由？
>
> > 如果一个页面文件中包含多个组件（即多个 `@Component` 声明），**只需要将你希望导航到的“目标页面组件”作为路由声明**，其余组件可以作为子组件被该页面引用，无需路由配置
>
> 
>
> * 在main_pages中声明路由的作用是什么？若不声明会出现什么情况？
>
> > 1. **注册合法的页面组件**
> >
> > 它将你希望通过 `router.push()`、`router.replace()` 等 API 跳转到的页面组件进行 **预注册**，让 ArkUI 路由系统能识别并加载这些页面。
> >
> > 2. **启用页面跳转功能**
> >
> > 只有被 `main_pages.pages` 显式声明过的组件，**才能作为目标页面使用路由跳转**。如果你没有在这里声明，`router.push()`到它时会失败。
> >
> > 3. **若未声明**
> >
> > 页面跳转失败、报错或黑屏，控制台输出类似错误：
> >
> > ```
> > Route failed: page LoginPage not found
> > ```





### this详解

**Q1：**在ArkUI中，this指什么？为什么在使用变量或函数时都需要使用this？若不使用this会怎样？

> ## ✅ `this` 在 ArkUI 中的含义
>
> 在 ArkUI 的组件（`@Component struct XXX {}`）中，`this` 代表的是**当前组件的实例对象**，就像类中访问成员变量和方法一样。
>
> ### 示例：
>
> ```
> ts复制编辑@Component
> struct MyComponent {
>   @State count: number = 0;
> 
>   increment() {
>     this.count++;
>   }
> 
>   build() {
>     Column() {
>       Text(`${this.count}`)
>       Button("Click").onClick(() => {
>         this.increment(); // 使用 this 来调用当前组件的方法
>       })
>     }
>   }
> }
> ```
>
> 在这个例子中：
>
> - `this.count` 访问的是当前组件的响应式状态变量。
> - `this.increment()` 是调用当前组件的方法。
>
> ------
>
> ## ❓为什么需要使用 `this`？
>
> 在 ArkTS 中，组件结构本质上是**类结构**（结构体 + 修饰符模拟类），为了避免变量作用域冲突，**访问组件内成员（变量/函数）必须显式用 `this.` 访问**。
>
> ### 原因：
>
> - 避免与局部变量重名。
> - 明确访问的是**组件实例的成员**。
> - 有利于提升代码可读性和调试性。
>
> ------
>
> ## 🚫 不使用 `this` 会怎样？
>
> ### 情况一：访问失败（找不到变量）
>
> ```
> ts复制编辑@Component
> struct Demo {
>   @State count: number = 0;
> 
>   build() {
>     Column() {
>       Text(`${count}`) // ❌ 报错：找不到变量 'count'
>     }
>   }
> }
> ```
>
> 解释：`count` 是组件成员，未在作用域中声明，ArkTS 会报错找不到该标识符。
>
> ------
>
> ### 情况二：调用失败（找不到函数）
>
> ```
> ts复制编辑@Component
> struct Demo {
>   sayHello() {
>     console.log("Hello");
>   }
> 
>   build() {
>     Button("Click").onClick(() => {
>       sayHello(); // ❌ 报错：Cannot find name 'sayHello'
>     })
>   }
> }
> ```
>
> 解释：你调用了一个“未知的全局函数”，而不是组件的 `sayHello()` 成员方法。应使用 `this.sayHello()`。

## UIAbility相关



#### UIAbility生命周期![0000000000011111111.20250314165828.88406085354074403858398137027521:50001231000000:2800:DCDBE530D6D7EAE37CDAFDCED901B07FCC64FBC2AFEE04E6F3D3DAB827FC8B8C.png](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250314165828.88406085354074403858398137027521:50001231000000:2800:DCDBE530D6D7EAE37CDAFDCED901B07FCC64FBC2AFEE04E6F3D3DAB827FC8B8C.png)

#### UIAbility的Context（上下文信息）

[UIAbility](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-app-ability-uiability-V5)类拥有自身的上下文信息，该信息为[UIAbilityContext](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-inner-application-uiabilitycontext-V5)类的实例，[UIAbilityContext](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-inner-application-uiabilitycontext-V5)类拥有abilityInfo、currentHapModuleInfo等属性。

<span style="color:violet">通过UIAbilityContext可以获取UIAbility的相关配置信息，如包代码路径、Bundle名称、Ability名称和应用程序需要的环境状态等属性信息，以及可以获取操作UIAbility实例的方法（如[startAbility()](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-inner-application-uiabilitycontext-V5#uiabilitycontextstartability)、[connectServiceExtensionAbility()](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-inner-application-uiabilitycontext-V5#uiabilitycontextconnectserviceextensionability)、[terminateSelf()](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-inner-application-uiabilitycontext-V5#uiabilitycontextterminateself)等）。</span>

如果需要在页面中获得当前Ability的Context，可调用[getContext](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-getcontext-V5#getcontext)接口获取当前页面关联的UIAbilityContext或[ExtensionContext](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-inner-application-extensioncontext-V5)。
