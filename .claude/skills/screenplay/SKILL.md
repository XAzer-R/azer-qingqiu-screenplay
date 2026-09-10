---
name: screenplay
description: 影视剧本创作主控技能包。覆盖故事骨架、人物档案、分集目录、场景写作、剧本审查、改编全流程。采用双 Agent 架构：编剧 Agent 负责创作，审查员 Agent 负责独立评分。支持原创剧本和改编项目，适用于系列剧、短剧、短片、创意广告等所有类型。
---

> Copyright (c) 2026 阿泽。依据根目录 LICENSE（MIT）使用。

# 剧本创作主控

## 架构概述

本系统采用**双 Agent + 多 Skill** 协作架构：

```
screenplay（主控，你正在读的这个文件）
  │
  ├── 创作阶段 → 调用 screenwriter agent（编剧）
  │                 └── 编剧调用各 Skill 执行创作
  │
  └── 审查阶段 → 调用 reviewer agent（审查员）
                   └── 不合格 → 主控调用编剧修改 → 再审
```

**主控职责：** 流程调度、状态管理、Agent 调用、用户交互

---

## 创作流程总览

```
/开始 → /孵化 → /故事骨架 → /人物 → /目录 → /场景 [E集-S场] → /审查 [E集] → /导出
```

每个阶段产出文件，下一个阶段读取上一阶段的文件，保持上下文连贯。

---

## 文件结构

```
{项目名}/
├── .drama-state.json          # 创作状态追踪
├── creative-seeds.md          # 创意种子卡（/孵化 阶段生成）
├── creative-plan.md           # 故事骨架（以 creative-seeds.md 为输入）
├── characters.md              # 人物档案
├── episode-directory.md       # 分集目录
├── CHANGELOG.md               # 系统/规则变更记录（产品经理系统维护）
├── episodes/                  # 分集剧本
│   ├── S1E01/                 # 该集分场子目录
│   │   ├── S1E01-S01_{场景名}.md
│   │   ├── S1E01-S02_{场景名}.md
│   │   └── ...
│   ├── S1E01.md               # 整集剧本
│   ├── S1E01_审稿日志.md       # 与剧本一一对应的审稿记录
│   └── ...
├── references/                # 大师范例库
│   ├── README.md              # 含领域术语闸门清单
│   ├── scenes/                # 各类型场景范例（审查对标基准）
│   ├── dialogue/              # Mamet / Sorkin / 宋方金
│   ├── structure/             # Truby 22 步真实落点
│   └── novels/                # 小说素材库（中文目录命名）
│       └── {书名}/
│           ├── raw/           # 原文（.gitignore 排除）
│           ├── INDEX.md       # 总索引
│           └── ...            # 各类对标卡片或 v{N} 标准化产物
├── archive/                   # 已归档的旧版剧本与过时文档
└── export/                    # 导出目录
    └── {剧名}-完整剧本.md

# 注：新增原著或参考素材，放入 references/novels/
# 或对应小说的 references/novels/{书名}/raw/ 目录。项目根不使用 source/。
```

---

## 命令定义

> **命名约定**：以下命令定义中的文件路径统一使用 `EP{NN}` 格式（如 `EP01.md`、`EP01_审稿日志.md`）。如需区分季可用 `S{N}E{NN}` 格式。

### /开始

**功能：** 项目立项，锁定创作方向。

**执行者：** 主控自身（不调用 Agent）

**流程：**

读取 `references/novels/`（当前作品根） 目录（如有文件则列出），然后提问：

```
Q1：项目基本信息
- 项目名称
- 类型：系列剧 / 短剧 / 短片 / 创意广告 / 改编项目
- 集数（如适用）
- 单集时长预估

Q2：核心创意
用一两句话描述：这个故事的核心吸引力是什么？
（如是改编项目，原著放在 references/novels/ 下，需说明改编方向）

Q3：故事调性（选择或描述）
- 严肃叙事·克制爆发（推荐科幻/现实主义/历史题材）
- 类型悬疑·信息博弈（推荐惊悚/犯罪/谍战题材）
- 短剧快节奏·情绪驱动（推荐短剧/竖屏短片）
- 或自由描述风格偏好

Q4：编剧风格包（可选）
- serious-drama（严肃叙事，克制中的爆发）
- schrader-style（Schrader 日常叙事，日记体 + 偶遇叙事）
- 无风格包（仅用 Kasdan+Gilroy 基础标准）
- 或描述风格偏好

Q5：项目目录
- 子项目目录名（将在仓库根目录下创建）
- 如果是改编项目，原著文本位置

Q6：附加 Skill 配置（可选）
- 场景写作时除基础 Skill 外，额外调用哪些 Skill？
- 项目是否有专属的必读参考文件？（将写入 session_reads）
```

