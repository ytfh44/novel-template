---
name: novel-clarify
description: 通过问答解决模糊性 — 交互式澄清规格文档中的不明确之处
---

# Novel Clarify — 交互式澄清

## 目的

扫描项目中所有标记为 `TBD`、存在矛盾或含糊不清的定义，通过结构化问答与用户逐一解决，并将答案回写到对应的源文件中。

## 触发方式

用户调用 `/novel-clarify`，可选参数：

- 无参数 — 全面扫描
- `<文件名>` — 只扫描指定文件
- `<关键词>` — 只针对包含该关键词的 TBD 项

## 前置条件

- 至少有一个非空的规格文件（constitution / specification / character）

## 工作流程

### 第一步：扫描

1. 遍历以下目录中的所有 YAML 文件：
   - `constitution/`
   - `specification/`
   - `characters/`
   - `plot/`
2. 收集所有值为 `TBD`、空字符串 `""`、`null`、或空列表 `[]` 的字段
3. 检测潜在矛盾（例如：constitution 说"不美化暴力"，但 constraints 的 rating 设为 R）
4. 将发现整理为一个优先级排序的问题列表

### 第二步：分类

将问题分为以下类别：

- **阻塞性** — 不解决将无法继续后续工作（如 core_thesis 为空）
- **重要** — 会影响质量但不阻塞（如角色的 backstory 为空）
- **可选** — 可以在写作中自然补充（如次要角色的 voice_tag）

### 第三步：交互式问答

按优先级从高到低，逐一向用户提问：

- 每次最多提 3 个相关问题
- 提供上下文（如："在 `specification/premise.yaml` 中，`logline` 当前为空"）
- 如果可能，提供建议选项
- 用户可以回答"跳过"来延后处理

### 第四步：回写

将用户的回答**直接写入对应的源文件**，不仅仅存在 session 记录中。

### 第五步：记录 Session

将本次问答的完整记录保存到 `clarification/sessions/<YYYYMMDD-HHMMSS>.yaml`，格式：

```yaml
timestamp: "2024-01-15T14:30:00"
scope: "full-scan"       # full-scan | file:<name> | keyword:<kw>
questions_total: 12
questions_answered: 10
questions_skipped: 2
entries:
  - question: "..."
    context: "file: specification/premise.yaml, field: logline"
    answer: "..."
    written_to: "specification/premise.yaml"
  - question: "..."
    context: "..."
    answer: "SKIPPED"
    written_to: null
```

## 输出文件

- 修改对应的源文件（constitution / specification / characters / plot）
- 新增 `clarification/sessions/<timestamp>.yaml`

## 注意事项

- 回写时使用 view_file + replace_file_content，精确修改对应字段，不要覆盖整个文件
- 如果用户的回答引发了新的矛盾，立即指出并请求澄清
- 记录中保留原问题和原答案，便于日后回溯
