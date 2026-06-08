---
name: xbatis-skills
description: 在使用 xbatis 的 Java 项目中实现、改写、审查或优化 ORM / Mapper / QueryChain / 联表 / 分页 / VO 映射 / 多租户 / 逻辑删除 / 乐观锁 / SQL 模板相关代码时使用。适用于 Spring Boot 2/3/4、Solon，以及需要让 AI Agent 严格遵循 xbatis 原生风格、优先使用框架能力并控制 XML / 原生 SQL 回退边界的任务。
---

# Xbatis Skills

使用这份 Skill 时，把 xbatis 当作自己的 ORM 体系处理。优先使用 xbatis 原生能力，不要套用 JPA、Hibernate、MyBatis XML 或其他 ORM 的默认写法。

## 使用边界

适用任务：

- 新增或修改 xbatis 实体、Mapper、DAO、Service、Controller 持久化代码
- 新增或修改 VO、QO、DTO、Model、OrderBy 等 xbatis 相关 POJO
- 接入 Spring Boot / Solon 运行环境、starter、数据源、Mapper 扫描、安全检查
- 实现 CRUD、分页、搜索、联表、VO 映射、对象转条件、对象转排序
- 接入逻辑删除、多租户、乐观锁、动态值、动态默认值、持久化枚举
- 使用 SQL 模板、动态 where、动态 query、原生 SQL 包装或 XML
- 审查现有 xbatis 代码是否偏离框架习惯

不适用任务：

- 纯 JPA / Hibernate 项目
- 不使用 xbatis 的普通 MyBatis 项目
- 仅做宣传介绍、安装教程、用户导览的任务

## 核心决策

始终按这个顺序选择实现方式：

1. DAO 层、BaseDao、内置 Mapper 方法
2. DAO 内部的 `queryChain()` / `updateChain()` / `insertChain()` / `deleteChain()`
3. 查询对象、Model、VO 自动映射、结果映射注解、`returnType`
4. SQL 模板、动态 where、动态 query、数据库函数
5. 少量 XML / 原生 SQL

先判断能否直接使用框架原生能力；再判断是否需要映射、对象条件或模板；最后才回退 XML 或原生 SQL。

避免默认倾向：

- 不要跳过 DAO 层直接在 Service / Controller 创建 Chain
- 不要为简单 CRUD、标准列表页、普通联表先写 XML
- 不要为列表查询手工维护 count + limit 两套 SQL
- 不要为联表返回先查实体再手工 copy VO
- 不要把“复杂”直接等同于“只能 XML”

## 启用流程

1. 识别任务类型：简单 CRUD、标准列表查询、搜索表单、联表 / VO 查询、写入 Model / POJO 改造、高复杂度报表 SQL、框架特性接入、代码审查。
2. 读取当前项目：确认 Spring Boot / Solon 版本、依赖、Mapper 模式、DAO 层、分页器、逻辑删除、多租户、乐观锁、VO / DTO 命名习惯。
3. 按需读取下面的 `references/`，不要一次性加载全部参考文件。
4. 先从当前项目依赖和已有源码确认真实类名、包名、泛型、方法签名和注解；仍不确定时，再使用本地 xbatis/starter 源码或下载源码确认。
5. 生成或审查代码时跟随项目已有模式；新项目或用户未指定时，优先推荐单 Mapper + DAO。
6. 提交答案前用相关 reference 的检查项复核，不确定的 API 不要写成确定代码。

xbatis / starter 源码位置需要时从这些仓库确认：

- xbatis: `https://github.com/xbatis/xbatis` 或 `https://gitee.com/xbatis/xbatis`
- Spring Boot starter: `https://github.com/xbatis/xbatis-spring-boot-parent` 或 `https://gitee.com/xbatis/xbatis-spring-boot-parent`
- Solon starter: `https://github.com/xbatis/xbatis-solon-plugin` 或 `https://gitee.com/xbatis/xbatis-solon-plugin`