确认后：
1. 创建项目子目录及标准结构（`.drama-state.json`、`STATUS.md`、`episodes/`）
2. 将 workflow 配置（`creation_skills`、`review_skills`）和 `session_reads` 写入 `.drama-state.json`
3. 自动进入 `/孵化`（如用户在 /开始 时已给出清晰的七根系要素，则跳过孵化，直接进入 /故事骨架）

---

### /孵化

**功能：** 创意孵化。在立项之后、故事骨架之前，通过多轮苏格拉底式追问帮助用户从碎片中生长出故事的七根系（Truby 有机结构论）。

**触发：** `/开始` 完成后自动进入。如用户在 `/开始` 时已给出清晰的七根系要素，可跳过。

**执行者：** → screenwriter agent

**主控传递给编剧：**
- 阶段：创意孵化
- 读取：`.drama-state.json`
- 调用 Skill：creative-incubation-skill

**编剧执行：**
1. **发散**：从人物入手（宋方金：人物先于情节），多轮追问挖掘碎片
2. **聚合**：将碎片对应到七根系（弱点与需求/欲望/对手/核心信念/关系引擎/世界质感/道德困境），输出根系对照表
3. **参考推荐**：推荐 3-5 部参考作品（**优先用已知经典作品，不必联网；冷门/新作没把握才单条核实**），按对白标杆/结构标杆/关系标杆/质感标杆四维度呈现，附具体台词和场景
4. **补缺**：按 Truby 因果链逐个解决空缺根系
5. **确认**：输出创意种子卡，前 4 根系到位即可放行

**输出：** 保存为 `creative-seeds.md`

---

### /故事骨架

**功能：** 生成完整故事骨架。

**执行者：** → screenwriter agent

**主控传递给编剧：**
- 阶段：故事骨架
- 读取：`creative-seeds.md`（如有）、`references/novels/`（当前作品根） 目录、`.drama-state.json`
- 调用 Skill：structuralist-skill + screenwriter-skill

**编剧生成内容：**

1. 项目信息（名称/类型/集数/风格）
2. 核心梗概（一句话）
3. 核心问题（主角追求什么？什么阻止他？代价是什么？）
4. 故事结构（按类型选择起承转合或三幕结构）
5. 主要人物列表（每人一句话）
6. 主题（用一个场景或动作，不用抽象词语）
7. 全季/全片钩子规划

**输出：** 保存为 `creative-plan.md`

---

### /人物

**功能：** 生成完整人物档案。

**执行者：** → screenwriter agent

**主控传递给编剧：**
- 阶段：人物设计
- 读取：`creative-plan.md`、`references/novels/`（当前作品根）
- 调用 Skill：dialogue-skill（语言档案）+ screenwriter-skill + 风格包

**编剧生成内容：**

**主要角色（每人包含）：**
- 姓名、年龄、外貌（2-3句，有具体细节）
- 核心信念 / 核心缺陷
- 外部目标 / 内部需求
- 说话方式（词汇/句式/口头禅/永远不会说的话）
- 标志性动作（2个）
- 人物弧线
- **语言档案**（由 dialogue-skill 生成）

**配角** + **人物关系图**（Mermaid 格式）

**输出：** 保存为 `characters.md`

---

### /目录

**功能：** 生成分集目录。

**执行者：** → screenwriter agent

**主控传递给编剧：**
- 阶段：分集目录
- 读取：`creative-plan.md`、`characters.md`
- 调用 Skill：structuralist-skill（节奏图谱 + 钩子编排）

**编剧生成内容：**

每集条目 + 标记 + 开场钩/结尾钩 + 叙事功能 + **节奏图谱**

**输出：** 保存为 `episode-directory.md`

---

### /场景 [E集-S场]

**功能：** 写一个具体场景。

**用法：** `/场景 S1E01-S01`

**执行者：** → screenwriter agent

