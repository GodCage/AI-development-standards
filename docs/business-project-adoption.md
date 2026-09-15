# 业务项目落地指南

状态：规范与接入模板；未在目标业务项目运行。本文是项目设计建议，不是 Spring Modulith 自带的技术底座。主规范见 [开发规范](ai-native-spring-boot-standard.md)。

## 1. 接入原则与交付边界

1. 必须先盘点已有实现，再决定保留、包装或迁移；禁止因为目录不符合规范而直接重写业务。
2. 默认一个 Maven 工程、一个 Spring Boot 应用，以 Java 包作为模块边界。目录隔离不是编译隔离，必须配套测试。
3. 核心域、支撑域是业务分类，不是技术调用栈。外部对接是协议边界，Platform 是技术能力集合。
4. 只在独立构建、跨项目复用、团队边界等真实需求出现时拆 Maven 模块；独立部署另行决策。
5. 必须提交项目接入清单、一个真实用例试点、架构及运行测试结果、分批迁移和回退记录，才能宣称已落地。
6. 本文代码和类名是约定示例。版本、包名、数据字段和认证方案必须经过项目确认，不视为可直接运行的脚手架。

## 2. 项目包与依赖契约

根包以 com.company.product 为占位符，接入时统一替换。ProductApplication 放根包，避免只扫描 bootstrap 子包。测试镜像生产包，模块级集成测试放所属模块包内。

| 来源 | 允许的目标 | 禁止 |
|---|---|---|
| 核心业务模块 | 已登记的支撑/核心模块 api、所需 Platform api | 其他 internal、具体外部适配实现 |
| 支撑业务模块 | 已登记的其他支撑模块 api、Platform api | 默认反向依赖核心域 |
| integrations/sap 等 | 业务模块 spi；入站集成可调用业务 api；所需 Platform api | 业务 internal、替业务做领域决策 |
| Platform | 无环的其他 Platform 契约、框架库 | 任何业务模块或具体集成 |
| 应用装配 | 受控的模块装配入口和实现连接 | 承载业务逻辑 |

所有跨模块依赖必须登记且无环。同组模块也不能随意互调。支撑模块需要通知核心模块时，优先发布自身事实事件，由核心模块监听；事件不是掩盖双向类型依赖的工具。

每个业务模块最低保留 api 和 internal：
- api：公开门面、调用参数、返回值、集成事件；不泄露数据库对象。
- internal/application：事务、用例编排、用例授权。
- internal/domain：按需启用，承载业务不变量；不得依赖 Spring、MyBatis、HTTP。
- internal/adapter/in：HTTP/MQ/Job 入站转换。
- internal/adapter/out/persistence：Entity、Mapper、Repository 实现。
- internal/application/port/out 或 internal/domain/repository：模块内端口。
- spi：只有模块外实现确有需要时建立；端口及传输对象必须自包含，不引用 internal。

Platform 能力包也要划分 api / internal，必要时有 spi。例如 platform.data.api.model.BaseEntity 对持久化适配器开放，platform.data.internal.mybatis.MyBatisConfiguration 不对业务代码开放。platform.core 的共享错误契约放 core.api.error；它仍必须保持纯 Java。

启用 Modulith 时给每个实际能力包独立 ID，例如 platformCore、platformData、platformSecurity、platformWeb；分类父包不标注模块。所有公开 api/spi 包及需要公开的子包显式声明 Named Interface。先验证模块发现清单，再验证依赖；不得把整个 Platform 标为 Open Module 以绕过检测。

## 3. Platform 能力登记与归属

| 能力 | 对业务公开 | 内部实现/装配 | 业务保留 |
|---|---|---|---|
| 数据访问 | data.api.model.BaseEntity（可选） | data.internal.mybatis.MyBatisConfiguration | Mapper、SQL、Entity、查询索引设计 |
| 审计 | 最小操作者/时钟契约 | data.internal.audit 审计填充 | 表是否需要审计 |
| 安全 | security.api.CurrentUserProvider | security.internal 安全链、拒绝处理 | 用例权限、对象归属判断 |
| 权限来源 | security.spi.PermissionLookup | 身份模块适配器实现该端口 | 用户、角色及权限分配 |
| 错误 | core.api.error.ErrorCode / BusinessException | web.internal.error.GlobalExceptionHandler | 各模块错误码与业务异常 |
| HTTP | web.api 的必要响应契约 | web.internal 序列化、参数错误映射 | Request/Response 和接口行为 |
| 可观测性 | 必要的观测接口 | observability.internal 日志脱敏、Tracing | 有业务语义的审计事件 |

平台不提供万能 BaseService、SpringUtils 或能够任意访问 Bean 的入口。公共工具按能力归属；纯函数可以是静态方法，有连接、配置和生命周期的能力使用受控 Bean。已有库够用时不重复包装。

## 4. 配置收口与启动装配

### 4.1 三类配置

| 类别 | 位置 | 变更要求 |
|---|---|---|
| 全局机制与默认值 | Platform 配置类 | 底座负责人评审 |
| 环境参数 | application.yml、环境变量、配置中心 | 运维/平台共同确认，敏感值不入库 |
| 业务策略 | 业务模块配置或数据库 | 对应模块负责，不得覆盖全局安全约束 |

