# Java 项目命名规范（示例）

面向 Spring Boot/Java 项目的通用命名约定，帮助保持一致性与可读性。

## 目录与包
- 业务分层包：`config/`、`security/`、`controller/`、`service/impl/`、`repository/`、`domain/entity/`、`dto/`、`exception/`、`util/`。
- 包名全小写，单词用点分隔：`com.company.project.module`。
- 避免在包名中使用复数或下划线；如需复合语义用子包：`controller/admin`。

## 类与接口
- 类名：`PascalCase`，名词或名词短语，表达职责，例如 `UserServiceImpl`、`MoodRecordController`。
- 接口名：`PascalCase`，行为/能力导向，例如 `UserService`、`MoodRecordService`；如需默认实现，用 `Impl` 后缀。
- 枚举：`PascalCase`；枚举常量使用 `UPPER_SNAKE_CASE`。
- 异常类：以 `Exception` 结尾，例如 `BusinessException`、`ValidationException`。
- 工具类：以 `Utils` 或 `Helper` 结尾，例如 `DateUtils`、`JwtHelper`。
- 配置类：以 `Config`、`Configuration` 结尾，例如 `SecurityConfig`、`WebMvcConfiguration`。
- 实体类：名词，避免带表后缀，必要时用领域名：`User`、`MoodRecord`。
- DTO/VO：以用途后缀，如 `UserRequest`、`UserResponse`、`MoodRecordDTO`；与 Entity 分离。
- 测试类：与被测类同名，加 `Test` 后缀，包路径与主代码保持一致。

## 变量、方法与常量
- 变量/方法：`lowerCamelCase`，方法用动词开头表达动作：`createUser`、`listByUserId`。
- 常量：`UPPER_SNAKE_CASE`，放在 `static final` 字段，尽量集中在靠近使用处或常量类。
- 布尔变量/方法：使用 `is/has/should/can` 等前缀：`isEnabled`、`hasPermission`。
- 集合命名用复数：`users`、`records`；单个对象用单数：`user`。
- 避免魔法值：用常量或枚举表达业务含义。

## 数据库与持久化
- 实体字段名与数据库列保持清晰映射；使用 `@Column(name = "column_name")` 解决命名差异。
- 数据库表名、列名推荐使用下划线小写（如团队约定）；在代码中保持驼峰命名并用注解映射。
- Repository 方法命名遵循 Spring Data 规则：`findByUsername`、`findByUserIdAndStatus`。

## 接口与路由
- REST 路由使用 kebab-case 或复数资源名：`/api/users`、`/api/mood-records/{id}`。
- 控制器方法以资源 + 动作命名：`listUsers`、`createUser`、`getMoodRecordsByUser`。

## 其他建议
- 避免缩写和模糊命名；如需缩写确保团队内广泛接受（如 `DTO`、`VO`、`ID`）。
- 新建文件/类时遵守现有项目的惯例，保持一致性优先于个人偏好。
- 文档/配置文件同样保持命名统一：`README.md`、`application-dev.properties`、`docker-compose.yml`。