**主控传递给编剧：**
- 阶段：场景写作
- 读取：`creative-plan.md`、`characters.md`、`episode-directory.md`、同集已完成场景
- **必读（存在则读、已读复用、缺失降级跳过，见 CLAUDE.md 省Token原则·四/五）：** `references/README.md`（"必须先问用户"清单；缺失时闸门行为仍生效）、对应集 `EP{NN}_审稿日志.md` **顶部「决策摘要节」**（已被推翻 + 已确认方向，**不读全文**）、`references/scenes/<对应类型>/` **该类型 1 份标杆范例**（不读全部）
- **调用 Skill（不再硬编码，按映射表派单）：** 主控先判定本场场景类型 → 按 CLAUDE.md 省Token原则·四的「场景类型→Skill 映射表」算出本场清单 = `screenwriter-skill`（基础）+ 该类型表行额外 Skill + 项目配置的风格包/`creation_skills`附加 → 显式把这份清单交给编剧。**不再恒载 structuralist+dialogue，不载"全部 references 子文件"。** 编剧只载清单、载齐清单，动笔前输出 `📦 本场加载` 声明供对账

**编剧执行：**
1. 读取审稿日志**顶部「决策摘要节」**，确认哪些创作方向已被否决（无此节则读全文一次并在更新时补建该节）
2. 输出 `📦 本场加载` 声明 → 场景结构定位（由清单内 Skill 提供）
3. 场景正文（剧本格式）
4. 场景自检（不输出给用户）
5. **保存后立即更新 `EP{NN}_审稿日志.md`**（维护顶部「决策摘要节」；本会话内此文件随即缓存失效，下次读必重读）

**输出：** 保存为 `episodes/EP{NN}/EP{NN}-S{M}_{场景名}.md`

---

### /审查 [E集]

**功能：** 独立审查一整集（对标范例驱动）。**92 分合格线（量化为 X/100）。**

**用法：** `/审查 S1E01`

**执行者：** → reviewer agent（独立评分）

**审查前置读取（reviewer 必须先读；存在则读、已读复用、缺失降级跳过不报错）：**
1. `references/README.md` — "必须先问用户"清单，作为黑话检测基准（缺失时红线扫描的黑话检测仍执行）
2. `episodes/EP{NN}_审稿日志.md` **顶部「决策摘要节」** — 历次修改决策，避免给出与已确认决策矛盾的建议（**只读决策节、不读全文**；本会话内该文件若被改写过则缓存失效、须重读）
3. `references/scenes/<本集出现类型>/_README.md` 及标杆范例 — 对标基准（只读本集用到的类型，缺失则 McKee/Seger 内功兜底）

**审查方法：** 不用抽象标准催生平庸修改——**对标具体范例**。指出差距在哪个具体维度（节奏/物件/视点/沉默）。

**审查-修改闭环：**

```
主控调用 reviewer agent → 前置读取 → 对标范例评估
    ↓
┌─ ≥ 92 分 → ✅ 通过
│   提示用户：当前得分 X/100，是否追求 100 分满分？
│   ├── 用户选择"精修" → reviewer 输出精修建议 → 主控调用 screenwriter 精修 → 再审
│   └── 用户选择"通过" → 确认，进入下一集
│
└─ < 92 分 → ⚠️ 不合格
    reviewer 输出修改指令（P0/P1/P2），对标范例指出差距
    → 主控调用 screenwriter agent 执行修改
    → screenwriter 修改后更新 EP{NN}_审稿日志.md
    → 主控再次调用 reviewer agent 重审
    → 循环（**≤2 轮自动循环上限·省 token 硬规则**，见 CLAUDE.md「审查-修改闭环」）
    → 2 轮后仍 <92 → 停止自动循环，把得分 + 剩余 P0/P1 交用户决定
```

**审查方法论（对标范例驱动 · 不再用抽象四维度打分）：**

详细规则见 `.claude/agents/reviewer.md`。摘要：

- **不再使用** "场景效率/对白质量/信息张力/钩子与节奏 各 25 分" 的抽象打分
- **改为** 逐场对标具体范例（`references/scenes/<类型>/`），描述"对标了谁、差在哪个维度"
- 92 分合格线保留，但是对标差距汇总后的量化结果，不是四维度加总
- 无对应范例的场景用 McKee 价值转变 + Gilroy 信息张力兜底
- 问题分级 P0（结构性）/ P1（功能性）/ P2（质感性），三级分明

---

### /改编

**功能：** 将 `references/novels/`（当前作品根） 目录中的原著改编为剧本。

**执行者：** → screenwriter agent

**主控传递给编剧：**
- 阶段：改编
- 读取：`references/novels/`（当前作品根） 目录所有文件
- 调用 Skill：adaptation-skill（4 个子技能依次调用）

**编剧执行：**
1. source-analysis（原著解析）
2. dramatic-extraction（戏剧点提炼）
3. structure-mapping（结构映射）
4. character-migration（人物迁移）
5. 综合生成《改编分析报告》

