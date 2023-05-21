---
marp: true
theme: academic
paginate: false
footer: OS Lab Report · Week 13
---

<!-- header: 第 13 周工作 -->
<!-- _footer: '' -->

<small>

1. 完成日志的 commit（即日志写回到磁盘）
   - 数据块（调用 `Journal::dirty_data`）：直接写回原位置（对应 ext3 的 ordered/writeback 模式）
   - 元数据块（调用 `Journal::dirty_metadata`）：写到日志中
     - 原有缓存保存到 `Shadow` 队列中，用于 checkpoint 时写回原位置
     - 复制一份保存到 `IO` 队列中，用于写入日志

      日志磁盘结构：

      ```
      D: Descriptor    M: Metadata    C: Commit
      | D | M | M | ... | M | D | M | M | C |
      ``` 

      - Descriptor 块：事务开始标记，记录每个 Metadata 块的信息
      - Metadata 块：保存元数据
      - Commit 块：事务结束标志（确保之前块写入磁盘后才写入）

</small>

---

<!-- _footer: '' -->

1. 对文件系统调用日志模块的过程进行测试（可以正常运行）
   ```rust
   // Initialize journal structure
   let journal = Journal::init_dev(...);

   // Write a random block.
   let buf = get_cache(...);
   buf.mark_dirty();

   // Create a handle for submitting buffers
   let mut handle = journal.start();
   // Record the buffer in the journal
   handle.get_write_access(&meta_buf);
   handle.dirty_metadata(&meta_buf);
   handle.stop();

   journal.commit_transaction();
   ```

---

<!-- _footer: '' -->

<small>

问题：
- 可能无法实现一些功能
  - 异步写磁盘（需要操作系统支持）
  - 多线程支持（时间可能不够）
- 该如何写测试验证正确性？
  - 比较难针对某一个接口进行测试（完整进行一次日志记录需要调用多个接口）
  - 可能需要额外提供测试用接口（如：模拟写日志过程中的 crash）

剩余部分（大概下周就能基本完成）

1. Checkpoint（将日志中的内容写回到文件系统）
2. Recover（从崩溃中恢复）


---

<!-- _header: 下周计划 -->

1. 完成 checkpoint、recover 功能
2. 尝试将日志模块接入 easyfs
3. 与另一位同学交流将日志模块接入 ext2
