---
name: novel-plan
description: 设计情节组和技术实现方案 — 桥段设计、伏笔布局、动态章节规划
---

# Novel Plan — 情节规划

## 目的

以"情节组"为基本单位设计故事结构。每个情节组包含 trigger → develop → climax → resolution 四个关键点，不指定绝对章节号，通过节奏标签实现动态编排。同时管理伏笔的种下与回收。

## 触发方式

用户调用 `/novel-plan`，子命令：

- `/novel-plan create <组名>` — 创建新情节组
- `/novel-plan edit <组名>` — 编辑已有情节组
- `/novel-plan order` — 查看/调整排列顺序
- `/novel-plan foreshadow` — 管理伏笔
- `/novel-plan overview` — 展示全局情节地图

无参数时，展示全局 overview。

## 前置条件

- `specification/premise.yaml` 已填写（至少 logline 非空）
- `constitution/chekhov-rules.yaml` 已就位

## 工作流程

### 创建情节组 (create)

1. 读取 `constitution/principles.yaml` — 确保情节服务于核心主旨
2. 读取 `specification/themes.yaml` — 确定此情节组推进哪个主题
3. 读取 `characters/_index.yaml` — 获取可用角色列表
4. 交互式填写：
   - **名称和概要** — 一句话描述此情节组
   - **目的** — 推进哪个主题、哪些角色的弧光、揭示哪些世界观
   - **四个关键点** — 每个 beat 的描述、涉及的角色、伏笔操作
   - **节奏标记** — compact / moderate / relaxed / transition
   - **依赖关系** — 必须排在哪些情节组之后
5. 保存到 `plot/groups/<组名>.yaml`
6. 自动在 `plot/_order.yaml` 的 sequence 中添加条目

### 伏笔管理 (foreshadow)

1. 读取 `constitution/chekhov-rules.yaml` 的 `foreshadowing_registry`
2. 展示所有伏笔的状态：planted / paid_off / abandoned
3. 支持操作：
   - **plant** — 在某个情节组的某个 beat 中种下新伏笔
   - **payoff** — 标记某条伏笔在某处得到回收
   - **abandon** — 标记某条伏笔已放弃（需要理由）
4. 每次操作同步更新：
   - `constitution/chekhov-rules.yaml` 的 `foreshadowing_registry`
   - 对应 `plot/groups/<组名>.yaml` 的 `foreshadowing_planted` / `foreshadowing_paid`

### 排列 (order)

1. 读取 `plot/_order.yaml`
2. 读取所有 `plot/groups/*.yaml`（排除 _template）
3. 展示当前排列 + 每个组的 pacing 标签
4. 用户可以：
   - 手动拖拽排列（指定新的顺序列表）
   - 切换到 auto 模式（系统按依赖关系 + pacing 交替排列）
5. 自动验证依赖关系是否满足

### 全局总览 (overview)

1. 读取所有情节组
2. 以列表形式展示：排列顺序 → 情节组名 → pacing → 涉及角色 → 伏笔状态
3. 标注潜在问题：
   - 连续多个 compact 节奏的组（读者可能疲劳）
   - 有 planted 但无 expected_payoff 的伏笔
   - 有依赖关系冲突的组

## 动态章节规划 — 核心逻辑

情节组不包含绝对章节号。章节号在 `/novel-write` 时按以下逻辑生成：

1. 读取 `plot/_order.yaml` 的 `sequence`
2. 按顺序遍历每个组的 beats（trigger → develop → climax → resolution）
3. 每个 beat 可以对应 1 个或多个章节（根据内容量和 pacing）
4. 为每个章节分配递增章节号
5. 记录到 `plot/_order.yaml` 的 `chapter_mapping`
6. 新增或删除情节组后，自动重新编排后续章节号

## 输出文件

- `plot/groups/<组名>.yaml`
- `plot/_order.yaml`
- `constitution/chekhov-rules.yaml`（伏笔部分）

## 注意事项

- 情节组文件中**不写绝对章节号**，只写相对位置和依赖关系
- 每个情节组必须有明确的 `purpose`，不允许"凑字数"的空白情节组
- 创建情节组时，自动检查相关角色的 arc 阶段是否连贯
- 如果依赖关系形成环，报错并请用户解决
