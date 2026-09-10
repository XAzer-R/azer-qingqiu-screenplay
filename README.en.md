# Azer AI Screenwriting System | Qingqiu Zhishi

[简体中文](README.md) | [English](README.en.md)

**Develop ideas, character relationships, story structures, scenes, and revisions.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-0.1.0-green.svg)](CHANGELOG.md)

Part of the Azer Shanhai Creative Series: AI filmmaking workflows, agents, Skills, prompting methods, and blank templates.

Bring your own creative goal and authorized materials. The coordinator organizes staged work, review, and revision. The package does not include completed screenplays, novels, production assets, or finished films.

## Language support

Chinese and English READMEs, getting-started instructions, runtime rules, and receipt documentation are available. You can ask the agent to communicate and create new content in English. **Underlying Agent and Skill instructions remain primarily Chinese**; this is not a full translation of every method or a verified English creative end-to-end run.

Keep file names, command tokens, IDs, JSON keys, and status values unchanged. Translate existing screenplay dialogue only when explicitly requested. The host's model must be able to read the supplied method instructions.

## Choose a project

| Project | Best starting point | Workflow |
|---|---|---|
| [Feilian — Director Express](https://github.com/XAzer-R/azer-feilian-director/blob/main/README.en.md) | You have a screenplay and want shot designs and video prompts | Director → art designer → checked asset bindings |
| [Zhulong — Director Pro](https://github.com/XAzer-R/azer-zhulong-director-pro/blob/main/README.en.md) | You need more control over identity references and keyframes | Director → art designer → storyboard refinement → coordinator handoff |
| [Qingqiu — Screenwriting](https://github.com/XAzer-R/azer-qingqiu-screenplay/blob/main/README.en.md) | You want to develop an idea or authorized adaptation | Incubation → structure and characters → scenes → independent review and revision |

These projects run independently. You may hand a screenplay to a director workflow without installing the other two systems as dependencies.

## Capabilities

- Screenwriter and reviewer roles, with separate work directories.
- 11 top-level Skills, loaded according to the current task.
- Explicit writer responsibilities, input versions, and review state.
- At most two automatic revision rounds; unresolved failures remain failures.
- Offline package validation and read-only review receipt checks.

## Requirements

- The original role layout targets Claude Code with project `.claude/agents` and `.claude/skills` support.
- Bring your own installed host and model access. No credentials, weights, prepaid usage, or subscriptions are included.
- Python 3.10+ is needed only for the offline tools; they have no third-party Python dependencies.
- Other hosts may adapt these methods, but automatic role registration and equivalent behavior are not generally verified.

## Installation

```shell
git clone https://github.com/XAzer-R/azer-qingqiu-screenplay.git
cd azer-qingqiu-screenplay
python -B tools/validate.py
claude
```

An identical Gitee mirror is available at [X-zer/azer-qingqiu-screenplay](https://gitee.com/X-zer/azer-qingqiu-screenplay); you can clone its HTTPS URL instead. Without Git, use the hosting site's ZIP download and start your host inside the extracted repository root.

The root `CLAUDE.md` loads the runtime contract and coordinator instructions. Opening a Markdown file does not install a program.

## First task

Tell the coordinator:

```text
Please communicate in English and help me create a new screenplay project.
Start by asking about the intended audience, format, and creative goal.
Keep technical paths and field names unchanged.
```

No existing work is a normal first-run state. The coordinator creates a new `projects/<ascii-name>/` directory after collecting the necessary information. Method files remain relative to the system root; story files and references belong to the selected work root.

| Conversation request | Purpose |
|---|---|
| Create or select a project | Establish the active work |
| Develop an idea, story structure, or characters | Build the creative foundation |
| Write, review, or revise a scene | Work on a specified episode and scene |
| Analyze or adapt source material | Use material you have permission to use |

Chinese slash-style examples in the method files are conversation conventions, not registered host commands. You can state the equivalent request in English. Do not store novels in the system root or inside Skill folders.

## Files and outputs

```text
README.md / README.en.md   Chinese and English project overviews
CLAUDE.md                  Host entry
AGENTS.md                  Repository maintenance instructions
.claude/CLAUDE.md           Coordinator workflow
.claude/agents/             Role definitions
.claude/skills/             Methods and templates
docs/en/                   English operating documentation
tools/                     Read-only validation tools
tests/                     Isolated regression tests
project.json               Release metadata
```

Work files live in `projects/`, including state, characters, episodes, and private references. Blank review templates live in `templates/`.

Work directories are ignored by Git. A few director `assets/` templates are already tracked: after you fill them in, their contents appear as changes. Review your diff and do not blindly commit all production files.

## Validation

```shell
python -B tools/validate.py
python -B -m unittest discover -s tests
python -B tools/workflow_guard.py unit EP01-S01
```

For actual review records, use the command and schema in [Review receipts](docs/en/REVIEW-RECEIPTS.md). Current file hashes establish version consistency, not artistic quality or cryptographic approval.

## What has and has not been verified

Package validation, offline regression tests, and clean-clone checks passed for the published baseline. See the [source runtime review](docs/RUNTIME-REVIEW.md), currently in Chinese, for recorded details.

No real-model creative run, full Claude Code workflow, media generation, or English-language end-to-end run is claimed. A readable method package is not a guarantee of one-click filmmaking. Platform limits and actual image/video outcomes require separate verification.

Read the [English runtime contract](docs/en/RUNTIME-CONTRACT.md) for review limits, stale results, file ownership, missing inputs, and execution permissions.

## Contributing, security, and license

See the [English contribution and security guide](docs/en/CONTRIBUTING-SECURITY.md). Provide minimal, sanitized reproduction steps. Do not upload unpublished scripts, complete novels, client data, unlicensed film frames, credentials, or private logs.

Author: **Azer (阿泽)**. Licensed under [MIT](LICENSE). Retain the copyright and license notice when using, modifying, or distributing the package. Methodological references do not imply third-party endorsement; see [NOTICE](NOTICE.md).

The series names use Chinese mythological imagery as branding; they are not presented as quotations or literal functional definitions from ancient texts.
