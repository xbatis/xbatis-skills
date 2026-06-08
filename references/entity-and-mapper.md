# Entity And Mapper

适用于以下场景：

- 新增或审查实体注解
- 判断 `@TableField` 是否必要
- 选择 `MybatisMapper<T>` / `BasicMapper` 的默认 CRUD 能力
- 新增或修改用于写入的 Model / POJO
- 单条数据先查后改，需要精准局部修改
- 处理主键或唯一键冲突写入

## 实体注解

实体类注解真实包名优先按本地 xbatis 源码确认；常见注解在 `cn.xbatis.db.annotations` 下。

- `@Table`
  - 定义表名、`schema`
  - 可设置列名规则和数据库大小写规则
- `@TableId`
  - 支持自增、生成器、SQL 生成等主键策略
  - 可按数据库类型差异化配置
- `@TableField`
  - 控制字段是否参与 `select` / `insert` / `update`
  - 可配置 `neverUpdate`、`exists = false`
  - 可配置 `jdbcType`、`typeHandler`
  - 可配置 `defaultValue` / `updateDefaultValue`
- `@LogicDelete`
  - 定义删除前后值
  - `afterValue` 支持动态值
  - 字段已有逻辑删除注解时，禁止通过 `@TableField` 配置 `defaultValue` / `updateDefaultValue` 等默认值相关属性
- `@LogicDeleteTime`
  - 记录逻辑删除时间
  - 字段已有逻辑删除注解时，禁止通过 `@TableField` 配置 `defaultValue` / `updateDefaultValue` 等默认值相关属性
- `@TenantId`
  - 标记租户字段
- `@Version`
  - 标记乐观锁字段

生命周期注解不在 `cn.xbatis.db.annotations` 下，真实包名是：

- `cn.xbatis.listener.annotations.OnInsert`
- `cn.xbatis.listener.annotations.OnUpdate`

## 枚举约定

新增、生成或改写持久化枚举时，枚举类必须完整遵循 xbatis 枚举契约，并实现/继承（Java 代码使用 `implements`）真实接口 `cn.xbatis.core.mybatis.typeHandler.EnumSupport<T>`：

```java
public enum Status implements EnumSupport<Integer> {

    ENABLED(1),
    DISABLED(0);

    private final Integer code;

    Status(Integer code) {
        this.code = code;
    }

    @Override
    public Integer getCode() {
        return code;
    }

    public static Status of(Integer code) {
        for (Status item : values()) {
            if (java.util.Objects.equals(item.getCode(), code)) {
                return item;
            }
        }
        return null;
    }
}
```

规则：

- 枚举通过 `getCode()` 返回数据库存储值
- 枚举生成时必须提供 `of(T code)` 静态方法，找不到匹配项返回 `null`
- `T` 按数据库字段实际类型选择，例如 `String`、`Integer`、`Long`
- 不要默认依赖 `ordinal()`、`name()` 或手写 TypeHandler
- 需要返回枚举名称或展示值时，优先配合 `@PutEnumValue`

## 字段注解规则

- 创建时间字段优先使用 `@TableField(defaultValue = "{NOW}", update = false)`
- 修改时间字段优先使用 `@TableField(defaultValue = "{NOW}", updateDefaultValue = "{NOW}", updateDefaultValueFillAlways = true)`
- 普通字段无特殊语义时，不要为了声明列名机械补 `@TableField`
- 只有需要控制默认值、更新行为、类型处理、查询参与、非表字段等特殊语义时才补 `@TableField`
- 实体类只保留数据库表字段；VO、QO、DTO、Model 中需要忽略的字段优先使用 `@Ignore` 或 `@Ignores`
- 实体注解只能写在实体类上，禁止写在 VO、DTO、QO、Model 等非实体类上

## Model 修改约定

新增或修改用于新增、更新的 POJO 时，优先判断是否应作为 xbatis Model 使用。

规则：

- 修改操作优先使用 Model 类作为传参载体，收敛本次业务允许写入的字段
- Model 可以像实体类一样直接参与 `save(Model)`、`update(Model)`
- 能使用 Model 直接写入时，不要再编写 DTO / Model / 实体之间的手工转换代码
- 只有接口契约隔离、跨上下文复用、聚合字段拆装等真实需求明确存在时，才保留额外 DTO 到 Model / 实体的转换层
- Model 中凡是不参与数据库操作、需要忽略的字段，优先使用 `@Ignore` 或 `@Ignores`

## Mapper 默认能力

多 Mapper 模式下，一个实体通常对应一个 `MybatisMapper<T>`：

```java
public interface SysUserMapper extends MybatisMapper<SysUser> {
}
```

