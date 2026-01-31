# Eval 指令

管理评估驱动开发工作流程。

## 使用方式

`/eval [define|check|report|list] [feature-name]`

## 定义 Evals

`/eval define feature-name`

建立新的 eval 定义：

1. 使用范本建立 `.claude/evals/feature-name.md`：

```markdown
## EVAL: feature-name
建立日期：$(date)

### 能力 Evals
- [ ] [能力 1 的描述]
- [ ] [能力 2 的描述]

### 回归 Evals
- [ ] [现有行为 1 仍然有效]
- [ ] [现有行为 2 仍然有效]

### 成功标准
- 能力 evals 的 pass@3 > 90%
- 回归 evals 的 pass^3 = 100%
```

2. 提示使用者填入具体标准

## 检查 Evals

`/eval check feature-name`

执行功能的 evals：

1. 从 `.claude/evals/feature-name.md` 读取 eval 定义
2. 对每个能力 eval：
   - 嘗试验证标准
   - 记录通过/失敗
   - 记录嘗试到 `.claude/evals/feature-name.log`
3. 对每个回归 eval：
   - 执行相关测试
   - 与基准比较
   - 记录通过/失敗
4. 報告目前状态：

```
EVAL 检查：feature-name
========================
能力：X/Y 通过
回归：X/Y 通过
状态：进行中 / 就绪
```

## 報告 Evals

`/eval report feature-name`

产生全面的 eval 報告：

```
EVAL 報告：feature-name
=========================
产生日期：$(date)

能力 EVALS
----------------
[eval-1]：通过（pass@1）
[eval-2]：通过（pass@2）- 需要重试
[eval-3]：失敗 - 參见备注

回归 EVALS
----------------
[test-1]：通过
[test-2]：通过
[test-3]：通过

指标
-------
能力 pass@1：67%
能力 pass@3：100%
回归 pass^3：100%

备注
-----
[任何问题、边界情况或观察]

建议
--------------
[发布 / 需要改进 / 阻擋]
```

## 列出 Evals

`/eval list`

显示所有 eval 定义：

```
EVAL 定义
================
feature-auth      [3/5 通过] 进行中
feature-search    [5/5 通过] 就绪
feature-export    [0/4 通过] 未开始
```

## 參数

$ARGUMENTS:
- `define <name>` - 建立新的 eval 定义
- `check <name>` - 执行并检查 evals
- `report <name>` - 产生完整報告
- `list` - 显示所有 evals
- `clean` - 移除旧的 eval 日志（保留最后 10 次执行）
