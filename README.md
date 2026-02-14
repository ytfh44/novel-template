# Novel Template

基于 Agent Skill 的小说创作模板项目。通过 7 个可调用的工作流，支持从构思到完稿的全流程管理。

## 快速开始

按以下顺序执行工作流：

1. `/novel-constitution` — 建立核心创作原则（美学、主旨、价值观、文风、契诃夫之枪规则）
2. `/novel-specify` — 定义故事需求（前提、主题、世界观、硬约束）
3. `/novel-clarify` — 交互式解决规格文档中的模糊点
4. `/novel-character` — 创建和管理角色（主要角色 / 次要角色）
5. `/novel-plan` — 设计情节组（桥段设计、伏笔布局、动态章节规划）
6. `/novel-tasks` — 将情节组分解为可执行的写作任务
7. `/novel-write` — 执行实际写作（自动加载角色数据、commit/uncommit）
8. `/novel-analyze` — 质量分析（OOC 检测、伏笔审计、主旨忠实度）

## 目录结构

```
constitution/        核心创作原则 (principles, chekhov-rules, style-guide)
specification/       故事需求 (premise, themes, world-building, constraints)
clarification/       问答记录
characters/          角色库 (main/ 每人一个文件夹, minor/ 每人一个文件)
plot/                情节组 + 动态排列
tasks/               可执行任务队列
chapters/            章节输出 + commit 状态
analysis/            分析报告
.agent/workflows/    工作流入口
.agent/skills/       Skill 实现
```

## 进阶特性

- **Commit / Uncommit** — 已定稿章节不可被 refine 修改，只能被引用
- **动态章节规划** — 情节组不含绝对章节号，按排列和节奏自动编排
- **角色 OOC 防护** — 写作前自动加载角色的 profile + arc 阶段 + voice 数据
- **契诃夫之枪审计** — 自动追踪伏笔的种下与回收
- **自动化分析** — 写完即检，覆盖情节完整性、主旨忠实度、结构合理度
