# CLAUDE.md

此文件为 Claude Code (claude.ai/code) 在此代码仓库中工作时提供指导。

## 开发命令

### 核心开发命令
- `npm run dev:frontend` - 启动 Nuxt 前端开发服务器 (localhost:3000)
- `npm run dev:backend` - 使用 Wrangler 启动 Hono 后端开发服务器
- `npm run build:all` - 同时构建前端和后端
- `npm run deploy:all` - 构建并部署两个服务到 Cloudflare（先部署后端，再部署前端）

### 单独包命令
- `npm run build:frontend` / `npm run build:backend` - 构建单独的包
- `npm run deploy:frontend` / `npm run deploy:backend` - 部署单独的包
- `npm run type-check` - 对两个包进行 TypeScript 验证

### 代码质量
- `npm run lint` - 对所有 TypeScript/Vue 文件运行 ESLint
- `npm run lint:fix` - 自动修复 ESLint 问题
- `npm run format` - 使用 Prettier 格式化代码
- `npm run format:check` - 检查格式化而不进行修改

## 架构概览

这是一个包含 Cloudflare 全栈应用的 **monorepo**，具有共享的 TypeScript 类型。

### 项目结构
```
├── packages/
│   ├── frontend/          # Nuxt 4 + Tailwind CSS 前端
│   └── backend/           # 用于 Cloudflare Workers 的 Hono 后端
└── shared/                # 共享的 TypeScript 类型和常量
```

### 技术栈
- **前端**: Nuxt 4, Vue 3, Tailwind CSS，部署到 Cloudflare Pages
- **后端**: Hono 框架, TypeScript，部署到 Cloudflare Workers
- **共享**: 前后端共享的 TypeScript 类型和 API 常量

### 关键架构决策

#### 后端 (Hono + Cloudflare Workers)
- **服务分层架构**: 路由层(Routes) → 服务层(Services) → 存储层(KV)
- **OAuth 2.0 PKCE 流程**: 完整的 Claude AI OAuth 认证实现
- **Claude API 代理**: 透明代理 Claude API 请求，自动处理认证和错误
- **定时任务支持**: 每6小时自动刷新 OAuth token (Cron: `0 */6 * * *`)
- **统一错误处理**: 全局异常捕获和标准化错误响应
- **开放 CORS 配置**: 支持所有来源访问，适配 Claude Code 客户端
- **TypeScript 类型安全**: 严格类型检查和共享类型定义

#### 前端 (Nuxt 4 + Cloudflare Pages)
- 针对 Cloudflare Pages 配置，具有优化的构建设置
- 使用手动代码分割（vendor chunks）以提升性能
- 首页在构建时预渲染
- 通过 `runtimeConfig` 管理环境变量

#### 共享代码策略
- **类型定义**: 认证相关(`auth.ts`)、Claude API(`claude.ts`)、通用 API(`api.ts`)
- **常量配置**: OAuth 配置(`oauth.ts`)、Claude API 配置(`claude.ts`)、错误码(`errors.ts`)
- **导入路径**: 从 packages 使用相对路径如 `../../../shared/types/auth`

### 环境配置

#### 后端环境变量 (wrangler.toml)
- `NODE_ENV` - 控制错误详细程度和 CORS 行为 (production/preview)
- `FRONTEND_URL` - 前端 URL，用于 CORS 配置
- `ADMIN_USERNAME` - 管理中心登录用户名 (默认: admin)
- `ADMIN_PASSWORD` - 管理中心登录密码 (默认: password123)
- `CLAUDE_KV` - KV Namespace 绑定，存储 OAuth token 和应用状态

#### 前端环境变量
- `NUXT_PUBLIC_API_BASE_URL` - 后端 API 基础 URL
- `NUXT_PUBLIC_APP_NAME` / `NUXT_PUBLIC_APP_VERSION` - 应用元数据

### 部署流程
1. **后端先部署** (`npm run deploy:backend`) - 确保 API 可用
2. **前端后部署** (`npm run deploy:frontend`) - 连接到线上 API
3. **完整部署** (`npm run deploy:all`) - 处理整个序列

### 开发工作流程
- 前端默认连接到已部署的后端（而非本地后端）
- 两个包都使用基于 workspace 的 npm 脚本以保持一致性
- 共享类型确保前后端 API 契约一致性
- 所有包都启用 TypeScript 严格模式
- 后端作为 Claude Code 请求转发代理，支持来自任意客户端的访问

## 后端 API 架构

