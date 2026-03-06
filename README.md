# MyServer - 分布式服务器监控系统

一个基于 Spring Boot + Vue 3 的轻量级分布式服务器监控解决方案，支持实时监控多台服务器的 CPU、内存、磁盘、网络等系统指标，并提供 Web SSH 终端功能。

## 项目架构

```
MyServer/
├── back-server/      # 后端服务器（Spring Boot）
├── back-client/      # 监控客户端（部署在被监控服务器）
├── front-web/        # 前端界面（Vue 3）
└── config/           # 配置文件
```

### 技术栈

**后端服务器 (back-server)**
- Spring Boot 3.1.2 + Java 21
- MyBatis-Plus + MySQL（用户和客户端信息）
- InfluxDB（时序数据存储）
- Redis（缓存和限流）
- RabbitMQ（异步消息队列）
- Spring Security + JWT（认证授权）
- WebSocket（实时通信）
- Swagger/OpenAPI（API 文档）

**监控客户端 (back-client)**
- Spring Boot 3.1.2 + Java 21
- OSHI（系统信息采集）
- Quartz（定时任务调度）
- FastJSON2（数据序列化）

**前端界面 (front-web)**
- Vue 3 + TypeScript
- Vite 6
- Element Plus（UI 组件库）
- ECharts（数据可视化）
- Pinia（状态管理）
- xterm.js（终端模拟器）

## 核心功能

### 1. 实时监控
- CPU 使用率
- 内存使用情况
- 磁盘读写速度和使用量
- 网络上传/下载速度
- 历史数据图表展示

### 2. 客户端管理
- Token 注册机制
- 客户端分组和重命名
- 客户端详情查看
- 批量管理和删除

### 3. 用户权限
- 管理员/普通用户角色
- 子账号管理
- 基于客户端的访问权限控制
- JWT 令牌认证

### 4. SSH 终端
- Web 端 SSH 连接
- 实时终端交互
- 支持多会话管理

### 5. 安全特性
- 邮件验证码
- 密码加密存储
- 请求频率限制
- CORS 跨域配置
- JWT 令牌过期机制

## 快速开始

### 环境要求

- JDK 21+
- Node.js 18+
- MySQL 8.0+
- Redis 6.0+
- RabbitMQ 3.x
- InfluxDB 2.x

### 1. 数据库配置

**MySQL 数据库**
```sql
CREATE DATABASE monitor CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

**InfluxDB 配置**
- 创建组织（Organization）
- 创建存储桶（Bucket）
- 生成访问令牌（Token）

### 2. 后端服务器部署

修改配置文件 `back-server/src/main/resources/application-dev.yml`：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/monitor
    username: root
    password: your_password

  data:
    redis:
      host: localhost
      port: 6379
      password: your_password

  rabbitmq:
    addresses: localhost
    username: your_username
    password: your_password

  influxdb:
    url: http://localhost:8086
    username: your_username
    password: your_password
    org: your_org
    bucket: your_bucket

  mail:
    host: smtp.126.com
    username: your_email@126.com
    password: your_email_password
```

启动服务器：
```bash
cd back-server
mvn clean package
java -jar target/back-server-0.0.1-SNAPSHOT.jar
```

服务器将在 `http://localhost:8081` 启动

### 3. 前端界面部署

```bash
cd front-web
npm install
npm run dev
```

前端将在 `http://localhost:5173` 启动

生产环境构建：
```bash
npm run build
```

### 4. 监控客户端部署

**步骤 1：获取注册 Token**
- 使用管理员账号登录前端界面
- 在客户端管理页面生成注册 Token

**步骤 2：配置客户端**

修改 `config/server.json`：
```json
{
  "address": "http://your-server-ip:8081",
  "networkInterface": "eth0",
  "token": "your_registration_token"
}
```

**步骤 3：启动客户端**
```bash
cd back-client
mvn clean package
java -jar target/back-client-0.0.1-SNAPSHOT.jar
```

客户端将自动注册并开始上报监控数据。

## API 文档

启动后端服务器后，访问 Swagger UI：
```
http://localhost:8081/swagger-ui.html
```

## 主要接口

### 认证接口
- `POST /api/auth/login` - 用户登录
- `GET /api/auth/ask-code` - 获取邮箱验证码
- `POST /api/auth/reset-confirm` - 重置密码

### 监控接口
- `GET /api/monitor/list` - 获取客户端列表
- `GET /api/monitor/details` - 获取客户端详情
- `GET /api/monitor/runtime-history` - 获取历史监控数据
- `GET /api/monitor/runtime-now` - 获取实时监控数据
- `GET /api/monitor/register` - 生成注册 Token
- `GET /api/monitor/delete` - 删除客户端

### 客户端接口
- `GET /monitor/register` - 客户端注册
- `POST /monitor/detail` - 上报基础信息
- `POST /monitor/runtime` - 上报运行时数据

### WebSocket
- `/terminal` - SSH 终端连接

## 项目配置

### 安全配置
```yaml
spring:
  security:
    jwt:
      key: 'your_secret_key'      # JWT 密钥
      expire: 72                   # 令牌过期时间（小时）
      limit:
        base: 9                    # 基础限制
        upgrade: 280               # 升级限制
        frequency: 30              # 频率限制
```

### 流量控制
```yaml
spring:
  web:
    flow:
      period: 5                    # 时间窗口（秒）
      limit: 500                   # 请求限制
      block: 30                    # 封禁时间（秒）
```

### 邮件验证
```yaml
spring:
  web:
    verify:
      mail-limit: 60               # 验证码有效期（秒）
```

## 默认账号

首次启动需要在数据库中手动创建管理员账号：

```sql
INSERT INTO db_account (username, email, password, role, register_time, clients)
VALUES ('admin', 'admin@example.com', '$2a$10$...', 'admin', NOW(), '[]');
```

注意：密码需要使用 BCrypt 加密。

## 开发说明

### 后端开发
```bash
cd back-server
mvn spring-boot:run
```

### 前端开发
```bash
cd front-web
npm run dev
```

### 客户端开发
```bash
cd back-client
mvn spring-boot:run
```

## 部署建议

### 生产环境配置

1. 修改 `application-prod.yml` 配置文件
2. 使用环境变量管理敏感信息
3. 配置 HTTPS 证书
4. 启用防火墙规则
5. 配置 Nginx 反向代理

### 性能优化

- Redis 缓存客户端信息
- InfluxDB 数据保留策略
- 定期清理历史数据
- 调整监控采集频率

## 常见问题

**Q: 客户端无法注册？**
A: 检查 Token 是否正确，网络是否可达，服务器地址是否配置正确。

**Q: 监控数据不更新？**
A: 检查客户端是否正常运行，InfluxDB 连接是否正常。

**Q: 邮件验证码收不到？**
A: 检查邮箱配置，确认 RabbitMQ 服务正常，查看邮件队列日志。

**Q: WebSocket 连接失败？**
A: 检查防火墙配置，确认 WebSocket 端口开放。

## 许可证

本项目仅供学习和研究使用。

## 作者

吾骨封灯

## 更新日志

### v0.0.1-SNAPSHOT
- 初始版本
- 实现基础监控功能
- 支持多客户端管理
- 集成 Web SSH 终端
