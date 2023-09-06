# rust windows ReadDirectoryChangesW

 [ReadDirectoryChangesW](https://learn.microsoft.com/zh-cn/windows/win32/api/winbase/nf-winbase-readdirectorychangesw) :官方给出的系统最低需求为 **windwos xp**。

用rust使用windows常用的 **ReadDirectoryChangesW** 函数实现文件监控动作。



### 同步示例

示例没有处理错误，没有考虑关闭问题。

Cargo.toml

```
[package]
name = "test_rs"
version = "0.1.0"
edition = "2021"

[dependencies]

[dependencies.windows-sys]
version = "0.48"
features = [
    "Win32_Foundation",
    "Win32_Security",
    "Win32_System_Threading",
    "Win32_UI_WindowsAndMessaging",
    "Win32_Storage_FileSystem",
    "Win32_System_IO",
]
```



main.rs

```
use std::{
    ffi::c_void,
    fs::OpenOptions,
    io::Error,
    os::windows::prelude::{AsRawHandle, OpenOptionsExt},
    path::Path,
    ptr::null_mut,
    slice::from_raw_parts,
};

use windows_sys::Win32::Storage::FileSystem::{
    ReadDirectoryChangesW, FILE_FLAG_BACKUP_SEMANTICS, FILE_LIST_DIRECTORY,
    FILE_NOTIFY_CHANGE_ATTRIBUTES, FILE_NOTIFY_CHANGE_CREATION, FILE_NOTIFY_CHANGE_DIR_NAME,
    FILE_NOTIFY_CHANGE_FILE_NAME, FILE_NOTIFY_CHANGE_LAST_WRITE, FILE_NOTIFY_CHANGE_SECURITY,
    FILE_NOTIFY_CHANGE_SIZE, FILE_SHARE_DELETE, FILE_SHARE_READ, FILE_SHARE_WRITE,
};

fn main() {
    let path = Path::new("ts");

    // rust在windwos平台时实现了std::os::windows::fs::OpenOptionsExt trait
    // 详情标准库或源码。
    let mut option = OpenOptions::new();
    
    // FILE_LIST_DIRECTORY在ReadeDirectoryChangesW中有说明需要或者GENERIC_READ
    // 对应CreateFile的dwDesiredAccess。
    option.access_mode(FILE_LIST_DIRECTORY); 
    
    // 对应CreateFile的dwShareMode.
    option.share_mode(FILE_SHARE_READ | FILE_SHARE_WRITE | FILE_SHARE_DELETE); 

    // CreateFile的 dwCreationDisposition是根据create, truncate, create_new判断自动得出的。
    // 源码地址C:\Users\mzha\Desktop\hobby\rs\rust\library\std\src\sys\windows\fs.rs 大约253行get_creation_mode

    // attributes | custom_flags | securite_qos_flags三个属性是合并一起的随便设置哪个
    // 对应CreateFile的另外几个设置，看源码了解详情。
    // FILE_FLAG_BACKUP_SEMANTICS在ReadeDirectoryChangesW中有说明要获得文件夹的handle需要它。
    // 我们时同步就不需要FILE_FLAG_OVERLAPPED标志了。
    option.custom_flags(FILE_FLAG_BACKUP_SEMANTICS); 

    // 如果用CreateFileW的话记得path要转宽字节。
    // rust包装后的handle实现了Drop trait会自动关闭资源。
    let handle = option.open(path).unwrap(); 

    let raw_handle = handle.as_raw_handle() as isize;
    // buf是FILE_NOTIFY_INFORMATION结构
    // 记录文件变化信息。
    // 两次ReadDirectoryChangesW调用之间的变化都会写入改buf.
    let mut buf = [0u8; 1024];
    let lpbuf = buf.as_mut_ptr() as *mut c_void;

    let notis = FILE_NOTIFY_CHANGE_FILE_NAME
        | FILE_NOTIFY_CHANGE_DIR_NAME
        | FILE_NOTIFY_CHANGE_ATTRIBUTES
        | FILE_NOTIFY_CHANGE_SIZE
        | FILE_NOTIFY_CHANGE_LAST_WRITE
        | FILE_NOTIFY_CHANGE_CREATION
        | FILE_NOTIFY_CHANGE_SECURITY;
    let mut bn = 0u32;
    let lpbn = &mut bn as *mut u32;

    loop {
        // 具体字段含义查看官方解释。
        let ok = unsafe {
            ReadDirectoryChangesW(
                raw_handle,      // handle
                lpbuf,           // buf
                buf.len() as _,  // buf长度
                1,               // 是否递归监控子目录
                notis,           // 监控的事项flags
                lpbn,            // buf写入了多少Bytes
                null_mut() as _, // 同步操作此值写NULL。
                None,            // 同步操作不需要此值写None.
            )
        };

        if !ok == 1 {
            panic!("{}", Error::last_os_error());
        }
        // buf溢出了返回0
        if bn == 0 {
            continue;
        }

        let data = &buf[..bn as usize];
        let mut lpdata = data.as_ptr() as *const u32;

        // 处理FILE_NOTIFY_INFORMATION结构
        loop {
            // offset得长度为 count*lpdata指向类型， read读取lpdata指向得类型得一个长度
            let action = match unsafe { lpdata.offset(1).read() } {
                1 => "新增",
                2 => "删除",
                3 => "变更",
                4 => "更名前名字",
                5 => "更名后名称",
                _ => "",
            };
            let length = unsafe { lpdata.offset(2).read() };
            let lptr_u16 = unsafe { lpdata.offset(3) } as *const u16;
            let name_u16 = unsafe { from_raw_parts(lptr_u16, length as usize / 2) };
            let name = String::from_utf16(name_u16).unwrap();

            println!("action: {:?}, name: {:?}", action, name);
            if unsafe { lpdata.read() } == 0 {
                break;
            } else {
                // 第一个u32字段(即read的值)记录得是多少Bytes, 而offset需要的是指针类型的多少个字 u32和byte转换下
                lpdata = unsafe { lpdata.offset(lpdata.read() as isize / 4) };
            }
        }
    }
}
```



### 异步示例

错误处理未考虑

Cargo.toml

```
[package]
name = "testa_rs"
version = "0.1.0"
edition = "2021"

[dependencies]
ctrlc = "3.4.1"

[dependencies.windows-sys]
version = "0.48"
features = [
    "Win32_Foundation",
    "Win32_Security",
    "Win32_System_Threading",
    "Win32_UI_WindowsAndMessaging",
    "Win32_Storage_FileSystem",
    "Win32_System_IO",
]
```



main.rs

```
use std::{
    collections::HashMap,
    ffi::c_void,
    fs::{File, OpenOptions},
    io::{self, Error},
    mem::{self},
    os::windows::prelude::{AsRawHandle, OpenOptionsExt},
    path::Path,
    slice::from_raw_parts, sync::mpsc::channel,
};

use windows_sys::Win32::{
    Storage::FileSystem::{
        ReadDirectoryChangesW, FILE_FLAG_BACKUP_SEMANTICS, FILE_FLAG_OVERLAPPED,
        FILE_LIST_DIRECTORY, FILE_NOTIFY_CHANGE_ATTRIBUTES, FILE_NOTIFY_CHANGE_CREATION,
        FILE_NOTIFY_CHANGE_DIR_NAME, FILE_NOTIFY_CHANGE_FILE_NAME, FILE_NOTIFY_CHANGE_LAST_WRITE,
        FILE_NOTIFY_CHANGE_SECURITY, FILE_NOTIFY_CHANGE_SIZE, FILE_SHARE_DELETE, FILE_SHARE_READ,
        FILE_SHARE_WRITE,
    },
    System::{
        Threading::{SleepEx, INFINITE, QueueUserAPC},
        IO::{OVERLAPPED, CancelIo},
    },
};
fn main() {

    let (tx, rx) = channel();

    // 整个逻辑放闭包要在一个单独线程执行不用关注方式。
    let bb = move || {
        let ps = vec![
            Path::new("C:\\Users\\mzha\\Desktop\\tz"),
            Path::new("C:\\Users\\mzha\\Desktop\\ts"),
        ];

        // 放需要信息的结构体
        let mut watching_dir = HashMap::new();

        let access = FILE_LIST_DIRECTORY;
        let share = FILE_SHARE_READ | FILE_SHARE_WRITE | FILE_SHARE_DELETE;

        let mut file = OpenOptions::new();
        file.access_mode(access);
        file.share_mode(share);
        // 加上FILE_FLAG_OVRELAPPED开启异步。
        file.custom_flags(FILE_FLAG_BACKUP_SEMANTICS | FILE_FLAG_OVERLAPPED);

        // 因为异步可以通知执行多个ReadDirecotryChangesW
        for path in ps {
            let handle = file.open(path).unwrap();

            // 这个rust默认的主进程stack是8M，heap因该也是没验证，线程的可以设置大小。
            // 不要重用buf
            let buf = Box::new([0u8; 1024]);
            // 异步要用的 重叠 结构体， 不要复用
            let overlapped = unsafe { Box::new(mem::zeroed::<OVERLAPPED>()) };

            let change_dir = ChangeDir {
                path,
                handle,
                buf,
                overlapped,
            };
            watching_dir.insert(path, change_dir);

            let change_dir = watching_dir.get_mut(path).unwrap();
            // 我们使用完成例程获得改变信息，官方文档说明了该模式下hEvent可自用，系统不会使用。
            // hEvent 指向包含本身的change_dir， 用来在回调函数中再次调用ReadDirectoryChangesW。
            change_dir.overlapped.hEvent = change_dir as *const ChangeDir as isize;

            read_dir_change(change_dir).unwrap();
        }

        // ctrl+c时跳出循环。
        while !rx.try_recv().is_ok() {
            // 挂起当前线程，直到满足指定的条件执行。
            // 就是等待变更事件发生并执行。 还有其他方式参考官方文档。
            unsafe { SleepEx(INFINITE, 1) };
        }
        
    };

    // 在子线程运行主逻辑
    let hj = std::thread::spawn(bb); 

    let ti = hj.as_raw_handle() as isize;

    // 使用了ctrlc库，捕获了ctrl+c
    let _ = ctrlc::set_handler(move|| {
        // 通道告知使跳出循环
        tx.send(1).unwrap();
        // 因为SleepEx已经在等待了，让他继续才能判断跳出循坏。
        // 可能会报错这里没有处理 _
        let _ = unsafe { QueueUserAPC(Some(wu), ti, 0) };
    });

    hj.join().unwrap();
}

#[allow(dead_code)]
struct ChangeDir<'a> {
    path: &'a Path,
    handle: File,
    buf: Box<[u8; 1024]>,
    overlapped: Box<OVERLAPPED>,
}

impl Drop for ChangeDir<'_> {
    fn drop(&mut self) {
        // 取消挂起的IO
        let cie = unsafe { CancelIo(self.handle.as_raw_handle() as isize) };

        if cie != 1 {
            // 要处理错误，我未处理直接panic了
            panic!("{}", Error::last_os_error());
        }
    }
}

fn read_dir_change(cfd: &mut ChangeDir) -> Result<(), io::Error> {
    let lpbuf = cfd.buf.as_mut_ptr() as *mut c_void;
    let mut bn: u32 = 0;
    let lpbn = &mut bn as *mut u32;
    let notis = FILE_NOTIFY_CHANGE_FILE_NAME
        | FILE_NOTIFY_CHANGE_DIR_NAME
        | FILE_NOTIFY_CHANGE_ATTRIBUTES
        | FILE_NOTIFY_CHANGE_SIZE
        | FILE_NOTIFY_CHANGE_CREATION
        | FILE_NOTIFY_CHANGE_LAST_WRITE
        | FILE_NOTIFY_CHANGE_SECURITY;

    let re = unsafe {
        ReadDirectoryChangesW(
            cfd.handle.as_raw_handle() as isize,
            lpbuf,
            cfd.buf.len() as u32,
            1,
            notis,
            lpbn, // 异步这个值写0就行
            (&mut *(cfd.overlapped)) as *mut OVERLAPPED, // 异步需填写
            Some(rev), // 回调函数， 我们利用它进行循环调用。
        )
    };

    if re == 0 {
        return Err(io::Error::last_os_error());
    }
    Ok(())
}

unsafe extern "system" fn wu(_parameter: usize) -> () {}

unsafe extern "system" fn rev(err_code: u32, number: u32, lpo: *mut OVERLAPPED) {
    let cfd = &mut *lpo;
    let he = &mut *(cfd.hEvent as *mut ChangeDir);
    if err_code != 0 {
        panic!("{:?}", Error::from_raw_os_error(err_code as i32));
    }

    // buf更新字节数为0，一般为buf溢出了
    if number == 0 {
        println!("{:?}", Error::last_os_error());
        read_dir_change(he).unwrap();
        return;
    }

    let mut lpdata = he.buf.as_ptr() as *const u32;
    loop {
        let action = match unsafe { lpdata.offset(1).read() } {
            1 => "新增",
            2 => "删除",
            3 => "变更",
            4 => "更名前名称",
            5 => "更名后名称",
            _ => "",
        };
        let length = unsafe { lpdata.offset(2).read() };
        let lptr_u16 = unsafe { lpdata.offset(3) } as *const u16;
        let name_u16 = unsafe { from_raw_parts(lptr_u16, length as usize / 2) };
        let name = String::from_utf16(name_u16).unwrap();

        println!("action: {:?}, name: {:?}", action, name);
        if unsafe { lpdata.read() } == 0 {
            break;
        } else {
            lpdata = unsafe { lpdata.offset(lpdata.read() as isize / 4) };
        }
    }
    // 处理后 再次调用以达到循环。
    read_dir_change(he).unwrap();
}
```



### 参考

[Undaerstanding ReadDirectoryChangesW](https://qualapps.blogspot.com/2010/05/understanding-readdirectorychangesw.html?m=1)

[理解 ReadDirectoryChangesW](https://blog.csdn.net/wzsy/article/details/6697613) 中文 源自上边文章

[notify - crates.io: Rust Package Registry](https://crates.io/crates/notify) 