### 核心 API 端点

#### OAuth 认证 API (`/api/auth/*`)
- `GET /api/auth/url` - 生成 OAuth 授权链接和 PKCE 参数
- `POST /api/auth/token` - 使用授权码交换 access token
- `GET /api/auth/token` - 检查当前 token 状态和有效性

#### Claude API 代理 (`/v1/*`)
- `POST /v1/messages` - 代理 Claude API 消息请求，自动处理认证

#### 管理中心 API (`/api/admin/*`)
- `POST /api/admin/auth` - 管理员认证登录
- `GET /api/admin/dashboard` - 获取仪表板数据（Claude 账号状态、供应商统计等）
- `GET /api/admin/providers` - 获取所有模型供应商
- `POST /api/admin/providers` - 添加新的模型供应商
- `DELETE /api/admin/providers/:id` - 删除模型供应商
- `GET /api/admin/models` - 获取可用模型列表
- `POST /api/admin/select-model` - 选择默认使用的模型

#### 系统端点
- `GET /health` - 应用健康检查和状态信息

### 服务层架构

#### AuthService (认证服务)
- **职责**: OAuth 2.0 PKCE 流程管理、token 生命周期管理
- **核心方法**:
  - `generateAuthUrl()` - 生成带 PKCE 的授权 URL
  - `exchangeToken(code, pkce)` - 授权码换取 token
  - `refreshToken()` - 自动刷新过期 token
  - `getValidToken()` - 获取有效的访问令牌
  - `getTokenStatus()` - 检查 token 状态

#### ClaudeProxyService (代理服务) 
- **职责**: Claude API 请求转发、错误处理、响应流处理
- **核心方法**:
  - `proxyRequest(request)` - 透明代理 Claude API 请求
- **特性**: 保持原始响应头、支持流式响应、智能错误处理

#### AdminService (管理服务)
- **职责**: 管理中心功能实现、模型供应商管理、系统配置
- **核心方法**:
  - `verifyAdmin(username, password)` - 验证管理员凭据
  - `getDashboardData()` - 获取仪表板统计数据
  - `getProviders()` / `addProvider()` / `deleteProvider()` - 供应商管理
  - `getAvailableModels()` / `selectModel()` - 模型选择和切换

### 数据存储设计

#### KV 存储结构
```typescript
// OAuth Token 存储
'claude_token': {
  access_token: string,
  refresh_token: string,
  expires_at: number,
  scope: string,
  // ... 其他 token 信息
}

// 管理中心数据存储
'admin_model_providers': ModelProvider[]  // 模型供应商列表
'admin_selected_model': SelectedModel     // 当前选中的模型
```

### 错误处理机制

#### 自定义错误类型
- `AuthError` - OAuth 认证相关错误
- `TokenExpiredError` - Token 过期错误  
- `ClaudeApiError` - Claude API 调用错误
- `ValidationError` - 请求参数验证错误

#### 统一错误响应格式
```typescript
{
  success: false,
  error: {
    type: 'ERROR_TYPE',
    message: 'Human readable message'
  },
  timestamp: string
}
```

### 自动化任务

#### 定时任务 (Cron Jobs)
- **频率**: 每6小时执行 (`0 */6 * * *`)
- **功能**: 自动检查并刷新即将过期的 OAuth token
- **实现**: Cloudflare Workers scheduled handler

### 需要理解的关键文件

#### 后端核心文件
- `packages/backend/src/index.ts` - Hono 应用入口，中间件和路由配置
- `packages/backend/src/routes/auth.ts` - OAuth 认证路由处理
- `packages/backend/src/routes/claude.ts` - Claude API 代理路由
- `packages/backend/src/services/auth.ts` - OAuth 认证服务实现
- `packages/backend/src/services/claude.ts` - Claude API 代理服务
- `packages/backend/src/types/env.ts` - 环境变量和绑定类型定义

#### 共享类型和常量
- `shared/types/auth.ts` - OAuth 认证相关类型定义
- `shared/types/claude.ts` - Claude API 请求/响应类型
- `shared/constants/oauth.ts` - OAuth 配置常量
- `shared/constants/claude.ts` - Claude API 配置常量

#### 配置文件
- `packages/backend/wrangler.toml` - Cloudflare Workers 部署配置
- `根目录/package.json` - Workspace 配置和统一脚本

### 开发最佳实践

