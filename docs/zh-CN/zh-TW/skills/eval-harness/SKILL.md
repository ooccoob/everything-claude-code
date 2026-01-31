---
name: eval-harness
description: Formal evaluation framework for Claude Code sessions implementing eval-driven development (EDD) principles
tools: Read, Write, Edit, Bash, Grep, Glob
---

# Eval Harness 技能

Claude Code 工作阶段的正式评估框架，实作 eval 驱动开发（EDD）原则。

## 理念

Eval 驱动开发将 evals 视为「AI 开发的单元测试」：
- 在实作前定义预期行为
- 开发期间持续执行 evals
- 每次变更追踪回归
- 使用 pass@k 指标进行可靠性测量

## Eval 类型

### 能力 Evals
测试 Claude 是否能做到以前做不到的事：
```markdown
[CAPABILITY EVAL: feature-name]
任务：Claude 应完成什麼的描述
成功标准：
  - [ ] 标准 1
  - [ ] 标准 2
  - [ ] 标准 3
预期输出：预期结果描述
```

### 回归 Evals
确保变更不会破坏现有功能：
```markdown
[REGRESSION EVAL: feature-name]
基准：SHA 或检查点名称
测试：
  - existing-test-1: PASS/FAIL
  - existing-test-2: PASS/FAIL
  - existing-test-3: PASS/FAIL
结果：X/Y 通过（先前为 Y/Y）
```

## 评分器类型

### 1. 基於程序代码的评分器
使用程序代码的确定性检查：
```bash
# 检查档案是否包含预期模式
grep -q "export function handleAuth" src/auth.ts && echo "PASS" || echo "FAIL"

# 检查测试是否通过
npm test -- --testPathPattern="auth" && echo "PASS" || echo "FAIL"

# 检查构建是否成功
npm run build && echo "PASS" || echo "FAIL"
```

### 2. 基於模型的评分器
使用 Claude 评估开放式输出：
```markdown
[MODEL GRADER PROMPT]
评估以下程序代码变更：
1. 它是否解决了陈述的问题？
2. 结构是否良好？
3. 边界案例是否被处理？
4. 错误处理是否适当？

分数：1-5（1=差，5=优秀）
理由：[解释]
```

### 3. 人工评分器
标记为手动审查：
```markdown
[HUMAN REVIEW REQUIRED]
变更：变更内容的描述
理由：为何需要人工审查
风险等级：LOW/MEDIUM/HIGH
```

## 指标

### pass@k
「k 次嘗试中至少一次成功」
- pass@1：第一次嘗试成功率
- pass@3：3 次嘗试内成功
- 典型目标：pass@3 > 90%

### pass^k
「所有 k 次试验都成功」
- 更高的可靠性标准
- pass^3：连续 3 次成功
- 用於关键路徑

## Eval 工作流程

### 1. 定义（编码前）
```markdown
## EVAL 定义：feature-xyz

### 能力 Evals
1. 可以建立新使用者帳戶
2. 可以验证电子邮件格式
3. 可以安全地杂凑密码

### 回归 Evals
1. 现有登入仍可运作
2. 工作阶段管理未变更
3. 登出流程完整

### 成功指标
- 能力 evals 的 pass@3 > 90%
- 回归 evals 的 pass^3 = 100%
```

### 2. 实作
撰寫程序代码以通过定义的 evals。

### 3. 评估
```bash
# 执行能力 evals
[执行每个能力 eval，记录 PASS/FAIL]

# 执行回归 evals
npm test -- --testPathPattern="existing"

# 产生報告
```

### 4. 報告
```markdown
EVAL 報告：feature-xyz
========================

能力 Evals：
  create-user:     PASS (pass@1)
  validate-email:  PASS (pass@2)
  hash-password:   PASS (pass@1)
  整体：           3/3 通过

回归 Evals：
  login-flow:      PASS
  session-mgmt:    PASS
  logout-flow:     PASS
  整体：           3/3 通过

指标：
  pass@1: 67% (2/3)
  pass@3: 100% (3/3)

状态：准备审查
```

## 整合模式

### 实作前
```
/eval define feature-name
```
在 `.claude/evals/feature-name.md` 建立 eval 定义档案

### 实作期间
```
/eval check feature-name
```
执行当前 evals 并報告状态

### 实作后
```
/eval report feature-name
```
产生完整 eval 報告

## Eval 保存

在专案中保存 evals：
```
.claude/
  evals/
    feature-xyz.md      # Eval 定义
    feature-xyz.log     # Eval 执行历史
    baseline.json       # 回归基准
```

## 最佳实务

1. **编码前定义 evals** - 強制清楚思考成功标准
2. **频繁执行 evals** - 及早捕捉回归
3. **随时间追踪 pass@k** - 监控可靠性趋勢
4. **可能时使用程序代码评分器** - 确定性 > 机率性
5. **安全性需人工审查** - 永远不要完全自动化安全检查
6. **保持 evals 快速** - 慢 evals 不会被执行
7. **与程序代码一起版本化 evals** - Evals 是一等工件

## 范例：新增认证

```markdown
## EVAL：add-authentication

### 阶段 1：定义（10 分钟）
能力 Evals：
- [ ] 使用者可以用电子邮件/密码註册
- [ ] 使用者可以用有效憑证登入
- [ ] 无效憑证被拒绝并显示适当错误
- [ ] 工作阶段在页面重新载入后持续
- [ ] 登出清除工作阶段

回归 Evals：
- [ ] 公开路由仍可存取
- [ ] API 回应未变更
- [ ] 资料库 schema 相容

### 阶段 2：实作（视情况而定）
[撰寫程序代码]

### 阶段 3：评估
执行：/eval check add-authentication

### 阶段 4：報告
EVAL 報告：add-authentication
==============================
能力：5/5 通过（pass@3：100%）
回归：3/3 通过（pass^3：100%）
状态：准备发佈
```
