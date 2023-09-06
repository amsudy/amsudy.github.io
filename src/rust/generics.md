# 泛型trait



### trait 对象

`Box<dyn Ma>`: 代表实现了`Ma trait`的类型。可以是多种类型。

trait 对象属于动态分发，会有性能损失。

trait 对象需要类型安全。

trait 方法符合下边规则就是对象安全的：

- 返回值不是`Self`
- 没有泛型类型的参数



### 关联类型

**关联类型**（*associated types*）是一个将类型占位符与 trait 相关联的方式，这样 trait 的方法签名中就可以使用这些占位符类型。

```
pub trait Iterator {
    type Item; // 关联类型类似泛型

    fn next(&mut self) -> Option<Self::Item>;
}
```

和用泛型实现的区别是：泛型可以多次实现`impl Iterator<String> for Counter`但是关联类型不行。



### 默认泛型类型

```
trait Add<Rhs=Self> { //默认的泛型类型为Self
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```



### 完全限定语法

类型优先调用自己的方法，如果实现了多个同名方法需要表明trait. `TaitName::method(&receiver)`

如果是关联函数，则需要使用**完全限定语法**`<Dog as Animal>::baby_name()`

```
<Type as Trait>::function(receiver_if_method, next_arg, ...); // 完全限定语法
```



### 父trait

子trait需要使用到父trait的功能。

```
trait Child: Parent {
	fn name();
}
```


