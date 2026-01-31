---
name: continuous-learning
description: Automatically extract reusable patterns from Claude Code sessions and save them as learned skills for future use.
---

# 持续学习技能

自动评估 Claude Code 工作阶段结束时的内容，提取可重用模式并保存为学习技能。

## 运作方式

此技能作为 **Stop hook** 在每个工作阶段结束时执行：

1. **工作阶段评估**：检查工作阶段是否有足夠讯息（预设：10+ 则）
2. **模式侦测**：从工作阶段识别可提取的模式
3. **技能提取**：将有用模式保存到 `~/.claude/skills/learned/`

## 设置

编辑 `config.json` 以自定义：

```json
{
  "min_session_length": 10,
  "extraction_threshold": "medium",
  "auto_approve": false,
  "learned_skills_path": "~/.claude/skills/learned/",
  "patterns_to_detect": [
    "error_resolution",
    "user_corrections",
    "workarounds",
    "debugging_techniques",
    "project_specific"
  ],
  "ignore_patterns": [
    "simple_typos",
    "one_time_fixes",
    "external_api_issues"
  ]
}
```

## 模式类型

| 模式 | 描述 |
|------|------|
| `error_resolution` | 特定错误如何被解决 |
| `user_corrections` | 来自使用者修正的模式 |
| `workarounds` | 框架/函数库怪异问题的解决方案 |
| `debugging_techniques` | 有效的调试方法 |
| `project_specific` | 专案特定慣例 |

## Hook 设置

新增到你的 `~/.claude/settings.json`：

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning/evaluate-session.sh"
      }]
    }]
  }
}
```

## 为什麼用 Stop Hook？

- **轻量**：工作阶段结束时只执行一次
- **非阻塞**：不会为每则讯息增加延迟
- **完整上下文**：可存取完整工作阶段记录

## 相关

- [Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 持续学习章节
- `/learn` 指令 - 工作阶段中手动提取模式

---

## 比较笔记（研究：2025 年 1 月）

### vs Homunculus (github.com/humanplane/homunculus)

Homunculus v2 採用更复杂的方法：

| 功能 | 我们的方法 | Homunculus v2 |
|------|----------|---------------|
| 观察 | Stop hook（工作阶段结束） | PreToolUse/PostToolUse hooks（100% 可靠） |
| 分析 | 主要上下文 | 背景 agent（Haiku） |
| 粒度 | 完整技能 | 原子「本能」 |
| 信心 | 无 | 0.3-0.9 加权 |
| 演化 | 直接到技能 | 本能 → 聚类 → 技能/指令/agent |
| 分享 | 无 | 导出/导入本能 |

**来自 homunculus 的关键见解：**
> "v1 依赖技能进行观察。技能是机率性的——它们触发约 50-80% 的时间。v2 使用 hooks 进行观察（100% 可靠），并以本能作为学习行为的原子单位。"

### 潜在 v2 增強

1. **基於本能的学习** - 较小的原子行为，帶信心评分
2. **背景观察者** - Haiku agent 并行分析
3. **信心衰减** - 如果被矛盾则本能失去信心
4. **领域标记** - code-style、testing、git、debugging 等
5. **演化路徑** - 将相关本能聚类为技能/指令

參见：`/Users/affoon/Documents/tasks/12-continuous-learning-v2.md` 完整规格。
