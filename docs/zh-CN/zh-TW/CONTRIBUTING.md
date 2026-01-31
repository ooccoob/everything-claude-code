# 贡献 Everything Claude Code

感谢您想要贡献。本存储库旨在成为 Claude Code 使用者的社群资源。

## 我们正在尋找什麼

### 代理程序（Agents）

能夠妥善处理特定任务的新代理程序：
- 特定语言审查员（Python、Go、Rust）
- 框架专家（Django、Rails、Laravel、Spring）
- DevOps 专家（Kubernetes、Terraform、CI/CD）
- 领域专家（ML 管线、资料工程、行动开发）

### 技能（Skills）

工作流程定义和领域知识：
- 语言最佳实务
- 框架模式
- 测试策略
- 架构指南
- 特定领域知识

### 指令（Commands）

调用实用工作流程的斜线指令：
- 部署指令
- 测试指令
- 文件指令
- 程序代码生成指令

### 钩子（Hooks）

实用的自动化：
- Lint/格式化钩子
- 安全检查
- 验证钩子
- 通知钩子

### 规则（Rules）

必须遵守的准则：
- 安全规则
- 程序代码风格规则
- 测试需求
- 命名慣例

### MCP 设置

新的或改进的 MCP 伺服器设置：
- 资料库整合
- 云端供应商 MCP
- 监控工具
- 通讯工具

---

## 如何贡献

### 1. Fork 存储库

```bash
git clone https://github.com/YOUR_USERNAME/everything-claude-code.git
cd everything-claude-code
```

### 2. 建立分支

```bash
git checkout -b add-python-reviewer
```

### 3. 新增您的贡献

将档案放置在适当的目录：
- `agents/` 用於新代理程序
- `skills/` 用於技能（可以是单一 .md 或目录）
- `commands/` 用於斜线指令
- `rules/` 用於规则档案
- `hooks/` 用於钩子设置
- `mcp-configs/` 用於 MCP 伺服器设置

### 4. 遵循格式

**代理程序**应包含 frontmatter：

```markdown
---
name: agent-name
description: What it does
tools: Read, Grep, Glob, Bash
model: sonnet
---

Instructions here...
```

**技能**应清晰且可操作：

```markdown
# Skill Name

## When to Use

...

## How It Works

...

## Examples

...
```

**指令**应说明其功能：

```markdown
---
description: Brief description of command
---

# Command Name

Detailed instructions...
```

**钩子**应包含描述：

```json
{
  "matcher": "...",
  "hooks": [...],
  "description": "What this hook does"
}
```

### 5. 测试您的贡献

在提交前确保您的设置能与 Claude Code 正常运作。

### 6. 提交 PR

```bash
git add .
git commit -m "Add Python code reviewer agent"
git push origin add-python-reviewer
```

然后开启一个 PR，包含：
- 您新增了什麼
- 为什麼它有用
- 您如何测试它

---

## 指南

### 建议做法

- 保持设置专注且模块化
- 包含清晰的描述
- 提交前先测试
- 遵循现有模式
- 记录任何相依性

### 避免做法

- 包含敏感资料（API 金钥、权杖、路徑）
- 新增过於复杂或小眾的设置
- 提交未测试的设置
- 建立重复的功能
- 新增需要特定付费服务但无替代方案的设置

---

## 档案命名

- 使用小寫加连字号：`python-reviewer.md`
- 具描述性：`tdd-workflow.md` 而非 `workflow.md`
- 将代理程序/技能名称与档名对应

---

## 有问题？

开启 issue 或在 X 上联系：[@affaanmustafa](https://x.com/affaanmustafa)

---

感谢您的贡献。让我们一起打造优质的资源。
