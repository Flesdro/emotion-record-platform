# Java 常用语法速查与示例（项目实战向）

覆盖日常项目中高频的语法/特性，按主题附短示例。示例基于 Java 17。

## 基本
- **变量与类型**：`int age = 18; double rate = 0.1; boolean ok = true;`
- **引用类型**：`String name = "Alice"; List<String> list = new ArrayList<>();`
- **类型推断 (局部)**：`var total = 10 + 5;`（只能用于局部变量）
- **常量**：`final int MAX = 10;`
- **包装类型**：`Integer`, `Long`, `Boolean` 等（可为空，用于集合/泛型）
- **字符串**：`String s = "hi" + name; s.toUpperCase(); s.contains("x");`
- **文本块**：多行字符串 `"""line1\nline2"""`（注意缩进对齐）

## 运算与控制流
- **算术/比较/逻辑**：`+ - * / %`，`== != < >`，`&& || !`
- **分支**：`if/else`，`switch`（支持字符串、枚举、模式匹配增强）
  ```java
  switch (status) {
      case "OK" -> handleOk();
      case "FAIL" -> handleFail();
      default -> throw new IllegalStateException();
  }
  ```
- **循环**：`for`、`while`、`do-while`
  ```java
  for (int i = 0; i < n; i++) {}
  for (String item : list) {}
  ```

## 方法与重载
- 定义：`public int add(int a, int b) { return a + b; }`
- 重载：同名不同参数。可选参数通过重载或建造者实现。
- 可变参数：`void log(String... msgs)` → `log("a", "b")`

## 类、封装、继承、多态
- 定义字段/方法，访问修饰符：`public`, `protected`, `private`, 包可见（缺省）
- 构造器：`public User(String name) { this.name = name; }`
- 继承：`class Student extends Person {}`；多态通过父类型引用调用子类实现
- `super` 访问父类成员；`final` 阻止继承或重写

## 接口与抽象类
- 接口定义能力：`interface Repository { void save(); }`
- 默认/静态方法：接口可带 `default` 方法，`static` 工具方法
- 抽象类可包含状态与部分实现：`abstract class BaseService { abstract void run(); }`

## 记录类型 (record)
- 用于不可变数据载体，自动生成构造/访问器：`public record UserDto(Long id, String name) {}`

## 枚举 (enum)
- 有限常量集，可含字段/方法：
  ```java
  public enum Role {
      USER("普通用户"), ADMIN("管理员");
      private final String label;
      Role(String label) { this.label = label; }
      public String label() { return label; }
  }
  ```

## 泛型
- 声明：`List<String>`，`Map<Long, User>`
- 泛型方法：`public static <T> T first(List<T> list) { return list.get(0); }`
- 通配符：`List<? extends Number>` 读，`List<? super Integer>` 写

## 集合与流 (Streams)
- 集合：`List`, `Set`, `Map`；不可变集合：`List.of(...)`
- 流式操作：
  ```java
  List<String> names = users.stream()
      .filter(u -> u.getAge() > 18)
      .map(User::getName)
      .sorted()
      .toList();
  ```
- Optional 处理空值：`Optional.ofNullable(x).orElse("default");`

## 异常
- 受检/非受检：`IOException`(受检) vs `RuntimeException`(非受检)
- 抛出：`throw new IllegalArgumentException("bad");`
- 捕获：`try { ... } catch (IOException e) { ... } finally { ... }`
- 自定义异常继承 `RuntimeException` 或 `Exception`

## 注解
- 标记/元信息：`@Override`, `@Deprecated`, `@SuppressWarnings`
- 自定义注解：定义 `@interface`，可用作元数据驱动 AOP/校验

## Lambda 与函数式接口
- 函数式接口：单抽象方法，如 `Runnable`, `Supplier<T>`, `Function<T,R>`
- Lambda 写法：`() -> doSomething()`；`(a, b) -> a + b`
- 方法引用：`User::getName` 等价于 `u -> u.getName()`

## I/O 与资源管理
- Try-with-resources 自动关闭：
  ```java
  try (var reader = Files.newBufferedReader(Path.of("a.txt"))) {
      return reader.readLine();
  }
  ```

## 并发基础
- 线程：`new Thread(() -> work()).start();`
- 线程池：`ExecutorService pool = Executors.newFixedThreadPool(4);`
- CompletableFuture：
  ```java
  CompletableFuture.supplyAsync(() -> fetch(), pool)
      .thenApply(this::convert)
      .exceptionally(ex -> fallback());
  ```
- 同步工具：`synchronized`、`ReentrantLock`、`CountDownLatch`、`Semaphore`

## 反射与注解处理
- 反射获取类/字段/方法：`Class<?> c = obj.getClass(); c.getDeclaredFields();`
- 常用于框架内部（如 Spring）；业务层尽量少用

## 模块与可见性 (简述)
- `module-info.java` 描述模块依赖/导出包（大多项目未开启 JPMS）

## Spring 项目高频注解/用法（简述）
- 核心：`@SpringBootApplication`
- Web：`@RestController`, `@RequestMapping`, `@GetMapping`, `@PostMapping`, `@RequestBody`, `@PathVariable`, `@RequestParam`
- 依赖注入：`@Component`, `@Service`, `@Repository`, `@Configuration`, `@Bean`, 构造注入（配合 `@RequiredArgsConstructor`）
- 校验：`@Valid`, `@NotNull`, `@NotBlank`, `@Email` 等
- 数据：`@Entity`, `@Id`, `@GeneratedValue`, `@Column`, `@Table`, `@ManyToOne`, `@OneToMany`, `JpaRepository`
- 事务：`@Transactional`
- 安全：`@EnableWebSecurity`, `UserDetailsService`, `PasswordEncoder`

## 风格与建议
- 优先使用不可变数据（record 或私有字段+无 setter）暴露读接口
- 避免空指针：用 `Optional` 或早期校验
- 控制作用域：最小可见性原则，`private` 优先
- 早抛异常，清晰报错信息；避免吞异常
- 结合单元测试验证逻辑，Mock 外部依赖