## Reference 路由

按任务选择最相关的文件读取：

- Spring Boot / Solon 接入、starter 依赖、数据源、启动类、`@MapperScan`、POJO 安全检查：`references/environment-setup.md`
- 实体注解、`@TableField`、持久化枚举、Mapper 默认能力、Model 写入、精准局部修改、写入冲突：`references/entity-and-mapper.md`
- 单 Mapper / 多 Mapper 判断、BaseDao、DAO 注入、Mapper 组织方式、自定义 Mapper 方法边界：`references/mapper-modes.md`
- 链式 DSL、`QueryChain`、`InsertChain`、`UpdateChain`、`DeleteChain`、对象驱动条件、结果映射注解、`@Fetch`：`references/query-dsl.md`
- 标准查询、分页、搜索、exists / count、联表、VO 返回、join 风险：`references/query-strategy.md`
- 逻辑删除、多租户、乐观锁、动态值、对象转条件、对象转排序、枚举展示：`references/framework-features.md`
- SQL 模板、动态 SQL、数据库函数、原生 SQL、跨数据库差异：`references/advanced-sql.md`
- 是否允许 XML / 原生 SQL、XML namespace、单 Mapper / 多 Mapper 下 XML 边界：`references/xml-boundaries.md`
- 代码审查、PR Review、实现偏差排查、严重问题排序：`references/review-checklist.md`

## 项目约束

- 开发环境默认开启 xbatis POJO 安全检查，用于检查 VO、Model、条件对象、排序对象的映射和注解缺口。
- Spring / Spring Boot 项目优先按当前 starter 支持的 `@XbatisPojoCheckScan` 接入；生成 import 前必须从当前项目依赖或本地 starter 源码确认真实包名。
- Solon 项目优先在 `solon.yml` 的 `mybatis.<数据源 bean 名称>.pojoCheck` 下配置扫描范围。
- POJO 安全检查扫描范围二选一：使用 `basePackages`，或使用 `modelPackages` / `resultEntityPackages` / `conditionTargetPackages` / `orderByTargetPackages` 等细分包路径；不要同时生成两种方案。
- 测试环境和生产环境不要求默认开启安全检查，除非项目明确要求。
- 生成新项目骨架、接入 xbatis、生成 VO / Model / QO / OrderBy 对象时，同步补开发环境安全检查扫描范围。
- 无特殊要求时项目集成 Lombok；实体、VO、QO、DTO、Model 优先使用 Lombok，并按需启用 `@FieldNameConstants` 支持字段常量引用。
- Controller 入参超过 2 个，或大概率会继续增加参数时，使用 QO、DTO、Model 等对象接收，并把对象传递到 Service 层。

## Mapper 与 DAO

- 修改代码前先判断项目是单 Mapper 还是多 Mapper；已有项目跟随现有模式，不混入另一套风格。
- 新项目默认优先单 Mapper + DAO。
- 有 DAO 层时，Service / Controller 不直接持有 Mapper，不直接创建 Chain，不拼 SQL / where / order by。
- 创建业务 DAO 前先创建并复用项目级 BaseDao。
- 单 Mapper 模式下项目 BaseDao 继承 `BasicDaoImpl<T, ID>`；多 Mapper 模式下项目 BaseDao 继承 `DaoImpl<T, ID>`。
- 业务 DAO 拆成接口和实现类；接口继承 `cn.xbatis.core.mvc.Dao<T, ID>`，不要继承 `IDao<T, ID>`。
- 项目 BaseDao 的 `setMapper(...)` 负责容器注入并调用父类 `setMapper(mapper)`；业务 DAO 实现类不重复声明 Mapper 字段、构造器注入或 `setMapper(...)`。
- 基础按 id、save、update、delete、count、exists 等能力直接使用 BaseDao / 内置 Mapper，业务 DAO 不写等价简单转发方法。
- 单 Mapper 模式下不主动生成或调用 `XbatisGlobalConfig.setSingleMapperClass(...)`。

