---
marp: true
theme: academic
paginate: false
footer: OS Lab Report · Week 11
---

<!-- _header: 第 11 周工作 -->
<!-- _footer: '' -->

基本实现 JBD 事务管理（缓存管理）部分的接口，包括 Linux `include/linux/jbd.h` 中的

<small>

```c
extern handle_t *journal_start(journal_t *, int nblocks);
extern int	 journal_restart (handle_t *, int nblocks);
extern int	 journal_extend (handle_t *, int nblocks);
extern int	 journal_get_write_access(handle_t *, struct buffer_head *);
extern int	 journal_get_create_access (handle_t *, struct buffer_head *);
extern int	 journal_get_undo_access(handle_t *, struct buffer_head *);
extern int	 journal_dirty_data (handle_t *, struct buffer_head *);
extern int	 journal_dirty_metadata (handle_t *, struct buffer_head *);
extern void	 journal_release_buffer (handle_t *, struct buffer_head *);
extern int	 journal_forget (handle_t *, struct buffer_head *);
extern void	 journal_sync_buffer (struct buffer_head *);
extern int	 journal_try_to_free_buffers(journal_t *, struct page *, gfp_t);
extern int	 journal_stop(handle_t *);
extern int	 journal_flush (journal_t *);
extern void	 journal_lock_updates (journal_t *);
extern void	 journal_unlock_updates (journal_t *);
```

</small>

---

<!-- _header: 下周工作 -->

- 实现日志的 commit 流程（`fs/jbd/commit.c`）
- 实现日志的 checkpoint 功能（`fs/jbd/checkpoint.c`）
- 实现日志的恢复（`fs/jbd/recovery.c`）
- 尝试对事务管理进行详细的用户态测试