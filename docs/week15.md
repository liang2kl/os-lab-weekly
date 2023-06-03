---
marp: true
theme: academic
paginate: false
footer: OS Lab Report · Week 15
---

<!-- header: 第 15 周工作 -->

1. 完成日志 Recovery 功能
2. 通过测试初步验证日志实现的正确性
3. （重新）将 EasyFS 作为用户程序接入 ArceOS
4. 为 EasyFS 添加日志功能（in progress...）

---

**1. 完成日志 Recovery 功能**

Recover 过程：

1. 第一遍（`Scan`）：确定日志起止位置
2. 第二遍（`Revoke`）：根据日志中的 Revoke 记录重建 Revoke 表（用于避免数据在 replay 时被覆盖）
3. 第三遍（`Replay`）：重做日志，即将日志内容写回对应位置

（日志模块完成）

---

<!-- _footer: '' -->

**2. 通过测试初步验证日志实现的正确性**

<small>

```rust
#[test]
fn test_recovery() {
    let journal = new_journal().unwrap();
    journal.create();

    // Do several filesystem transactions
    do_one_transaction(...);
    do_one_transaction(...);
    ...

    // Recreate the journal without checkpointing the old one. (Crash!)
    let journal = new_journal().unwrap();
    // Load & recover
    journal.load();

    // The data should have been written to the disk now.
    ...
}
```

</small>

---

**3. 为 EasyFS 添加日志功能（in progress...）**

**主要问题**：JBD 只考虑单线程情形，在 Rust 中与多线程实现的文件系统（如目前的 Ext2）很难兼容

<small>

如：文件系统的 Cache manager 返回 `Arc<Mutex<Buffer>>`，而 JBD 接口参数为 `Rc<RefCell<Buffer>>`
（C 语言此问题较容易解决）

</small>

<u>目前：退而求其次，将 EasyFS 变成只支持单线程</u>

---

**4.（重新）将 EasyFS 作为用户程序接入 ArceOS**

<div class="twocols">

使用 ArceOS 提供的自定义文件系统接口（之前版本需要修改 `axfs`）

```rust
#[crate_interface::impl_interface]
impl MyFileSystemIf for MyFileSystemIfImpl {
    fn new_myfs(disk: Disk) -> Arc<dyn VfsOps> {
        Arc::new(EasyFileSystemWrapper::new(...))
    }
}
```

<p class="break"></p>

```
apps/fs/logshell
├── Cargo.toml
├── easy_fs
│   ├── Cargo.toml
│   └── src
│       ├── bitmap.rs
│       ├── block_cache.rs
│       ├── block_dev.rs
│       ├── config.rs
│       ├── efs.rs
│       ├── journal.rs
│       ├── layout.rs
│       ├── lib.rs
│       └── vfs.rs
└── src
    ├── cmd.rs
    ├── fat32.img
    └── main.rs
```

</div>



---

<!-- _header: 下周计划 -->

1. 完成带有日志功能的 EasyFS 并进行测试
2. 将单线程版本 JBD 以及对应的 EasyFS 用户程序提交到上游
