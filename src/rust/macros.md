# 宏

[The Little Book of Rust Macros](https://veykril.github.io/tlborm/)

宏是一种为了写其他代码而写代码的方式，即**元编程**(metaprogramming)。

在一个文件里调用宏 **之前** 必须**定义**它，或将其**引入**作用域。

**声明**(declarative)宏使用 `macro_rules!` 声明。

**过程**(procedural)宏分三类：

- 自定义 `#[derive]` 宏在结构体和枚举上指定通过 `derive` 属性添加的代码
- 类属性（Attribute-like）宏定义可用于任意项的自定义属性
- 类函数宏看起来像函数不过作用于作为参数传递的 token



### macro_rules!宏

Rust 最常用的宏形式是 **声明宏**（*declarative macros*）。别名 macros by example、macro_rules!宏、macros.

核心感念是类似 `match` 的模式匹配，但是与其模式匹配语法有所不同。宏的模式语法请查阅[rust参考]([Macros By Example - The Rust Reference (rust-lang.org)](https://doc.rust-lang.org/reference/macros-by-example.html))

下为 `vec!` 的简化定义：

```
#[macro_export] // 该宏可以被导入
macro_rules! vec { // 定义一个macro宏 命名为vec
    ( $( $x:expr ),* ) => { // ()为一个分支的模式，可有多个分支 逗号是匹配可有可无的分割符 *为0到多次。
        {
            let mut temp_vec = Vec::new();
            $( // 匹配的表达式使用
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```



### 过程宏

**过程宏**（*procedural macros*），因为它们更像函数（一种过程类型）。

创建过程宏时，其定义**必须**驻留在它们自己的具有特殊 crate 类型的 crate 中。

大致样子：

```
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```



#### derive宏

顺序：创建trait的crate --> lib声明trait --> 在本crate创建宏crate --> 编写宏

```
$ cargo new hello_macro --lib
```

文件src/lib.rs中声明trait

```
pub trait HelloMacro {
    fn hello_macro();
}
```

过程宏必须在自己的ctate中

派生过程宏包为当前cratename_derive 此为惯例。

```
$ cargo new hello_macro_derive --lib
```

文件hello_macro_derive/src/Cargo.toml

```
[lib]
proc-macro = true

[dependencies]
syn = "1.0"
quote = "1.0"
```

文件hello_macro_derive/src/lib.rs

```
use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)] // 注解宏类型和名称HelloMacro
pub fn hello_macro_derive(input: TokenStream) -> TokenStream { // 该函数负责解析TokenStream
    
    let ast = syn::parse(input).unwrap(); // 返回DeriveInput结构体

    impl_hello_macro(&ast)
}

// 负责转换语法树及实现trait的逻辑
fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! { // 在该宏中编写希望返回的代码
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}!", stringify!(#name)); // stringify转换为字符常量
            }
        }
    };
    gen.into() // 转为TokenStream
}
```

另外一种组织方式可以将`hello_macro_derive` 在 `hello_macro` 中 `pub use` 重导出下。



#### 类属性宏

类属性宏能让你创建新的属性。

实现方式类似derive宏

```
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {}
// attr相当如下例的 GET, "/"  item相当如fn index() {}
// 使用宏
#[route(GET, "/")]
fn index() {}
```



#### 类函数宏

类函数（Function-like）宏的定义看起来像函数调用的宏。

```
#[proc_macro]
pub fn make_answer(_item: TokenStream) -> TokenStream {
    "fn answer() -> u32 { 42 }".parse().unwrap()
}

// main.rs
make_answer!();
fn main() {
    println!("{}", answer());
}
```

