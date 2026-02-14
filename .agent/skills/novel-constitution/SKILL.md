---
name: novel-constitution
description: 建立核心创作原则 — 定义美学、主旨、价值观、契诃夫之枪规则和文风
---

# Novel Constitution — 建立核心创作原则

## 目的

在项目初始阶段，与用户交互式地建立整部小说的"宪法"——不可违背的核心创作原则。这些原则将指导后续所有环节。

## 触发方式

用户调用 `/novel-constitution`

## 前置条件

无。这通常是创建新小说项目的第一步。

## 工作流程

### 第一步：读取现有文件

1. 读取 `constitution/principles.yaml`，检查是否已有内容
2. 读取 `constitution/chekhov-rules.yaml`
3. 读取 `constitution/style-guide.yaml`
4. 如果已有内容，展示给用户并询问是"修改"还是"重建"

### 第二步：交互式采集

通过**分轮问答**的方式与用户建立以下内容（不要一次性问所有问题，每轮 2-3 个问题）：

**轮次 1 — 美学与基调**

- 你想要的整体基调是什么？（现有选项参考：沉郁、明快、黑色幽默、诗意、冷峻…）
- 文笔倾向偏向哪种？（简洁凝练 / 华丽繁复 / 口语化 / 混合）
- 有没有你喜欢的参考作品/作家风格？

**轮次 2 — 核心主旨**

- 这部作品最终想要表达什么？（一句话描述核心命题）
- 有哪些主题关键词？
- 有没有绝对不能违背的价值观底线？

**轮次 3 — 文风细节**

- 叙事视角？（第一/第三人称有限/全知/交替）
- 时态？（过去时/现在时）
- 对话风格偏好？（潜台词丰富 vs 直抒胸臆）
- 有没有禁忌的表达方式？

**轮次 4 — 契诃夫之枪**

- 确认是否启用标准契诃夫之枪规则
- 是否有额外的伏笔管理规则
- 确认伏笔登记簿的初始状态

### 第三步：生成文件

将采集到的内容写入以下文件：

1. `constitution/principles.yaml` — 填入美学、核心主旨、价值观底线、主题关键词
2. `constitution/style-guide.yaml` — 填入叙事视角、时态、语言风格、对话风格、场景描写偏好、禁忌清单
3. `constitution/chekhov-rules.yaml` — 确认全局规则，初始化伏笔登记簿

### 第四步：确认

将生成的三个文件内容清晰展示给用户，请求用户确认。如用户提出修改，就地更新对应文件。

## 输出文件

- `constitution/principles.yaml`
- `constitution/style-guide.yaml`
- `constitution/chekhov-rules.yaml`

## 注意事项

- 每个字段都要用用户的**原话或贴近原意**的表述填写，不要过度美化或抽象化
- `chekhov-rules.yaml` 中的 `foreshadowing_registry` 初始为空列表，后续由 `/novel-plan` 和 `/novel-write` 填充
- 如果用户无法回答某些问题，标记为 `TBD` 并建议通过 `/novel-clarify` 后续补充
