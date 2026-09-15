# Project Rules

复制到业务仓库根 AGENTS.md。实施前填写 docs/architecture/project-adoption.md，替换包名、模块清单和验证命令。

## 开始任务前

1. 阅读本文件、目标模块的 `AGENTS.md` 和当前任务的 `.ai/tasks/<task-id>.yaml`。
2. 确认业务归属、公开契约、数据归属和允许修改范围。
3. 先给出最小变更计划，再开始编码。

## 架构规则

1. 默认单 Maven 工程，domains/core 与 domains/supporting 只做业务分类；integrations 放对接，platform 放技术机制。目录不等于 Maven 模块。
2. 跨业务模块调用只能依赖目标模块的 `api`。
3. 禁止从模块外访问其他模块的 `internal`。
4. `spi` 只供适配器实现，不作为其他业务模块的调用入口。
5. Domain 不依赖 Spring、MyBatis、JPA、Redis、MQ、HTTP Client 或第三方 SDK；不继承持久化 BaseEntity。
6. Controller、MQ Consumer 和 Job 只负责协议适配，不直接访问 Repository。
7. 外部系统通过 Adapter 接入，外部模型不得进入 Domain。
8. 基础技术能力优先使用 Platform；Platform 不得依赖业务模块。
9. 禁止新增无明确语义的顶层 `common`、`utils`、`manager` 包。
10. Mapper、SQL、业务Entity留在模块；全局MyBatis配置、安全链、异常映射只在Platform受控注册。不得自行新增全局覆盖。
11. 新增第三方依赖、跨模块依赖或全局配置时必须说明原因和影响。

## 修改范围

1. 默认只修改 Task Scope 的 `allowed` 路径。
2. `readonly` 路径可以读取，不得修改。
3. 不得修改 forbidden 路径；readonly/forbidden 优先于 allowed，未声明的路径默认不可修改。
4. Scope 本身及门禁使用可信基线，禁止自我修改范围以通过校验。
5. 如完成任务必须越界，暂停修改并请求调整 Scope，不得自行扩大范围。
6. 不顺手重构、不格式化或重命名任务无关代码。

## 质量要求

1. 修复缺陷时优先增加能够复现问题的测试。
2. 新增业务规则时优先编写 Domain 单元测试。
3. 修改公开 API 时检查所有调用方和兼容性。
4. 提交前执行项目锁定的 verify 全量验证及 Scope 检查；按任务补充授权负向、审计、数据迁移和接口兼容测试。不得用 compile 代替验证。
5. 不通过删除测试、放宽规则或增加永久例外来规避失败。

## 完成报告

完成后必须说明：

- 修改了哪些文件及原因；
- 执行了哪些验证及结果；
- 是否改变公开契约、依赖、配置或数据库；
- 尚未解决的风险和建议后续动作。
