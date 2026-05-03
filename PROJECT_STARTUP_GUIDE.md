# AI 爆款文章创作器 - 项目启动流程指南

> 文档版本：v1.0  
> 适用对象：`ai-passage-creator` Java 后端项目  
> 最后更新：2026年4月9日

---

## 目录

1. [环境要求](#1-环境要求)
2. [快速启动（推荐）](#2-快速启动推荐)
3. [手动启动流程](#3-手动启动流程)
4. [开发环境配置](#4-开发环境配置)
5. [常见问题排查](#5-常见问题排查)
6. [验证与测试](#6-验证与测试)

---

## 1. 环境要求

### 1.1 基础环境

| 组件 | 版本要求 | 用途 | 下载地址 |
|------|----------|------|----------|
| **JDK** | 21+ | Java运行环境 | https://adoptium.net/ |
| **Maven** | 3.9+ | 项目构建 | https://maven.apache.org/ |
| **Node.js** | 18+ | 前端构建 | https://nodejs.org/ |
| **MySQL** | 8.0+ | 数据存储 | https://dev.mysql.com/ |
| **Redis** | 7.x | 缓存/会话 | https://redis.io/ |
| **Docker** | 20.10+ | 容器化部署（可选） | https://www.docker.com/ |

### 1.2 验证环境安装

```bash
# 验证 JDK
java -version
# 预期输出：openjdk version "21" 或更高

# 验证 Maven
mvn -version
# 预期输出：Apache Maven 3.9.x

# 验证 Node.js
node --version
# 预期输出：v18.x.x 或更高

# 验证 MySQL
mysql --version
# 预期输出：Ver 8.0.x

# 验证 Redis
redis-cli --version
# 预期输出：redis-cli 7.x.x
```

### 1.3 必需的外部服务

| 服务 | 用途 | 获取方式 |
|------|------|----------|
| **DashScope API Key** | 通义千问大模型调用 | https://bailian.console.aliyun.com |
| **Pexels API Key** | 图片库检索 | https://www.pexels.com/api/ |
| **腾讯云 COS** | 图片上传存储 | https://console.cloud.tencent.com/cos |
| **Stripe 账户** | 支付功能（可选） | https://dashboard.stripe.com |

---

## 2. 快速启动（推荐）

### 2.1 Docker 一键启动

**适用场景**：快速体验、本地开发、演示环境

```bash
# 1. 进入项目根目录
cd ai-passage-creator

# 2. 复制环境变量模板
cp .env.example .env

# 3. 编辑 .env 文件，填入必需的 API Key
# 必填：DASHSCOPE_API_KEY 和 PEXELS_API_KEY
vim .env

# 4. 启动所有服务（后台运行）
docker compose up -d --build

# 5. 查看服务状态
docker compose ps
```

**启动成功后访问**：
- 前端页面：http://localhost
- 后端 API：http://localhost:8123/api
- 接口文档：http://localhost:8123/api/doc.html

### 2.2 环境变量配置模板

```bash
# .env 文件内容示例

# ============================================
# 必需配置（不配置服务无法启动）
# ============================================
DASHSCOPE_API_KEY=your-dashscope-api-key-here
PEXELS_API_KEY=your-pexels-api-key-here

# ============================================
# 数据库配置（默认即可，生产环境建议修改）
# ============================================
MYSQL_ROOT_PASSWORD=123456
MYSQL_DATABASE=ai_passage_creator

# ============================================
# 端口配置（可选，默认即可）
# ============================================
BACKEND_PORT=8123
FRONTEND_PORT=80

# ============================================
# VIP 功能配置（可选）
# ============================================
NANO_BANANA_API_KEY=your-gemini-api-key-here
STRIPE_API_KEY=sk_test_your-stripe-key-here
STRIPE_WEBHOOK_SECRET=whsec_your-webhook-secret-here

# ============================================
# 腾讯云 COS 配置（可选，不配图片上传到本地）
# ============================================
TENCENT_COS_SECRET_ID=your-secret-id
TENCENT_COS_SECRET_KEY=your-secret-key
TENCENT_COS_REGION=ap-guangzhou
TENCENT_COS_BUCKET=your-bucket-name
```

---

## 3. 手动启动流程

### 3.1 数据库初始化

```bash
# 1. 登录 MySQL
mysql -u root -p

# 2. 创建数据库（或在 SQL 脚本中自动创建）
CREATE DATABASE IF NOT EXISTS ai_passage_creator 
    DEFAULT CHARACTER SET utf8mb4 
    DEFAULT COLLATE utf8mb4_unicode_ci;

# 3. 执行建表脚本
USE ai_passage_creator;
SOURCE sql/create_table.sql;

# 4. 执行增量更新脚本（如有）
SOURCE sql/add_article_fields.sql;
SOURCE sql/add_vip_payment.sql;

# 5. 验证表创建成功
SHOW TABLES;
# 预期输出：agent_log, article, payment_record, user
```

### 3.2 后端启动

#### 方式一：IDEA 启动（推荐开发使用）

```
1. 打开 IDEA，导入项目（选择 ai-passage-creator/pom.xml）
2. 等待 Maven 依赖下载完成
3. 复制配置文件：
   cp src/main/resources/application-local.yml.example \
      src/main/resources/application-local.yml
4. 编辑 application-local.yml，填入 API Key
5. 点击 MainApplication.java 的运行按钮
```

#### 方式二：命令行启动

```bash
# 1. 进入后端项目目录
cd ai-passage-creator

# 2. 复制并编辑配置文件
cp src/main/resources/application-local.yml.example \
   src/main/resources/application-local.yml
# 编辑 application-local.yml 填入 API Key

# 3. 编译项目
mvn clean compile

# 4. 启动应用
mvn spring-boot:run

# 或使用打包后的 jar 启动
mvn clean package -DskipTests
java -jar target/ai-passage-creator-0.0.1-SNAPSHOT.jar
```

#### 方式三：多环境配置启动

```bash
# 开发环境
mvn spring-boot:run -Dspring-boot.run.profiles=local

# 测试环境
mvn spring-boot:run -Dspring-boot.run.profiles=test

# 生产环境
java -jar target/ai-passage-creator-0.0.1-SNAPSHOT.jar \
     --spring.profiles.active=prod
```

### 3.3 前端启动

```bash
# 1. 进入前端目录
cd frontend

# 2. 安装依赖
npm install

# 3. 启动开发服务器
npm run dev

# 4. 浏览器访问
# http://localhost:5173
```

### 3.4 配置文件详解

#### application.yml（主配置）

```yaml
server:
  port: 8567
  servlet:
    context-path: /api

spring:
  application:
    name: ai-passage-creator
  
  # 数据源配置
  datasource:
    url: jdbc:mysql://localhost:3306/ai_passage_creator?useUnicode=true&characterEncoding=utf-8
    username: root
    password: ${MYSQL_PASSWORD:123456}
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.zaxxer.hikari.HikariDataSource
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 30000
  
  # Redis 配置
  redis:
    host: localhost
    port: 6379
    database: 0
    timeout: 5000
  
  # 会话配置
  session:
    store-type: redis
    timeout: 30d

# Spring AI Alibaba 配置
spring:
  ai:
    alibaba:
      dashscope:
        api-key: ${DASHSCOPE_API_KEY:}
        chat:
          options:
            model: qwen-max

# 日志配置
logging:
  level:
    com.yupi.template: DEBUG
    com.alibaba.cloud.ai: INFO
```

#### application-local.yml（本地开发覆盖配置）

```yaml
# 本地开发覆盖配置
spring:
  datasource:
    password: 123456  # 本地密码可能不同
  
  ai:
    alibaba:
      dashscope:
        api-key: sk-your-local-api-key

# 本地调试：关闭部分外部服务
pexels:
  enabled: true
  api-key: your-local-pexels-key

cos:
  enabled: false  # 本地开发可关闭 COS，图片保存到本地
```

---

## 4. 开发环境配置

### 4.1 IDEA 推荐配置

```
1. 安装插件：
   - Lombok（必需）
   - MyBatis-Flex（可选，代码生成辅助）
   - Maven Helper（依赖分析）

2. 代码风格设置：
   - Editor → Code Style → Java
   - 导入代码风格配置文件：.idea/codeStyles/Project.xml

3. 运行配置：
   - Main class: com.yupi.template.MainApplication
   - VM options: -Dspring.profiles.active=local
   - Working directory: $MODULE_WORKING_DIR$
```

### 4.2 热加载配置

```xml
<!-- pom.xml 已包含 devtools -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

```yaml
# application-local.yml 启用热加载
spring:
  devtools:
    restart:
      enabled: true
      additional-paths: src/main/java
```

### 4.3 调试配置

```bash
# IDEA 调试模式启动
# 1. 点击 Debug 按钮（Shift+F9）
# 2. 自动开启 5005 调试端口

# 远程调试（用于 Docker 环境）
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005 \
     -jar target/ai-passage-creator-0.0.1-SNAPSHOT.jar
```

---

## 5. 常见问题排查

### 5.1 启动失败问题

| 问题现象 | 可能原因 | 解决方案 |
|----------|----------|----------|
| `Connection refused: localhost:3306` | MySQL 未启动 | `sudo systemctl start mysql` 或检查端口 |
| `Redis connection failed` | Redis 未启动 | `redis-server` 或检查端口 6379 |
| `Invalid API Key` | DashScope Key 无效 | 检查 `application-local.yml` 中的 key |
| `Port 8567 already in use` | 端口被占用 | `lsof -i:8567` 然后 `kill -9 PID` |
| `Table doesn't exist` | 未执行 SQL 脚本 | 运行 `sql/create_table.sql` |

### 5.2 依赖下载问题

```bash
# Maven 依赖下载失败
mvn clean install -U

# 切换阿里云镜像（settings.xml）
<mirrors>
  <mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
  </mirror>
</mirrors>

# 跳过测试快速编译
mvn clean package -DskipTests
```

### 5.3 Docker 启动问题

```bash
# 查看容器日志
docker compose logs -f backend
docker compose logs -f mysql
docker compose logs -f redis

# 重新构建（代码修改后）
docker compose up -d --build

# 完全重置（清空数据）
docker compose down -v
docker compose up -d --build

# 检查容器状态
docker ps
docker compose ps
```

### 5.4 API 调用问题

```bash
# 测试后端是否启动
curl http://localhost:8567/api/health
# 预期：{"code":0,"data":"ok","message":"ok"}

# 测试接口文档
open http://localhost:8567/api/doc.html

# 查看详细错误日志
tail -f logs/error.log
```

---

## 6. 验证与测试

### 6.1 服务健康检查

```bash
#!/bin/bash
# health-check.sh

echo "=== 服务健康检查 ==="

# 1. 检查 MySQL
if mysql -u root -p123456 -e "SELECT 1" > /dev/null 2>&1; then
    echo "✅ MySQL: 运行正常"
else
    echo "❌ MySQL: 连接失败"
fi

# 2. 检查 Redis
if redis-cli ping | grep -q "PONG"; then
    echo "✅ Redis: 运行正常"
else
    echo "❌ Redis: 连接失败"
fi

# 3. 检查后端服务
if curl -s http://localhost:8567/api/health | grep -q "ok"; then
    echo "✅ 后端服务: 运行正常"
else
    echo "❌ 后端服务: 未启动"
fi

# 4. 检查前端服务
if curl -s -o /dev/null -w "%{http_code}" http://localhost:5173 | grep -q "200"; then
    echo "✅ 前端服务: 运行正常"
else
    echo "❌ 前端服务: 未启动"
fi
```

### 6.2 核心功能测试

```bash
# 1. 创建文章任务
curl -X POST http://localhost:8567/api/article/create \
  -H "Content-Type: application/json" \
  -H "Cookie: session=your-session-id" \
  -d '{
    "topic": "人工智能发展趋势",
    "style": "tech"
  }'

# 2. 获取文章列表
curl http://localhost:8567/api/article/list \
  -H "Cookie: session=your-session-id"

# 3. 查看 SSE 流（需要浏览器或专用客户端）
curl http://localhost:8567/api/article/stream/{taskId} \
  -H "Cookie: session=your-session-id"
```

### 6.3 测试账号

```
| 账号   | 密码     | 角色   | 用途         |
|--------|----------|--------|--------------|
| admin  | 12345678 | 管理员 | 后台管理测试 |
| user   | 12345678 | 普通用户 | 功能测试   |
| test   | 12345678 | 测试账号 | 自动化测试 |
```

### 6.4 启动成功标志

```
✅ 后端控制台输出：
   "Started MainApplication in X.XXX seconds"
   
✅ 接口文档可访问：
   http://localhost:8567/api/doc.html
   
✅ 健康检查通过：
   curl http://localhost:8567/api/health
   
✅ 前端页面可访问：
   http://localhost:5173 (开发) 或 http://localhost (Docker)
   
✅ 可正常创建文章任务并看到 SSE 实时推送
```

---

## 附录

### A. 常用命令速查

```bash
# 快速启动（Docker）
docker compose up -d

# 查看日志
docker compose logs -f backend

# 停止服务
docker compose down

# Maven 快速编译
mvn clean package -DskipTests

# 后端开发模式
mvn spring-boot:run

# 前端开发模式
npm run dev
```

### B. 配置文件优先级

```
优先级从高到低：
1. 命令行参数 (--spring.profiles.active=prod)
2. 环境变量 (SPRING_DATASOURCE_PASSWORD)
3. application-{profile}.yml
4. application.yml
5. 默认配置
```

### C. 相关文档链接

- [项目 README](README.md)
- [技术分析报告](Java_Backend_Technical_Analysis_Report.md)
- [VIP 功能说明](VIP_FEATURES.md)
- [Stripe 支付配置](STRIPE_SETUP.md)

---

*文档结束 - 祝开发顺利！*
