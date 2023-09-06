# 并发

**并发编程**（*Concurrent programming*），代表程序的不同部分相互独立的执行。

**并行编程**（*parallel programming*）代表程序不同部分于同时执行。

---

### 多线程

程序执行再**进程**(process)中，同一进程中可以有多个线程(threads).

`thread::spawn` 需传一个闭包，闭包即为新线程运行代码。

`spawn`返回`JoinHandle`, `JoinHandle`有一个`join`方法，来等待线程运行结束。



### 进程间通信



#### 消息传递

**消息传递**(message passing) rust通过**信道**(channel)进行消息传递, 信道由 **发送者**(transmitter) 和**接收者**(receiver)组成.

`std::sync::mpsc::channel` 生成一个信道 返回一个元组 (**发送者, 接受者**)。

**`mpsc`** 为**多生产者，单消费者**（*multiple producer, single consumer*）的缩写。

发送者可以通过 **`发送者.clone`** 来获得多个发送者。



#### 共享状态

**共享**：让多个线程拥有相同的共享数据。

**互斥器**（*mutex*）是 *mutual exclusion* 的缩写，任意时刻只允许一个线程访问某些数据。要访问互斥器中的数据首先需要获得**锁**(lock)。

**`Mutex<T>`** 内部可变



### Sync和Send trait

`Send trait`: 实现了 `Send` 的类型值的所有权可以在线程间传送。

`Sync trait`: 实现了 `Sync` 的类型可以安全的在多个线程中拥有其值的引用。