# 验证循环技能

Claude Code 工作阶段的完整验证系统。

## 何时使用

在以下情况呼叫此技能：
- 完成功能或重大程序代码变更后
- 建立 PR 前
- 想确保品质门槛通过时
- 重构后

## 验证阶段

### 阶段 1：构建验证
```bash
# 检查专案是否构建
npm run build 2>&1 | tail -20
# 或
pnpm build 2>&1 | tail -20
```

如果构建失敗，停止并在继续前修复。

### 阶段 2：型別检查
```bash
# TypeScript 专案
npx tsc --noEmit 2>&1 | head -30

# Python 专案
pyright . 2>&1 | head -30
```

報告所有型別错误。继续前修复关键错误。

### 阶段 3：Lint 检查
```bash
# JavaScript/TypeScript
npm run lint 2>&1 | head -30

# Python
ruff check . 2>&1 | head -30
```

### 阶段 4：测试套件
```bash
# 执行帶覆盖率的测试
npm run test -- --coverage 2>&1 | tail -50

# 检查覆盖率门槛
# 目标：最低 80%
```

報告：
- 总测试数：X
- 通过：X
- 失敗：X
- 覆盖率：X%

### 阶段 5：安全掃描
```bash
# 检查密钥
grep -rn "sk-" --include="*.ts" --include="*.js" . 2>/dev/null | head -10
grep -rn "api_key" --include="*.ts" --include="*.js" . 2>/dev/null | head -10

# 检查 console.log
grep -rn "console.log" --include="*.ts" --include="*.tsx" src/ 2>/dev/null | head -10
```

### 阶段 6：差异审查
```bash
# 显示变更内容
git diff --stat
git diff HEAD~1 --name-only
```

审查每个变更的档案：
- 非预期变更
- 缺少错误处理
- 潜在边界案例

## 输出格式

执行所有阶段后，产生验证報告：

```
验证報告
==================

构建：     [PASS/FAIL]
型別：     [PASS/FAIL]（X 个错误）
Lint：     [PASS/FAIL]（X 个警告）
测试：     [PASS/FAIL]（X/Y 通过，Z% 覆盖率）
安全性：   [PASS/FAIL]（X 个问题）
差异：     [X 个档案变更]

整体：     [READY/NOT READY] for PR

待修复问题：
1. ...
2. ...
```

## 持续模式

对於长时间工作阶段，每 15 分钟或重大变更后执行验证：

```markdown
设置心理检查点：
- 完成每个函数后
- 完成元件后
- 移至下一个任务前

执行：/verify
```

## 与 Hooks 整合

此技能补充 PostToolUse hooks 但提供更深入的验证。
Hooks 立即捕捉问题；此技能提供全面审查。
