# Spring Boot 项目结构规范（示例）

建议的目录/包划分及作用说明，便于团队统一约定和快速上手。

```
emotion-record-platform/
├─ pom.xml                        # Maven 依赖与插件
├─ mvnw*                          # Maven Wrapper，保证版本一致
├─ src/
│  ├─ main/java/
│  │  └─ com/jue/emotion_record_platform/
│  │     ├─ EmotionRecordPlatformApplication.java  # 启动类
│  │     ├─ config/              # 全局配置（CORS、WebMvc、Beans）
│  │     ├─ security/            # 安全与鉴权配置（过滤器、JWT、WebSecurity）
│  │     ├─ controller/         # 接口层，处理请求/响应
│  │     ├─ dto/                 # 请求/响应模型，含校验注解
│  │     ├─ domain/
│  │     │  └─ entity/           # JPA/领域实体
│  │     ├─ repository/          # 数据访问接口（JPA Repositories）
│  │     ├─ service/             # 业务接口
│  │     │  └─ impl/             # 业务实现
│  │     ├─ exception/           # 全局异常、业务异常定义
│  │     ├─ util/                # 工具类/辅助组件
│  ├─ main/resources/
│  │  ├─ application.properties  # 应用配置（可分 profile: application-*.properties）
│  │  └─ static|templates        # 前端静态资源或模板（可选）
│  └─ test/java/                 # 测试代码（单测/集成测试）
└─ target/                       # 构建输出（忽略提交）
```

## 关键规范与建议
- **分层职责清晰**：Controller 只做接口编排，业务放 Service，数据访问放 Repository。
- **DTO 与 Entity 分离**：外部交互使用 DTO，内部持久化用 Entity，避免耦合。
- **统一返回结构**：封装 `ApiResponse<T>` 统一 `success/data/message`。
- **参数校验**：在 DTO 上使用 `jakarta.validation` 注解；结合 `@Valid` 与全局异常处理。
- **异常处理**：`@RestControllerAdvice` 统一转换异常，避免重复 try/catch。
- **配置分环境**：使用 `application-{profile}.properties`，本地/测试/生产独立配置。
- **安全策略**：在 `security/` 配置认证与权限（如 JWT Filter、密码加密、权限注解）。
- **事务管理**：在 Service 层使用 `@Transactional` 包裹需要的写操作。
- **命名与包结构**：按业务域拆包，避免“大杂烩”包；工具类集中在 `util/`。
- **测试**：为业务逻辑和接口编写单测/集成测试，保持可回归性。
