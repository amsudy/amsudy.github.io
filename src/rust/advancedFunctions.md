# 函数和闭包



### 函数指针

`fn` 被称为函数指针(function pointer)。例：fn(i32) -> i32

注意函数 `fn` 和闭包trait `Fn` 。

函数指针实现了 `Fn` `FnMut` `FnOnce` 三个闭包trait, 所以期望闭包的参数也可以用函数指针。



### 返回闭包

因为编译器要知道类型的大小也就是 `Sized` 所以如下：

```
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

