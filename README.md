# URL Proxy

一个高性能的 HTTP 代理服务器,用 Go 语言编写,支持流式传输和连接复用

## 功能特性
- ✅ 可配置 Keep-Alive 连接复用
- ✅ 支持 HTTP 代理
- ✅ 完整的 CORS 支持
- ✅ 可配置超时和请求体大小限制

## 快速开始

### 1. 安装依赖

确保已安装 Go 1.16 或更高版本

### 2. 配置环境变量

复制 `.env` 文件并根据需要修改配置:

```env
# 服务端口
PORT=3000

# 目标 API 地址
API_URL=https://ai.megallm.io

# HTTP 代理(可选)
HTTP_PROXY=

# Keep-Alive 开关
# true  = 启用连接复用(高性能)
# false = 禁用连接复用(支持 IP 轮换)
ENABLE_KEEPALIVE=false

# 请求超时时间(秒,默认 300)
# TIMEOUT=300

# 最大请求体大小(MB,默认 100)
# MAX_BODY_SIZE=100
```

### 3. 运行服务

```bash
go run main.go
```

或编译后运行:

```bash
go build -o proxy main.go
./proxy
```

## 配置说明

### 环境变量

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `PORT` | 服务监听端口 | `8000` |
| `API_URL` | 目标 API 地址 | `https://api.openai.com` |
| `HTTP_PROXY` | HTTP 代理地址 | 无 |
| `ENABLE_KEEPALIVE` | 是否启用 Keep-Alive | `true` |
| `TIMEOUT` | 请求超时时间(秒) | `300` |
| `MAX_BODY_SIZE` | 最大请求体大小(MB) | `100` |

### Keep-Alive 说明

- **启用 Keep-Alive** (`ENABLE_KEEPALIVE=true`):
  - 连接复用,性能更高
  - 适合固定 IP 场景

- **禁用 Keep-Alive** (`ENABLE_KEEPALIVE=false`):
  - 每次请求新建连接
  - 支持代理 IP 轮换

## 使用示例

启动服务后,将反代URL替换原URl即可:

```bash
curl http://localhost:3000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "claude-sonnet-4-5-20250929",
    "messages": [
        {
            "role": "user",
             "content": "Hello"
        }
    ],
    "stream": true
  }'
```