特点：

- `QueryChain.of(sysUserMapper)` 可以自动推断实体类型
- 对当前实体的简单查询，通常可以省略 `select` / `from` / `returnType`
- 更适合中小项目和边界清晰的业务模块

`MybatisMapper<T>` 默认覆盖：

- 查询：`get`、`list`、`exists`、`count`、`paging`、`cursor`、`mapWithKey`
- 新增：`save`、`saveBatch`、`saveOrUpdate`、`saveModel`
- 更新：`partialUpdate`、`update`、`updateBatch`
- 删除：`deleteById`、`deleteByIds`、`delete(where)`
- 原生 SQL：`execute`、`select`、`selectList`
- 多库差异化：`dbAdapt(...)`

按 id 查询优先复用内置方法，不默认写 `QueryChain`：

- 返回完整实体：`getById(id)`
- 返回单表部分列：`getById(id, Entity::getName, Entity::getStatus)`
- 返回单列值：`getValueById(id, Entity::getName)`

单 Mapper 模式下，统一 Mapper 继承 `BasicMapper`，链式 API 里显式传入实体类型：

```java
QueryChain.of(xbatisMapper, SysUser.class)
        .eq(SysUser::getId, 1)
        .list();
```

单 Mapper 模式不要主动生成或调用 `XbatisGlobalConfig.setSingleMapperClass(...)`。

## 精准局部修改

`partialUpdate(...)` 适合单条数据“先查询出来，再改部分字段”的场景。

```java
SysUser sysUser = sysUserMapper.getById(id);

int cnt = sysUserMapper.partialUpdate(sysUser, o -> {
    o.setUserName(updateModel.getUserName());
    o.setMobile(updateModel.getMobile());
});
```

规则：

- 第一个参数传已查询出的实体对象
- 回调里只 `set` 本次业务允许修改的字段
- `setXxx(null)` 表示把该字段更新为数据库 `NULL`
- 不要把全量实体直接交给 `update(entity)` 代替局部修改

显式置空示例：

```java
SysUser sysUser = sysUserMapper.getById(id);

int cnt = sysUserMapper.partialUpdate(sysUser, o -> {
    o.setUserName(null);
});
```

不推荐写法：

```java
SysUser sysUser = sysUserMapper.getById(id);
sysUser.setUserName(updateModel.getUserName());
sysUser.setMobile(updateModel.getMobile());

sysUserMapper.update(sysUser);
```

原因：这类先查后改的单条局部更新应使用 `partialUpdate(...)` 收窄更新字段，避免普通 `update(entity)` 携带不该由当前业务修改的字段。

## DAO 注入约定

如果项目使用 DAO 层，先创建项目级 BaseDao：

- 单 Mapper 模式：项目 BaseDao 继承 `BasicDaoImpl<T, ID>`
- 多 Mapper 模式：项目 BaseDao 继承 `DaoImpl<T, ID>`
- 业务 DAO 必须拆成接口和实现类
- 业务 DAO 接口继承 `cn.xbatis.core.mvc.Dao<T, ID>`
- 业务 DAO 实现类继承项目 BaseDao，并实现对应业务 DAO 接口
- 不要让业务 DAO 接口继承 `cn.xbatis.core.mvc.IDao<T, ID>`
- 项目 BaseDao 的 `setMapper(...)` 方法必须加当前容器框架的自动注入注解
- `setMapper(...)` 内部调用父类 `setMapper(mapper)` 完成绑定
- 业务 DAO 实现类只继承项目 BaseDao，不需要也不应重复重写 `setMapper(...)`
- 业务 DAO 只保留有业务语义、组合查询、事务边界或复用价值的方法

容器注解跟随当前项目：

- Spring 项目跟随 `@Autowired`、`@Resource` 等既有规范
- Solon 项目跟随 `@Inject`、`@Db` 等既有规范

## 写入冲突策略

`save` / `saveBatch` / `InsertChain` 可按项目实际依赖确认是否支持 `onConflict`：

- 冲突忽略：`doNothing()`
- 冲突更新：`doUpdate(...)`

这是跨数据库封装能力，适合主键或唯一键冲突场景。生成代码前必须确认当前 xbatis 版本和数据库方言是否支持。

## 生成代码时的默认选择

- 除非仓库已经统一使用 `BasicMapper`，否则默认生成 `MybatisMapper<T>`
- 实体字段优先用注解表达映射规则，不要把映射逻辑分散到业务层
- 默认使用 Lambda 属性引用，不写硬编码列名
- 不知道类路径、真实类名、方法名、注解包名或泛型签名时，先用本地 xbatis 源码确认，不要猜测
