# 更新文件

从单一真相来源同步文件：

1. 读取 package.json scripts 区段
   - 产生 scripts 參考表
   - 包含注释中的描述

2. 读取 .env.example
   - 撷取所有环境变量
   - 记录用途和格式

3. 产生 docs/CONTRIB.md，包含：
   - 开发工作流程
   - 可用的 scripts
   - 环境设置
   - 测试程序

4. 产生 docs/RUNBOOK.md，包含：
   - 部署程序
   - 监控和警報
   - 常见问题和修复
   - 回滚程序

5. 识别过时的文件：
   - 找出 90 天以上未修改的文件
   - 列出供手动审查

6. 显示差异摘要

单一真相来源：package.json 和 .env.example
