# Cargo 和 crates.io

- Cargo -- 包管理工具。参考文档 [Cargo Book](https://doc.rust-lang.org/cargo/)
- crates.io -- 官方crate库。地址 [crates.io](https://crates.io/)

---

### 发布到crates.io

#### 创建账号

需要[crates.io](https://crates.io/)创建账号， 并复制自己的API token使用 `cargo login` 关联

```
$ cargo login your_API_token
```

该API token会记录在~/.cargo/credentials.

#### 添加元信息

具体信息请参考：[Mamifest](https://doc.rust-lang.org/cargo/reference/manifest.html)

如下边的Cargo.toml

```
[package]
// 以下为必须项
name = "crate_name" // 该名字在crates.io上必须唯一
version = “0.1.0”
edition = "2021"
description = "balabala"
license = "MIT OR Apache-2.0" // 请参考下方的SPDX, 使用不在SPDX列表的需要放一个license文件在根目录，该行换成 license-file

[dependencies]
```

[SPDX License List | Software Package Data Exchange (SPDX)](https://spdx.org/licenses/)

#### 发布

发布为永久性的不可被覆盖或删除，然而，被发布的版本号没有限制。

```
$ cargo publish
```

#### 发布新版本

秩序修改Cargo.toml中的 `version`即可重新 `cargo publish`

`version`使用 [SemVer](https://semver.org/)

#### yank

可以撤回(yanking)某个版本，以阻止心项目加入依赖该版本，该操作不影响旧有使用：

```
$ cargo yank --vers 1.0.1
```

撤回之后可以撤销这个动作：

```
$ cargo yank --vers 1.0.1 --undo
```



### Cargo 工作空间

可管理多个相关的相同开发包。

#### 创建工作空间

在工作空间文件夹下建Cargo.toml

```
[workspace]

members = [
    "adder",
]
```

新建`adder`crate

```
$ cargo new adder
```

同一工作空间使用同一个构建目录target.

#### 增加成员

变动Cargo.toml

```
[workspace]

members = [
    "adder",
    "add_one",
]
```

新建增加的库

```
$ cargo new add_one --lib
```

如果两个crate之间有依赖关系，需要手动在crate自己的Cargo.toml中增加依赖关系。

在工作空间运行对应包

```
$ cargo run -p adder
```

#### 外部依赖

工作空间只维护一个Cargo.lock来确保成员使用的同一个库的版本兼容性，即只使用同一个版本。

#### test

可直接所有`cargo test`

也可以指定库`cargo test -p add_one`

### cargo

支持install二进制文件 同时可以安装cargo扩展命令