---
name: novel-tasks
description: 分解为可执行的写作任务步骤
---

# Novel Tasks — 任务分解

## 目的

将情节组分解为具体的可执行写作任务，自动排序依赖关系，理清每个任务需要提前加载的角色文件和参考章节。

## 触发方式

用户调用 `/novel-tasks`，子命令：

- `/novel-tasks generate` — 根据 plot groups 自动生成任务
- `/novel-tasks list` — 列出所有任务
- `/novel-tasks next` — 显示下一个应执行的任务
- `/novel-tasks done <task-id>` — 标记任务为完成
- `/novel-tasks reorder` — 手动调整优先级

无参数时，显示当前任务概览。

## 前置条件

- `plot/groups/` 下至少有一个非 _template 的情节组文件
- `plot/_order.yaml` 的 sequence 非空

## 工作流程

### 生成任务 (generate)

1. 读取 `plot/_order.yaml` 获取排列顺序
2. 遍历每个情节组的每个 beat
3. 为每个 beat 生成一个写作任务，包含：
   - **id** — `task-<NNN>` 格式
   - **type** — write（首次撰写）
   - **description** — 基于情节组的 beat 描述生成
   - **target** — 关联的 plot_group 和 beat
   - **prerequisites**:
     - `characters_to_load` — 从 beat 的 `characters_involved` 提取，映射到实际文件路径
     - `chapters_to_reference` — 从依赖关系推导出需要参考的已完成章节
   - **priority** — 根据排列顺序和依赖关系自动分配
   - **status** — 初始为 pending
4. 将高优先级任务写入 `tasks/active.yaml`，其余写入 `tasks/backlog.yaml`
5. active 队列最多保持 5 个任务

### 查看下一个任务 (next)

1. 读取 `tasks/active.yaml`
2. 找到第一个 status=pending 的任务
3. 展示完整信息，包括：
   - 任务描述
   - 需要加载的角色文件列表（含完整路径）
   - 需要参考的章节列表
   - 对应的情节组上下文
4. 提示用户执行 `/novel-write` 开始撰写

### 完成任务 (done)

1. 将指定任务的 status 设为 done
2. 检查是否有被此任务阻塞的后续任务
3. 如有，将其从 backlog 移入 active

### 重排 (reorder)

1. 展示当前 active 队列
2. 用户指定新顺序或修改优先级
3. 更新 `tasks/active.yaml`

## 输出文件

- `tasks/active.yaml`
- `tasks/backlog.yaml`

## 注意事项

- 生成任务时不覆盖已有任务（status=done 的保留），只添加新任务
- `characters_to_load` 是写作前必须读取的角色文件列表，这是 OOC 防护的入口
- 如果角色的 arc 没有为对应阶段定义 behavior_patterns，发出警告并建议先执行 `/novel-character arc <角色名>`
