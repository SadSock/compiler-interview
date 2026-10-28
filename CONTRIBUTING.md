# 贡献指南

## Git 提交信息

提交信息使用以下格式：

```text
<领域>: <动词> <简要说明>
```

`<领域>` 表示本次修改所属的主题，使用小写英文。当前约定的领域包括：

- `cfa`：控制流分析（Control-Flow Analysis）
- `isa`：指令集架构（Instruction Set Architecture）
- `repo`：仓库级维护，例如贡献规范或目录调整

新增主题时，可以使用新的小写英文领域名；多个单词使用连字符连接。

由于本仓库本身以文档为主，不使用泛化的 `docs` 前缀。

`<动词>` 使用英文原形动词，例如 `add`、`design`、`revise`、`clarify`、`reorganize` 或 `remove`。

`<简要说明>` 应简洁说明修改的题目或文件：使用小写英文开头，末尾不加句号。

每个提交只应包含一个逻辑变更。

### 示例

```text
cfa: add control-flow analysis questions
isa: add carry and overflow question
isa: add integer arithmetic question
repo: clarify contribution guidelines
```

避免使用信息量不足的提交信息，例如：

```text
update docs
fix stuff
```
