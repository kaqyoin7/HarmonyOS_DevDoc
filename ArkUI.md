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



### build()

`build()`用于**定义自定义组件**的声明式UI描述，自定义组件必须定义build()函数。(build()函数内部进行相应的声明式UI描述)

```typescript
@Entry
@Component
struct MyComponent {
  build() {
  }
}
```



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

下面的示例代码将父组件中的函数传递给子组件，并在子组件中调用

```typescript
@Entry
@Component
struct Parent {
  @State cnt: number = 0
	//此处submit: () => void的写法是为将成员类型显示声明为函数类型
    //同样可以依赖类型的自动推断而省略声明，直接 submit = () => {}，但在
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



### 补充说明：

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

- 不允许在UI描述里直接使用console.info，但允许在方法或者函数里使用，反例如下。

```typescript
build() {
  // 反例：不允许本地作用域
  {
    // ...
  }
}
```

* 不允许调用没有用 @Builder 装饰的方法（在将该方法作为UI组件使用的情况下不允许；若是作为事件监听函数，则可以通过箭头函数的形式不经 @Builder 修饰直接在 build() 中出现，详见后文build()中三种函数调用的对比），允许系统组件的参数是TS方法的返回值。

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





基本页面结构：

![image-20250612192022915](C:\Users\kaqyoin\AppData\Roaming\Typora\typora-user-images\image-20250612192022915.png)



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





## UIAbility相关



#### UIAbility生命周期![0000000000011111111.20250314165828.88406085354074403858398137027521:50001231000000:2800:DCDBE530D6D7EAE37CDAFDCED901B07FCC64FBC2AFEE04E6F3D3DAB827FC8B8C.png](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250314165828.88406085354074403858398137027521:50001231000000:2800:DCDBE530D6D7EAE37CDAFDCED901B07FCC64FBC2AFEE04E6F3D3DAB827FC8B8C.png)

#### UIAbility的Context（上下文信息）

[UIAbility](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-app-ability-uiability-V5)类拥有自身的上下文信息，该信息为[UIAbilityContext](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-inner-application-uiabilitycontext-V5)类的实例，[UIAbilityContext](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-inner-application-uiabilitycontext-V5)类拥有abilityInfo、currentHapModuleInfo等属性。

<span style="color:violet">通过UIAbilityContext可以获取UIAbility的相关配置信息，如包代码路径、Bundle名称、Ability名称和应用程序需要的环境状态等属性信息，以及可以获取操作UIAbility实例的方法（如[startAbility()](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-inner-application-uiabilitycontext-V5#uiabilitycontextstartability)、[connectServiceExtensionAbility()](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-inner-application-uiabilitycontext-V5#uiabilitycontextconnectserviceextensionability)、[terminateSelf()](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-inner-application-uiabilitycontext-V5#uiabilitycontextterminateself)等）。</span>

如果需要在页面中获得当前Ability的Context，可调用[getContext](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-getcontext-V5#getcontext)接口获取当前页面关联的UIAbilityContext或[ExtensionContext](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-inner-application-extensioncontext-V5)。
