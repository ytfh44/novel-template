---
name: novel-write
description: 执行实际写作 — 按任务撰写章节, 支持 commit/uncommit 锁定
---

# Novel Write — 章节撰写

## 目的

执行实际的章节写作。写作前自动加载角色数据确保不 OOC，写作后自动更新 manifest 和章节号，支持 commit/uncommit 章节锁定。

## 触发方式

用户调用 `/novel-write`，子命令：

- `/novel-write` — 撰写下一个待写任务
- `/novel-write <task-id>` — 撰写指定任务
- `/novel-write refine <chapter>` — 修改已有章节（受 commit 保护）
- `/novel-write commit <chapter>` — 将章节标记为已定稿
- `/novel-write uncommit <chapter>` — 取消章节的定稿标记
- `/novel-write status` — 显示所有章节的 commit 状态

## 前置条件

- `tasks/active.yaml` 中有 status=pending 或 status=in_progress 的任务
- 对应的角色文件已存在

## 工作流程

### 撰写 (write)

#### 0. 预检

1. 读取 `tasks/active.yaml`，找到目标任务
2. 将任务 status 设为 `in_progress`

#### 1. 上下文加载（关键步骤）

按以下顺序读取所有必要文件，构建写作上下文：

1. **创作原则**
   - `constitution/principles.yaml` — 核心主旨和价值观
   - `constitution/style-guide.yaml` — 文风要求
   - `constitution/chekhov-rules.yaml` — 伏笔状态

2. **角色数据**（OOC 防护核心）
   对于任务中 `characters_to_load` 列出的每个角色：
   - 主要角色：读取 `profile.yaml` + `arc.yaml`（定位当前阶段）+ `voice.yaml`（定位当前阶段的语言变化）
   - 次要角色：读取对应的 `.yaml` 文件
   - **确定每个角色在此章节中所处的 arc 阶段**
   - 提取该阶段的 `behavior_patterns` 和 `belief_at_stage` 作为行为约束

3. **情节上下文**
   - 对应的 `plot/groups/<组名>.yaml` — 当前 beat 的描述
   - `plot/_order.yaml` — 前后情节组的上下文

4. **已有章节**（参考用）
   - 任务中 `chapters_to_reference` 列出的章节
   - **注意**：committed 章节只读取，不可修改

#### 2. 章节号分配

1. 读取 `chapters/_manifest.yaml`
2. 读取 `plot/_order.yaml` 的 `chapter_mapping`
3. 根据当前情节组和 beat 在排列中的位置，确定章节号
4. 如果是插入到现有章节之间，自动重新编排后续章节号
5. 更新 `chapter_mapping`

#### 3. 写作

根据加载的上下文**执行写作**：

- 遵循 `style-guide.yaml` 的文风要求
- 角色的言行举止必须符合其当前 arc 阶段的 `behavior_patterns`
- 角色的对话必须符合 `voice.yaml` 中的语言特征
- 如果情节组中标记了需要种下的伏笔，在合适位置自然植入
- 如果标记了需要回收的伏笔，确保合理回收
- 字数参考 `constraints.yaml` 的 `per_chapter` 设定

#### 4. 后处理

1. 将章节保存到 `chapters/drafts/ch<NNN>.md`
2. 在 `chapters/_manifest.yaml` 中添加/更新条目：
   - `status: uncommitted`
   - `characters_appearing` — 实际出场角色
   - `foreshadowing_planted` / `foreshadowing_paid` — 伏笔操作
   - `word_count`
3. 同步更新 `constitution/chekhov-rules.yaml` 的 `foreshadowing_registry`
4. 将任务 status 设为 `done`
5. **自动触发 `/novel-analyze`** 对此章节进行检查

### 修改 (refine)

1. 读取 `chapters/_manifest.yaml`，检查目标章节的 `status`
2. **如果 status=committed**：
   - 拒绝修改
   - 提示："此章节已定稿 (committed)，如需修改请先执行 `/novel-write uncommit <chapter>`"
   - 该章节可被读取引用，但内容不可更改
3. **如果 status=uncommitted**：
   - 加载与首次写作相同的上下文
   - 读取当前章节内容
   - 与用户交互确定修改范围
   - 使用 replace_file_content 精确修改（不重写整章）
   - 更新 manifest 中的 word_count 和其他元数据
   - 重新触发 `/novel-analyze`

### Commit / Uncommit

**commit** — 将章节标记为已定稿：

1. 检查该章节是否通过了 `/novel-analyze` 检查（`analysis_passed: true`）
2. 如未通过，警告用户并确认是否强制 commit
3. 设置 `status: committed`, `committed_at: <当前时间>`

**uncommit** — 取消定稿标记：

1. 设置 `status: uncommitted`, `committed_at: null`
2. 警告用户：uncommit 后此章节将可被 refine 修改

## 输出文件

- `chapters/drafts/ch<NNN>.md` — 章节正文
- `chapters/_manifest.yaml` — 章节元数据
- `plot/_order.yaml` — 章节映射
- `constitution/chekhov-rules.yaml` — 伏笔更新
- `tasks/active.yaml` — 任务状态更新

## 注意事项

- **OOC 防护是强制性的**：跳过角色文件加载视为流程违规
- committed 章节的保护是**内容级别**的，不影响读取和引用
- 章节号是动态生成的，如果中间插入新情节组，后续章节号会自动递增
- 写作时遵循"展示，不要叙述"(show, don't tell) 的基本原则，除非 style-guide 另有指定
- 每章写完后，列出本章实际涉及的伏笔操作供用户确认
