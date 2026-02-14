---
name: novel-character
description: 人物形象和角色弧光的管理 — 创建/编辑主要和次要角色
---

# Novel Character — 角色管理

## 目的

创建、编辑和维护故事中的角色。主要角色有独立文件夹（profile + arc + relationships + voice），次要角色为单文件。所有角色统一在 `_index.yaml` 中索引。

## 触发方式

用户调用 `/novel-character`，子命令：

- `/novel-character create <名字>` — 创建新角色
- `/novel-character edit <名字>` — 编辑已有角色
- `/novel-character list` — 列出所有角色
- `/novel-character promote <名字>` — 次要角色升级为主要角色
- `/novel-character arc <名字>` — 专门编辑角色弧光

无参数时，列出所有角色并提示操作。

## 前置条件

- `constitution/principles.yaml` 已建立（用于确保角色设计符合价值观）

## 工作流程

### 创建角色 (create)

1. 询问用户角色是**主要角色**还是**次要角色**
2. 如果是**主要角色**：
   - 复制 `characters/main/_template/` 为 `characters/main/<角色名>/`
   - 交互式填写 `profile.yaml`（基础信息、性格内核、背景故事、标志性特征）
   - 交互式填写 `arc.yaml`（弧光类型、起始状态、弧光阶段、终态）
   - 交互式填写 `relationships.yaml`（与已有角色的关系）
   - 交互式填写 `voice.yaml`（语言特征、内心独白风格）
3. 如果是**次要角色**：
   - 复制 `characters/minor/_template.yaml` 为 `characters/minor/<角色名>.yaml`
   - 交互式填写精简版字段
4. 在 `characters/_index.yaml` 中添加条目

### 编辑角色 (edit)

1. 读取该角色的所有文件
2. 展示当前内容
3. 用户指定要修改的部分
4. 使用 replace_file_content 精确更新

### 角色弧光编辑 (arc)

这是一个专门的深度编辑模式：

1. 读取角色的 `arc.yaml`
2. 读取 `plot/_order.yaml` 和相关 `plot/groups/*.yaml`
3. 展示当前弧光阶段与情节组的映射关系
4. 与用户交互式调整弧光阶段
5. 确保每个弧光阶段的 `behavior_patterns` 字段充分填写 — **这是 OOC 检测的基础**
6. 更新 `arc.yaml`

### 升级角色 (promote)

1. 读取 `characters/minor/<角色名>.yaml`
2. 在 `characters/main/` 下创建文件夹
3. 将已有信息拆分到 profile / arc / relationships / voice 四个文件
4. 交互式补充主要角色需要的额外信息
5. 删除 `characters/minor/<角色名>.yaml`
6. 更新 `characters/_index.yaml`

## OOC 防护机制

角色文件的关键作用是在写作时防止 OOC。以下字段是 OOC 检测的核心数据源：

- `profile.yaml` → `personality.core_trait`, `personality.flaws`, `personality.moral_alignment`
- `arc.yaml` → `stages[当前阶段].behavior_patterns`, `stages[当前阶段].belief_at_stage`
- `voice.yaml` → `speech_patterns`, `voice_evolution[当前阶段].changes`

**规则**：在 `/novel-write` 写作任何包含该角色的章节前，必须先读取该角色的 profile + arc(当前阶段) + voice(当前阶段) 文件。

## 输出文件

- `characters/main/<角色名>/profile.yaml`
- `characters/main/<角色名>/arc.yaml`
- `characters/main/<角色名>/relationships.yaml`
- `characters/main/<角色名>/voice.yaml`
- 或 `characters/minor/<角色名>.yaml`
- `characters/_index.yaml`（始终更新）

## 注意事项

- 角色名在文件系统中使用原名（中文/英文均可），不做拉丁化转写
- `arc.yaml` 的 `stages` 数组按故事进程排序，第一个元素是故事开始时的状态
- `behavior_patterns` 字段要具体到"在X情境下会Y"，不能过于模糊
- 创建新角色时，自动检查与已有角色的潜在关系，建议用户填写 relationships
