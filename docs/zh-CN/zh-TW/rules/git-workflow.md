# Git 工作流程

## Commit 讯息格式

```
<type>: <description>

<optional body>
```

类型：feat、fix、refactor、docs、test、chore、perf、ci

注意：归屬透过 ~/.claude/settings.json 全域停用。

## Pull Request 工作流程

建立 PR 时：
1. 分析完整 commit 历史（不只是最新 commit）
2. 使用 `git diff [base-branch]...HEAD` 查看所有变更
3. 起草全面的 PR 摘要
4. 包含帶 TODO 的测试计划
5. 如果是新分支，使用 `-u` flag 推送

## 功能实作工作流程

1. **先规划**
   - 使用 **planner** Agent 建立实作计划
   - 识别相依性和风险
   - 拆解为阶段

2. **TDD 方法**
   - 使用 **tdd-guide** Agent
   - 先撰寫测试（RED）
   - 实作使测试通过（GREEN）
   - 重构（IMPROVE）
   - 验证 80%+ 覆盖率

3. **程序代码审查**
   - 撰寫程序代码后立即使用 **code-reviewer** Agent
   - 处理关键和高优先问题
   - 盡可能修复中优先问题

4. **Commit 与推送**
   - 详细的 commit 讯息
   - 遵循 conventional commits 格式
