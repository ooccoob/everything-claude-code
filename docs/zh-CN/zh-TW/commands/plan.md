---
description: Restate requirements, assess risks, and create step-by-step implementation plan. WAIT for user CONFIRM before touching any code.
---

# Plan 指令

此指令呼叫 **planner** Agent，在撰寫任何程序代码前建立全面的实作计划。

## 此指令的功能

1. **重述需求** - 厘清需要建构什麼
2. **识别风险** - 揭示潜在问题和阻碍
3. **建立步骤计划** - 将实作拆解为阶段
4. **等待确认** - 必须在继续前收到使用者批准

## 何时使用

在以下情况使用 `/plan`：
- 开始新功能
- 进行重大架构变更
- 进行复杂重构
- 多个档案/元件会受影响
- 需求不清楚或模糊

## 运作方式

planner Agent 会：

1. **分析请求**并以清晰的语言重述需求
2. **拆解为阶段**，包含具体、可执行的步骤
3. **识别相依性**元件之间的关係
4. **评估风险**和潜在阻碍
5. **估计复杂度**（高/中/低）
6. **呈现计划**并等待您明确确认

## 使用范例

```
使用者：/plan 我需要在市場解决时新增即时通知

Agent（planner）：
# 实作计划：即时市場解决通知

## 需求重述
- 当使用者关注的市場解决时发送通知
- 支援多个通知管道（应用内、Email、Webhook）
- 确保通知可靠送达
- 包含市場结果和使用者持倉结果

## 实作阶段

### 阶段 1：资料库结构描述
- 新增 notifications 表格，字段：id、user_id、market_id、type、status、created_at
- 新增 user_notification_preferences 表格用於管道偏好
- 在 user_id 和 market_id 上建立索引以优化效能

### 阶段 2：通知服务
- 在 lib/notifications.ts 建立通知服务
- 使用 BullMQ/Redis 实作通知队列
- 为失敗的送达新增重试逻辑
- 建立通知范本

### 阶段 3：整合点
- 连接到市場解决逻辑（当状态变更为 "resolved"）
- 查询所有在该市場有持倉的使用者
- 为每个使用者加入通知队列

### 阶段 4：前端元件
- 在标题建立 NotificationBell 元件
- 新增 NotificationList 彈窗
- 透过 Supabase 订阅实作即时更新
- 新增通知偏好设置页面

## 相依性
- Redis（用於队列）
- Email 服务（SendGrid/Resend）
- Supabase 即时订阅

## 风险
- 高：Email 送达率（需要 SPF/DKIM）
- 中：每个市場 1000+ 使用者时的效能
- 中：如果市場频繁解决可能造成通知垃圾讯息
- 低：即时订阅的开销

## 估计复杂度：中
- 后端：4-6 小时
- 前端：3-4 小时
- 测试：2-3 小时
- 总计：9-13 小时

**等待确认**：继续此计划？（是/否/修改）
```

## 重要提醒

**关键**：planner Agent **不会**撰寫任何程序代码，直到您明确以「是」、「继续」或类似肯定回应确认计划。

如果您想要修改，回应：
- "修改：[您的变更]"
- "不同的方法：[替代方案]"
- "跳过阶段 2，先做阶段 3"

## 与其他指令的整合

计划后：
- 使用 `/tdd` 以测试驱动开发实作
- 如果发生构建错误，使用 `/build-fix`
- 使用 `/code-review` 审查完成的实作

## 相关 Agent

此指令呼叫位於以下位置的 `planner` Agent：
`~/.claude/agents/planner.md`
