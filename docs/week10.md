---
marp: true
theme: academic
paginate: false
---

<!-- footer: OS Lab Report · Week 10 -->
<!-- _header: 第 10 周工作 -->

1. 实现 JBD 内部管理 buffer（Linux 中的 `journal_head`）
2. 实现 `journal_get_create_access` 和 `journal_get_write_access`（添加对应缓存到 JBD）
3. 增加用户态测试（通过实现系统抽象层）

[Git diff](https://github.com/liang2kl/jbd-rs/compare/904f08117a188ed8a141419e07d1e39168259736...2ed80ccc102a06ee249f03b7e2a82e6eee556588) (642++/149--)

---

<!-- _header: 下周工作 -->

1. 实现其余的缓存管理部分
2. 补充更多测试
