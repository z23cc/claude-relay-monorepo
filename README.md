# Claude Relay Monorepo

一个现代化的 Claude API 代理服务，基于 Cloudflare Workers 构建，让您安全便捷地使用 Claude Code。

## 🌟 主要特性

- 🔐 **OAuth 认证** - 安全的 OAuth 2.0 PKCE 流程，无需手动管理 API 密钥
- 🌐 **全球部署** - 基于 Cloudflare Workers，享受全球边缘网络的低延迟
- 💻 **现代化管理界面** - 直观的 Web 界面管理您的 Claude 账号和配置
- 🚀 **自动化运维** - 自动刷新 Token，智能错误处理和重试机制
- 📊 **完善的监控** - 实时查看使用情况和系统状态
- 🔧 **自由添加 LLM 供应商** - 支持添加 魔搭 等第三方模型供应商，灵活切换使用

## 🏃‍♂️ 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/your-username/claude-relay-monorepo.git
cd claude-relay-monorepo
npm install
```

### 2. 配置环境

创建 `packages/backend/wrangler.toml` 文件：

```toml
name = "claude-relay-backend"
main = "src/index.ts"
compatibility_date = "2024-12-01"

[vars]
NODE_ENV = "production"
FRONTEND_URL = "https://your-frontend.pages.dev"
ADMIN_USERNAME = "admin"
ADMIN_PASSWORD = "your-secure-password"

[[kv_namespaces]]
binding = "CLAUDE_KV"
id = "your-kv-namespace-id"
```

### 3. 部署服务

```bash
# 一键部署前后端
npm run deploy:all

# 或分别部署
npm run deploy:backend  # 先部署后端
npm run deploy:frontend # 再部署前端
```

### 4. 配置 Claude Code

部署成功后，配置 Claude Code 使用您的代理服务：

```bash
export ANTHROPIC_BASE_URL="https://your-backend.workers.dev/"
export ANTHROPIC_AUTH_TOKEN="placeholder"
```

### 5. 开始使用

访问管理界面完成 OAuth 授权：

1. 打开 `https://your-frontend.pages.dev/admin`
2. 使用配置的管理员账号登录
3. 在 Claude 账号页面添加账号并完成授权
4. 开始使用 Claude Code！

```bash
claude
```

## 📁 项目结构

```
claude-relay-monorepo/
├── packages/
│   ├── frontend/          # Nuxt 4 前端应用
│   └── backend/           # Hono 后端服务
└── shared/                # 共享类型定义
```

## 🛠️ 开发指南

### 本地开发

```bash
# 启动前端开发服务器 (localhost:3000)
npm run dev:frontend

# 启动后端开发服务器
npm run dev:backend

# 代码检查和格式化
npm run lint
npm run format
```

### 常用命令

- `npm run build:all` - 构建前后端
- `npm run deploy:all` - 部署整个应用
- `npm run type-check` - TypeScript 类型检查

## 🔧 高级配置

### 管理中心功能

管理中心提供以下功能：

- **Claude 账号管理** - 添加、删除和管理多个 Claude 账号
- **模型供应商** - 配置第三方 AI 模型供应商
- **系统监控** - 查看账号状态和使用统计

### 环境变量说明

后端环境变量（wrangler.toml）：
- `NODE_ENV` - 运行环境（production/preview）
- `FRONTEND_URL` - 前端地址，用于 CORS 配置
- `ADMIN_USERNAME` - 管理员用户名
- `ADMIN_PASSWORD` - 管理员密码

前端环境变量：
- `NUXT_PUBLIC_API_BASE_URL` - 后端 API 地址

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request！

## 📄 许可证

本项目采用 MIT 许可证。

---

⭐ 如果这个项目对您有帮助，请给我们一个星标！