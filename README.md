# AI Development Standards

一套面向 AI 协作开发场景的 Java / Spring Boot 工程规范。

它关注的不是“目录看起来整齐”，而是让架构边界同时对研发和编码 Agent 清晰、可验证：

- AI 改动可控：任务允许修改、只读和禁止修改的范围均显式声明；
- 任务可隔离：业务按模块切分，可按模块分配给不同研发或 Agent；
- 代码可读：入口、业务规则、数据访问和外部适配职责明确；
- 扩展可持续：跨模块只依赖公开契约，外部系统通过适配器接入；
- 技术可收口：通用技术能力由 Platform 统一提供，架构规则由自动化测试强制执行。

## 如何用于业务项目

1. 先阅读[业务项目落地指南](docs/business-project-adoption.md)。
2. 将[项目接入清单](templates/project-adoption.md)复制到业务仓库并填写实际版本、模块和底座现状。
3. 使用根/模块/Platform AGENTS.md 模板和任务 Scope，先完成一条真实业务用例试点。
4. 按验收矩阵接入架构测试、安全测试和 CI，再分批迁移。

本仓库是规范与可复制模板，不是可运行的 Spring Boot 脚手架，也尚未接入你的具体业务仓库。默认单 Maven 工程、目录模块化；复杂或需复用时再拆 Maven 模块。Spring Modulith 可直接用于目录模块，无需等待物理拆分。

## 核心方案

默认采用以下组合：

```text
Spring Boot
  + Spring Modulith（业务模块边界）
  + api / spi / internal（代码访问边界）
  + ArchUnit（架构规则自动校验）
  + AGENTS.md（AI 行为与目录规则）
  + Task Scope（单次任务修改边界）
```

一句话原则：

> 业务按领域纵向切分，模块内部按职责分层；跨模块只走 API，外部适配只实现 SPI，基础技术进入 Platform，架构规则进入 Governance，AI 修改范围通过 Task Scope 显式声明。

## 文档导航

- [AI 友好型 Spring Boot 开发规范](docs/ai-native-spring-boot-standard.md)
- [根目录 AGENTS.md 模板](templates/root-AGENTS.md)
- [业务模块 AGENTS.md 模板](templates/module-AGENTS.md)
- [项目接入清单](templates/project-adoption.md)
- [Platform AGENTS.md 模板](templates/platform-AGENTS.md)
- [AI 任务范围模板](templates/task-scope.yaml)

## 推荐的最小落地路径

1. 先按业务能力重组包结构，并建立 `api/internal` 边界；
2. 在仓库根目录和业务模块目录增加 `AGENTS.md`；
3. 接入 Spring Modulith 与 ArchUnit，把关键依赖规则变成测试；
4. 把日志、异常、安全、数据访问等能力逐步收口到 Platform；
5. 使用 Task Scope 分配 AI 任务，并在 CI 中检查越界修改。

本规范推荐“够用的 DDD + 六边形边界”，不要求为了形式创建大量抽象，也不要求项目一开始就拆成大量 Maven 模块。

