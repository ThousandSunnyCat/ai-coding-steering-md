# Kiro 安全检查 Agent Hooks 系统

这是一套完整的代码安全检查系统，通过 Kiro Agent Hooks 自动监控和防护项目中的敏感信息泄露。

## 🛡️ 系统概览

### 核心组件

| Hook 名称 | 触发方式 | 主要功能 |
|-----------|----------|----------|
| `security-scanner` | 文件保存时 | 检测硬编码的 API 密钥、密码等敏感信息 |
| `pre-commit-security` | 手动触发 | Git 提交前的安全检查，防止敏感信息进入版本库 |
| `env-file-validator` | .env 文件保存时 | 验证环境变量文件的安全性和完整性 |
| `dependency-security-check` | 依赖文件保存时 | 检查第三方依赖中的安全漏洞 |
| `config-security-audit` | 配置文件保存时 | 审计各类配置文件的安全设置 |
| `master-security-check` | 手动触发 | 执行全面的项目安全审计和报告生成 |

## 🚀 快速开始

### 1. 启用 Hooks

在 Kiro 中打开命令面板（Cmd/Ctrl + Shift + P），搜索 "Open Kiro Hook UI"，然后：

1. 导入 `.kiro/hooks/` 目录下的所有 hook 文件
2. 根据项目需要启用相应的 hooks
3. 配置触发条件和目标文件模式

### 2. 基础配置

确保项目包含以下安全基础文件：

```bash
# 创建 .gitignore（如果不存在）
echo ".env
.env.*
!.env.example
*.key
*.pem
config/secrets.json" >> .gitignore

# 创建环境变量模板
cp .env .env.example
# 手动将敏感值替换为占位符
```

### 3. 运行首次安全检查

使用 `master-security-check` hook 进行全面扫描：

1. 在 Kiro 中触发 "master-security-check" hook
2. 查看生成的安全报告
3. 按优先级修复发现的问题

## 📋 使用指南

### 日常开发流程

1. **编码阶段**：`security-scanner` 自动检查每个保存的文件
2. **配置修改**：相关 hooks 自动验证配置文件安全性
3. **提交前**：手动运行 `pre-commit-security` 进行最终检查
4. **定期审计**：每周运行 `master-security-check` 生成安全报告

### 常见问题处理

#### 🚨 发现硬编码密钥

```javascript
// ❌ 问题代码
const apiKey = "sk-1234567890abcdef";

// ✅ 修复方案
const apiKey = process.env.OPENAI_API_KEY;
```

#### ⚠️ 依赖包有漏洞

```bash
# 查看具体漏洞信息
npm audit

# 自动修复（谨慎使用）
npm audit fix

# 手动升级特定包
npm install package-name@latest
```

#### 🔧 配置文件不安全

```yaml
# ❌ 不安全配置
database:
  password: "hardcoded_password"

# ✅ 安全配置  
database:
  password: ${DB_PASSWORD}
```

## 🎯 安全检查覆盖范围

### 敏感信息检测

- **API 密钥**：OpenAI, AWS, Google, GitHub 等
- **数据库凭据**：连接字符串、用户名密码
- **加密密钥**：JWT secret, 加密密钥
- **私钥文件**：SSL 证书、SSH 密钥

### 配置安全检查

- **Web 服务器**：Nginx, Apache 安全配置
- **容器化**：Docker, docker-compose 安全实践
- **CI/CD**：GitHub Actions, GitLab CI 权限配置
- **数据库**：MongoDB, Redis, MySQL 安全设置

### 依赖安全监控

- **已知漏洞**：CVE 数据库匹配
- **恶意包**：供应链攻击检测
- **版本过时**：安全更新提醒
- **许可证**：开源许可证合规检查

## 📊 安全评分系统

| 评分范围 | 安全等级 | 说明 |
|----------|----------|------|
| 90-100 | 🟢 优秀 | 无安全风险，遵循最佳实践 |
| 70-89 | 🟡 良好 | 有轻微问题，但不影响安全 |
| 50-69 | 🟠 警告 | 存在中等风险，需要关注 |
| 0-49 | 🔴 危险 | 存在严重安全风险，需立即修复 |

## 🔧 自定义配置

### 添加新的敏感信息模式

编辑 `security-scanner.md`，在检测模式中添加：

```regex
# 自定义 API 密钥模式
CUSTOM_API_[A-Z0-9]{32}
```

### 排除特定文件

在 hook 配置中修改 `Target` 参数：

```yaml
Target: "**/*,!test/**,!docs/**"
```

### 调整安全等级

修改各个 hook 中的风险评分权重。

## 🚨 应急响应

### 发现敏感信息已提交

```bash
# 1. 立即撤销提交（如果还未推送）
git reset --hard HEAD~1

# 2. 如果已推送，使用 git filter-branch 清理历史
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch path/to/sensitive/file' \
  --prune-empty --tag-name-filter cat -- --all

# 3. 强制推送（谨慎操作）
git push origin --force --all
```

### 密钥泄露处理

1. **立即撤销**泄露的 API 密钥
2. **生成新密钥**并更新环境变量
3. **检查日志**确认是否有异常使用
4. **通知团队**相关安全事件

## 📚 最佳实践

### 环境变量管理

```bash
# 开发环境
.env.development

# 生产环境（不提交到版本库）
.env.production

# 示例模板（提交到版本库）
.env.example
```

### 密钥轮换策略

- **定期轮换**：每 90 天更换一次密钥
- **权限最小化**：只授予必要的 API 权限
- **监控使用**：设置异常使用告警

### 团队协作

- **代码审查**：重点检查安全相关变更
- **安全培训**：定期进行安全意识培训
- **文档维护**：及时更新安全策略文档

## 🔗 相关资源

- [OWASP 安全编码实践](https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/)
- [GitHub 安全最佳实践](https://docs.github.com/en/code-security)
- [Docker 安全指南](https://docs.docker.com/engine/security/)
- [Node.js 安全检查清单](https://blog.risingstack.com/node-js-security-checklist/)

## 📞 支持与反馈

如果在使用过程中遇到问题或有改进建议，请：

1. 检查本文档的常见问题部分
2. 查看 Kiro 官方文档
3. 在项目中创建 issue 反馈

---

**⚠️ 重要提醒**：安全是一个持续的过程，请定期运行安全检查并及时修复发现的问题。