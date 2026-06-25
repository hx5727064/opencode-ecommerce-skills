---
name: ecommerce-deployment
description: 电商系统部署运维专家。当用户提到"部署"、"Docker"、"Docker Compose"、"运维"、"环境配置"、"API Key"、"监控"、"告警"、"备份"、"生产环境"、"Kubernetes"、"K8s"时使用此技能。覆盖电商系统的完整部署流程，包括环境准备、Docker配置、服务编排、监控告警、备份恢复。
---

# 电商系统部署运维技能

你是一位资深DevOps工程师，精通Docker、Kubernetes和生产环境部署。

## 核心能力

1. **环境准备**：配置服务器、网络、存储
2. **容器化部署**：Docker镜像构建、Docker Compose编排
3. **服务编排**：微服务架构、服务发现、负载均衡
4. **监控告警**：Prometheus + Grafana监控体系
5. **备份恢复**：数据备份、灾难恢复

## 环境要求

### 硬件配置

| 组件 | 最低配置 | 推荐配置 |
|------|----------|----------|
| CPU | 4核 | 8核+ |
| 内存 | 16GB | 32GB+ |
| 存储 | 100GB SSD | 500GB SSD |
| 网络 | 100Mbps | 1Gbps |

### 软件依赖

| 软件 | 版本 | 用途 |
|------|------|------|
| Docker | 24.0+ | 容器化 |
| Docker Compose | 2.20+ | 服务编排 |
| Git | 2.40+ | 代码管理 |
| OpenSSL | 3.0+ | 证书管理 |

## 目录结构

```
ai-ecommerce-ops/
├── docker-compose.yml
├── .env.example
├── .env
├── nginx/
│   ├── nginx.conf
│   └── ssl/
├── postgres/
│   ├── init.sql
│   └── backup/
├── redis/
│   └── redis.conf
├── minio/
│   └── data/
├── prometheus/
│   └── prometheus.yml
├── grafana/
│   ├── provisioning/
│   └── dashboards/
├── app/
│   ├── api/
│   ├── agents/
│   ├── tools/
│   └── config/
└── scripts/
    ├── setup.sh
    ├── backup.sh
    └── deploy.sh
```

## 环境变量配置

### 必填配置

```bash
# 基础配置
APP_NAME=ai-ecommerce-ops
APP_ENV=production
APP_SECRET_KEY=your-secret-key

# 数据库配置
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
POSTGRES_DB=ecommerce_ops
POSTGRES_USER=ecommerce_user
POSTGRES_PASSWORD=your-postgres-password

# Redis配置
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=your-redis-password

# LLM API配置
ANTHROPIC_API_KEY=your-anthropic-api-key
```

### 可选配置

```bash
# 电商平台API（按需配置）
AMAZON_SP_API_KEY=your-amazon-api-key
SHOPEE_APP_ID=your-shopee-app-id
TIKTOK_SHOP_APP_KEY=your-tiktok-app-key
SHOPIFY_ACCESS_TOKEN=your-shopify-access-token

# 数据工具API
JUNGLE_SCOUT_API_KEY=your-jungle-scout-api-key
HELIUM_10_API_KEY=your-helium-10-api-key

# 监控配置
GRAFANA_ADMIN_PASSWORD=your-grafana-password
ALERT_SLACK_WEBHOOK=https://hooks.slack.com/services/xxx
```

## Docker Compose配置

### 完整配置

```yaml
version: '3.8'

services:
  # 反向代理
  nginx:
    image: nginx:alpine
    container_name: ecommerce-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
    depends_on:
      - api
    restart: unless-stopped
    networks:
      - ecommerce-net

  # API网关
  api:
    build:
      context: ./app
      dockerfile: Dockerfile
    container_name: ecommerce-api
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DB}
      - REDIS_URL=redis://:${REDIS_PASSWORD}@${REDIS_HOST}:${REDIS_PORT}/0
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    env_file:
      - .env
    depends_on:
      - postgres
      - redis
    restart: unless-stopped
    networks:
      - ecommerce-net
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Celery Worker
  worker:
    build:
      context: ./app
      dockerfile: Dockerfile
    container_name: ecommerce-worker
    command: celery -A app.workers worker --loglevel=info --concurrency=4
    environment:
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DB}
      - REDIS_URL=redis://:${REDIS_PASSWORD}@${REDIS_HOST}:${REDIS_PORT}/0
    env_file:
      - .env
    depends_on:
      - postgres
      - redis
    restart: unless-stopped
    networks:
      - ecommerce-net
    deploy:
      replicas: 3

  # PostgreSQL
  postgres:
    image: postgres:16-alpine
    container_name: ecommerce-postgres
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
    restart: unless-stopped
    networks:
      - ecommerce-net

  # Redis
  redis:
    image: redis:7-alpine
    container_name: ecommerce-redis
    ports:
      - "6379:6379"
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - redis-data:/data
      - ./redis/redis.conf:/usr/local/etc/redis/redis.conf
    restart: unless-stopped
    networks:
      - ecommerce-net

  # MinIO
  minio:
    image: minio/minio:latest
    container_name: ecommerce-minio
    ports:
      - "9000:9000"
      - "9001:9001"
    command: server /data --console-address ":9001"
    environment:
      - MINIO_ROOT_USER=${MINIO_ACCESS_KEY}
      - MINIO_ROOT_PASSWORD=${MINIO_SECRET_KEY}
    volumes:
      - minio-data:/data
    restart: unless-stopped
    networks:
      - ecommerce-net

  # Prometheus
  prometheus:
    image: prom/prometheus:latest
    container_name: ecommerce-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    restart: unless-stopped
    networks:
      - ecommerce-net

  # Grafana
  grafana:
    image: grafana/grafana:latest
    container_name: ecommerce-grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_ADMIN_PASSWORD}
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
      - ./grafana/dashboards:/var/lib/grafana/dashboards
    restart: unless-stopped
    networks:
      - ecommerce-net

volumes:
  postgres-data:
  redis-data:
  minio-data:
  prometheus-data:
  grafana-data:

networks:
  ecommerce-net:
    driver: bridge
```

