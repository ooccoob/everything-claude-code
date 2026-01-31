---
description: Configure your preferred package manager (npm/pnpm/yarn/bun)
disable-model-invocation: true
---

# 套件管理器设置

为此专案或全域设置您偏好的套件管理器。

## 使用方式

```bash
# 侦测目前的套件管理器
node scripts/setup-package-manager.js --detect

# 设置全域偏好
node scripts/setup-package-manager.js --global pnpm

# 设置专案偏好
node scripts/setup-package-manager.js --project bun

# 列出可用的套件管理器
node scripts/setup-package-manager.js --list
```

## 侦测优先顺序

决定使用哪个套件管理器时，按以下顺序检查：

1. **环境变量**：`CLAUDE_PACKAGE_MANAGER`
2. **专案设置**：`.claude/package-manager.json`
3. **package.json**：`packageManager` 字段
4. **Lock 档案**：是否存在 package-lock.json、yarn.lock、pnpm-lock.yaml 或 bun.lockb
5. **全域设置**：`~/.claude/package-manager.json`
6. **备援**：第一个可用的套件管理器（pnpm > bun > yarn > npm）

## 设置档

### 全域设置
```json
// ~/.claude/package-manager.json
{
  "packageManager": "pnpm"
}
```

### 专案设置
```json
// .claude/package-manager.json
{
  "packageManager": "bun"
}
```

### package.json
```json
{
  "packageManager": "pnpm@8.6.0"
}
```

## 环境变量

设置 `CLAUDE_PACKAGE_MANAGER` 以覆盖所有其他侦测方法：

```bash
# Windows (PowerShell)
$env:CLAUDE_PACKAGE_MANAGER = "pnpm"

# macOS/Linux
export CLAUDE_PACKAGE_MANAGER=pnpm
```

## 执行侦测

要查看目前套件管理器侦测结果，执行：

```bash
node scripts/setup-package-manager.js --detect
```
