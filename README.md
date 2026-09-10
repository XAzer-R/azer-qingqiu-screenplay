# 阿泽 AI剧本创作系统｜青丘织事

**组织创意孵化、人物关系、故事结构、分场写作和审稿修订。**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-0.1.0-green.svg)](CHANGELOG.md)
[![Host](https://img.shields.io/badge/host-Claude_Code-orange.svg)](docs/HOST-COMPATIBILITY.md)

阿泽山海创作系列 · AI 影视创作工作流 · Agent / Skills / 提示词 / 空白模板

本项目提供可阅读、可修改的创作方法与角色协作规则。你提供自己的创作目标和授权材料，由主控组织分阶段执行、审核与修订。仓库不附带具体剧本、小说原文、图片、视频或历史创作产物。

## 三个项目怎么选

| 项目 | 适合的需求 | 流程 |
|---|---|---|
| [飞廉开镜](https://github.com/XAzer-R/azer-feilian-director) | 已有剧本，希望快速得到镜头与视频提示词 | 导演 → 服化道 → 素材引用回填 |
| [烛龙造境](https://github.com/XAzer-R/azer-zhulong-director-pro) | 已有剧本，希望进一步控制画面材质、光影与关键帧 | 导演 → 服化道 → 分镜师精修 → 主控回填 |
| [青丘织事](https://github.com/XAzer-R/azer-qingqiu-screenplay) | 从想法或授权改编材料开始写剧本 | 孵化 → 结构与人物 → 场景 → 独立审稿与修订 |

三个系统独立运行，不需要互相安装；剧本可作为导演系统的输入，由用户选择交接。

## 能做什么

- 编剧 + 审查员，按作品隔离输入输出。
- 11 个一级 Skill，按当前任务加载专业方法，避免无关内容堆进上下文。
- 明确角色读写边界、输入版本与审核状态；修改上游后重新核验下游结果。
- 自动修订最多两轮，仍失败就保留问题并交用户决定，失败不会被改成“通过”。
- 提供本地只读校验工具与回归测试，便于检查分发包是否完整。

## 环境要求

- 原生配置面向支持项目 `.claude/agents` 与 `.claude/skills` 的 Claude Code。
- 使用者自行准备宿主和模型访问权限；仓库没有账号、密钥、预付额度或模型权重。
- Python 3.10+ 只用于离线校验，不是运行创作提示词的必需依赖；校验工具无第三方 Python 依赖。
- 其他 Agent 宿主可参考或适配方法，目前没有跨宿主自动注册与端到端等价保证。

## 安装与首次使用

```shell
git clone https://github.com/XAzer-R/azer-qingqiu-screenplay.git
cd azer-qingqiu-screenplay
python -B tools/validate.py
claude
```

没有 Git 时也可通过 GitHub 的 **Code → Download ZIP** 下载解压，并在项目根启动宿主。不要双击 Markdown 期待自动安装；不要向聊天粘贴密钥。

根目录 `CLAUDE.md` 导入运行合同和主控。进入新会话后直接说明目标，例如：

```text
请新建一个剧本项目
```

首次启动没有作品是正常状态，主控会引导创建 `projects/<ASCII项目名>/`，采集目标与创作范围。系统方法仍从仓库根读取；作品内容写入当前作品目录。不要在系统根或 Skill 目录保存原著。

| 自然语言入口 | 用途 |
|---|---|
| 新建项目 / 开始 | 建立并选择当前作品 |
| 孵化 / 故事骨架 / 人物 | 形成创意、结构与人物关系 |
| 场景 / 审查 / 修改 | 按当前作品和集号写作、审稿与修订 |
| 改编 / 消化 | 使用自己有权使用的材料进行分析 |

这些是主控的对话约定。文档中的 `/开始` 等不是预装的宿主 slash command；如果宿主拦截斜杠命令，直接用自然语言输入。

## 输入、输出与文件结构

```text
CLAUDE.md                  宿主入口
AGENTS.md                  仓库维护规则
.claude/CLAUDE.md           主控流程
.claude/agents/             角色合同
.claude/skills/             专业方法与空白模板
docs/                      运行合同、使用边界与检查报告
tools/                     只读验证工具
tests/                     隔离回归测试
project.json               发行元数据
```

作品位于 `projects/`，包括状态、人物、剧本和作品私有参考；审稿日志模板位于 `templates/`。

作品目录默认被 Git 忽略。导演版少量已跟踪的空白 assets 模板填写后仍会出现在 Git 改动中；发布前必须检查差异，不能用 `git add .` 无差别提交作品。

## 验证与状态检查

```shell
python -B tools/validate.py
python -B -m unittest discover -s tests
python -B tools/workflow_guard.py unit EP01-S01
```

对实际审核记录可运行：

```shell
python -B tools/workflow_guard.py review outputs/review.json --root .
```

剧本系统应把 `--root` 指向当前 `projects/<项目名>`，回执路径相对该作品根。回执格式见 [审核记录](docs/REVIEW-RECEIPTS.md)。工具只验证当前文件版本与审核记录一致性，不证明艺术质量，也不是防篡改签名。

## 当前验证边界

本发行版完成的检查与发现见 [运行机制检查](docs/RUNTIME-REVIEW.md)。离线测试不等于真实模型创作成功；本轮没有调用模型、生成媒体或完成 Claude Code 全流程 E2E。因此本项目以 **0.1.0 工作流发行版**发布，不承诺一键出片或所有宿主开箱即用。

用户必须确认创作方向、处理剩余审核问题并自行管理素材权利。平台时长、分辨率和接口会变化，方法文件不构成平台能力保证。

## 贡献与反馈

欢迎提交明确的复现步骤、通用方法改进和兼容性补丁。请勿上传客户材料、完整小说、未授权影视帧、密钥或私人会话。

参见 [贡献指南](CONTRIBUTING.md)、[安全反馈](SECURITY.md) 和 [变更记录](CHANGELOG.md)。

## 作者与许可

作者：**阿泽（Azer）**。采用 [MIT License](LICENSE)，允许使用、修改和再分发，需保留版权与许可声明。第三方理论名称与方法参考见 [NOTICE](NOTICE.md)。

项目名称采用山海神话意象作为品牌命名，不宣称是古籍中的原句或职能定义。