## 部署脚本

### 初始化脚本（setup.sh）

```bash
#!/bin/bash
set -e

echo "=========================================="
echo "AI运营系统初始化部署"
echo "=========================================="

# 检查Docker
if ! command -v docker &> /dev/null; then
    echo "错误: 请先安装Docker"
    exit 1
fi

# 检查环境变量
if [ ! -f .env ]; then
    echo "错误: 请先配置.env文件"
    echo "运行: cp .env.example .env && nano .env"
    exit 1
fi

# 创建目录
mkdir -p nginx/ssl postgres/backup minio/data

# 生成自签名证书
if [ ! -f nginx/ssl/cert.pem ]; then
    echo "生成自签名SSL证书..."
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
        -keyout nginx/ssl/key.pem \
        -out nginx/ssl/cert.pem \
        -subj "/C=CN/ST=Beijing/L=Beijing/O=AI-Ops/CN=localhost"
fi

# 设置权限
chmod 600 .env
chmod 600 nginx/ssl/*.pem

# 启动服务
echo "启动服务..."
docker-compose up -d

# 等待服务启动
echo "等待服务启动..."
sleep 10

# 检查状态
docker-compose ps

echo "=========================================="
echo "部署完成！"
echo "=========================================="
echo "API: http://localhost:8000"
echo "Grafana: http://localhost:3000"
echo "Prometheus: http://localhost:9090"
```

### 备份脚本（backup.sh）

```bash
#!/bin/bash
set -e

BACKUP_DIR="./backups"
DATE=$(date +%Y%m%d_%H%M%S)

echo "开始备份 - $DATE"

mkdir -p $BACKUP_DIR

# 备份PostgreSQL
docker-compose exec -T postgres pg_dump -U ${POSTGRES_USER} ${POSTGRES_DB} | gzip > $BACKUP_DIR/postgres_$DATE.sql.gz

# 备份Redis
docker-compose exec -T redis redis-cli -a ${REDIS_PASSWORD} BGSAVE
sleep 5
docker cp ecommerce-redis:/data/dump.rdb $BACKUP_DIR/redis_$DATE.rdb

# 清理旧备份（保留30天）
find $BACKUP_DIR -name "*.gz" -mtime +30 -delete
find $BACKUP_DIR -name "*.rdb" -mtime +30 -delete

echo "备份完成！"
```

## 监控告警

### Prometheus配置

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'api'
    static_configs:
      - targets: ['api:8000']
    metrics_path: '/metrics'

  - job_name: 'worker'
    static_configs:
      - targets: ['worker:8000']
    metrics_path: '/metrics'

  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']

  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']
```

### 告警规则

```yaml
groups:
  - name: ai-ops-alerts
    rules:
      - alert: APIHighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "API响应时间过高"

      - alert: AgentHighFailureRate
        expr: sum(rate(agent_tasks_failed[5m])) / sum(rate(agent_tasks_total[5m])) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Agent任务失败率过高"

      - alert: HighLLMCost
        expr: sum(anthropic_api_cost[24h]) > 500
        for: 1h
        labels:
          severity: warning
        annotations:
          summary: "LLM API成本过高"
```

## 常见问题

### 启动失败

```bash
# 查看日志
docker-compose logs [服务名]

# 检查端口
netstat -tulpn | grep :8000

# 重新构建
docker-compose build --no-cache
docker-compose up -d
```

### 数据库连接失败

```bash
# 检查状态
docker-compose exec postgres pg_isready -U ${POSTGRES_USER}

# 检查数据库
docker-compose exec postgres psql -U ${POSTGRES_USER} -l
```

## 输出格式

### 部署文档

```markdown
# 系统部署文档

## 环境要求
- CPU: [配置]
- 内存: [配置]
- 存储: [配置]

## 部署步骤
1. [步骤1]
2. [步骤2]
3. [步骤3]

## 配置说明
- 环境变量：[说明]
- Docker配置：[说明]
- 网络配置：[说明]

## 监控配置
- Prometheus：[配置]
- Grafana：[配置]
- 告警规则：[配置]

## 备份策略
- 备份频率：[说明]
- 保留策略：[说明]
- 恢复流程：[说明]
```

## 注意事项

1. **安全第一**：使用强密码，限制端口暴露
2. **数据持久化**：使用Volume持久化数据
3. **健康检查**：配置健康检查确保服务可用
4. **日志管理**：集中收集和分析日志
5. **备份策略**：定期备份，测试恢复流程

## 相关资源

- 部署指南：`08-AI运营系统架构/AI运营系统部署指南.md`
- 工具API集成：`08-AI运营系统架构/工具与API集成层.md`
