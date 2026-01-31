# Checkpoint 指令

在您的工作流程中建立或验证检查点。

## 使用方式

`/checkpoint [create|verify|list] [name]`

## 建立检查点

建立检查点时：

1. 执行 `/verify quick` 确保目前状态是乾净的
2. 使用检查点名称建立 git stash 或 commit
3. 将检查点记录到 `.claude/checkpoints.log`：

```bash
echo "$(date +%Y-%m-%d-%H:%M) | $CHECKPOINT_NAME | $(git rev-parse --short HEAD)" >> .claude/checkpoints.log
```

4. 報告检查点已建立

## 验证检查点

针对检查点进行验证时：

1. 从日志读取检查点
2. 比较目前状态与检查点：
   - 检查点后新增的档案
   - 检查点后修改的档案
   - 现在 vs 当时的测试通过率
   - 现在 vs 当时的覆盖率

3. 報告：
```
检查点比较：$NAME
============================
变更档案：X
测试：+Y 通过 / -Z 失敗
覆盖率：+X% / -Y%
构建：[通过/失敗]
```

## 列出检查点

显示所有检查点，包含：
- 名称
- 时间戳
- Git SHA
- 状态（目前、落后、领先）

## 工作流程

典型的检查点流程：

```
[开始] --> /checkpoint create "feature-start"
   |
[实作] --> /checkpoint create "core-done"
   |
[测试] --> /checkpoint verify "core-done"
   |
[重构] --> /checkpoint create "refactor-done"
   |
[PR] --> /checkpoint verify "feature-start"
```

## 參数

$ARGUMENTS:
- `create <name>` - 建立命名检查点
- `verify <name>` - 针对命名检查点验证
- `list` - 显示所有检查点
- `clear` - 移除旧检查点（保留最后 5 个）
