# URL Proxy

一个高性能的 HTTP 代理服务器，用 Go 语言编写，支持多路径映射、流式传输和连接复用

## 功能特性
- ✅ 多路径前缀映射（支持代理多个不同的 API）
- ✅ 可配置 Keep-Alive 连接复用
- ✅ 完整的 CORS 支持
- ✅ 可配置超时和请求体大小限制

## 快速开始

### 方法一：Docker 部署（推荐）

```bash
docker run -d \
  -p 8000:8000 \
  -e API_PROXY_CONFIG='{"openai":"https://api.openai.com","anthropic":"https://api.anthropic.com"}' \
  -e ENABLE_KEEPALIVE=true \
  ghcr.io/rfym21/url-proxy:latest
```

或使用 docker-compose：

```bash
cd docker
docker-compose up -d
```

### 方法二：源码运行

#### 1. 安装依赖

确保已安装 Go 1.21 或更高版本

#### 2. 配置环境变量

创建 `.env` 文件并根据需要修改配置：

```env
# 服务端口
PORT=8000

# API 路径映射配置（JSON 格式）
# 格式：{"路径前缀":"目标API地址"}
API_PROXY_CONFIG={"openai":"https://api.openai.com","anthropic":"https://api.anthropic.com"}

# HTTP 上游代理（可选）
HTTP_PROXY=

# Keep-Alive 开关
# true  = 启用连接复用（高性能）
# false = 禁用连接复用（支持 IP 轮换）
ENABLE_KEEPALIVE=true

# 请求超时时间（秒，默认 300）
TIMEOUT=300

# 最大请求体大小（MB，默认 100）
MAX_BODY_SIZE=100
```

#### 3. 运行服务

```bash
go run main.go
```

或编译后运行：

```bash
go build -o proxy main.go
./proxy
```

## 配置说明

### 环境变量

| 变量名 | 说明 | 默认值 | 示例 |
|--------|------|--------|------|
| `PORT` | 服务监听端口 | `8000` | `8000` |
| `API_PROXY_CONFIG` | API 路径映射配置 | `{"openai":"https://api.openai.com"}` | 见下方说明 |
| `HTTP_PROXY` | HTTP 上游代理地址 | 无 | `http://proxy.example.com:8080` |
| `ENABLE_KEEPALIVE` | 是否启用 Keep-Alive | `true` | `true` / `false` |
| `TIMEOUT` | 请求超时时间（秒） | `300` | `120` |
| `MAX_BODY_SIZE` | 最大请求体大小（MB） | `100` | `50` |

### API_PROXY_CONFIG 配置说明

`API_PROXY_CONFIG` 是一个 JSON 对象，定义了路径前缀到目标 API 的映射关系。

**配置格式：**
```json
{
  "路径前缀1": "目标API地址1",
  "路径前缀2": "目标API地址2"
}
```

**示例：**
```json
{
  "openai": "https://api.openai.com",
  "anthropic": "https://api.anthropic.com",
  "custom": "https://custom-api.example.com"
}
```

**访问方式：**
- 请求 `http://localhost:8000/openai/v1/chat/completions` → 转发到 `https://api.openai.com/v1/chat/completions`
- 请求 `http://localhost:8000/anthropic/v1/messages` → 转发到 `https://api.anthropic.com/v1/messages`

### Keep-Alive 说明

Keep-Alive 控制是否复用 TCP 连接，不同场景需要不同配置：

- **启用 Keep-Alive** (`ENABLE_KEEPALIVE=true`)：
  - 连接复用，性能更高，延迟更低
  - 适合固定 IP 场景
  - 适合内网环境或无限速要求的场景
  - **推荐场景**：基础部署、开发测试

- **禁用 Keep-Alive** (`ENABLE_KEEPALIVE=false`)：
  - 每次请求新建连接，支持上游代理的 IP 轮换
  - 配合 ProxyFlow 等代理池实现请求级 IP 切换
  - 有效绕过基于 IP 的 API 限速
  - **推荐场景**：高频请求、需要绕过限速、配合代理池使用

## 使用示例

### 示例 1：代理 Anthropic API

假设配置为 `API_PROXY_CONFIG={"anthropic":"https://api.anthropic.com"}`

```bash
curl http://localhost:8000/anthropic/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-sonnet-4-5-20250929",
    "messages": [{"role": "user", "content": "Hello"}],
    "stream": true,
    "max_tokens": 1024
  }'
```

### 示例 2：同时代理多个 API

配置：`API_PROXY_CONFIG={"openai":"https://api.openai.com","anthropic":"https://api.anthropic.com"}`

```bash
# 访问 OpenAI
curl http://localhost:8000/openai/v1/chat/completions ...

# 访问 Anthropic
curl http://localhost:8000/anthropic/v1/messages ...
```

## Docker 部署

### 使用 Docker Compose

项目提供了两个 docker-compose 配置文件：

1. **docker-compose.yml** - 基础配置，适合单一 IP 或固定代理场景
2. **docker-compose-proxyflow.yml** - 配合 ProxyFlow 实现请求级 IP 轮换，有效绕过 API 限速

#### 基础部署

```bash
cd docker
docker-compose up -d
```

#### 使用 ProxyFlow 实现 IP 轮换（推荐用于高频请求场景）

ProxyFlow 是一个智能代理池管理工具，可实现请求级别的 IP 轮换，有效应对以下场景：
- API 速率限制（Rate Limiting）
- IP 级别的访问限制
- 高并发请求分散

**部署步骤：**

1. 准备代理列表文件 `proxy.txt`，格式为每行一个代理：
```
http://user:pass@ip1:port
http://user:pass@ip2:port
socks5://user:pass@ip3:port
```

2. 将 `proxy.txt` 放置在 `docker` 目录下

3. 启动服务：
```bash
cd docker
docker-compose -f docker-compose-proxyflow.yml up -d
```

**工作原理：**
- URL Proxy 设置 `ENABLE_KEEPALIVE=false`，确保每次请求使用新连接
- ProxyFlow 接收请求后，从代理池中轮换选择不同的代理 IP
- 每个请求使用不同的出口 IP，有效绕过基于 IP 的限速策略

### 自定义构建

```bash
# 构建镜像
docker build -f docker/Dockerfile -t url-proxy .

# 运行容器
docker run -d \
  -p 8000:8000 \
  -e API_PROXY_CONFIG='{"openai":"https://api.openai.com"}' \
  url-proxy
```