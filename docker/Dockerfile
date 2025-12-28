# 多阶段构建：编译阶段
FROM --platform=$BUILDPLATFORM golang:1.21-alpine AS builder

# 使用 buildx 提供的架构变量
ARG TARGETOS
ARG TARGETARCH

# 设置 Go 代理加速依赖下载
ENV GOPROXY=https://goproxy.cn,https://proxy.golang.org,direct

# 设置工作目录
WORKDIR /build

# 复制 go.mod 和 go.sum
COPY go.mod go.sum ./

# 下载依赖（利用 Docker 缓存）
RUN go mod download

# 复制源代码
COPY main.go ./

# 编译二进制文件（静态链接，优化体积）
# 使用原生平台编译，交叉编译到目标平台
RUN CGO_ENABLED=0 GOOS=${TARGETOS} GOARCH=${TARGETARCH} go build \
    -ldflags="-w -s" \
    -o proxy \
    main.go

# 运行阶段：使用最小镜像
FROM alpine:latest

# 安装 CA 证书（HTTPS 请求需要）
RUN apk --no-cache add ca-certificates tzdata

# 设置时区
ENV TZ=Asia/Shanghai

# 创建非 root 用户
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

# 设置工作目录
WORKDIR /app

# 从构建阶段复制二进制文件
COPY --from=builder /build/proxy .

# 复制 .env 文件（可选）
COPY .env* ./

# 修改文件所有者
RUN chown -R appuser:appuser /app

# 切换到非 root 用户
USER appuser

# 暴露端口
EXPOSE 8000

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8000/ || exit 1

# 启动服务
CMD ["./proxy"]
