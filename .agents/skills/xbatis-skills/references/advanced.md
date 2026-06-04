# 多库、租户、分页与高级能力

## 多数据库差异化

`QueryChain` / `InsertChain` / `UpdateChain` / `DeleteChain` 以及 Mapper 都支持 `dbAdapt(...)`。

适用场景：

- 同一逻辑在 MySQL、PostgreSQL、H2、SQL Server 上要生成不同 SQL
- 冲突处理、分页、函数调用需要按库区分

默认做法：

- 先用统一 DSL
- 只有数据库方言差异真的存在时，再包一层 `dbAdapt(...)`

## 动态数据源

README 记录了外部能力 `cn.xbatis:xbatis-datasource-routing`：

- 支持 `@DS("master")`
- 支持主从、分组、连接池与配置解密
- 事务切换时要注意传播行为

这部分属于外部扩展，不在当前仓库模块内实现，使用前先确认依赖是否已接入。

## 多租户

真实类型：

- 注解：`cn.xbatis.db.annotations.TenantId`
- 上下文：`cn.xbatis.core.tenant.TenantContext`

常见约定：

- 实体租户字段标 `@TenantId`
- 启动阶段注册 `TenantContext.registerTenantGetter(...)`
- 需要临时关闭时，让 getter 返回 `null`，或在上层做显式控制

## 逻辑删除

真实类型：

- 注解：`@LogicDelete`、`@LogicDeleteTime`
- 开关：`cn.xbatis.core.logicDelete.LogicDeleteSwitch`
- 工具：`cn.xbatis.core.logicDelete.LogicDeleteUtil`

要点：

- Mapper 删除方法会走逻辑删除能力
- `DeleteChain` 默认是物理删除
- 可用 `XbatisGlobalConfig.setLogicDeleteInterceptor(...)` 补充删除人等字段
- 局部关闭逻辑删除可用 `LogicDeleteSwitch.with(false)` 或 `LogicDeleteUtil.execute(false, ...)`

## 乐观锁

真实注解：`cn.xbatis.db.annotations.Version`

行为约定：

- `save` 时默认写入初始版本
- `update` / `delete` 会带上版本条件
- 成功更新后自动递增

## 分表

真实类型：

- `@SplitTable`
- `@SplitTableKey`
- `TableSplitter`

要点：

- `TableSplitter#support(Class<?>)` 声明支持的分片键类型
- `TableSplitter#split(String, Object)` 计算真实表名
- 链式查询和更新与普通实体基本一致
- 条件中必须带上分表键，否则无法定位真实表

## SQL 函数与模板

优先静态导入：

```java
import static db.sql.api.impl.cmd.Methods.*;
```

常用能力：

- 聚合和数学函数：`count`、`sum`、`avg`、`round`、`pow`
- 字符串函数：`concat`、`upper`、`lower`、`substring`
- 日期函数：`currentDate`、`dateDiff`、`dateAdd`
- 模板：
  - `tpl(...)`
  - `fTpl(...)`
  - `cTpl(...)`

默认策略：

- 能用标准函数就不用模板
- 模板只在表达复杂片段时使用，避免 SQL 字符串到处散落

## XML 整合

单 Mapper 模式下，可通过 `withSqlSession(...)` 调用 XML：

- 命名空间通常是 `MybatisBasicMapper`
- statement 命名可按 `EntityName:method`

适合场景：

- 已有复杂 XML 资产
- 特殊 SQL 很难用 DSL 表达
- 需要和现有 MyBatis 语句直接复用

## 分页与性能

真实分页类型是 `cn.xbatis.core.mybatis.mapper.context.Pager`。

建议：

- 用 `Pager.of(pageNo, pageSize)` 驱动分页
- 默认信任框架的分页优化能力
- 遇到特殊数据库分页语法，再通过 `XbatisGlobalConfig.setPagingProcessor(...)` 定制

README 说明框架会自动优化：

- `COUNT(*)`
- 冗余 `LEFT JOIN`
- 冗余 `ORDER BY`

## 代码生成与安全检查

README 中记录了两类能力，但当前仓库源码未直接提供实现：

- `GeneratorConfig`
- `org.mybatis.spring.boot.autoconfigure.XbatisPojoCheckScan`

因此使用时要区分：

- 可以把它们当成 xbatis 生态里的“接入约定”
- 不能把它们当成当前仓库源码已存在的类型

AI 生成代码时的默认建议：

- 开发环境启用 POJO 安全检查
- 显式配置 `databaseId`
- 查询接口优先 `.forSearch(true)`
- VO 优先 `@ResultEntity` 驱动
- 日志排查时开启 `cn.xbatis` 的 `trace`