详细规则见 `references/mapper-modes.md` 和 `references/entity-and-mapper.md`。

## 实体、POJO 与枚举

- 实体类只承载数据库表字段；实体注解只能写在实体类上，不能写在 VO、DTO、QO、Model 等非实体类上。
- 默认使用 `@Table`、`@TableId`；普通字段无特殊语义时不要机械补 `@TableField("列名")`。
- 只有需要控制默认值、更新行为、类型处理、查询参与、非表字段等特殊语义时才补 `@TableField`。
- 创建时间字段优先使用 `@TableField(defaultValue = "{NOW}", update = false)`。
- 修改时间字段优先使用 `@TableField(defaultValue = "{NOW}", updateDefaultValue = "{NOW}", updateDefaultValueFillAlways = true)`。
- 字段已使用 `@LogicDelete` / `@LogicDeleteTime` 等逻辑删除注解时，禁止再通过 `@TableField` 配置默认值或更新默认值。
- 新增或修改用于新增、更新的 POJO 时，优先判断能否作为 xbatis Model 直接参与 `save(Model)` / `update(Model)`，减少 DTO / Model / 实体之间的手工转换。
- VO、QO、DTO、Model 中凡是不参与数据库操作、需要忽略的字段，优先使用 `@Ignore` 或 `@Ignores`。
- 持久化枚举实现 `cn.xbatis.core.mybatis.typeHandler.EnumSupport<T>`，提供稳定 `getCode()` 和 `of(T code)`；找不到匹配项返回 `null`，不要默认依赖 `name()`、`ordinal()` 或手写 TypeHandler。

详细规则见 `references/entity-and-mapper.md` 和 `references/framework-features.md`。

## 查询与写入

- 按 id 返回实体或单表部分列时优先 `getById`；按 id 返回单列值时优先 `getValueById`。
- 常规多条件查询优先使用查询对象转条件，QO 实现 `cn.xbatis.core.mvc.QO`，在 `where()` 中优先 `WhereUtil.where(this)`。
- 查询条件对象优先使用 `@ConditionTarget`、`@Condition`、`@Conditions`、`@ConditionGroup`；动态排序对象优先使用 `@OrderByTarget` / `@OrderBy`。
- 只有搜索表单、后台筛选页、关键词查询等搜索查询才优先使用 `forSearch(true)`；非搜索场景使用 predicate 重载、boolean `when` 重载或显式业务判断。
- 成组 `or` 条件使用 `.nested(...)`；调用 `.or()` 后需要时用 `.and()` 切回。
- 适合“先查 A，顺带带出 B”的场景优先考虑 `@Fetch`；真正需要 join 时先 `.from(...)` 明确主表，并优先使用方法引用 join。
- 返回 VO 时优先 `returnType(VO.class)`、`select(VO.class)` 和结果映射注解；需要枚举展示值时优先 `@PutEnumValue`。
- 修改操作优先使用 Model 类作为传参载体，并收敛本次业务允许更新的字段。
- 单条数据先查后改部分字段时优先 `partialUpdate(...)` 精准修改。
- `UpdateChain`、`InsertChain`、`DeleteChain` 在 DAO 内创建，最终统一通过 `execute()` 执行。

详细规则见 `references/query-strategy.md`、`references/query-dsl.md` 和 `references/entity-and-mapper.md`。

## 框架特性与 SQL 边界

- 逻辑删除、多租户、乐观锁、动态值、动态默认值优先使用框架能力，不在业务代码里重复手工拼条件或填字段。
- 需要 SQL 模板时，命令模板优先 `Methods.tpl(...)`，函数模板优先 `Methods.fTpl(...)`，条件模板优先 `Methods.cTpl(...)`。
- XML / 原生 SQL 是后置选项，只用于极重报表 SQL、xbatis 原生表达明显不清晰、数据库语义很强、或现有系统已成熟使用 XML 且要求延续的场景。
- 选择 XML 前必须先判断 QueryChain、对象条件、结果映射、SQL 模板是否足够表达。