当前普通目录方案使用 Spring 配置类和明确扫描/导入完成装配，不需要先开发 starter。业务 Mapper 扫描仅匹配约定注解/路径，避免扫描 api/spi 接口。应用启动测试必须验证所有预期模块 Bean、Mapper 和安全链存在。

可配置不等于可以任意覆盖。为参数登记默认值、合法范围、是否允许环境覆盖、负责人。关键安全参数缺失或非法时启动失败。禁止业务模块新增第二套全局数据源、安全链、ObjectMapper 或全局异常处理器；确有多数据源等需求时由 Platform 统一登记和配置路由。

后续抽取 starter 时再采用自动配置机制，并验证依赖版本、条件装配和覆盖行为。不要把自动配置的“允许用户覆盖”直接当作安全保证。

### 4.2 MyBatis 与 MyBatis-Plus

- 接入清单必须明确使用 MyBatis 还是 MyBatis-Plus，不能混用两者的配置前缀和扩展接口。
- 原生 MyBatis Starter 的属性与 ConfigurationCustomizer 是统一设置的入口；MyBatis 插件/类型处理器由底座登记。
- 字段自动填充、逻辑删除、租户 SQL 等不是继承 BaseEntity 就能获得的能力。使用 MyBatis-Plus 时按选定版本实现其扩展；原生 MyBatis 需另行实现或显式 SQL。
- 全局确定映射规则、超时、分页最大值、插件顺序。SQL 日志不得泄漏凭证或敏感字段。
- 模块 SQL/XML 留在 src/main/resources/mybatis/<module>/；禁止跨模块直接写表。
- 多租户、分页和数据权限必须覆盖自定义 SQL、批量 SQL、联表及允许绕过场景的负向测试。
- 业务事务在 Application 用例层声明；传播、回滚规则和跨模块一致性策略显式设计。不要用一个全局 AOP 事务规则包住所有方法。

## 5. BaseEntity 和模型边界

BaseEntity 是可选持久化契约，只放真正通用的字段，例如统一类型的主键、创建/修改时间。审计字段可独立组合或建立明确的 AuditedEntity；不是所有表都需要 tenantId、deleted、version。

必须遵守：
- 只有持久化 Entity 继承数据库基类；领域 Order 不继承它，API 也不返回它。
- 字段类型、时区、主键生成、自动填充和批量操作行为写入项目接入清单。
- createdBy 等审计字段从可信身份上下文取得，不信任前端传入值。
- tenantId 存在不代表已经隔离；必须验证查询、更新、删除和后台任务路径。
- BaseEntity 变更按全库影响评估，不属于普通订单任务可自行修改的范围。
- 涉及现有表字段变更必须有迁移脚本、兼容窗口、索引评估和恢复方案。

## 6. 认证、授权和拦截

Platform Security 负责凭证校验、身份上下文、全局请求策略、方法授权机制及 401/403 响应。身份模块负责用户/角色/授权数据，业务模块负责订单归属等资源级判断。

单向依赖示例：
- 平台声明 security.spi.PermissionLookup。
- 身份模块的适配器实现该端口。
- 平台依赖接口，在运行时由 Spring 注入；平台不 import 身份模块。
- 当前用户接口只暴露最小身份信息，不暴露完整用户 Entity。

请求策略必须覆盖全部路径；未明确公开的接口默认要求认证。公开接口白名单集中审查；应用独有白名单由受控装配配置声明。方法级/用例级权限负责更细粒度授权，不能只依赖“已登录”。

| 入口 | 必须检查 |
|---|---|
| HTTP | 无凭证、过期凭证、有效但无权限、跨用户/跨租户 |
| 内部调用 | 用例级授权不能因未经过 Controller 而失效 |
| MQ / Job | 明确机器身份、租户来源和权限，不伪造管理员用户 |
| 异步任务 | 显式传递必要上下文并清理，不假设 ThreadLocal 自动传播 |

Servlet 安全由 Spring Security 过滤链负责，不以 MVC Interceptor 作为唯一安全边界。普通 Web 拦截、SQL 拦截、审计切面按技术能力归属，禁止所有拦截器堆在根 interceptor 包。注册顺序由底座统一维护，并通过启动和请求测试验证。

## 7. 统一错误协议

平台定义纯 Java 错误码接口和必要异常基类；业务模块定义 ORDER_ALREADY_SHIPPED 等具体码。不要建立包含全部业务细节的全局枚举。

- HTTP 层将业务错误映射到稳定响应结构，包含 code、message、traceId；HTTP 状态码必须表达语义。
- 原有成功/错误协议有消费者时，先做兼容评估，不直接改成新格式。
- 未知异常对外返回通用错误，对内保留带 traceId 的堆栈并脱敏。
- 认证过滤链使用自己的认证入口/拒绝处理器，共享错误序列化协议；不能假设 ControllerAdvice 捕获所有过滤器异常。
- MQ 区分可重试和不可重试错误，定义重试上限、死信、幂等和人工处理。
- Job 记录失败状态与可恢复信息，不把 HTTP 错误对象作为所有入口的通用结果。
- 避免每层重复记录同一堆栈，也不得 catch 后吞掉异常或一律返回成功。

