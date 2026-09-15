# <Module Name> Module

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

## 所需 SPI

- `<XxxRepository>`：<用途>
- `<ExternalXxxPort>`：<用途及实现位置>

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

## 验证命令

```bash
<module build command>
<module test command>
```
