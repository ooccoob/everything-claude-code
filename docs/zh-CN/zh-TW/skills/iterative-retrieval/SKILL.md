---
name: iterative-retrieval
description: Pattern for progressively refining context retrieval to solve the subagent context problem
---

# 迭代检索模式

解决多 agent 工作流程中的「上下文问题」，其中子 agents 在开始工作之前不知道需要什麼上下文。

## 问题

子 agents 以有限上下文产生。它们不知道：
- 哪些档案包含相关程序代码
- 程序代码库中存在什麼模式
- 专案使用什麼术语

标准方法失敗：
- **传送所有内容**：超过上下文限制
- **不传送内容**：Agent 缺乏关键资讯
- **猜测需要什麼**：经常错误

## 解决方案：迭代检索

一个渐进精炼上下文的 4 阶段循环：

```
┌─────────────────────────────────────────────┐
│                                             │
│   ┌──────────┐      ┌──────────┐            │
│   │ DISPATCH │─────▶│ EVALUATE │            │
│   └──────────┘      └──────────┘            │
│        ▲                  │                 │
│        │                  ▼                 │
│   ┌──────────┐      ┌──────────┐            │
│   │   LOOP   │◀─────│  REFINE  │            │
│   └──────────┘      └──────────┘            │
│                                             │
│        最多 3 个循环，然后继续               │
└─────────────────────────────────────────────┘
```

### 阶段 1：DISPATCH

初始廣泛查询以收集候选档案：

```javascript
// 从高层意圖开始
const initialQuery = {
  patterns: ['src/**/*.ts', 'lib/**/*.ts'],
  keywords: ['authentication', 'user', 'session'],
  excludes: ['*.test.ts', '*.spec.ts']
};

// 派遣到检索 agent
const candidates = await retrieveFiles(initialQuery);
```

### 阶段 2：EVALUATE

评估检索内容的相关性：

```javascript
function evaluateRelevance(files, task) {
  return files.map(file => ({
    path: file.path,
    relevance: scoreRelevance(file.content, task),
    reason: explainRelevance(file.content, task),
    missingContext: identifyGaps(file.content, task)
  }));
}
```

评分标准：
- **高（0.8-1.0）**：直接实作目标功能
- **中（0.5-0.7）**：包含相关模式或类型
- **低（0.2-0.4）**：间接相关
- **无（0-0.2）**：不相关，排除

### 阶段 3：REFINE

基於评估更新搜索标准：

```javascript
function refineQuery(evaluation, previousQuery) {
  return {
    // 新增在高相关性档案中发现的新模式
    patterns: [...previousQuery.patterns, ...extractPatterns(evaluation)],

    // 新增在程序代码库中找到的术语
    keywords: [...previousQuery.keywords, ...extractKeywords(evaluation)],

    // 排除确认不相关的路徑
    excludes: [...previousQuery.excludes, ...evaluation
      .filter(e => e.relevance < 0.2)
      .map(e => e.path)
    ],

    // 针对特定缺口
    focusAreas: evaluation
      .flatMap(e => e.missingContext)
      .filter(unique)
  };
}
```

### 阶段 4：LOOP

以精炼标准重复（最多 3 个循环）：

```javascript
async function iterativeRetrieve(task, maxCycles = 3) {
  let query = createInitialQuery(task);
  let bestContext = [];

  for (let cycle = 0; cycle < maxCycles; cycle++) {
    const candidates = await retrieveFiles(query);
    const evaluation = evaluateRelevance(candidates, task);

    // 检查是否有足夠上下文
    const highRelevance = evaluation.filter(e => e.relevance >= 0.7);
    if (highRelevance.length >= 3 && !hasCriticalGaps(evaluation)) {
      return highRelevance;
    }

    // 精炼并继续
    query = refineQuery(evaluation, query);
    bestContext = mergeContext(bestContext, highRelevance);
  }

  return bestContext;
}
```

## 实际范例

### 范例 1：Bug 修复上下文

```
任务：「修复认证 token 过期 bug」

循环 1：
  DISPATCH：在 src/** 搜索 "token"、"auth"、"expiry"
  EVALUATE：找到 auth.ts (0.9)、tokens.ts (0.8)、user.ts (0.3)
  REFINE：新增 "refresh"、"jwt" 关键字；排除 user.ts

循环 2：
  DISPATCH：搜索精炼术语
  EVALUATE：找到 session-manager.ts (0.95)、jwt-utils.ts (0.85)
  REFINE：足夠上下文（2 个高相关性档案）

结果：auth.ts、tokens.ts、session-manager.ts、jwt-utils.ts
```

### 范例 2：功能实作

```
任务：「为 API 端点增加速率限制」

循环 1：
  DISPATCH：在 routes/** 搜索 "rate"、"limit"、"api"
  EVALUATE：无匹配 - 程序代码库使用 "throttle" 术语
  REFINE：新增 "throttle"、"middleware" 关键字

循环 2：
  DISPATCH：搜索精炼术语
  EVALUATE：找到 throttle.ts (0.9)、middleware/index.ts (0.7)
  REFINE：需要路由器模式

循环 3：
  DISPATCH：搜索 "router"、"express" 模式
  EVALUATE：找到 router-setup.ts (0.8)
  REFINE：足夠上下文

结果：throttle.ts、middleware/index.ts、router-setup.ts
```

## 与 Agents 整合

在 agent 提示中使用：

```markdown
为此任务检索上下文时：
1. 从廣泛关键字搜索开始
2. 评估每个档案的相关性（0-1 尺度）
3. 识别仍缺少的上下文
4. 精炼搜索标准并重复（最多 3 个循环）
5. 回传相关性 >= 0.7 的档案
```

## 最佳实务

1. **从廣泛开始，逐渐缩小** - 不要过度指定初始查询
2. **学习程序代码库术语** - 第一个循环通常会揭示命名慣例
3. **追踪缺失内容** - 明确的缺口识别驱动精炼
4. **在「足夠好」时停止** - 3 个高相关性档案勝过 10 个普通档案
5. **自信地排除** - 低相关性档案不会变得相关

## 相关

- [Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 子 agent 协调章节
- `continuous-learning` 技能 - 用於随时间改进的模式
- `~/.claude/agents/` 中的 Agent 定义
