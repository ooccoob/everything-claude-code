# Orchestrate 指令

复杂任务的循序 Agent 工作流程。

## 使用方式

`/orchestrate [workflow-type] [task-description]`

## 工作流程类型

### feature
完整的功能实作工作流程：
```
planner -> tdd-guide -> code-reviewer -> security-reviewer
```

### bugfix
Bug 调查和修复工作流程：
```
explorer -> tdd-guide -> code-reviewer
```

### refactor
安全重构工作流程：
```
architect -> code-reviewer -> tdd-guide
```

### security
以安全性为焦点的审查：
```
security-reviewer -> code-reviewer -> architect
```

## 执行模式

对工作流程中的每个 Agent：

1. **呼叫 Agent**，帶入前一个 Agent 的上下文
2. **收集输出**作为结构化交接文件
3. **传递给下一个 Agent**
4. **彙整结果**为最终報告

## 交接文件格式

Agent 之间，建立交接文件：

```markdown
## 交接：[前一个 Agent] -> [下一个 Agent]

### 上下文
[完成事项的摘要]

### 发现
[关键发现或决策]

### 修改的档案
[触及的档案列表]

### 开放问题
[下一个 Agent 的未解决项目]

### 建议
[建议的后续步骤]
```

## 最终報告格式

```
协调報告
====================
工作流程：feature
任务：新增使用者验证
Agents：planner -> tdd-guide -> code-reviewer -> security-reviewer

摘要
-------
[一段摘要]

AGENT 输出
-------------
Planner：[摘要]
TDD Guide：[摘要]
Code Reviewer：[摘要]
Security Reviewer：[摘要]

变更的档案
-------------
[列出所有修改的档案]

测试结果
------------
[测试通过/失敗摘要]

安全性状态
---------------
[安全性发现]

建议
--------------
[发布 / 需要改进 / 阻擋]
```

## 平行执行

对於独立的检查，平行执行 Agents：

```markdown
### 平行阶段
同时执行：
- code-reviewer（品质）
- security-reviewer（安全性）
- architect（设计）

### 合并结果
将输出合并为单一報告
```

## 參数

$ARGUMENTS:
- `feature <description>` - 完整功能工作流程
- `bugfix <description>` - Bug 修复工作流程
- `refactor <description>` - 重构工作流程
- `security <description>` - 安全性审查工作流程
- `custom <agents> <description>` - 自定义 Agent 序列

## 自定义工作流程范例

```
/orchestrate custom "architect,tdd-guide,code-reviewer" "重新设计快取层"
```

## 提示

1. **复杂功能从 planner 开始**
2. **合并前总是包含 code-reviewer**
3. **对验证/支付/PII 使用 security-reviewer**
4. **保持交接简洁** - 专注於下一个 Agent 需要的内容
5. **如有需要，在 Agents 之间执行 verification**
