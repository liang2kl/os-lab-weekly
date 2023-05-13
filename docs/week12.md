---
marp: true
theme: academic
paginate: false
footer: OS Lab Report · Week 12
---

<!-- _header: 第 12 周工作 -->

1. 实现/重构 JBD 事务管理（缓存管理）部分的接口（本周+上周）
   - 包括 `journal_start`、`journal_get_write_access`、`journal_get_create_access`、`journal_get_undo_access`、`journal_dirty_data`、`journal_dirty_metadata` 等
   - 基本可以支持 ext3 的开发（虽然暂时还跑不了，只是完成了了文件系统需要的接口）
2. 基本实现日志的 commit（即日志写回到磁盘）
3. 重构用户态测试

---

<!-- _header: 下周工作 -->

1. 与另一位同学交流接入 JBD 的方法
2. 继续完成日志的 commit
3. 完成 checkpoint 功能