#### 代码组织原则
- **分层架构**: 严格的路由→服务→存储分层
- **类型安全**: 全链路 TypeScript 类型约束
- **错误优先**: 完善的错误处理和用户友好提示
- **无状态设计**: 利用 KV 存储实现状态持久化

#### 性能优化策略
- **响应流**: 直接转发 Claude API 流式响应，减少延迟
- **智能缓存**: OAuth token 本地缓存，减少不必要的 API 调用
- **并发控制**: 异步处理和适当的超时控制

代码库设计重点关注简洁性、类型安全和 Cloudflare 平台深度集成，为 Claude Code 提供稳定可靠的 API 代理服务。

## 管理中心功能

### 功能概述

管理中心提供了一个完整的 Web 界面来管理 Claude Code 的配置、第三方模型供应商和多个 Claude Code 账号。

#### 核心功能
- **🔐 管理员认证**: 基于环境变量的简单认证机制
- **📊 系统仪表板**: 显示 Claude 账号状态、供应商统计和 Claude Code 账号统计
- **👥 Claude Code 账号管理**: 添加、删除和管理多个 Claude Code 账号，支持 OAuth 认证
- **🔧 模型供应商管理**: 添加、删除第三方 AI 模型供应商（魔搭、智谱 AI 等）
- **🎯 模型选择**: 在官方 Claude 和第三方模型间切换默认使用的模型

#### 预设供应商支持
- **魔搭 Qwen**: 阿里云魔搭社区的 Qwen 系列模型
- **智谱 AI**: GLM-4 等先进语言模型  
- **OpenAI Compatible**: 兼容 OpenAI API 的其他服务

### 访问方式

#### 开发环境
- 前端本地开发: `http://localhost:3000/admin`
- 后端本地开发: `http://localhost:8787` (Wrangler 默认端口)
- 本地开发时，前端自动使用 `http://localhost:8787` 作为 API 基础 URL

#### 生产环境
- 管理中心: `https://claude-relay-frontend.pages.dev/admin`
- 默认登录凭据: `admin` / `password123`

### Claude Code 账号管理功能

#### OAuth 认证流程
1. **添加账号**: 在管理中心点击"添加账号"，输入账号名称和描述
2. **OAuth 授权**: 点击"授权登录"，系统生成授权链接
3. **完成认证**: 在新窗口中完成 Claude Code OAuth 授权
4. **输入授权码**: 从浏览器地址栏复制授权码并粘贴
5. **自动管理**: 系统自动管理 token 生命周期和刷新

#### 账号状态监控
- **活跃状态**: Token 有效，可正常使用
- **过期状态**: Token 已过期，需要刷新
- **未授权状态**: 尚未完成 OAuth 认证

#### 数据存储
- **加密存储**: OAuth token 使用 AES 加密存储在 KV
- **临时数据**: PKCE 参数临时存储，10分钟自动过期
- **状态同步**: 前后端实时同步账号状态

### 页面结构

```
/                         # 首页（自动重定向到管理中心）
/admin                    # 登录页面
├── /admin/dashboard      # 主仪表板
│   ├── Claude 账号标签页   # 管理多个 Claude Code 账号（默认页面）
│   ├── 模型供应商标签页    # 管理第三方AI模型供应商
│   └── 模型选择标签页      # 选择默认使用的模型
└── /admin/add-provider   # 添加供应商页面
```

### 文件组织

#### 前端文件
```
packages/frontend/pages/admin/
├── index.vue           # 登录页面
├── dashboard.vue       # 主仪表板  
└── add-provider.vue    # 添加供应商页面
```

#### 后端文件
```
packages/backend/src/
├── routes/admin.ts     # 管理 API 路由
└── services/admin.ts   # 管理服务实现
```

#### 共享类型
```
shared/
├── types/admin.ts      # 管理中心类型定义
└── constants/admin.ts  # 管理中心常量配置
```

### 部署注意事项

1. **环境变量配置**: 确保在 `wrangler.toml` 中正确设置 `ADMIN_USERNAME` 和 `ADMIN_PASSWORD`
2. **安全考虑**: 生产环境中应修改默认密码
3. **KV 存储**: 管理中心数据存储在与 Claude token 相同的 KV namespace 中

### 开发建议

- 管理中心采用简化设计，专注核心功能
- 认证机制简单但安全，基于环境变量验证
- UI 设计复用原型，保持与项目整体风格一致
- API 设计遵循现有模式，确保类型安全
- Claude Code 账号管理集成完整的 OAuth 2.0 PKCE 流程
- 支持多账号管理，方便大型团队使用