## 8. AI 边界和项目治理

模板入口：
- [项目接入清单](../templates/project-adoption.md)
- [根规则](../templates/root-AGENTS.md)
- [模块规则](../templates/module-AGENTS.md)
- [Platform 规则](../templates/platform-AGENTS.md)
- [任务 Scope](../templates/task-scope.yaml)

模板必须替换占位符并确认后使用。业务任务默认 Platform、pom.xml、全局配置、架构测试及 Scope 本身只读/禁止修改；需要修改时单独审批任务范围。依赖白名单、公开 API 变更权、迁移脚本范围都应显式登记。

AGENTS.md 是行为约定，不是安全沙箱。CI 门禁和仓库保护由项目实际配置；文档本身不会自动阻止越界。根规则不能仅放 .ai/AGENTS.md。

Scope 检查必须覆盖文件新增、删除、重命名两端以及本地未跟踪文件。CI 比较目标分支可信基线和任务变更；任务不能修改 YAML 来自我授权。共享仓库中的多任务集成应由集成负责人分配合并后的明确范围，不简单关闭检查。

## 9. 最低验收矩阵

下表是业务项目必须实现的测试，不是本规范仓库已执行的测试。

| 门禁 | 正向验收 | 负向验收 |
|---|---|---|
| 模块识别 | 检测集合与登记表一致 | 漏标新模块/误标父包应失败 |
| 模块边界 | 仅调用登记 API | 引用其他模块 internal 应失败 |
| 分层 | Mapper 仅由持久化层调用 | Domain 引入 MyBatis/Spring 应失败 |
| 平台依赖 | 平台只引用平台契约 | 平台 import 身份业务类应失败 |
| 配置归属 | 受控位置装配全局 Bean | 业务新增全局安全链应失败 |
| 鉴权 | 合法用户完成用例 | 匿名、越权、跨租户请求拒绝 |
| 持久化 | 审计、分页、迁移正确 | 前端伪造审计字段无效 |
| 异常 | 业务码与状态一致 | 未知异常不泄漏 SQL/堆栈 |
| Scope | 允许范围内变更通过 | 未声明新文件、删除/重命名越界失败 |
| 兼容性 | 原调用方、原数据继续工作 | 破坏性 API/DDL 无审批不能合并 |

验证命令在业务仓库锁定：
- Linux/macOS：./mvnw verify
- Windows：mvnw.cmd verify
- 没有 Wrapper 时先使用项目已锁定的 Maven 命令，不能声称 Wrapper 已存在。
- 配置单元测试、模块测试、集成测试、ArchUnit 和 Modulith 全部纳入 verify；只运行 compile 或挑选 Order 测试不等于全部通过。
- JDK/依赖版本、测试数量、成功/失败/跳过及原因写入交付报告。外部测试环境不可用必须标明未验证。

架构规则使用 ArchUnit/Modulith，配置实际效果和安全规则使用启动/集成测试。BOM 管理依赖版本，不管理 JDK 本身；JDK 用编译器 release、Toolchains/Enforcer 和 CI 环境共同约束。

## 10. 从存量项目开始

| 步骤 | 产物 | 完成条件 |
|---|---|---|
| 盘点 | 填写接入清单 | 版本、模块、接口、表、底座现状明确 |
| 定边界 | 模块登记表和依赖图 | 负责人确认，无计划中的循环依赖 |
| 保留既有行为 | 特征测试和 API 快照 | 改目录前有可比较基线 |
| 做一个真实试点 | 一条完整业务用例 | HTTP→用例→DB→错误/授权全部验收 |
| 固化底座 | 技术能力登记及受控配置 | 旧配置不重复装配，调用方兼容 |
| 启用门禁 | CI 和评审规则 | 正向通过、故意违规会失败 |
| 分批迁移 | 每批变更与回退记录 | 无新增越界，历史例外逐项清理 |

历史违规可以使用有负责人、原因、到期时间的精确基线；只禁止新增违规，不无限期豁免整个包。迁移粒度按业务需求，避免先做全库移动导致并行任务冲突。

试点建议选择已有的小型写入用例，不凭空创建示例业务替代真实验证。例如已有订单取消：先保持接口和表不变，迁入 order 包，补充授权和状态规则测试，再验证 Mapper、审计和错误协议。库存恢复及外部通知若包含异步一致性，需验证持久化事件/outbox、幂等与故障恢复，不能假设发布内存事件就可靠。

回退必须分别记录代码、配置、数据库和消息四类。代码可回退不代表数据可逆；破坏性 DDL 优先采用兼容扩展后收缩，不能默认靠自动反向 SQL 恢复。

## 11. 参考

以下来源解释框架机制；本文目录与团队门禁属于项目建议。
- [Spring Modulith 模块与公开接口](https://docs.spring.io/spring-modulith/reference/fundamentals.html)
- [MyBatis Spring Boot 集成与配置扩展](https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)
- [Spring Security Servlet 架构](https://docs.spring.io/spring-security/reference/servlet/architecture.html)

