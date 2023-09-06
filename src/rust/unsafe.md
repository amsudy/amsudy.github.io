# unsafe

使用 `unsafe` 开启不安全块。

不安全情况：

- 解引用裸指针
- 调用不安全的函数或方法
- 访问或修改可变静态变量
- 实现不安全 trait
- 访问 `union` 的字段

`unsafe` 块内除了上述五种情况，其他代码编译器仍然会做安全检查。



### 裸指针

**raw pointers** 有两种， 可变 **`*mut T`** 不可变 **`*const T`** .

裸指针与引用和智能指针的区别在于

- 允许忽略借用规则，可以同时拥有不可变和可变的指针，或多个指向相同位置的可变指针
- 不保证指向有效的内存
- 允许为空
- 不能实现任何自动清理功能



### 使用extern函数调用外部代码

`extern` 帮助与**外部函数接口**（*Foreign Function Interface*，FFI）交互。它的代码在rust看来总是不安全的，

需要在unsafe块执行。

**外部函数接口**是一个编程语言用以定义函数的方式，其允许不同（外部）编程语言调用这些函数。

```
extern "C" { // "C"定义了外部函数所使用的 应用二进制接口（application binary interface，ABI）
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

**ABI** 定义了如何在汇编语言层面调用此函数。

让函数可在其他语言使用：

```
#[no_mangle] // 告诉编译器不要对函数名mangling。
pub extern "C" fn call_from_c() { // "C"指定所用ABI
    println!("Just called a Rust function from C!");
}
```



### 静态变量和常量

|   类型   | 关键字 |      使用      | 可变 |    safe    |
| :------: | :----: | :------------: | :--: | :--------: |
| 静态变量 | static |  同一地址寻找  |  是  | 写时不安全 |
|   常量   | const  | 复制硬编码数据 |  否  |    安全    |



### 不安全trait

```
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}
```

