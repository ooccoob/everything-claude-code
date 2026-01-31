# Hook 系统

## Hook 类型

- **PreToolUse**：工具执行前（验证、參数修改）
- **PostToolUse**：工具执行后（自动格式化、检查）
- **Stop**：工作阶段结束时（最终验证）

## 目前 Hooks（在 ~/.claude/settings.json）

### PreToolUse
- **tmux 提醒**：建议对长时间执行的指令使用 tmux（npm、pnpm、yarn、cargo 等）
- **git push 审查**：推送前开启 Zed 进行审查
- **文件阻擋器**：阻擋建立不必要的 .md/.txt 档案

### PostToolUse
- **PR 建立**：记录 PR URL 和 GitHub Actions 状态
- **Prettier**：编辑后自动格式化 JS/TS 档案
- **TypeScript 检查**：编辑 .ts/.tsx 档案后执行 tsc
- **console.log 警告**：警告编辑档案中的 console.log

### Stop
- **console.log 稽核**：工作阶段结束前检查所有修改档案中的 console.log

## 自动接受权限

谨慎使用：
- 对受信任、定义明确的计划启用
- 对探索性工作停用
- 绝不使用 dangerously-skip-permissions flag
- 改为在 `~/.claude.json` 中设置 `allowedTools`

## TodoWrite 最佳实务

使用 TodoWrite 工具来：
- 追踪多步骤任务的进度
- 验证对指示的理解
- 启用即时调整
- 显示细粒度实作步骤

待办清单揭示：
- 顺序错误的步骤
- 缺少的项目
- 多余的不必要项目
- 错误的粒度
- 误解的需求
