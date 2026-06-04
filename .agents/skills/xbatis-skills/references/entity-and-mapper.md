# 实体与 Mapper 约定

## 实体注解

这些注解的真实包名都在 `cn.xbatis.db.annotations` 下。

- `@Table`
  - 定义表名、`schema`
  - 可设置 `columnNameRule` 与 `databaseCaseRule`
- `@TableId`
  - 支持自增、生成器、SQL 生成等主键策略
  - 可按数据库类型差异化配置
- `@TableField`
  - 控制字段是否参与 `select` / `insert` / `update`
  - `neverUpdate` 可彻底禁止字段更新
  - `exists = false` 可标记非表字段
  - 支持 `jdbcType`、`typeHandler`
  - 支持 `defaultValue` / `updateDefaultValue`
- `@LogicDelete`
  - 定义删除前后值
  - `afterValue` 支持动态值
- `@LogicDeleteTime`
  - 记录逻辑删除时间
- `@TenantId`
  - 标记租户字段
- `@Version`
  - 标记乐观锁字段

生命周期注解不在 `cn.xbatis.db.annotations`，真实包名是：

- `cn.xbatis.listener.annotations.OnInsert`
- `cn.xbatis.listener.annotations.OnUpdate`

## 默认 Mapper 模式

默认推荐每个实体一个 Mapper：

```java
public interface SysUserMapper extends MybatisMapper<SysUser> {
}
```

特点：

- `QueryChain.of(sysUserMapper)` 可以自动推断实体类型
- 对当前实体的简单查询，通常可以省略 `select` / `from` / `returnType`
- 更适合中小项目和边界清晰的业务模块

`MybatisMapper<T>` 默认覆盖的能力包括：

- 查询：`get`、`list`、`exists`、`count`、`paging`、`cursor`、`mapWithKey`
- 新增：`save`、`saveBatch`、`saveOrUpdate`、`saveModel`
- 更新：`update`、`updateBatch`
- 删除：`deleteById`、`deleteByIds`、`delete(where)` 等
- 原生 SQL：`execute`、`select`、`selectList`
- 多库差异化：`dbAdapt(...)`

## 单 Mapper 模式

当实体很多、希望统一访问层时，再使用 `BasicMapper`：

```java
public interface MybatisBasicMapper extends BasicMapper {
}
```

使用要求：

1. `@MapperScan(..., markerInterface = BasicMapper.class)`
2. 启动阶段执行 `XbatisGlobalConfig.setSingleMapperClass(MybatisBasicMapper.class)`
3. 链式 API 里显式传入实体类型

示例：

```java
QueryChain.of(mybatisBasicMapper, SysUser.class)
        .eq(SysUser::getId, 1)
        .list();
```

单 Mapper 模式下的要点：

- DSL 必须知道实体类，否则无法推断表和字段
- 复杂 XML 调用通过 `withSqlSession(...)`
- 如果代码库已经采用单 Mapper，就继续沿用，不要混入大量实体 Mapper

## 写入冲突策略

README 说明 `save` / `saveBatch` / `InsertChain` 支持 `onConflict`：

- 冲突忽略：`doNothing()`
- 冲突更新：`doUpdate(...)`

这是跨数据库封装能力，适合主键或唯一键冲突场景。

## 生成代码时的默认选择

- 除非仓库已经统一使用 `BasicMapper`，否则默认生成 `MybatisMapper<T>`
- 实体字段优先用注解表达映射规则，不要把映射逻辑分散到业务层
- 默认使用 Lambda 属性引用，不写硬编码列名
