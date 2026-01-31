| name | description |
|------|-------------|
| cloud-infrastructure-security | Use this skill when deploying to cloud platforms, configuring infrastructure, managing IAM policies, setting up logging/monitoring, or implementing CI/CD pipelines. Provides cloud security checklist aligned with best practices. |

# 云端与基础设施安全技能

此技能确保云端基础设施、CI/CD 管线和部署设置遵循安全最佳实务并符合业界标准。

## 何时启用

- 部署应用程序到云端平台（AWS、Vercel、Railway、Cloudflare）
- 设置 IAM 角色和权限
- 设置 CI/CD 管线
- 实作基础设施即程序代码（Terraform、CloudFormation）
- 设置日志和监控
- 在云端环境管理密钥
- 设置 CDN 和边缘安全
- 实作災难还原和备份策略

## 云端安全检查清单

### 1. IAM 与存取控制

#### 最小权限原则

```yaml
# ✅ 正确：最小权限
iam_role:
  permissions:
    - s3:GetObject  # 只有读取存取
    - s3:ListBucket
  resources:
    - arn:aws:s3:::my-bucket/*  # 只有特定 bucket

# ❌ 错误：过於廣泛的权限
iam_role:
  permissions:
    - s3:*  # 所有 S3 动作
  resources:
    - "*"  # 所有资源
```

#### 多因素认证（MFA）

```bash
# 总是为 root/admin 帳戶启用 MFA
aws iam enable-mfa-device \
  --user-name admin \
  --serial-number arn:aws:iam::123456789:mfa/admin \
  --authentication-code1 123456 \
  --authentication-code2 789012
```

#### 验证步骤

- [ ] 生产环境不使用 root 帳戶
- [ ] 所有特权帳戶启用 MFA
- [ ] 服务帳戶使用角色，非长期憑证
- [ ] IAM 政策遵循最小权限
- [ ] 定期进行存取审查
- [ ] 未使用憑证已轮換或移除

### 2. 密钥管理

#### 云端密钥管理器

```typescript
// ✅ 正确：使用云端密钥管理器
import { SecretsManager } from '@aws-sdk/client-secrets-manager';

const client = new SecretsManager({ region: 'us-east-1' });
const secret = await client.getSecretValue({ SecretId: 'prod/api-key' });
const apiKey = JSON.parse(secret.SecretString).key;

// ❌ 错误：寫死或只在环境变量
const apiKey = process.env.API_KEY; // 未轮換、未稽核
```

#### 密钥轮換

```bash
# 为资料库憑证设置自动轮換
aws secretsmanager rotate-secret \
  --secret-id prod/db-password \
  --rotation-lambda-arn arn:aws:lambda:region:account:function:rotate \
  --rotation-rules AutomaticallyAfterDays=30
```

#### 验证步骤

- [ ] 所有密钥保存在云端密钥管理器（AWS Secrets Manager、Vercel Secrets）
- [ ] 资料库憑证启用自动轮換
- [ ] API 金钥至少每季轮換
- [ ] 程序代码、日志或错误讯息中无密钥
- [ ] 密钥存取启用稽核日志

### 3. 网路安全

#### VPC 和防火墙设置

```terraform
# ✅ 正确：限制的安全群组
resource "aws_security_group" "app" {
  name = "app-sg"

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]  # 只有内部 VPC
  }

  egress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # 只有 HTTPS 输出
  }
}

# ❌ 错误：对网际网路开放
resource "aws_security_group" "bad" {
  ingress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # 所有埠、所有 IP！
  }
}
```

#### 验证步骤

- [ ] 资料库不可公开存取
- [ ] SSH/RDP 埠限制为 VPN/堡壘机
- [ ] 安全群组遵循最小权限
- [ ] 网路 ACL 已设置
- [ ] VPC 流量日志已启用

### 4. 日志与监控

#### CloudWatch/日志设置

```typescript
// ✅ 正确：全面日志记录
import { CloudWatchLogsClient, CreateLogStreamCommand } from '@aws-sdk/client-cloudwatch-logs';

const logSecurityEvent = async (event: SecurityEvent) => {
  await cloudwatch.putLogEvents({
    logGroupName: '/aws/security/events',
    logStreamName: 'authentication',
    logEvents: [{
      timestamp: Date.now(),
      message: JSON.stringify({
        type: event.type,
        userId: event.userId,
        ip: event.ip,
        result: event.result,
        // 永远不要记录敏感资料
      })
    }]
  });
};
```

#### 验证步骤

- [ ] 所有服务启用 CloudWatch/日志记录
- [ ] 失敗的认证嘗试被记录
- [ ] 管理员动作被稽核
- [ ] 日志保留已设置（合规需 90+ 天）
- [ ] 可疑活动设置警報
- [ ] 日志集中化且防篡改

### 5. CI/CD 管线安全

#### 安全管线设置

