---
marp: true
theme: academic
paginate: false
footer: OS Lab Report · Week 15
---

<!-- header: 第 15 周工作 -->

1. 完成日志 Recovery 功能
2. 通过测试初步验证日志实现的正确性
3. 为 EasyFS 添加日志功能（in progress...）
4. （重新）将 EasyFS 作为用户程序接入 ArceOS

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

<!-- _footer: '' -->

**3. 为 EasyFS 添加日志功能（in progress...）**

**主要问题**：JBD 只考虑单线程情形，与 Rust 实现的多线程文件系统（如目前的 Ext2）很难兼容

<small>

如：文件系统的 cache manager 返回 `Arc<Mutex<...>>`：

```rust
pub fn get_block_cache(&mut self, block_id: usize, block_device: Arc<dyn BlockDevice>,) -> Arc<Mutex<BlockCache>>
```

而 JBD 接口参数为 `Rc<...>`：

```rust
pub fn get_write_access(&self, buf: &Rc<dyn Buffer>) -> JBDResult
```

（C 语言此问题较容易解决）

</small>

<u>目前：退而求其次，将 EasyFS 变成只支持单线程</u>

---

```rust
#[cfg(feature = "journal")]
let mut journal = jbd::Journal::init_dev(
    provider,
    block_device.clone(),
    block_device.clone(),
    total_blocks,
    JOURNAL_SIZE,
)
.unwrap();

#[cfg(feature = "journal")]
journal.create().unwrap();

let mut efs = Self {
    block_device: Rc::clone(&block_device),
    inode_bitmap,
    data_bitmap,
    inode_area_start_block: 1 + inode_bitmap_blocks,
    data_area_start_block: 1 + inode_total_blocks + data_bitmap_blocks,
    journal_start_block: total_blocks,
    journal_size: JOURNAL_SIZE,
    #[cfg(feature = "journal")]
    journal: Rc::new(RefCell::new(journal)),
};
```

---

```rust
pub fn increase_size(
    &mut self,
    new_size: u32,
    new_blocks: Vec<u32>,
    block_device: &Rc<dyn BlockDevice>,
    #[cfg(feature = "journal")] disk_inode_block_id: u32,
    #[cfg(feature = "journal")] handle: &mut jbd::Handle,
) {
    ...
    #[cfg(feature = "journal")]
    let buf = get_buffer_dyn(block_device, self.indirect1 as usize).unwrap();
    #[cfg(feature = "journal")]
    handle.get_write_access(&buf).unwrap();

    // fill indirect1
    get_block_cache(self.indirect1 as usize, Rc::clone(block_device)).modify(...);

    #[cfg(feature = "journal")]
    handle.dirty_metadata(&buf).unwrap();
    ...
}
```

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

<!-- _header: 目前成果 & 下周计划 -->

**目前成果**

1. 参考 Linux 完成单线程 JBD 层，为文件系统提供日志支持
2. 在 JBD 内部进行若干功能测试

**下周计划**

1. 完成带有日志功能的 EasyFS 并进行测试（正确性、性能）
2. 将单线程版本 JBD 以及对应的 EasyFS 用户程序提交到上游
