---
name: novel-specify
description: 定义故事需求 — 前提、主题、世界观、约束
---

# Novel Specify — 定义故事需求

## 目的

基于已建立的 constitution，引导用户定义故事的具体需求：前提概要、主题、世界观和硬约束。

## 触发方式

用户调用 `/novel-specify`

## 前置条件

- `constitution/principles.yaml` 已填写（至少 `core_thesis` 非空）
- 如未满足，提示用户先执行 `/novel-constitution`

## 工作流程

### 第一步：加载上下文

1. 读取 `constitution/principles.yaml` — 获取核心主旨和主题关键词
2. 读取 `specification/` 下所有文件，检查已有内容
3. 如果已有内容，展示并询问是"补充"还是"重建"

### 第二步：故事前提 (premise.yaml)

引导用户定义：

- **Logline** — 一句话概括故事核心冲突（格式建议：谁 + 在什么处境下 + 想要什么 + 面对什么阻碍）
- **Synopsis** — 200~500 字的长概要
- **Genre** — 主类型 + 副类型
- **目标读者**
- **预计体量**

确保 logline 与 `principles.yaml` 中的 `core_thesis` 一致。如果不一致，指出并请用户调和。

### 第三步：主题 (themes.yaml)

- 从 `principles.yaml` 的 `theme_keywords` 出发，与用户逐一确认每个关键词的具体含义
- 为每个主题确定权重（primary / secondary / minor）
- 初步关联角色和情节组（可留空，后续由 `/novel-character` 和 `/novel-plan` 填充）

### 第四步：世界观 (world-building.yaml)

根据故事类型引导用户定义：

- 时代背景
- 地理/空间
- 社会结构
- 魔法/科技体系（如适用）
- 重要历史事件

对于现实题材，此部分可以精简。

### 第五步：硬约束 (constraints.yaml)

- 字数约束
- 结构约束（卷数、章节命名）
- 内容分级和敏感话题
- 发布平台和更新频率

### 第六步：一致性检查

在所有文件写入后，自动执行以下检查：

- logline 的核心冲突是否呼应 core_thesis
- themes 是否与 theme_keywords 一致
- 世界观设定是否支撑故事前提
- 约束条件之间是否矛盾

将不一致项报告给用户。

### 第七步：确认

展示完整的 specification 内容，请用户确认。

## 输出文件

- `specification/premise.yaml`
- `specification/themes.yaml`
- `specification/world-building.yaml`
- `specification/constraints.yaml`

## 注意事项

- 每个文件独立更新，不要因为改动 premise 而覆盖已有的 world-building
- 如用户在此过程中发现需要修改 constitution，建议用户先执行 `/novel-constitution` 再回来
- 标记所有未确定的字段为 `TBD`，建议后续用 `/novel-clarify` 解决
