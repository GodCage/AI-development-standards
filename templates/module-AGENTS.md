# <Module Name> Module

- 模块唯一ID：<填写>
- 分组及完整包路径：<core/supporting/integrations/platform；填写真实路径>
- 负责人：<填写>
- 状态：<试点/已接入/迁移中>

## 职责

- <本模块负责的业务能力>

## 不负责

- <明确不属于本模块的能力及其归属模块>

## 主要入口

- Command：`<CreateXxxHandler>`
- Query：`<GetXxxHandler>`
- Web / MQ / Job：`<入口类>`

## 对外 API

- `<XxxCommandFacade>`：<用途>
- `<XxxQueryFacade>`：<用途>
- `<XxxEvent>`：<发布条件与语义>

除 `api` 中明确公开的类型外，其他模块不得依赖本模块实现。

## 所需 SPI（仅模块外实现时公开）

- `<ExternalXxxPort>`：<用途及实现位置>

模块内 Repository 放 internal 的端口包，不公开内部领域模型。

## 外部依赖

- `<OtherModule api>`：<调用原因>
- `<Platform capability>`：<使用的基础能力>

不得新增未在此处登记的跨模块依赖。确需新增时，先更新说明并通过架构评审。

## 数据归属

- 表：`<table_name>`
- 缓存：`<cache key prefix>`
- Topic：`<topic name>`

其他模块不得直接修改本模块拥有的数据。

## 业务不变量

- <任何情况下都必须成立的业务规则>

## 模块局部约束

- <只能在本模块适用的补充规则>

## 底座使用与权限

- MyBatis/Entity/审计/错误协议采用的Platform契约：<填写>
- 接口或用例所需权限、资源归属和租户约束：<填写>
- 不得定义全局安全链、数据源或异常Advice；确需扩展先审批。
- 模块级集成测试与生产代码同包；记录需加载或替换的Platform依赖。

## 验证命令

```bash
<module build command>
<module test command>
```

