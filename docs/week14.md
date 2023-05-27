---
marp: true
theme: academic
paginate: false
footer: OS Lab Report · Week 14
---

<!-- header: 第 14 周工作 -->

<small>

1. 完成日志的 checkpoint 功能（将日志中的内容写回到文件系统）
2. 完成日志的 revoke 功能
   - 对应情形：
     - 元数据可能被存在数据块中，如文件夹 / indirect block（**写日志**）
     - 删除这些数据后数据块可被复用
     - 如果被新文件的数据（**不写日志**）覆盖，会出现问题：
     - 在 redo 时，旧的元数据覆盖了新的数据，新的数据永远无法恢复
   - 解决方法：
     - 删除储存在数据块中的元数据时，在日志中增加一个 revoke 块
     - 避免对 revoke 块进行 redo
3. 开始接入 ext2

</small>

---

<!-- _header: 下周计划 -->

1. 完成 recover 功能
2. 完成测试并尝试合并入上游
3. 合作将日志模块接入 ext2