详细规则见 `references/framework-features.md`、`references/advanced-sql.md` 和 `references/xml-boundaries.md`。

## 始终记住这些规则

1. 先识别项目现有 xbatis 风格，再落具体代码；已有项目跟随当前 Mapper、DAO、分页、VO / DTO 习惯。
2. 新项目优先单 Mapper + DAO；有 DAO 层时，数据库访问和事务边界优先收敛在 DAO 方法。
3. 简单 CRUD 用 DAO / BaseDao / 内置 Mapper；按 id 查询优先 `getById`，按 id 查单列值优先 `getValueById`。
4. 大多数业务查询使用 QO 对象转条件，并在 DAO 内通过 `queryChain(qo.where())` 执行。
5. 搜索查询才使用 `forSearch(true)`；非搜索条件忽略优先 predicate 重载，其次 boolean `when` 重载。
6. 成组 `or` 条件使用 `.nested(...)`；调用 `.or()` 后需要时用 `.and()` 切回。
7. 适合“先查 A，顺带带出 B”的场景优先考虑 `@Fetch`；需要 join 时先 `.from(...)`，再使用方法引用 join。
8. 接口返回优先 VO；联表、展示、聚合结果优先 `returnType(VO.class)`、`select(VO.class)` 和结果映射注解。
9. 实体类只承载数据库表字段；普通字段不机械补 `@TableField("列名")`，非数据库操作字段放到 VO、QO、Model 并用 `@Ignore` / `@Ignores`。
10. 新增或修改写入 POJO 时优先考虑 xbatis Model；单条先查后改部分字段优先 `partialUpdate(...)`。
11. `UpdateChain`、`InsertChain`、`DeleteChain` 在 DAO 内创建，并最终调用 `execute()`。
12. 逻辑删除、多租户、乐观锁、动态值、动态默认值优先框架能力；XML / 原生 SQL 只作为后置选项。
13. 开发环境默认开启 xbatis POJO 安全检查；扫描范围在 `basePackages` 和细分包路径中二选一。
14. 不确定类名、包名、注解、方法签名或泛型时，先查当前项目依赖或本地 xbatis/starter 源码，不要凭空生成。

## 审查重点

做代码审查时，先读 `references/review-checklist.md`，并优先指出这些高风险问题：

- 有 DAO 层却绕过 DAO，在 Service / Controller 直接创建 Chain、持有 Mapper 或拼 SQL。
- 简单 CRUD、常规分页、普通联表直接写 XML 或原生 SQL。
- Mapper 模式混用，或 BaseDao 基类与单 Mapper / 多 Mapper 模式不匹配。
- 业务 DAO 接口继承 `IDao<T, ID>`，或业务 DAO 堆满基础方法的简单转发。
- `forSearch(true)` 用在非搜索查询，或条件忽略逻辑散落在 Service 层。
- QO 未使用对象转条件体系，或 `where()` 没有优先 `WhereUtil.where(this)`。
- Controller / API 直接返回实体类，或为了响应字段污染实体类。
- 普通字段机械补 `@TableField("列名")`，或实体类混入非数据库表字段。
- 逻辑删除字段已有逻辑删除注解，还叠加 `@TableField` 默认值配置。
- 写入 POJO 未优先考虑 xbatis Model，或更新字段范围过大。
- 单条数据先查后改却不用 `partialUpdate(...)`。
- 写入 Chain 漏掉最终 `execute()`。
- 不确定 API、类名、包名、注解或泛型签名时凭空生成代码。

## 输出要求

- 实现代码时，说明读取了哪些项目文件和哪些 reference。
- 代码审查时，按严重程度列出问题，并给出文件 / 行号。
- 不确定的 xbatis API 明确标注“需从当前项目依赖或本地源码确认”，不要写成确定结论。
- 如果生成结果违反上述规则，继续收敛实现方案。
