# smart-food-platform

融合“苍穹外卖业务流 + 黑马点评Redis能力”的 Spring Boot 后端项目，覆盖用户、店铺、订单支付、社交互动、秒杀、签到积分与数据统计能力。

## 功能列表

- 用户模块：注册、密码登录、短信验证码登录、登出、分布式 Session（Redis）
- 店铺模块：店铺详情缓存、缓存预热、缓存降级、GEO 附近搜索
- 订单模块：购物车下单、库存扣减、防重复提交、延迟取消、订单状态机
- 支付模块：微信支付 mock/notify 链路、状态同步
- 营销模块：优惠券秒杀（Lua 原子扣减 + 异步入库）、签到积分、排行榜
- 社交模块：点赞、关注、共同关注、Feed 流（推模式）
- 统计模块：营业额/用户/订单/热门菜品/评分分布、UV HyperLogLog

## 技术栈

- Java 17
- Spring Boot 2.7.18
- MyBatis-Plus + JPA
- MySQL 8.x
- Redis 6/7
- JWT（jjwt）
- Swagger（springdoc-openapi）
- JUnit 5 + Mockito

## 快速开始

### 1. 环境准备

- JDK 17
- Maven 3.8+
- MySQL 8.x
- Redis 6/7

### 2. 配置

编辑 `src/main/resources/application.yml`：

- `spring.datasource.*`：数据库连接
- `spring.redis.*`：Redis 连接
- `app.jwt.secret`：JWT 密钥
- `app.jwt.expiration-days`：JWT 有效天数
- `app.statistics.daily-cron`：统计任务 cron（可选）
- `app.statistics.preheat-days`：统计预热天数（可选）

### 3. 启动

```bash
mvn -DskipTests spring-boot:run
```

如果你只执行上面这一步，当前只会启动后端服务，所以你看到的是 Swagger。

启动后访问：

- Swagger UI：`http://localhost:8080/swagger-ui.html`

如需看到管理端前端页面，还需要单独启动前端（`admin-web`）：

```bash
cd admin-web
npm install
npm run dev
```

启动后访问：

- 管理端前端页面：`http://localhost:5173`

## 三角色入口与体验路径

### 1) 平台管理员（后台运营）

- 入口页面：`http://localhost:5173/login`
- 登录账号：`admin / admin123`
- 主要页面路径：
  - 仪表盘：`/dashboard`
  - 用户管理：`/users`
  - 店铺管理：`/shops`
  - 订单管理：`/orders`
  - 运营设置（点评审核/举报处理/公告配置）：`/settings`

### 2) 商家（B端用户）

- 入口页面：`http://localhost:5173/login`
- 登录账号示例：`merchant_1 / merchant123`
- 主要体验方式：
  - 可复用管理端页面查看订单/店铺数据
  - 商家专属接口（Swagger 中可见）：
    - `GET /merchant/shop`
    - `PUT /merchant/shop`
    - `GET /merchant/dishes`
    - `POST /merchant/dishes`
    - `PUT /merchant/dishes/{id}`
    - `DELETE /merchant/dishes/{id}`

### 3) 消费者（C端用户）

- 当前仓库未提供独立 C 端前端页面（App/H5），C 端链路通过后端接口体验。
- 体验入口：`http://localhost:8080/swagger-ui.html`
- 推荐体验顺序（已覆盖完整核心链路）：
  1. 用户登录：`/user/register`、`/user/login`、`/user/sms/login`
  2. 地址管理：`/app/address`（增删改查/默认地址）
  3. 店铺与菜品：`/shop/nearby`、`/app/shops/search`、`/app/dishes/search`、`/app/shops/{shopId}/dishes`
  4. 购物车：`/app/cart`
  5. 下单与支付：`/order/create` + 微信支付相关接口
  6. 订单追踪：`/app/orders`、`/app/orders/{orderNo}`、`/order/{orderNo}/...`
  7. 评价互动：`/review`、`/review/shop/{shopId}`、`/review/{id}/like`

## 数据库设计说明

核心表：

- `user`
- `shop`
- `dish`
- `cart`
- `order`
- `order_detail`
- （可扩展）`coupon_order`、`order_operation_log`

说明：实体映射在 `src/main/java/com/smartfood/entity` 下，结合 MyBatis-Plus Mapper 与部分 `JdbcTemplate` 聚合 SQL 使用。

## 接口文档

- Swagger 在线文档：`/swagger-ui.html`
- Postman 集合：`docs/postman/smart-food-platform.postman_collection.json`

## 测试说明

单元测试位于 `src/test/java`，已覆盖关键路径：

- `UserServiceImplTest`：注册/登录/短信登录关键逻辑
- `OrderServiceImplTest`：下单主流程与异常路径
- `SmsCodeServiceTest`：验证码发送/校验异常场景
- `RedisLoginSessionServiceTest`：Redis 会话写入与校验

执行测试：

```bash
mvn test
```

## 部署说明

### 本地部署

1. 准备 MySQL/Redis 并导入基础表结构
2. 配置 `application.yml`
3. 打包并运行：

```bash
mvn clean package -DskipTests
java -jar target/smart-food-platform-0.0.1-SNAPSHOT.jar
```

### 生产部署建议

- 使用 `prod` profile
- JWT 密钥与数据库密码放环境变量/配置中心
- Redis 开启持久化与监控
- Nginx 反向代理 + HTTPS
- 配置 JVM 参数（如 `-Xms -Xmx`）并开启 GC 日志

### Docker 一键部署（推荐联调）

项目已提供：

- `docker-compose.yml`
- 后端镜像构建文件 `Dockerfile`
- 管理端镜像构建文件 `admin-web/Dockerfile`
- 管理端网关配置 `admin-web/nginx.conf`

启动：

```bash
docker compose up -d --build
```

访问：

- 管理端：`http://localhost`
- 后端接口：`http://localhost:8080`

默认联调账号：

- 平台管理员：`admin / admin123`
- 商家账号：`merchant_{shopId} / merchant123`（例如 `merchant_1`）

> 注意：默认密码仅用于本地开发联调，请在生产环境替换并接入正式账号体系。

## 常见问题（FAQ）

### 1. Swagger 打不开？

确认服务已启动，访问 `http://localhost:8080/swagger-ui.html`，并检查 `springdoc` 依赖版本是否正确。

### 2. Redis 相关接口报错？

确认 Redis 可达、配置正确；开发环境可先用本地 Redis 单实例。

### 3. 登录后 token 无效？

检查：

- `app.jwt.secret` 一致性
- `Authorization: Bearer <token>` 格式
- Redis session key `login:user:{token}` 是否存在

### 4. 统计数据为空？

统计接口支持实时回源与缓存；若仍为空，请确认订单/用户基础数据是否存在，并检查时间窗口口径。

## 附加文档

- 统计性能优化建议：`docs/数据统计性能优化建议.md`

