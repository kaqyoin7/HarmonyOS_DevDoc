## 自定义组件的基本结构



### struct

​	自定义组件基于struct实现，**struct + 自定义组件名 + {...}**的组合构成**自定义组件**，不能有继承关系。对于struct的实例化，可以省略new。

> [!note]
>
> 自定义组件名、类名、函数名不能和系统组件名相同。
>
> 系统组件名：Text、Button、Column、Row 等



### @Component

`@Component`装饰器仅能装饰`struct`关键字声明的数据结构。`@Component`可以接受一个可选的bool类型参数。

> [!Important]
>
> `struct`被`@Component`装饰后具备**组件化**的能力，**需要实现`build`方法**描述 UI，一个`struct`只能被一个`@Component`装饰。

```typescript
@Component
struct MyComponent {
}
```



### @Entry

`@Entry`装饰的自定义组件将作为UI页面的入口。在单个UI页面中，最多可以使用@Entry装饰一个自定义组件。@Entry可以接受一个可选的[LocalStorage](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-localstorage-V5)的参数。

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

`build()`用于**定义自定义组件**的声明式UI描述，自定义组件必须定义build()函数。

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



#### 补充说明：

所有声明在`build()`函数的语句，我们统称为**UI描述**，UI描述需要遵循以下规则：

- `@Entry`装饰的自定义组件，其`build()`函数下的**根节点唯一且必要**，且**必须为容器组件**，其中ForEach禁止作为根节点。
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





## 自定义组件的基本用法

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

