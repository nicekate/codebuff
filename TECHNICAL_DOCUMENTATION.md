# Codebuff 技术文档

## 目录
- [项目概述](#项目概述)
- [技术栈](#技术栈)
- [系统架构](#系统架构)
- [目录结构](#目录结构)
- [安装和运行指南](#安装和运行指南)
- [核心功能模块](#核心功能模块)
- [数据流程](#数据流程)
- [API接口文档](#api接口文档)
- [数据库设计](#数据库设计)
- [配置文件说明](#配置文件说明)
- [开发指南](#开发指南)
- [常见问题](#常见问题)

## 项目概述

Codebuff 是一个基于多智能体架构的AI编程助手，通过自然语言指令来编辑代码库。与传统的单一模型工具不同，Codebuff 协调多个专门的智能体协同工作，以理解项目并进行精确的代码修改。

### 核心特性
- **多智能体协作**：文件探索器、规划器、实现器、审查器等专门智能体
- **深度定制化**：支持TypeScript生成器创建复杂的智能体工作流
- **多模型支持**：通过OpenRouter支持Claude、GPT、Qwen、DeepSeek等多种模型
- **完整SDK**：提供TypeScript SDK用于生产环境集成
- **智能体市场**：可重用和发布自定义智能体

### 工作原理
当用户请求"为我的API添加身份验证"时，Codebuff可能会调用：
1. **文件探索器智能体** - 扫描代码库了解架构并找到相关文件
2. **规划器智能体** - 规划需要修改的文件和修改顺序
3. **实现智能体** - 进行精确的代码编辑
4. **审查智能体** - 验证修改的正确性

## 技术栈

### 前端技术栈
- **框架**: Next.js 14.2.13 (React 18)
- **样式**: Tailwind CSS + Radix UI组件
- **状态管理**: TanStack Query (React Query)
- **认证**: NextAuth.js 4.24.11
- **主题**: next-themes (支持暗黑模式)
- **动画**: Framer Motion
- **图标**: Lucide React
- **3D渲染**: React Three Fiber + Drei
- **内容管理**: Contentlayer (MDX支持)

### 后端技术栈
- **运行时**: Bun (主要) + Node.js
- **框架**: Express.js 4.19.2
- **WebSocket**: ws 8.18.0 (实时通信)
- **数据库**: PostgreSQL + Drizzle ORM
- **认证**: NextAuth.js + Drizzle适配器
- **AI模型**: 
  - Anthropic Claude (主要)
  - OpenAI GPT系列
  - Google Gemini
  - DeepSeek
  - 通过OpenRouter支持更多模型

### 数据库和存储
- **主数据库**: PostgreSQL
- **ORM**: Drizzle ORM
- **迁移**: Drizzle Kit
- **会话存储**: 数据库会话
- **文件存储**: 本地文件系统

### 开发工具
- **包管理**: Bun + pnpm (monorepo)
- **类型检查**: TypeScript 5.5.4
- **代码格式化**: Prettier 3.3.2
- **代码检查**: ESLint + TypeScript ESLint
- **测试**: Jest + Playwright
- **构建工具**: Next.js + Bun

## 系统架构

```mermaid
graph TB
    subgraph "客户端层"
        CLI[CLI工具<br/>npm-app]
        WEB[Web界面<br/>Next.js]
        SDK[SDK<br/>TypeScript]
    end
    
    subgraph "服务层"
        WS[WebSocket服务器<br/>Express + ws]
        API[REST API<br/>Express]
        AUTH[认证服务<br/>NextAuth.js]
    end
    
    subgraph "智能体层"
        AGENT_MGR[智能体管理器]
        FILE_EXPLORER[文件探索器]
        PLANNER[规划器]
        IMPLEMENTER[实现器]
        REVIEWER[审查器]
    end
    
    subgraph "工具层"
        TOOLS[工具处理器]
        FILE_OPS[文件操作]
        TERMINAL[终端命令]
        SEARCH[代码搜索]
        WEB_SEARCH[网络搜索]
    end
    
    subgraph "AI模型层"
        ANTHROPIC[Anthropic Claude]
        OPENAI[OpenAI GPT]
        GEMINI[Google Gemini]
        OPENROUTER[OpenRouter<br/>其他模型]
    end
    
    subgraph "数据层"
        DB[(PostgreSQL<br/>用户/会话/消息)]
        FILES[本地文件系统]
    end
    
    CLI --> WS
    WEB --> API
    WEB --> WS
    SDK --> WS
    
    WS --> AGENT_MGR
    API --> AUTH
    AUTH --> DB
    
    AGENT_MGR --> FILE_EXPLORER
    AGENT_MGR --> PLANNER
    AGENT_MGR --> IMPLEMENTER
    AGENT_MGR --> REVIEWER
    
    FILE_EXPLORER --> TOOLS
    PLANNER --> TOOLS
    IMPLEMENTER --> TOOLS
    REVIEWER --> TOOLS
    
    TOOLS --> FILE_OPS
    TOOLS --> TERMINAL
    TOOLS --> SEARCH
    TOOLS --> WEB_SEARCH
    
    AGENT_MGR --> ANTHROPIC
    AGENT_MGR --> OPENAI
    AGENT_MGR --> GEMINI
    AGENT_MGR --> OPENROUTER
    
    WS --> DB
    FILE_OPS --> FILES
```

## 目录结构

```
codebuff/
├── .agents/                    # 智能体定义文件
│   ├── factory/               # 智能体工厂
│   └── *.ts                   # 自定义智能体
├── backend/                   # 后端服务
│   ├── src/
│   │   ├── tools/            # 工具处理器
│   │   ├── templates/        # 智能体模板
│   │   ├── websockets/       # WebSocket处理
│   │   ├── llm-apis/         # LLM API集成
│   │   └── index.ts          # 服务器入口
│   └── package.json
├── web/                       # Web前端
│   ├── src/
│   │   ├── app/              # Next.js App Router
│   │   ├── components/       # React组件
│   │   ├── lib/              # 工具库
│   │   └── styles/           # 样式文件
│   └── package.json
├── npm-app/                   # CLI应用
│   ├── src/
│   │   ├── cli-handlers/     # CLI命令处理
│   │   ├── tool-handlers.ts  # 工具处理器
│   │   └── index.ts          # CLI入口
│   └── package.json
├── sdk/                       # TypeScript SDK
│   ├── src/
│   │   ├── client.ts         # SDK客户端
│   │   └── tools/            # SDK工具
│   └── package.json
├── common/                    # 共享代码
│   ├── src/
│   │   ├── db/               # 数据库模式
│   │   ├── types/            # 类型定义
│   │   └── util/             # 工具函数
│   └── package.json
├── packages/                  # 内部包
│   ├── bigquery/             # BigQuery集成
│   ├── billing/              # 计费系统
│   └── internal/             # 内部工具
└── scripts/                   # 构建和部署脚本
```

## 安装和运行指南

### 环境要求
- **Node.js**: >= 20.0.0
- **Bun**: >= 1.2.11 (推荐)
- **PostgreSQL**: >= 14
- **Git**: 最新版本

### 依赖安装

1. **克隆仓库**
```bash
git clone https://github.com/your-org/codebuff.git
cd codebuff
```

2. **安装依赖**
```bash
# 使用Bun安装所有工作区依赖
bun install

# 或使用pnpm
pnpm install
```

3. **环境配置**
```bash
# 复制环境变量模板
cp .env.example .env.local

# 编辑环境变量
vim .env.local
```

### 必需的环境变量
```bash
# 数据库
DATABASE_URL="postgresql://user:password@localhost:5432/codebuff"

# AI模型API密钥
ANTHROPIC_API_KEY="your-anthropic-key"
OPENAI_API_KEY="your-openai-key"
GEMINI_API_KEY="your-gemini-key"
OPEN_ROUTER_API_KEY="your-openrouter-key"

# 认证
NEXTAUTH_SECRET="your-nextauth-secret"
NEXTAUTH_URL="http://localhost:3000"

# GitHub OAuth (可选)
CODEBUFF_GITHUB_ID="your-github-client-id"
CODEBUFF_GITHUB_SECRET="your-github-client-secret"

# 服务端口
PORT=8080
NEXT_PUBLIC_WEB_PORT=3000
```

### 数据库设置

1. **启动PostgreSQL**
```bash
# 使用Docker启动PostgreSQL
bun run start-db

# 或手动启动本地PostgreSQL服务
```

2. **运行数据库迁移**
```bash
cd common
bun run db:migrate
```

3. **查看数据库(可选)**
```bash
bun run start-studio
```

### 启动服务

1. **开发模式启动所有服务**
```bash
bun run dev
```

2. **分别启动各个服务**
```bash
# 启动后端服务器
bun run start-server

# 启动Web前端
bun run start-web

# 启动CLI工具
bun run start-bin
```

### CLI工具使用

1. **全局安装CLI**
```bash
npm install -g codebuff
```

2. **在项目中使用**
```bash
cd your-project
codebuff
```

3. **初始化智能体**
```bash
codebuff init-agents
```

### SDK使用示例

```typescript
import { CodebuffClient } from '@codebuff/sdk'

const client = new CodebuffClient({
  apiKey: 'your-api-key',
  cwd: '/path/to/your/project',
  onError: (error) => console.error('Codebuff error:', error.message),
})

const result = await client.run({
  agent: 'base',
  prompt: 'Add comprehensive error handling to all API endpoints',
  handleEvent: (event) => {
    console.log('Progress', event)
  },
})
```

## 核心功能模块

### 智能体系统

Codebuff的核心是多智能体协作系统，每个智能体都有特定的职责：

#### 基础智能体类型

1. **文件探索器 (File Explorer)**
   - 扫描代码库结构
   - 理解项目架构
   - 识别相关文件和依赖关系

2. **文件选择器 (File Picker)**
   - 根据特定需求选择文件
   - 支持并行文件搜索
   - 提供精确的文件匹配

3. **规划器 (Planner)**
   - 分析用户需求
   - 制定修改计划
   - 确定修改顺序和依赖关系

4. **实现器 (Implementer)**
   - 执行具体的代码修改
   - 处理文件读写操作
   - 确保代码语法正确

5. **审查器 (Reviewer)**
   - 验证代码修改
   - 检查潜在问题
   - 确保修改符合最佳实践

#### 智能体定义示例

```typescript
// .agents/custom-agent.ts
export default {
  id: 'custom-agent',
  displayName: 'Custom Agent',
  model: 'openai/gpt-4',
  toolNames: ['read_files', 'write_file', 'run_terminal_command'],

  instructionsPrompt: 'You are a specialized agent for...',

  async *handleSteps({ prompt, params }) {
    // 分析需求
    yield { tool: 'read_files', paths: ['package.json'] }

    // 执行操作
    yield { tool: 'write_file', path: 'new-file.ts', content: '...' }

    // 验证结果
    yield { tool: 'run_terminal_command', command: 'npm test' }
  },
}
```

### 工具系统

Codebuff提供丰富的工具集，支持各种代码操作：

#### 核心工具

1. **文件操作工具**
   - `read_files`: 读取文件内容
   - `write_file`: 写入新文件
   - `str_replace`: 字符串替换编辑

2. **代码分析工具**
   - `code_search`: 代码搜索和模式匹配
   - `find_files`: 智能文件查找

3. **执行工具**
   - `run_terminal_command`: 执行终端命令
   - `run_file_change_hooks`: 运行文件变更钩子

4. **智能体协作工具**
   - `spawn_agents`: 并行生成多个智能体
   - `spawn_agent_inline`: 内联生成智能体

5. **信息工具**
   - `web_search`: 网络搜索
   - `read_docs`: 读取文档

#### 工具处理流程

```mermaid
graph LR
    A[智能体请求工具] --> B[工具调用解析]
    B --> C[权限验证]
    C --> D[工具执行]
    D --> E[结果处理]
    E --> F[返回智能体]

    D --> G[客户端工具]
    D --> H[服务端工具]

    G --> I[文件操作]
    G --> J[终端命令]

    H --> K[AI模型调用]
    H --> L[数据库操作]
```

### WebSocket通信系统

实时通信是Codebuff的核心特性，通过WebSocket实现：

#### 通信协议

```typescript
// 客户端 -> 服务端
interface ClientMessage {
  type: 'user_input' | 'tool_result' | 'file_context'
  data: any
  userInputId: string
  clientSessionId: string
}

// 服务端 -> 客户端
interface ServerMessage {
  type: 'response_chunk' | 'tool_call' | 'agent_status'
  data: any
  userInputId: string
}
```

#### 连接管理

```typescript
// 连接建立
const ws = new WebSocket('ws://localhost:8080')

// 消息处理
ws.on('message', (data) => {
  const message = JSON.parse(data.toString())
  handleServerMessage(message)
})

// 发送用户输入
ws.send(JSON.stringify({
  type: 'user_input',
  data: { prompt: 'Add authentication to my API' },
  userInputId: generateId(),
  clientSessionId: sessionId
}))
```

## 数据流程

### 用户请求处理流程

```mermaid
sequenceDiagram
    participant U as 用户
    participant C as 客户端
    participant W as WebSocket服务器
    participant A as 智能体管理器
    participant T as 工具处理器
    participant M as AI模型
    participant D as 数据库

    U->>C: 输入自然语言指令
    C->>W: 发送用户输入
    W->>A: 创建智能体会话
    A->>D: 记录会话开始

    loop 智能体步骤
        A->>M: 发送消息到AI模型
        M->>A: 返回工具调用
        A->>T: 执行工具调用
        T->>C: 请求客户端工具(如需要)
        C->>T: 返回工具结果
        T->>A: 返回工具结果
        A->>W: 发送响应块
        W->>C: 流式传输响应
        C->>U: 显示实时进度
    end

    A->>D: 记录会话完成
    W->>C: 发送最终结果
    C->>U: 显示完成状态
```

### 文件修改流程

```mermaid
graph TD
    A[用户请求修改] --> B[文件探索器分析]
    B --> C[规划器制定计划]
    C --> D[实现器执行修改]
    D --> E[审查器验证]
    E --> F{修改是否正确?}
    F -->|是| G[应用修改]
    F -->|否| H[回滚并重试]
    H --> D
    G --> I[运行测试]
    I --> J[完成]
```

## API接口文档

### REST API端点

#### 用户认证
```http
POST /api/auth/signin
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password"
}
```

#### 使用情况查询
```http
POST /api/usage
Authorization: Bearer <token>
Content-Type: application/json

{
  "fingerprintId": "client-fingerprint",
  "authToken": "user-auth-token",
  "orgId": "organization-id"
}
```

#### 智能体名称验证
```http
GET /api/agents/validate-name?name=my-agent
Authorization: Bearer <token>
```

#### 组织仓库覆盖检查
```http
POST /api/orgs/is-repo-covered
Authorization: Bearer <token>
Content-Type: application/json

{
  "repoUrl": "https://github.com/user/repo",
  "orgId": "organization-id"
}
```

### WebSocket API

#### 连接建立
```javascript
const ws = new WebSocket('ws://localhost:8080')

// 认证
ws.send(JSON.stringify({
  type: 'auth',
  token: 'your-auth-token'
}))
```

#### 发送用户输入
```javascript
ws.send(JSON.stringify({
  type: 'user_input',
  data: {
    prompt: 'Add error handling to the API',
    fileContext: {
      files: [...],
      projectRoot: '/path/to/project'
    }
  },
  userInputId: 'unique-request-id',
  clientSessionId: 'client-session-id'
}))
```

#### 接收响应
```javascript
ws.on('message', (data) => {
  const message = JSON.parse(data)

  switch (message.type) {
    case 'response_chunk':
      // 处理流式响应
      console.log(message.data.content)
      break

    case 'tool_call':
      // 处理工具调用请求
      handleToolCall(message.data)
      break

    case 'agent_status':
      // 处理智能体状态更新
      updateAgentStatus(message.data)
      break
  }
})
```

## 数据库设计

### 核心表结构

#### 用户表 (user)
```sql
CREATE TABLE user (
  id TEXT PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT,
  email TEXT UNIQUE NOT NULL,
  password TEXT,
  email_verified TIMESTAMP,
  image TEXT,
  stripe_customer_id TEXT UNIQUE,
  stripe_price_id TEXT,
  next_quota_reset TIMESTAMP DEFAULT (now() + INTERVAL '1 month'),
  created_at TIMESTAMP NOT NULL DEFAULT now(),
  referral_code TEXT UNIQUE DEFAULT ('ref-' || gen_random_uuid()),
  referral_limit INTEGER NOT NULL DEFAULT 5,
  discord_id TEXT UNIQUE,
  handle TEXT UNIQUE,
  auto_topup_enabled BOOLEAN NOT NULL DEFAULT false,
  auto_topup_threshold INTEGER,
  auto_topup_amount INTEGER
);
```

#### 组织表 (org)
```sql
CREATE TABLE org (
  id TEXT PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  description TEXT,
  owner_id TEXT NOT NULL REFERENCES user(id) ON DELETE CASCADE,
  stripe_customer_id TEXT UNIQUE,
  stripe_subscription_id TEXT,
  current_period_start TIMESTAMP WITH TIME ZONE,
  current_period_end TIMESTAMP WITH TIME ZONE,
  auto_topup_enabled BOOLEAN NOT NULL DEFAULT false,
  auto_topup_threshold INTEGER NOT NULL,
  auto_topup_amount INTEGER NOT NULL,
  credit_limit INTEGER,
  billing_alerts BOOLEAN NOT NULL DEFAULT true,
  usage_alerts BOOLEAN NOT NULL DEFAULT true,
  weekly_reports BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now()
);
```

#### 消息表 (message)
```sql
CREATE TABLE message (
  id TEXT PRIMARY KEY,
  finished_at TIMESTAMP NOT NULL,
  client_id TEXT NOT NULL,
  client_request_id TEXT NOT NULL,
  model TEXT NOT NULL,
  request JSONB NOT NULL,
  last_message JSONB GENERATED ALWAYS AS (request -> -1),
  response JSONB NOT NULL,
  input_tokens INTEGER NOT NULL DEFAULT 0,
  cache_creation_input_tokens INTEGER NOT NULL DEFAULT 0,
  cache_read_input_tokens INTEGER NOT NULL DEFAULT 0,
  output_tokens INTEGER NOT NULL,
  cost NUMERIC(100,20) NOT NULL,
  credits INTEGER NOT NULL,
  latency_ms INTEGER,
  user_id TEXT REFERENCES user(id) ON DELETE CASCADE,
  org_id TEXT REFERENCES org(id) ON DELETE CASCADE,
  repo_url TEXT
);

CREATE INDEX message_user_id_idx ON message(user_id);
CREATE INDEX message_finished_at_user_id_idx ON message(finished_at, user_id);
CREATE INDEX message_org_id_idx ON message(org_id);
CREATE INDEX message_org_id_finished_at_idx ON message(org_id, finished_at);
```

#### 智能体运行表 (agent_run)
```sql
CREATE TABLE agent_run (
  id TEXT PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id TEXT REFERENCES user(id) ON DELETE CASCADE,
  org_id TEXT REFERENCES org(id) ON DELETE CASCADE,
  agent_id TEXT NOT NULL,
  ancestor_run_ids TEXT[] DEFAULT '{}',
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now(),
  completed_at TIMESTAMP WITH TIME ZONE,
  duration_ms INTEGER GENERATED ALWAYS AS (
    CASE WHEN completed_at IS NOT NULL
    THEN EXTRACT(EPOCH FROM (completed_at - created_at)) * 1000
    ELSE NULL END::integer
  ),
  total_steps INTEGER DEFAULT 0,
  direct_credits NUMERIC(10,6) DEFAULT '0',
  total_credits NUMERIC(10,6) DEFAULT '0'
);
```

### 数据关系图

```mermaid
erDiagram
    USER ||--o{ ORG_MEMBER : belongs_to
    USER ||--o{ MESSAGE : creates
    USER ||--o{ AGENT_RUN : executes
    USER ||--o{ CREDIT_LEDGER : has
    USER ||--o{ REFERRAL : refers

    ORG ||--o{ ORG_MEMBER : contains
    ORG ||--o{ ORG_REPO : manages
    ORG ||--o{ ORG_INVITE : sends
    ORG ||--o{ MESSAGE : uses
    ORG ||--o{ AGENT_RUN : executes

    AGENT_RUN ||--o{ AGENT_STEP : contains

    PUBLISHER ||--o{ AGENT_CONFIG : publishes

    USER {
        text id PK
        text email UK
        text name
        text stripe_customer_id UK
        timestamp created_at
        text referral_code UK
        boolean auto_topup_enabled
    }

    ORG {
        text id PK
        text slug UK
        text name
        text owner_id FK
        text stripe_customer_id UK
        boolean auto_topup_enabled
        timestamp created_at
    }

    MESSAGE {
        text id PK
        text user_id FK
        text org_id FK
        text model
        jsonb request
        jsonb response
        integer input_tokens
        integer output_tokens
        numeric cost
        integer credits
        timestamp finished_at
    }

    AGENT_RUN {
        text id PK
        text user_id FK
        text org_id FK
        text agent_id
        text[] ancestor_run_ids
        timestamp created_at
        timestamp completed_at
        integer total_steps
        numeric total_credits
    }
```

## 配置文件说明

### 环境变量配置

#### 核心环境变量 (.env.local)

```bash
# 数据库配置
DATABASE_URL="postgresql://username:password@localhost:5432/codebuff"

# AI模型API密钥
ANTHROPIC_API_KEY="sk-ant-api03-..."           # Claude模型
ANTHROPIC_API_KEY2="sk-ant-api03-..."          # 备用Claude密钥
OPENAI_API_KEY="sk-..."                        # OpenAI GPT模型
GEMINI_API_KEY="AIza..."                       # Google Gemini
DEEPSEEK_API_KEY="sk-..."                      # DeepSeek模型
OPEN_ROUTER_API_KEY="sk-or-v1-..."            # OpenRouter聚合服务
RELACE_API_KEY="..."                           # Relace AI服务
LINKUP_API_KEY="..."                           # Linkup服务

# 认证配置
NEXTAUTH_SECRET="your-super-secret-key"        # NextAuth.js密钥
NEXTAUTH_URL="http://localhost:3000"           # 应用URL

# GitHub OAuth (可选)
CODEBUFF_GITHUB_ID="your-github-client-id"
CODEBUFF_GITHUB_SECRET="your-github-client-secret"

# 服务端口
PORT=8080                                      # 后端服务端口
NEXT_PUBLIC_WEB_PORT=3000                     # 前端服务端口

# 分析和监控
HELICONE_API_KEY="sk-helicone-..."            # Helicone监控
GOOGLE_CLOUD_PROJECT_ID="your-gcp-project"    # Google Cloud项目

# 环境标识
NEXT_PUBLIC_CB_ENVIRONMENT="dev"              # 环境: dev/staging/prod
```

#### 生产环境额外配置

```bash
# Stripe支付
STRIPE_SECRET_KEY="sk_live_..."
STRIPE_WEBHOOK_SECRET="whsec_..."
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY="pk_live_..."

# 邮件服务
LOOPS_API_KEY="..."                           # Loops邮件服务

# 安全配置
ENCRYPTION_KEY="32-character-encryption-key"   # 数据加密密钥
```

### 包配置文件

#### 根目录 package.json
```json
{
  "name": "codebuff-project",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "workspaces": [
    "common",
    "backend",
    "npm-app",
    "web",
    "packages/*",
    "scripts",
    "evals",
    "sdk",
    ".agents"
  ],
  "scripts": {
    "dev": "bash scripts/dev.sh",
    "start-db": "bun --cwd common db:start",
    "start-web": "bun start-db && bun --cwd web dev",
    "start-server": "bun --cwd backend dev",
    "typecheck": "bun --filter='*' run typecheck",
    "test": "bun --filter='{@codebuff/backend,@codebuff/common,@codebuff/npm-app}' run test"
  }
}
```

#### TypeScript配置 (tsconfig.json)
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["dom", "dom.iterable", "es6"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@codebuff/*": ["./packages/*/src", "./*/src"]
    }
  },
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts"
  ],
  "exclude": ["node_modules"]
}
```

#### Bun配置 (bunfig.toml)
```toml
[install]
# 使用pnpm风格的node_modules结构
auto = "bun"
exact = true
production = false

[install.cache]
# 缓存配置
dir = "~/.bun/install/cache"

[run]
# 运行时配置
bun = true
```

### 数据库配置

#### Drizzle配置 (common/src/db/drizzle.config.ts)
```typescript
import { defineConfig } from 'drizzle-kit'
import { env } from '@codebuff/internal'

export default defineConfig({
  dialect: 'postgresql',
  schema: './src/db/schema.ts',
  out: './src/db/migrations',
  dbCredentials: {
    url: env.DATABASE_URL,
  },
  tablesFilter: ['*', '!pg_stat_statements', '!pg_stat_statements_info'],
})
```

### Next.js配置

#### next.config.mjs
```javascript
import createMDX from '@next/mdx'
import { withContentlayer } from 'next-contentlayer'

const withMDX = createMDX({
  extension: /\.mdx?$/,
  options: {
    remarkPlugins: [],
    rehypePlugins: [],
  },
})

const nextConfig = {
  eslint: {
    ignoreDuringBuilds: true,
  },
  webpack: (config) => {
    config.resolve.fallback = { fs: false, net: false, tls: false }
    config.externals.push(
      { 'thread-stream': 'commonjs thread-stream', pino: 'commonjs pino' },
      'pino-pretty',
      'encoding'
    )
    return config
  },
  experimental: {
    serverComponentsExternalPackages: ['pino', 'pino-pretty'],
  },
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '**',
      },
    ],
  },
}

export default withContentlayer(withMDX(nextConfig))
```

## 开发指南

### 开发环境设置

#### 1. 代码编辑器配置

**VS Code推荐扩展:**
```json
{
  "recommendations": [
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next",
    "unifiedjs.vscode-mdx",
    "ms-playwright.playwright"
  ]
}
```

**VS Code设置 (.vscode/settings.json):**
```json
{
  "typescript.preferences.includePackageJsonAutoImports": "on",
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "files.associations": {
    "*.mdx": "mdx"
  }
}
```

#### 2. Git钩子设置

```bash
# 安装husky
bun add --dev husky

# 设置pre-commit钩子
echo "bun run typecheck && bun run test" > .husky/pre-commit
chmod +x .husky/pre-commit
```

### 代码规范

#### TypeScript规范

1. **严格类型检查**
```typescript
// ✅ 好的做法
interface UserData {
  id: string
  email: string
  name?: string
}

function createUser(data: UserData): Promise<User> {
  return userService.create(data)
}

// ❌ 避免使用any
function badFunction(data: any): any {
  return data.whatever
}
```

2. **使用类型断言谨慎**
```typescript
// ✅ 使用类型守卫
function isUser(obj: unknown): obj is User {
  return typeof obj === 'object' && obj !== null && 'id' in obj
}

// ✅ 使用zod验证
const userSchema = z.object({
  id: z.string(),
  email: z.string().email(),
})

// ❌ 避免强制类型断言
const user = data as User // 危险
```

#### React组件规范

1. **组件结构**
```typescript
// ✅ 推荐的组件结构
interface Props {
  title: string
  onSubmit: (data: FormData) => void
  children?: React.ReactNode
}

export function MyComponent({ title, onSubmit, children }: Props) {
  const [loading, setLoading] = useState(false)

  const handleSubmit = useCallback(async (data: FormData) => {
    setLoading(true)
    try {
      await onSubmit(data)
    } finally {
      setLoading(false)
    }
  }, [onSubmit])

  return (
    <div className="p-4">
      <h1>{title}</h1>
      {children}
    </div>
  )
}
```

2. **Hooks使用**
```typescript
// ✅ 自定义Hook
function useUserData(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    enabled: !!userId,
  })
}

// ✅ 使用useCallback优化性能
const memoizedCallback = useCallback(
  (value: string) => {
    doSomething(value)
  },
  [dependency]
)
```

### 测试指南

#### 单元测试

```typescript
// __tests__/utils.test.ts
import { describe, it, expect } from '@jest/globals'
import { formatCurrency } from '../utils'

describe('formatCurrency', () => {
  it('should format USD currency correctly', () => {
    expect(formatCurrency(1234.56, 'USD')).toBe('$1,234.56')
  })

  it('should handle zero values', () => {
    expect(formatCurrency(0, 'USD')).toBe('$0.00')
  })
})
```

#### 集成测试

```typescript
// __tests__/api.test.ts
import { describe, it, expect, beforeEach } from '@jest/globals'
import { testClient } from '../test-utils'

describe('API Integration', () => {
  beforeEach(async () => {
    await setupTestDatabase()
  })

  it('should create user successfully', async () => {
    const response = await testClient.post('/api/users', {
      email: 'test@example.com',
      name: 'Test User'
    })

    expect(response.status).toBe(201)
    expect(response.data.email).toBe('test@example.com')
  })
})
```

#### E2E测试 (Playwright)

```typescript
// __tests__/e2e/auth.spec.ts
import { test, expect } from '@playwright/test'

test('user can sign in', async ({ page }) => {
  await page.goto('/login')

  await page.fill('[data-testid="email"]', 'user@example.com')
  await page.fill('[data-testid="password"]', 'password')
  await page.click('[data-testid="submit"]')

  await expect(page).toHaveURL('/dashboard')
  await expect(page.locator('[data-testid="user-menu"]')).toBeVisible()
})
```

### 性能优化

#### 前端优化

1. **代码分割**
```typescript
// 动态导入大型组件
const HeavyComponent = lazy(() => import('./HeavyComponent'))

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  )
}
```

2. **图片优化**
```typescript
import Image from 'next/image'

function OptimizedImage() {
  return (
    <Image
      src="/hero.jpg"
      alt="Hero image"
      width={800}
      height={600}
      priority // 关键图片
      placeholder="blur" // 模糊占位符
    />
  )
}
```

#### 后端优化

1. **数据库查询优化**
```typescript
// ✅ 使用索引和限制
const users = await db
  .select()
  .from(userTable)
  .where(eq(userTable.email, email))
  .limit(1)

// ✅ 批量操作
const users = await db
  .insert(userTable)
  .values(userData)
  .returning()
```

2. **缓存策略**
```typescript
// Redis缓存示例
async function getCachedUser(userId: string): Promise<User | null> {
  const cached = await redis.get(`user:${userId}`)
  if (cached) {
    return JSON.parse(cached)
  }

  const user = await db.query.user.findFirst({
    where: eq(userTable.id, userId)
  })

  if (user) {
    await redis.setex(`user:${userId}`, 3600, JSON.stringify(user))
  }

  return user
}
```

### 部署指南

#### Docker部署

```dockerfile
# Dockerfile
FROM oven/bun:1 as base
WORKDIR /app

# 安装依赖
COPY package.json bun.lockb ./
RUN bun install --frozen-lockfile

# 构建应用
COPY . .
RUN bun run build

# 生产环境
FROM oven/bun:1-slim
WORKDIR /app
COPY --from=base /app .

EXPOSE 3000
CMD ["bun", "start"]
```

#### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/codebuff
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: codebuff
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

#### 环境变量管理

```bash
# 生产环境部署脚本
#!/bin/bash

# 设置环境变量
export NODE_ENV=production
export DATABASE_URL=$PROD_DATABASE_URL
export NEXTAUTH_SECRET=$PROD_NEXTAUTH_SECRET

# 构建和启动
bun run build
bun run start
```

## 常见问题和故障排除

### 安装和启动问题

#### Q: 安装依赖时出现权限错误
```bash
# 错误信息
EACCES: permission denied, mkdir '/usr/local/lib/node_modules'

# 解决方案
# 1. 使用nvm管理Node.js版本
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 20
nvm use 20

# 2. 或者使用Bun (推荐)
curl -fsSL https://bun.sh/install | bash
```

#### Q: 数据库连接失败
```bash
# 错误信息
Error: connect ECONNREFUSED 127.0.0.1:5432

# 解决方案
# 1. 检查PostgreSQL是否运行
sudo systemctl status postgresql

# 2. 启动PostgreSQL
sudo systemctl start postgresql

# 3. 检查连接字符串
echo $DATABASE_URL

# 4. 测试连接
psql $DATABASE_URL
```

#### Q: 端口被占用
```bash
# 错误信息
Error: listen EADDRINUSE: address already in use :::3000

# 解决方案
# 1. 查找占用端口的进程
lsof -i :3000

# 2. 终止进程
kill -9 <PID>

# 3. 或者使用不同端口
NEXT_PUBLIC_WEB_PORT=3001 bun run start-web
```

### 开发环境问题

#### Q: TypeScript类型错误
```typescript
// 错误: Property 'xxx' does not exist on type 'yyy'

// 解决方案
// 1. 更新类型定义
bun add --dev @types/node@latest

// 2. 重启TypeScript服务器 (VS Code)
Ctrl+Shift+P -> "TypeScript: Restart TS Server"

// 3. 清理构建缓存
rm -rf .next node_modules/.cache
bun install
```

#### Q: 热重载不工作
```bash
# 解决方案
# 1. 检查文件监听限制 (Linux)
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# 2. 重启开发服务器
bun run dev

# 3. 清理Next.js缓存
rm -rf .next
```

#### Q: 环境变量未加载
```bash
# 检查环境变量文件
ls -la .env*

# 确保文件名正确
.env.local          # 本地开发
.env.development    # 开发环境
.env.production     # 生产环境

# 重启服务器以加载新的环境变量
```

### 运行时问题

#### Q: WebSocket连接失败
```javascript
// 错误信息
WebSocket connection failed

// 解决方案
// 1. 检查后端服务是否运行
curl http://localhost:8080/healthz

// 2. 检查防火墙设置
sudo ufw status

// 3. 检查WebSocket URL配置
console.log(process.env.NEXT_PUBLIC_WS_URL)
```

#### Q: AI模型API调用失败
```bash
# 错误信息
API key invalid or expired

# 解决方案
# 1. 检查API密钥
echo $ANTHROPIC_API_KEY | head -c 20

# 2. 验证API密钥有效性
curl -H "Authorization: Bearer $ANTHROPIC_API_KEY" \
     https://api.anthropic.com/v1/messages

# 3. 检查配额和计费
# 登录相应的AI服务提供商控制台检查
```

#### Q: 文件操作权限错误
```bash
# 错误信息
EACCES: permission denied, open '/path/to/file'

# 解决方案
# 1. 检查文件权限
ls -la /path/to/file

# 2. 修改权限
chmod 644 /path/to/file

# 3. 检查目录权限
chmod 755 /path/to/directory
```

### 性能问题

#### Q: 应用响应缓慢
```bash
# 诊断步骤
# 1. 检查内存使用
free -h
top

# 2. 检查数据库性能
# 在PostgreSQL中执行
EXPLAIN ANALYZE SELECT * FROM message WHERE user_id = 'xxx';

# 3. 检查网络延迟
ping api.anthropic.com

# 4. 启用性能监控
# 在代码中添加性能标记
console.time('operation')
// ... 操作代码
console.timeEnd('operation')
```

#### Q: 数据库查询慢
```sql
-- 检查慢查询
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- 添加索引
CREATE INDEX CONCURRENTLY idx_message_user_created
ON message(user_id, created_at);

-- 分析表统计信息
ANALYZE message;
```

### 部署问题

#### Q: Docker构建失败
```dockerfile
# 常见问题: 依赖安装失败
# 解决方案: 使用多阶段构建和缓存

FROM node:20-alpine as deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine as builder
WORKDIR /app
COPY . .
COPY --from=deps /app/node_modules ./node_modules
RUN npm run build

FROM node:20-alpine as runner
WORKDIR /app
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=deps /app/node_modules ./node_modules
COPY package*.json ./

CMD ["npm", "start"]
```

#### Q: 生产环境环境变量问题
```bash
# 检查环境变量是否正确设置
printenv | grep -E "(DATABASE_URL|NEXTAUTH_SECRET|ANTHROPIC_API_KEY)"

# 使用Docker secrets (推荐)
echo "your-secret" | docker secret create db_password -

# 在docker-compose.yml中使用
services:
  app:
    secrets:
      - db_password
    environment:
      - DATABASE_PASSWORD_FILE=/run/secrets/db_password
```

### 调试技巧

#### 1. 启用详细日志
```typescript
// 在开发环境启用调试日志
if (process.env.NODE_ENV === 'development') {
  console.log('Debug info:', { userId, sessionId, timestamp: new Date() })
}

// 使用结构化日志
import { logger } from './util/logger'

logger.info('User action', {
  userId,
  action: 'file_read',
  filename: 'example.ts',
  timestamp: new Date().toISOString()
})
```

#### 2. 使用浏览器开发工具
```javascript
// 在浏览器控制台中调试WebSocket
const ws = new WebSocket('ws://localhost:8080')
ws.onmessage = (event) => {
  console.log('Received:', JSON.parse(event.data))
}
ws.onerror = (error) => {
  console.error('WebSocket error:', error)
}
```

#### 3. 数据库调试
```sql
-- 启用查询日志
ALTER SYSTEM SET log_statement = 'all';
SELECT pg_reload_conf();

-- 查看活动连接
SELECT pid, usename, application_name, client_addr, state, query
FROM pg_stat_activity
WHERE state = 'active';
```

### 监控和维护

#### 健康检查端点
```typescript
// backend/src/health.ts
export async function healthCheck() {
  const checks = {
    database: await checkDatabase(),
    redis: await checkRedis(),
    ai_apis: await checkAIAPIs(),
    timestamp: new Date().toISOString()
  }

  const isHealthy = Object.values(checks).every(check =>
    typeof check === 'boolean' ? check : check.status === 'ok'
  )

  return { healthy: isHealthy, checks }
}

async function checkDatabase() {
  try {
    await db.select().from(userTable).limit(1)
    return { status: 'ok' }
  } catch (error) {
    return { status: 'error', message: error.message }
  }
}
```

#### 性能监控
```typescript
// 添加性能指标收集
import { performance } from 'perf_hooks'

function measurePerformance<T>(
  name: string,
  fn: () => Promise<T>
): Promise<T> {
  return new Promise(async (resolve, reject) => {
    const start = performance.now()
    try {
      const result = await fn()
      const duration = performance.now() - start
      logger.info(`Performance: ${name}`, { duration })
      resolve(result)
    } catch (error) {
      const duration = performance.now() - start
      logger.error(`Performance: ${name} failed`, { duration, error })
      reject(error)
    }
  })
}
```

### 学习资源

#### 官方文档
- [Next.js文档](https://nextjs.org/docs)
- [Drizzle ORM文档](https://orm.drizzle.team/)
- [Bun文档](https://bun.sh/docs)
- [Anthropic API文档](https://docs.anthropic.com/)

#### 社区资源
- [Codebuff Discord社区](https://discord.gg/codebuff)
- [GitHub讨论区](https://github.com/codebuff/codebuff/discussions)
- [Stack Overflow标签](https://stackoverflow.com/questions/tagged/codebuff)

#### 相关技术学习
- [TypeScript深入理解](https://www.typescriptlang.org/docs/)
- [React最佳实践](https://react.dev/learn)
- [PostgreSQL性能调优](https://www.postgresql.org/docs/current/performance-tips.html)
- [WebSocket编程指南](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)

---

## 贡献指南

### 提交代码
1. Fork项目仓库
2. 创建功能分支: `git checkout -b feature/amazing-feature`
3. 提交更改: `git commit -m 'Add amazing feature'`
4. 推送分支: `git push origin feature/amazing-feature`
5. 创建Pull Request

### 报告问题
- 使用GitHub Issues报告bug
- 提供详细的复现步骤
- 包含环境信息和错误日志

### 功能请求
- 在GitHub Discussions中讨论新功能
- 提供详细的用例说明
- 考虑向后兼容性

---

*本文档持续更新中，如有问题请联系 [support@codebuff.com](mailto:support@codebuff.com)*
```
```
