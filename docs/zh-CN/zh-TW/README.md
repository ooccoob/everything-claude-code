# Everything Claude Code

[![Stars](https://img.shields.io/github/stars/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/stargazers)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Shell](https://img.shields.io/badge/-Shell-4EAA25?logo=gnu-bash&logoColor=white)
![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?logo=typescript&logoColor=white)
![Go](https://img.shields.io/badge/-Go-00ADD8?logo=go&logoColor=white)
![Markdown](https://img.shields.io/badge/-Markdown-000000?logo=markdown&logoColor=white)

**来自 Anthropic 黑客松冠军的完整 Claude Code 设置集合。**

经过 10 个月以上密集日常使用、打造真实产品所淬炼出的生产就绪代理程序、技能、钩子、指令、规则和 MCP 设置。

---

## 指南

本存储库仅包含原始程序代码。指南会解释所有内容。

<table>
<tr>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2012378465664745795">
<img src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" alt="Everything Claude Code 简明指南" />
</a>
</td>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2014040193557471352">
<img src="https://github.com/user-attachments/assets/c9ca43bc-b149-427f-b551-af6840c368f0" alt="Everything Claude Code 完整指南" />
</a>
</td>
</tr>
<tr>
<td align="center"><b>简明指南</b><br/>设置、基础、理念。<b>请先阅读此指南。</b></td>
<td align="center"><b>完整指南</b><br/>权杖最佳化、记忆持久化、评估、平行处理。</td>
</tr>
</table>

| 主题 | 学习内容 |
|------|----------|
| 权杖最佳化 | 模型选择、系统提示精简、背景程序 |
| 记忆持久化 | 自动跨工作阶段保存/载入上下文的钩子 |
| 持续学习 | 从工作阶段自动撷取模式并转化为可重用技能 |
| 验证回圈 | 检查点 vs 持续评估、评分器类型、pass@k 指标 |
| 平行处理 | Git worktrees、串联方法、何时扩展实例 |
| 子代理程序协调 | 上下文问题、渐进式检索模式 |

---

## 跨平台支援

此外挂程序现已完整支援 **Windows、macOS 和 Linux**。所有钩子和脚本已使用 Node.js 重寫以获得最佳相容性。

### 套件管理器侦测

外挂程序会自动侦测您偏好的套件管理器（npm、pnpm、yarn 或 bun），优先顺序如下：

1. **环境变量**：`CLAUDE_PACKAGE_MANAGER`
2. **专案设置**：`.claude/package-manager.json`
3. **package.json**：`packageManager` 字段
4. **锁定档案**：从 package-lock.json、yarn.lock、pnpm-lock.yaml 或 bun.lockb 侦测
5. **全域设置**：`~/.claude/package-manager.json`
6. **备援方案**：第一个可用的套件管理器

设置您偏好的套件管理器：

```bash
# 透过环境变量
export CLAUDE_PACKAGE_MANAGER=pnpm

# 透过全域设置
node scripts/setup-package-manager.js --global pnpm

# 透过专案设置
node scripts/setup-package-manager.js --project bun

# 侦测目前设置
node scripts/setup-package-manager.js --detect
```

或在 Claude Code 中使用 `/setup-pm` 指令。

---

## 内容概览

本存储库是一个 **Claude Code 外挂程序** - 可直接安装或手动复制元件。

```
everything-claude-code/
|-- .claude-plugin/   # 外挂程序和市集清单
|   |-- plugin.json         # 外挂程序中继资料和元件路徑
|   |-- marketplace.json    # 用於 /plugin marketplace add 的市集目录
|
|-- agents/           # 用於委派任务的专门子代理程序
|   |-- planner.md           # 功能实作规划
|   |-- architect.md         # 系统设计决策
|   |-- tdd-guide.md         # 测试驱动开发
|   |-- code-reviewer.md     # 品质与安全审查
|   |-- security-reviewer.md # 弱点分析
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright E2E 测试
|   |-- refactor-cleaner.md  # 无用程序代码清理
|   |-- doc-updater.md       # 文件同步
|   |-- go-reviewer.md       # Go 程序代码审查（新增）
|   |-- go-build-resolver.md # Go 构建错误解决（新增）
|
|-- skills/           # 工作流程定义和领域知识
|   |-- coding-standards/           # 程序语言最佳实务
|   |-- backend-patterns/           # API、资料库、快取模式
|   |-- frontend-patterns/          # React、Next.js 模式
|   |-- continuous-learning/        # 从工作阶段自动撷取模式（完整指南）
|   |-- continuous-learning-v2/     # 基於本能的学习与信心评分
|   |-- iterative-retrieval/        # 子代理程序的渐进式上下文精炼
|   |-- strategic-compact/          # 手动压缩建议（完整指南）
|   |-- tdd-workflow/               # TDD 方法论
|   |-- security-review/            # 安全性检查清单
|   |-- eval-harness/               # 验证回圈评估（完整指南）
|   |-- verification-loop/          # 持续验证（完整指南）
|   |-- golang-patterns/            # Go 惯用语法和最佳实务（新增）
|   |-- golang-testing/             # Go 测试模式、TDD、基准测试（新增）
|
|-- commands/         # 快速执行的斜线指令
|   |-- tdd.md              # /tdd - 测试驱动开发
|   |-- plan.md             # /plan - 实作规划
|   |-- e2e.md              # /e2e - E2E 测试生成
|   |-- code-review.md      # /code-review - 品质审查
|   |-- build-fix.md        # /build-fix - 修复构建错误
|   |-- refactor-clean.md   # /refactor-clean - 移除无用程序代码
|   |-- learn.md            # /learn - 工作阶段中撷取模式（完整指南）
|   |-- checkpoint.md       # /checkpoint - 保存验证状态（完整指南）
|   |-- verify.md           # /verify - 执行验证回圈（完整指南）
|   |-- setup-pm.md         # /setup-pm - 设置套件管理器
|   |-- go-review.md        # /go-review - Go 程序代码审查（新增）
|   |-- go-test.md          # /go-test - Go TDD 工作流程（新增）
|   |-- go-build.md         # /go-build - 修复 Go 构建错误（新增）
|
|-- rules/            # 必须遵守的准则（复制到 ~/.claude/rules/）
|   |-- security.md         # 強制性安全检查
|   |-- coding-style.md     # 不可变性、档案组织
|   |-- testing.md          # TDD、80% 覆盖率要求
|   |-- git-workflow.md     # 提交格式、PR 流程
|   |-- agents.md           # 何时委派给子代理程序
|   |-- performance.md      # 模型选择、上下文管理
|
|-- hooks/            # 基於触发器的自动化
|   |-- hooks.json                # 所有钩子设置（PreToolUse、PostToolUse、Stop 等）
|   |-- memory-persistence/       # 工作阶段生命周期钩子（完整指南）
|   |-- strategic-compact/        # 压缩建议（完整指南）
|
|-- scripts/          # 跨平台 Node.js 脚本（新增）
|   |-- lib/                     # 共用工具
|   |   |-- utils.js             # 跨平台档案/路徑/系统工具
|   |   |-- package-manager.js   # 套件管理器侦测与选择
|   |-- hooks/                   # 钩子实作
|   |   |-- session-start.js     # 工作阶段开始时载入上下文
|   |   |-- session-end.js       # 工作阶段结束时保存状态
|   |   |-- pre-compact.js       # 压缩前状态保存
|   |   |-- suggest-compact.js   # 策略性压缩建议
|   |   |-- evaluate-session.js  # 从工作阶段撷取模式
|   |-- setup-package-manager.js # 互动式套件管理器设置
|
|-- tests/            # 测试套件（新增）
|   |-- lib/                     # 函数库测试
|   |-- hooks/                   # 钩子测试
|   |-- run-all.js               # 执行所有测试
|
|-- contexts/         # 动态系统提示注入上下文（完整指南）
|   |-- dev.md              # 开发模式上下文
|   |-- review.md           # 程序代码审查模式上下文
|   |-- research.md         # 研究/探索模式上下文
|
|-- examples/         # 范例设置和工作阶段
|   |-- CLAUDE.md           # 专案层级设置范例
|   |-- user-CLAUDE.md      # 使用者层级设置范例
|
|-- mcp-configs/      # MCP 伺服器设置
|   |-- mcp-servers.json    # GitHub、Supabase、Vercel、Railway 等
|
|-- marketplace.json  # 自托管市集设置（用於 /plugin marketplace add）
```

---

## 生态系统工具

### ecc.tools - 技能建立器

从您的存储库自动生成 Claude Code 技能。

[安装 GitHub App](https://github.com/apps/skill-creator) | [ecc.tools](https://ecc.tools)

分析您的存储库并建立：
- **SKILL.md 档案** - 可直接用於 Claude Code 的技能
- **本能集合** - 用於 continuous-learning-v2
- **模式撷取** - 从您的提交历史学习

```bash
# 安装 GitHub App 后，技能会出现在：
~/.claude/skills/generated/
```

与 `continuous-learning-v2` 技能无缝整合以继承本能。

---

## 安装

### 选项 1：以外挂程序安装（建议）

使用本存储库最简单的方式 - 安装为 Claude Code 外挂程序：

```bash
# 将此存储库新增为市集
/plugin marketplace add affaan-m/everything-claude-code

# 安装外挂程序
/plugin install everything-claude-code@everything-claude-code
```

或直接新增到您的 `~/.claude/settings.json`：

```json
{
  "extraKnownMarketplaces": {
    "everything-claude-code": {
      "source": {
        "source": "github",
        "repo": "affaan-m/everything-claude-code"
      }
    }
  },
  "enabledPlugins": {
    "everything-claude-code@everything-claude-code": true
  }
}
```

这会让您立即存取所有指令、代理程序、技能和钩子。

---

### 选项 2：手动安装

如果您偏好手动控制安装内容：

```bash
# 复制存储库
git clone https://github.com/affaan-m/everything-claude-code.git

# 将代理程序复制到您的 Claude 设置
cp everything-claude-code/agents/*.md ~/.claude/agents/

# 复制规则
cp everything-claude-code/rules/*.md ~/.claude/rules/

# 复制指令
cp everything-claude-code/commands/*.md ~/.claude/commands/

# 复制技能
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

#### 将钩子新增到 settings.json

将 `hooks/hooks.json` 中的钩子复制到您的 `~/.claude/settings.json`。

#### 设置 MCP

将 `mcp-configs/mcp-servers.json` 中所需的 MCP 伺服器复制到您的 `~/.claude.json`。

**重要：** 将 `YOUR_*_HERE` 佔位符替換为您实际的 API 金钥。

---

## 核心概念

### 代理程序（Agents）

子代理程序以有限范围处理委派的任务。范例：

```markdown
---
name: code-reviewer
description: Reviews code for quality, security, and maintainability
tools: ["Read", "Grep", "Glob", "Bash"]
model: opus
---

You are a senior code reviewer...
```

### 技能（Skills）

技能是由指令或代理程序调用的工作流程定义：

```markdown
# TDD Workflow

1. Define interfaces first
2. Write failing tests (RED)
3. Implement minimal code (GREEN)
4. Refactor (IMPROVE)
5. Verify 80%+ coverage
```

### 钩子（Hooks）

钩子在工具事件时触发。范例 - 警告 console.log：

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[Hook] Remove console.log' >&2"
  }]
}
```

### 规则（Rules）

规则是必须遵守的准则。保持模块化：

```
~/.claude/rules/
  security.md      # 禁止寫死密钥
  coding-style.md  # 不可变性、档案限制
  testing.md       # TDD、覆盖率要求
```

---

## 执行测试

外挂程序包含完整的测试套件：

```bash
# 执行所有测试
node tests/run-all.js

# 执行个別测试档案
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

---

## 贡献

**欢迎并鼓勵贡献。**

本存储库旨在成为社群资源。如果您有：
- 实用的代理程序或技能
- 巧妙的钩子
- 更好的 MCP 设置
- 改进的规则

请贡献！详见 [CONTRIBUTING.md](CONTRIBUTING.md) 的指南。

### 贡献想法

- 特定语言的技能（Python、Rust 模式）- Go 现已包含！
- 特定框架的设置（Django、Rails、Laravel）
- DevOps 代理程序（Kubernetes、Terraform、AWS）
- 测试策略（不同框架）
- 特定领域知识（ML、资料工程、行动开发）

---

## 背景

我从实验性推出就开始使用 Claude Code。2025 年 9 月与 [@DRodriguezFX](https://x.com/DRodriguezFX) 一起使用 Claude Code 打造 [zenith.chat](https://zenith.chat)，赢得了 Anthropic x Forum Ventures 黑客松。

这些设置已在多个生产应用程序中经过实战测试。

---

## 重要注意事项

### 上下文窗口管理

**关键：** 不要同时启用所有 MCP。启用过多工具会让您的 200k 上下文窗口缩减至 70k。

经验法则：
- 设置 20-30 个 MCP
- 每个专案启用少於 10 个
- 启用的工具少於 80 个

在专案设置中使用 `disabledMcpServers` 来停用未使用的 MCP。

### 自定义

这些设置适合我的工作流程。您应该：
1. 从您认同的部分开始
2. 根据您的技术堆叠修改
3. 移除不需要的部分
4. 添加您自己的模式

---

## Star 历史

[![Star History Chart](https://api.star-history.com/svg?repos=affaan-m/everything-claude-code&type=Date)](https://star-history.com/#affaan-m/everything-claude-code&Date)

---

## 连结

- **简明指南（从这里开始）：** [Everything Claude Code 简明指南](https://x.com/affaanmustafa/status/2012378465664745795)
- **完整指南（进阶）：** [Everything Claude Code 完整指南](https://x.com/affaanmustafa/status/2014040193557471352)
- **追踪：** [@affaanmustafa](https://x.com/affaanmustafa)
- **zenith.chat：** [zenith.chat](https://zenith.chat)

---

## 授权

MIT - 自由使用、依需求修改、如可能请回馈贡献。

---

**如果有幫助请为本存储库加星。阅读兩份指南。打造偉大的作品。**