**用户确认后 → 进入正常创作流程**（/故事骨架 → /人物 → /目录 → /场景）

---

### /消化 {书名}

**功能：** 将小说原文消化为系统可用的参考素材。

**用法：** `/消化 三体`

**前置条件：** 小说文本已放入 `references/novels/{书名}/raw/` 目录

**执行者：** → screenwriter agent

**主控传递给编剧：**
- 阶段：小说消化
- 读取：`references/novels/{书名}/raw/` 全部文件
- 调用 Skill：novel-digester（三层消化流程）

**编剧执行：**
1. 第一层：骨架扫描 — 逐章提取结构功能，标记值得精读的段落
2. 第二层：定向精读 — 按结构/对白/场景/叙事四维度提取素材
3. 第三层：合成输出 — 生成 INDEX.md / dialogue.md / scenes.md / narrative.md / project-mapping.md

**输出：** 保存到 `references/novels/{书名}/` 目录下

---

### /导出

**功能：** 将所有场景合并为完整剧本文件。

**执行者：** 主控自身

读取所有 `episodes/` 目录下的场景文件，按顺序合并，输出为 `export/{剧名}-完整剧本.md`。

---

### /状态

**功能：** 查看当前创作进度。

**执行者：** 主控自身

读取 `.drama-state.json` 和 `episodes/` 目录，输出进度信息。

---

## Agent 与 Skill 调用矩阵

| 创作阶段 | 主控调用 | Agent/Skill 调用链 |
|---------|---------|-------------------|
| `/开始` | 主控自身 | — |
| `/孵化` | → screenwriter agent | → creative-incubation-skill |
| `/故事骨架` | → screenwriter agent | → structuralist-skill + screenwriter-skill |
| `/人物` | → screenwriter agent | → dialogue-skill + screenwriter-skill + 风格包 |
| `/目录` | → screenwriter agent | → structuralist-skill |
| `/场景` | → screenwriter agent | → screenwriter-skill（基础）+ 按「场景类型→Skill 映射表」派单的额外 Skill + 风格包/creation_skills（见 CLAUDE.md 省Token原则·四，**不恒载全部**） |
| `/审查` | → reviewer agent | → screenwriter-skill + director-skill + 风格包 |
| `/改编` | → screenwriter agent | → adaptation-skill（4 子技能） |
| `/消化` | → screenwriter agent | → novel-digester（三层消化） |
| `/导出` | 主控自身 | — |

---

## 创作原则

1. **source 优先** — 每次创作开始时检查 references/novels/ 目录
2. **状态追踪** — 每次完成一个阶段更新 `.drama-state.json`
3. **逐场审查** — 编剧每场写完自检，审查员整集审查
4. **分层加载** — 只在需要时读取对应 Skill
5. **风格一致** — 整个项目保持同一风格包
6. **审查闭环** — 92 分合格线，不合格自动修改重审，审查对标具体范例
7. **必须先问用户** — 遇到领域专用术语、专有名词、世界观设定、新角色命名等，立即停下问用户。完整清单见 `references/README.md`
8. **审稿日志必更新** — 任何修改 EP{NN}.md 后，立即更新 `EP{NN}_审稿日志.md`
9. **创作前必读（按需·缺失降级）** — 场景创作前读：`references/README.md`（缺失则降级，闸门行为仍生效）+ 对应集审稿日志**顶部决策摘要节**（不读全文）+ `references/scenes/<对应类型>/` **该类型 1 份标杆范例**（不读全部、缺失则内功兜底）
10. **加载按表派单** — 场景写作不恒载全部 Skill，由主控按「场景类型→Skill 映射表」派单、编剧只载清单并输出 `📦 本场加载` 声明（见 CLAUDE.md 省Token原则·四/五）

---

## 初始化

项目启动时执行 CLAUDE.md「工作目录（多项目初始化）」流程：

1. 扫描根目录 + 所有一级子目录的 `.drama-state.json`
2. 多项目时显示选择菜单，单项目时直接进入
3. 锁定活跃项目后显示：

```
🎬 剧本创作系统就绪

活跃项目：{project名}（{目录}）
阶段：{phase}
状态：{status}
架构：编剧 Agent + 审查员 Agent + 多 Skill 协作
风格包：{style_pack}
创作 Skill：{workflow.creation_skills 列表}
审查标准：对标范例驱动，92 分合格线（量化为 X/100）

输入 /开始 创建新子项目
输入 /帮助 查看所有命令
切换项目：说"切换到 {项目名}"
```
