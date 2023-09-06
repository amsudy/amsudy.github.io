# 高级类型



### newtype

`struct NewName(i32)` NewName类型会隐藏所有内部类型的方法，实现 `Deref trait` 可以拥有所有内部方法。

由于NewName是本地类型所以可以实现其他crate的trait.



### 类型别名

`type Kilometers = i32` 创建类型的同义词。别名不是新的类型，会被当做原类型对待。

可用来减少重复，将**复杂的类型名**创建一个**简单**的类型别名。

```
type Thunk = Box<dyn Fn() + Send + 'static>;
```



### 从不返回的never type

**`!`** empty type rust一般叫做never type  代表没有值。

从不返回的函数被称为 **发散函数**（*diverging functions*）。



### 动态大小类型

**动态大小类型**（*dynamically sized types*） 被称作DST或unsized types

编译器必须知道类型的大小，并且同一个类型大小不许一致。所以动态类型要置于某种指针之后。

与之相反有一个 `Sized trait`

**`Sized trait`** 系统自动为一直大小类型实现。同时给函数隐式增加了 `Sized` bound:

```
fn generic<T>(t: T) {
    // --snip--
}
// 实际上被当作如下处理
fn generic<T: Sized>(t: T) {
    // --snip--
}
// 可以使用如下特殊语法来放宽这个限制
fn generic<T: ?Sized>(t: &T) { // 注意T要加&
    // --snip--
}
```

