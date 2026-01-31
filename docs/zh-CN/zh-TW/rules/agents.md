# Agent 协调

## 可用 Agents

位於 `~/.claude/agents/`：

| Agent | 用途 | 何时使用 |
|-------|------|----------|
| planner | 实作规划 | 复杂功能、重构 |
| architect | 系统设计 | 架构决策 |
| tdd-guide | 测试驱动开发 | 新功能、Bug 修复 |
| code-reviewer | 程序代码审查 | 撰寫程序代码后 |
| security-reviewer | 安全性分析 | 提交前 |
| build-error-resolver | 修复构建错误 | 构建失敗时 |
| e2e-runner | E2E 测试 | 关键使用者流程 |
| refactor-cleaner | 无用程序代码清理 | 程序代码维护 |
| doc-updater | 文件 | 更新文件 |

## 立即使用 Agent

不需要使用者提示：
1. 复杂功能请求 - 使用 **planner** Agent
2. 剛撰寫/修改程序代码 - 使用 **code-reviewer** Agent
3. Bug 修复或新功能 - 使用 **tdd-guide** Agent
4. 架构决策 - 使用 **architect** Agent

## 平行任务执行

对独立操作总是使用平行 Task 执行：

```markdown
# 好：平行执行
平行启动 3 个 agents：
1. Agent 1：auth.ts 的安全性分析
2. Agent 2：快取系统的效能审查
3. Agent 3：utils.ts 的型別检查

# 不好：不必要的循序
先 agent 1，然后 agent 2，然后 agent 3
```

## 多观点分析

对於复杂问题，使用分角色子 agents：
- 事实审查者
- 资深工程师
- 安全专家
- 一致性审查者
- 冗余检查者
