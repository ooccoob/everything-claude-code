---
name: postgres-patterns
description: PostgreSQL database patterns for query optimization, schema design, indexing, and security. Based on Supabase best practices.
---

# PostgreSQL 模式

PostgreSQL 最佳实务快速參考。详细指南请使用 `database-reviewer` agent。

## 何时启用

- 撰寫 SQL 查询或 migrations
- 设计资料库 schema
- 疑难排解慢查询
- 实作 Row Level Security
- 设置连线池

## 快速參考

### 索引速查表

| 查询模式 | 索引类型 | 范例 |
|---------|---------|------|
| `WHERE col = value` | B-tree（预设） | `CREATE INDEX idx ON t (col)` |
| `WHERE col > value` | B-tree | `CREATE INDEX idx ON t (col)` |
| `WHERE a = x AND b > y` | 复合 | `CREATE INDEX idx ON t (a, b)` |
| `WHERE jsonb @> '{}'` | GIN | `CREATE INDEX idx ON t USING gin (col)` |
| `WHERE tsv @@ query` | GIN | `CREATE INDEX idx ON t USING gin (col)` |
| 时间序列范围 | BRIN | `CREATE INDEX idx ON t USING brin (col)` |

### 资料类型快速參考

| 使用情况 | 正确类型 | 避免 |
|---------|---------|------|
| IDs | `bigint` | `int`、随机 UUID |
| 字串 | `text` | `varchar(255)` |
| 时间戳 | `timestamptz` | `timestamp` |
| 金额 | `numeric(10,2)` | `float` |
| 旗标 | `boolean` | `varchar`、`int` |

### 常见模式

**复合索引顺序：**
```sql
-- 等值字段优先，然后是范围字段
CREATE INDEX idx ON orders (status, created_at);
-- 适用於：WHERE status = 'pending' AND created_at > '2024-01-01'
```

**覆盖索引：**
```sql
CREATE INDEX idx ON users (email) INCLUDE (name, created_at);
-- 避免 SELECT email, name, created_at 时的表格查询
```

**部分索引：**
```sql
CREATE INDEX idx ON users (email) WHERE deleted_at IS NULL;
-- 更小的索引，只包含活跃使用者
```

**RLS 政策（优化）：**
```sql
CREATE POLICY policy ON orders
  USING ((SELECT auth.uid()) = user_id);  -- 用 SELECT 包装！
```

**UPSERT：**
```sql
INSERT INTO settings (user_id, key, value)
VALUES (123, 'theme', 'dark')
ON CONFLICT (user_id, key)
DO UPDATE SET value = EXCLUDED.value;
```

**游标分页：**
```sql
SELECT * FROM products WHERE id > $last_id ORDER BY id LIMIT 20;
-- O(1) vs OFFSET 是 O(n)
```

**队列处理：**
```sql
UPDATE jobs SET status = 'processing'
WHERE id = (
  SELECT id FROM jobs WHERE status = 'pending'
  ORDER BY created_at LIMIT 1
  FOR UPDATE SKIP LOCKED
) RETURNING *;
```

### 反模式侦测

```sql
-- 找出未建索引的外键
SELECT conrelid::regclass, a.attname
FROM pg_constraint c
JOIN pg_attribute a ON a.attrelid = c.conrelid AND a.attnum = ANY(c.conkey)
WHERE c.contype = 'f'
  AND NOT EXISTS (
    SELECT 1 FROM pg_index i
    WHERE i.indrelid = c.conrelid AND a.attnum = ANY(i.indkey)
  );

-- 找出慢查询
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
WHERE mean_exec_time > 100
ORDER BY mean_exec_time DESC;

-- 检查表格膨脹
SELECT relname, n_dead_tup, last_vacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC;
```

### 设置范本

```sql
-- 连线限制（依 RAM 调整）
ALTER SYSTEM SET max_connections = 100;
ALTER SYSTEM SET work_mem = '8MB';

-- 逾时
ALTER SYSTEM SET idle_in_transaction_session_timeout = '30s';
ALTER SYSTEM SET statement_timeout = '30s';

-- 监控
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- 安全预设值
REVOKE ALL ON SCHEMA public FROM public;

SELECT pg_reload_conf();
```

## 相关

- Agent：`database-reviewer` - 完整资料库审查工作流程
- Skill：`clickhouse-io` - ClickHouse 分析模式
- Skill：`backend-patterns` - API 和后端模式

---

*基於 [Supabase Agent Skills](https://github.com/supabase/agent-skills)（MIT 授权）*
