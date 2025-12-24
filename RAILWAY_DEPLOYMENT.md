# Railway 部署指南（使用 Railpack）

Railway 现在使用 **Railpack** 作为默认构建系统，自动检测语言并零配置部署。

## 部署架构

本项目采用**前后端分离部署**：
- **后端服务**：Python FastAPI（自动检测 `pyproject.toml`）
- **前端服务**：Bun + Vite（自动检测 `package.json`）
- **通信方式**：前端通过环境变量配置后端 API URL

## 快速开始

### 1. 推送代码到 GitHub

```bash
git add .
git commit -m "Add Railpack configuration"
git push origin main
```

### 2. 部署后端服务

1. 访问 [railway.app](https://railway.app) 并登录
2. 点击 **New Project** → **Deploy from GitHub repo**
3. 选择你的仓库
4. Railway 会自动检测到 Python 项目
5. 设置 **Root Directory** 为 `backend`
6. 配置环境变量（Settings → Variables）：
   - `OPENAI_API_KEY`（必需）- OpenAI API 密钥
   - `OPENAI_BASE_URL`（可选）- 自定义 API 地址
   - `CORS_ORIGINS=*`（推荐）- 允许跨域访问
7. 部署完成后，复制后端 URL（如 `https://backend-xxx.up.railway.app`）

### 3. 部署前端服务

1. 在同一个 Railway 项目中，点击 **New Service** → **GitHub Repo**
2. 选择相同的仓库
3. 设置 **Root Directory** 为 `frontend`
4. Railway 会自动检测到 Node.js 项目
5. 配置环境变量：
   - `VITE_API_URL=<后端URL>`（从步骤 2.7 获取）
6. 部署完成

### 4. 访问应用

前端部署完成后，访问 Railway 提供的前端 URL 即可使用应用。

## 配置文件说明

### `backend/railpack.json`
```json
{
  "$schema": "https://schema.railpack.com",
  "deploy": {
    "startCommand": "uv run uvicorn app.main:app --host 0.0.0.0 --port $PORT"
  }
}
```
- Railpack 自动检测 Python 和 uv
- 自定义启动命令使用 uvicorn

### `frontend/railpack.json`
```json
{
  "$schema": "https://schema.railpack.com",
  "steps": {
    "install": {
      "commands": ["bun install"]
    },
    "build": {
      "inputs": [{ "step": "install" }],
      "commands": ["bun run build"]
    }
  },
  "deploy": {
    "startCommand": "bun run preview -- --host 0.0.0.0 --port $PORT"
  }
}
```
- 使用 **bun** 作为包管理器和运行时
- 安装步骤：`bun install`
- 构建步骤：`bun run build`
- 启动命令：使用 Vite preview 服务器

## 环境变量配置

### 后端服务
| 变量名 | 必需 | 说明 | 默认值 |
|--------|------|------|--------|
| `OPENAI_API_KEY` | ✅ | OpenAI API 密钥 | - |
| `OPENAI_BASE_URL` | ❌ | 自定义 API 地址 | - |
| `CORS_ORIGINS` | ❌ | CORS 允许源 | `*` |
| `LOG_LEVEL` | ❌ | 日志级别 | `INFO` |
| `DB_QUERY_DATA_DIR` | ❌ | 数据目录 | `~/.db_query` |

### 前端服务
| 变量名 | 必需 | 说明 |
|--------|------|------|
| `VITE_API_URL` | ✅ | 后端 API 完整 URL |

## Railpack 特性

### 零配置部署
Railpack 自动检测：
- **Python**：检测 `pyproject.toml`、`requirements.txt` 或 `Pipfile`
- **Bun/Node.js**：检测 `package.json`
- **依赖管理**：自动识别 uv、pip、bun、npm、yarn 等

### 自动优化
- 智能缓存依赖
- 多阶段构建
- 最小化镜像大小

### 健康检查
Railway 自动配置健康检查：
- 后端：`/health` 端点
- 前端：HTTP 200 响应

## 故障排除

### 后端构建失败
1. 检查 `backend/pyproject.toml` 是否存在
2. 确认 Python 版本兼容（需要 3.12）
3. 查看 Railway 构建日志中的错误
4. 验证 `uv.lock` 文件是否最新

### 前端构建失败
1. 检查 `frontend/package.json` 是否正确
2. 确认所有依赖都已声明
3. 本地运行 `npm run build` 测试
4. 查看构建日志中的具体错误

### API 调用失败
1. 确认 `VITE_API_URL` 设置正确（包含 `https://`）
2. 检查后端 `CORS_ORIGINS` 配置
3. 在浏览器开发者工具查看网络请求
4. 验证后端服务是否正常运行

### 数据库连接问题
1. 确认数据库 URL 格式：
   ```
   postgresql://user:pass@host:port/db?sslmode=disable
   ```
2. 检查数据库防火墙规则
3. 验证数据库凭据是否正确
4. 查看后端日志中的连接错误

## 本地开发

### 使用 Makefile
```bash
# 启动后端（终端 1）
make dev-backend

# 启动前端（终端 2）
make dev-frontend
```

### 手动启动
```bash
# 后端
cd backend
uv run uvicorn app.main:app --reload --port 8000

# 前端
cd frontend
bun run dev
```

访问 http://localhost:5173 查看应用。

## 更新部署

推送代码后 Railway 自动重新部署：

```bash
git add .
git commit -m "Update application"
git push origin main
```

Railway 会检测更改并自动重新构建和部署相应的服务。

## 监控和日志

### 查看日志
在 Railway Dashboard 中：
1. 选择服务（backend 或 frontend）
2. 点击 **Logs** 标签
3. 实时查看应用日志

### 监控指标
Railway 自动提供：
- CPU 使用率
- 内存使用率
- 网络流量
- 请求响应时间

## 扩展和优化

### 自动扩展
Railway 根据负载自动扩展服务。

### 自定义域名
在 Service Settings → Domains 中添加自定义域名。

### 环境变量管理
使用 Railway 的 Variables 功能管理不同环境的配置。

## 从 Nixpacks 迁移

如果你的服务使用旧的 Nixpacks：
1. 在 Service Settings 中选择 **Railpack**
2. 删除 `nixpacks.toml`（如果存在）
3. 添加 `railpack.json`（可选，用于自定义配置）
4. 重新部署

## 支持

- [Railway 文档](https://docs.railway.com)
- [Railpack 文档](https://railpack.com)
- [Railway Discord](https://discord.gg/railway)
- [GitHub Issues](https://github.com/railwayapp/railpack)
