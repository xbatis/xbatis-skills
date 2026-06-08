# Xbatis AI Skill

面向 AI Agent 的 xbatis 编码与审查规则集，用于在 Java 项目中实现、改写、审查或优化 xbatis 相关持久化代码。

这份 skill 的目标是让 Agent 优先使用 xbatis 原生能力，而不是把项目写成传统 MyBatis XML、JPA 或其他 ORM 风格。

## 适用场景

- Spring Boot / Solon 项目接入 xbatis
- 实体、Mapper、DAO、Service 持久化代码生成或改造
- QueryChain / UpdateChain / InsertChain / DeleteChain 使用
- 分页、搜索、联表、VO 映射、对象转条件、对象转排序
- 逻辑删除、多租户、乐观锁、动态值、动态默认值
- SQL 模板、原生 SQL、XML 回退边界判断
- xbatis 代码审查和风格偏差排查

## 结构

- `SKILL.md`：Agent 触发后读取的主入口，保留核心流程、约束和 reference 路由。
- `references/`：按任务拆分的详细规则，Agent 只在需要时读取相关文件。
- `agents/openai.yaml`：UI 展示和默认调用提示元数据。

## 使用方式

在支持 Codex skills 的环境中安装或启用本仓库后，可以显式调用：

```text
Use $xbatis-skills to implement or review xbatis Java persistence code.
```

Agent 会先读取 `SKILL.md`，再根据任务类型按需读取 `references/` 中的细分规则。

## 设计原则

- 优先 DAO、BaseDao、内置 Mapper 和 xbatis Chain。
- 优先对象转条件、VO 自动映射、结果映射注解和 SQL 模板。
- XML / 原生 SQL 只作为后置选项。
- 不确定的类名、包名、方法签名和注解必须从当前项目依赖或本地 xbatis 源码确认。
