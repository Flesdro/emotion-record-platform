# 登录/注册全流程示例（分层操作清单）

以“用户注册 + 登录”功能为例，说明各层如何协同，从数据库到 API 的完整走法。可按此清单落地并替换为你们的具体命名。

## 1) domain/entity
- 定义 `User` 实体：字段 `id`、`username`（唯一）、`passwordHash`、`role`、`createdAt`、`updatedAt`。
- 如需 Token 黑名单/刷新令牌，另建 `RefreshToken` 实体。

## 2) repository
- `UserRepository`：
  - `Optional<User> findByUsername(String username);`
  - 可选：`boolean existsByUsername(String username);`
- 如有刷新令牌：
  - `RefreshTokenRepository`：`findByToken`、`deleteByUserId`。

## 3) dto
- 请求：
  - `RegisterRequest { @NotBlank username; @NotBlank password; }`
  - `LoginRequest { @NotBlank username; @NotBlank password; }`
- 响应：
  - `UserResponse { id; username; role; }`
  - `AuthResponse { accessToken; refreshToken; user: UserResponse; }`

## 4) security
- 密码加密：`PasswordEncoder` Bean（如 `BCryptPasswordEncoder`）。
- JWT/Session：
  - 若用 JWT：定义 `JwtUtil`（生成/解析）、`JwtAuthenticationFilter`（解析 Header，构建 Authentication）。
  - 若用 Session：启用 Spring Security 默认 Session，登录成功后由框架维护。
- `SecurityConfig`：
  - 配置放行路由：`/api/auth/register`、`/api/auth/login`。
  - 保护其他路由，需要携带认证信息（JWT Header 或 Session）。

## 5) service
- `AuthService` 接口：
  - `UserResponse register(RegisterRequest req);`
  - `AuthResponse login(LoginRequest req);`
  - 如 JWT：`AuthResponse refresh(String refreshToken);`
- `AuthServiceImpl`：
  - `register`：检查重名 → 加密密码 → 保存用户 → 返回 `UserResponse`。
  - `login`：查用户 → 校验密码 → 生成 Token（或启动 Session）→ 返回 `AuthResponse`。
  - 若有刷新：校验刷新令牌有效性，颁发新 Access Token。

## 6) controller
- `AuthController` (`@RestController @RequestMapping("/api/auth")`)：
  - `POST /register` → `AuthService.register`
  - `POST /login` → `AuthService.login`
  - `POST /refresh`（可选）→ `AuthService.refresh`
- 返回统一响应包，如 `ApiResponse<AuthResponse>`。

## 7) exception
- 业务异常：`UserAlreadyExistsException`、`InvalidCredentialsException`。
- 全局处理：`GlobalExceptionHandler` 将异常转为 400/401/403/500 等标准响应。

## 8) config
- `WebConfig`：如需 CORS。
- `SecurityConfig`：注册 `PasswordEncoder`，配置放行/保护的 URL，挂载 JWT 过滤器（如果使用）。

## 9) util
- `JwtUtil`：生成/验证 Token，提取用户名/过期时间。
- `DateUtils` 等辅助按需添加。

## 10) test
- `AuthControllerTest`（集成测试）：启动应用，用 MockMvc 调用 `/register`、`/login`，断言状态码和返回体。
- `AuthServiceTest`（单测）：Mock 仓库与编码器，验证逻辑分支。

## 11) 数据库/SQL
- 建表（示例 MySQL）：
  ```sql
  CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(64) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(32) NOT NULL DEFAULT 'USER',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
  );
  ```
  如用刷新令牌，再建 `refresh_tokens`，包含 `token`、`user_id`、`expires_at`、`revoked`。

## 12) 典型调用链（注册）
1. `POST /api/auth/register` 携带 `username/password`。
2. `AuthController.register` 校验 DTO，调用 `AuthService.register`。
3. `AuthServiceImpl.register`：
   - `existsByUsername` 检查重名，重名则抛业务异常。
   - `PasswordEncoder.encode` 生成哈希。
   - 保存 `User`，返回 `UserResponse`。
4. Controller 包装 `ApiResponse` 返回。

## 13) 典型调用链（登录）
1. `POST /api/auth/login` 携带 `username/password`。
2. `AuthController.login` → `AuthService.login`。
3. `AuthServiceImpl.login`：
   - `findByUsername` 获取用户，不存在抛登录失败异常。
   - `PasswordEncoder.matches` 校验密码。
   - 生成 Access Token（和 Refresh Token，若需要）。
   - 返回 `AuthResponse`（含用户信息和令牌）。
4. 前端后续请求在 Header 携带 `Authorization: Bearer <token>`（或使用 Session Cookie）。

## 14) 常见注意事项
- 密码永不明文存储；使用强哈希（BCrypt/Argon2）。
- 登录失败提示模糊化，避免泄漏用户是否存在。
- Token 设置合理过期时间；刷新令牌可单独存储并支持吊销。
- 控制器层保持瘦；业务放在 Service，数据访问放 Repository。
- 为关键路径编写测试，防回归。
