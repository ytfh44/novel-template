---
name: novel-analyze
description: 验证质量和一致性 — 情节完整性、主旨忠实度、角色OOC检测、伏笔审计
---

# Novel Analyze — 质量分析

## 目的

对已撰写的章节执行多维度质量检查：情节完整性、主旨忠实度、价值观一致度、结构合理度、角色 OOC 检测、契诃夫之枪审计。可由 `/novel-write` 自动触发，也可手动调用。

## 触发方式

- **自动触发**：每次 `/novel-write` 完成一章后自动执行
- **手动触发**：用户调用 `/novel-analyze`，子命令：
  - `/novel-analyze <chapter>` — 分析指定章节
  - `/novel-analyze all` — 分析所有 uncommitted 章节
  - `/novel-analyze chekhov` — 只做伏笔审计
  - `/novel-analyze ooc <角色名>` — 只对指定角色做 OOC 检测
  - `/novel-analyze structure` — 只做结构分析

无参数时，分析所有 uncommitted 章节。

## 前置条件

- `chapters/drafts/` 下至少有一个章节文件
- `constitution/` 下的文件已建立

## 工作流程

### 第一步：加载基准数据

1. `constitution/principles.yaml` — 核心主旨、价值观底线、主题关键词
2. `constitution/style-guide.yaml` — 文风基准
3. `constitution/chekhov-rules.yaml` — 伏笔状态
4. `chapters/_manifest.yaml` — 章节列表和元数据
5. 所有涉及角色的 profile / arc / voice 文件

### 第二步：逐章分析

对每个目标章节执行以下检查：

#### A. 角色 OOC 检测

对章节中每个出场角色：

1. 确定该角色在此章节时处于 arc 的哪个阶段
2. 读取该阶段的 `behavior_patterns` 和 `belief_at_stage`
3. 审查角色在本章的行为和对话是否符合：
   - 性格特质 (`profile.yaml` → `personality`)
   - 当前信念 (`arc.yaml` → 当前阶段 `belief_at_stage`)
   - 行为模式 (`arc.yaml` → 当前阶段 `behavior_patterns`)
   - 语言特征 (`voice.yaml` → `speech_patterns` + 当前阶段 `voice_evolution`)
4. 如发现偏差，记录为 `character_ooc` issue，包含：
   - 预期行为 vs 实际行为
   - 严重程度（error = 完全违背性格; warning = 轻微偏差）
   - 修改建议

**特殊情况**：如果 OOC 行为恰好发生在弧光转折点（从一个 stage 过渡到下一个），视为合理——角色正在成长。但需要验证这个转变是否有足够的铺垫。

#### B. 主旨忠实度

1. 从 `principles.yaml` 提取 `core_thesis` 和 `theme_keywords`
2. 从 `themes.yaml` 提取每个主题的描述
3. 评估本章内容与核心主旨的关联度
4. 如果一章完全不涉及任何主题，记录为 info 级别提醒
5. 如果一章的内容与核心主旨矛盾，记录为 warning

#### C. 价值观一致度

1. 从 `principles.yaml` 提取 `value_boundaries`
2. 审查本章内容是否违反任何价值观底线
3. 违反底线记录为 error 级别

#### D. 结构合理度

1. 读取 `plot/_order.yaml`，检查当前排列的节奏分布
2. 标注问题：
   - 连续3个以上 compact 节奏的情节组
   - 连续3个以上 relaxed 节奏的情节组
   - 高潮 (climax) 之后缺少缓冲
   - 开头连续多章无冲突

#### E. 契诃夫之枪审计

1. 读取 `chekhov-rules.yaml` 的 `foreshadowing_registry`
2. 检查：
   - **未回收的伏笔** — status=planted 且 expected_payoff 时间已过
   - **无铺垫的转折** — 本章使用了之前未被 plant 的关键元素
   - **遗弃的伏笔** — status=abandoned 的合理性
3. 对每个问题标注严重性和建议

#### F. 情节完整性

1. 读取所有 `plot/groups/*.yaml`
2. 检查每个情节组的四个 beat 是否都有对应的章节
3. 标注缺失的 beat

### 第三步：汇总报告

将所有发现写入分析文件：

1. `analysis/consistency.yaml` — OOC + 连续性问题
2. `analysis/completeness.yaml` — 情节完整性 + 伏笔审计 + 主旨覆盖
3. `analysis/suggestions.yaml` — 综合改进建议

### 第四步：更新 Manifest

在 `chapters/_manifest.yaml` 中更新每个被分析章节的 `analysis_passed` 字段：

- 无 error 级别问题 → `true`
- 有 error 级别问题 → `false`

### 第五步：向用户展示摘要

按以下格式展示分析结果：

```
📊 分析完成: 第X章

✅ 通过的检查: A, B, C
⚠️ 警告 (N个):
  - [OOC] 角色X在第3段的行为与当前arc阶段不符...
  - [伏笔] "旧相框"在第Y章种下, 预期在此处回收但未出现...
❌ 错误 (N个):
  - [价值观] 此段落违反了"不美化无意义暴力"的底线...

💡 建议 (N个):
  - 本章节奏偏快, 建议在战斗场景后加入一段角色内心独白...
```

## 输出文件

- `analysis/consistency.yaml`
- `analysis/completeness.yaml`
- `analysis/suggestions.yaml`
- `chapters/_manifest.yaml`（更新 analysis_passed）

## 注意事项

- 分析是**非破坏性**的——只产出报告，不自动修改章节内容
- 弧光转折期的"OOC"需要特殊处理：如果行为变化发生在 stage 过渡处，应视为成长而非 OOC
- 伏笔审计应考虑"长周期伏笔"——有些伏笔在几十章后才回收是正常的
- 分析结果中的建议应具体可行，避免泛泛的"可以改进"
