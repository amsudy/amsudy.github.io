# 智能指针

### 指针

指针：一个包含内存地址的变量。

智能指针：一类数据结构，他们的表现类似指针，但是也拥有额外的元数据和功能。

#### Deref trait

**Deref** trait: 使实现者可以重载解引用运算符**`*`**，像常规引用一样处理该实现者。

**`*`**： 底层转换为 **`*(x.deref())`** 且只会发生一次不会递归转换。

隐式Deref强制转换：将实现了 `Deref` trait 的类型的引用转换为另一种类型的引用。例如：`&String` 转为 `&str`

`DerefMut` trait:用于重载可变引用的 `*` 运算符.

Rust 在发现类型和 trait 实现满足三种情况时会进行 Deref 强制转换：

- 当 `T: Deref<Target=U>` 时从 `&T` 到 `&U`。
- 当 `T: DerefMut<Target=U>` 时从 `&mut T` 到 `&mut U`。
- 当 `T: Deref<Target=U>` 时从 `&mut T` 到 `&U`。

#### Drop trait

Drop trait: 使实现者失效之前，执行drop方法中的操作。一般为释放资源

`std::mem::drop` 可提前执行drop方法中的操作。

### Box\<T\>

Box\<T\>:在堆上存储数据。

使用场景：

- 当有一个在编译时未知大小的类型，而又想要在需要确切大小的上下文中使用这个类型值的时候
- 当有大量数据并希望在确保数据不被拷贝的情况下转移所有权的时候
- 当希望拥有一个值并只关心它的类型是否实现了特定 trait 而不是其具体类型的时候

### Rc\<T\>

`Rc\<T\>`: Rc为*reference counting* 的缩写，可以多所有权，只读。

`Rc::clone(var)`或`var.clone()` 返回一个强引用同时计数增加，引用为0时清理。前者可以更好的表示意图（推荐）。

`Rc::downgrade` 弱引用。不影响数据清理。

`Weak\<T\>` donwgrade返回类型，可以调用upgrade获取值。

`Rc::strong_count(var)` 返回强引用计数。

`Rc::weak_count(var)` 返回弱引用计数。

### RefCell\<T\>

概述：内部可变，运行时检查借用规则，唯一所有权。

`var.borrow_mut()` 可变借用。

`var.boorow()` 不可变借用。

### 对比

|     类型     | 所有者 | 借用检查 | 可变性检查 |
| :----------: | :----: | :------: | :--------: |
|   Box\<T\>   |   单   |  编译时  |    r/w     |
|   Rc\<T\>    |   多   |  编译时  |     r      |
| RefCell\<T\> |   单   |  运行时  |    r/w     |