```yaml
# ✅ 正确：安全的 GitHub Actions 工作流程
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read  # 最小权限

    steps:
      - uses: actions/checkout@v4

      # 掃描密钥
      - name: Secret scanning
        uses: trufflesecurity/trufflehog@main

      # 依赖稽核
      - name: Audit dependencies
        run: npm audit --audit-level=high

      # 使用 OIDC，非长期 tokens
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/GitHubActionsRole
          aws-region: us-east-1
```

#### 供应链安全

```json
// package.json - 使用 lock 档案和完整性检查
{
  "scripts": {
    "install": "npm ci",  // 使用 ci 以获得可重现构建
    "audit": "npm audit --audit-level=moderate",
    "check": "npm outdated"
  }
}
```

#### 验证步骤

- [ ] 使用 OIDC 而非长期憑证
- [ ] 管线中的密钥掃描
- [ ] 依赖漏洞掃描
- [ ] 容器映像掃描（如适用）
- [ ] 強制执行分支保护规则
- [ ] 合并前需要程序代码审查
- [ ] 強制执行签署 commits

### 6. Cloudflare 与 CDN 安全

#### Cloudflare 安全设置

```typescript
// ✅ 正确：帶安全标头的 Cloudflare Workers
export default {
  async fetch(request: Request): Promise<Response> {
    const response = await fetch(request);

    // 新增安全标头
    const headers = new Headers(response.headers);
    headers.set('X-Frame-Options', 'DENY');
    headers.set('X-Content-Type-Options', 'nosniff');
    headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
    headers.set('Permissions-Policy', 'geolocation=(), microphone=()');

    return new Response(response.body, {
      status: response.status,
      headers
    });
  }
};
```

#### WAF 规则

```bash
# 启用 Cloudflare WAF 管理规则
# - OWASP 核心规则集
# - Cloudflare 管理规则集
# - 速率限制规则
# - Bot 保护
```

#### 验证步骤

- [ ] WAF 启用 OWASP 规则
- [ ] 速率限制已设置
- [ ] Bot 保护启用
- [ ] DDoS 保护启用
- [ ] 安全标头已设置
- [ ] SSL/TLS 嚴格模式启用

### 7. 备份与災难还原

#### 自动备份

```terraform
# ✅ 正确：自动 RDS 备份
resource "aws_db_instance" "main" {
  allocated_storage     = 20
  engine               = "postgres"

  backup_retention_period = 30  # 30 天保留
  backup_window          = "03:00-04:00"
  maintenance_window     = "mon:04:00-mon:05:00"

  enabled_cloudwatch_logs_exports = ["postgresql"]

  deletion_protection = true  # 防止意外删除
}
```

#### 验证步骤

- [ ] 已设置自动每日备份
- [ ] 备份保留符合合规要求
- [ ] 已启用时间点还原
- [ ] 每季执行备份测试
- [ ] 災难还原计划已记录
- [ ] RPO 和 RTO 已定义并测试

## 部署前云端安全检查清单

任何生产云端部署前：

- [ ] **IAM**：不使用 root 帳戶、启用 MFA、最小权限政策
- [ ] **密钥**：所有密钥在云端密钥管理器并有轮換
- [ ] **网路**：安全群组受限、无公开资料库
- [ ] **日志**：CloudWatch/日志启用并有保留
- [ ] **监控**：异常设置警報
- [ ] **CI/CD**：OIDC 认证、密钥掃描、依赖稽核
- [ ] **CDN/WAF**：Cloudflare WAF 启用 OWASP 规则
- [ ] **加密**：资料静态和传输中加密
- [ ] **备份**：自动备份并测试还原
- [ ] **合规**：符合 GDPR/HIPAA 要求（如适用）
- [ ] **文件**：基础设施已记录、建立操作手册
- [ ] **事件回应**：安全事件计划就位

## 常见云端安全错误设置

### S3 Bucket 暴露

```bash
# ❌ 错误：公开 bucket
aws s3api put-bucket-acl --bucket my-bucket --acl public-read

# ✅ 正确：私有 bucket 并有特定存取
aws s3api put-bucket-acl --bucket my-bucket --acl private
aws s3api put-bucket-policy --bucket my-bucket --policy file://policy.json
```

### RDS 公开存取

```terraform
# ❌ 错误
resource "aws_db_instance" "bad" {
  publicly_accessible = true  # 绝不这样做！
}

# ✅ 正确
resource "aws_db_instance" "good" {
  publicly_accessible = false
  vpc_security_group_ids = [aws_security_group.db.id]
}
```

## 资源

- [AWS Security Best Practices](https://aws.amazon.com/security/best-practices/)
- [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services)
- [Cloudflare Security Documentation](https://developers.cloudflare.com/security/)
- [OWASP Cloud Security](https://owasp.org/www-project-cloud-security/)
- [Terraform Security Best Practices](https://www.terraform.io/docs/cloud/guides/recommended-practices/)

**记住**：云端错误设置是资料外泄的主要原因。单一暴露的 S3 bucket 或过於寬鬆的 IAM 政策可能危及你的整个基础设施。总是遵循最小权限原则和深度防御。
