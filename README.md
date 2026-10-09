# Matt Pocock Agent Skills 中文版 · 精选定制集合

本仓库是 [`vinvcn/mattpocock-skills-zh-CN`](https://github.com/vinvcn/mattpocock-skills-zh-CN)（[`mattpocock/skills`](https://github.com/mattpocock/skills) 的简体中文本地化版）的**独立维护的精选发布集合**，用于在多个 AI Coding 工具（Claude Code、Codex、ZCode、VS Code Copilot、OpenCode 等）和不同项目中复用 Matt Pocock Skills。

与上游的区别只有两点，都是有意为之：

1. **只发布 14 个精选技能**（engineering 11 + productivity 3），其余上游技能不随本仓库发布。
2. **保留 4 处经验证的最小回填**（相对中文版上游），见 [BACKPORTS.md](./BACKPORTS.md)。

技能名称、目录名、路径、frontmatter 字段和工具标识全部保持与上游一致，以免破坏安装和运行行为；只做简体中文本地化与上述最小回填。

## 14 个技能清单

### Engineering（11）

**User-invoked**（只能由人显式调用）

- **[grill-with-docs](./skills/engineering/grill-with-docs/SKILL.md)** - 追问式访谈，同时构建项目的 domain model、打磨术语，并内联更新 `GLOSSARY.md` 与 ADRs。
- **[setup-matt-pocock-skills](./skills/engineering/setup-matt-pocock-skills/SKILL.md)** - 配置 issue tracker、triage labels 与 domain docs 布局。每个 repo 运行一次。
- **[to-spec](./skills/engineering/to-spec/SKILL.md)** - 把当前对话整理成 spec 并发布到 issue tracker。
- **[to-tickets](./skills/engineering/to-tickets/SKILL.md)** - 把 plan、spec 或 conversation 拆成 tracer-bullet tickets，每个 ticket 声明 blocking edges。
- **[implement](./skills/engineering/implement/SKILL.md)** - 基于 spec 或 ticket 集合实现一段工作，在预先认可的 seams 上调用 Skill 工具执行 `tdd`，提交前调用 Skill 工具执行 `code-review` 收尾。
- **[retro](./skills/engineering/retro/SKILL.md)** - 一次 session 结束后，针对 coding agent 的环境提出改进建议，最严重的排最前。

**Model-invoked**（模型和用户都可以调用）

- **[diagnosing-bugs](./skills/engineering/diagnosing-bugs/SKILL.md)** - 面向棘手 bug 和性能回退的纪律化诊断循环。
- **[tdd](./skills/engineering/tdd/SKILL.md)** - 使用 red-green-refactor 循环做 test-driven development。
- **[domain-modeling](./skills/engineering/domain-modeling/SKILL.md)** - 主动构建和打磨项目的 domain model，并内联更新 `GLOSSARY.md` 与 ADRs。
- **[codebase-design](./skills/engineering/codebase-design/SKILL.md)** - 设计 deep modules 的共享纪律和词汇。
- **[code-review](./skills/engineering/code-review/SKILL.md)** - 对固定点之后的 diff 做双轴 review（Standards + Spec），作为并行 sub-agents 运行。

### Productivity（3）

**User-invoked**

- **[handoff](./skills/productivity/handoff/SKILL.md)** - 把当前对话压缩成 handoff document，让另一个 agent 可以继续。

**Model-invoked**

- **[grilling](./skills/productivity/grilling/SKILL.md)** - 围绕计划、decision 或 idea 持续访谈用户，直到 design tree 的每个分支都被解决。
- **[writing-for-agents](./skills/productivity/writing-for-agents/SKILL.md)** - 为 agent 编写文档：skills、`AGENTS.md`/`CLAUDE.md`，以及任何 agent 通过指针触达的文档。

> 未纳入本集合的上游技能：`ask-matt`、`triage`、`improve-codebase-architecture`、`implement-spec`、`wayfinder`、`pr`、`prototype`、`research`、`wizard`、`grill-me`、`teach`、`to-questionnaire`、`wait-what`，以及 `misc/`、`in-progress/`、`deprecated/` 三个 bucket。需要时请从上游或中文版上游安装。

## 安装

### 方式一：skills.sh installer（推荐，支持多种 agent）

```bash
# 从本仓库安装（私有仓库需先具备访问权限）
npx skills@latest add Vin535LFJ/mattpocock-skills-zh-CN-custom -s setup-matt-pocock-skills grill-with-docs domain-modeling \
  to-spec to-tickets implement tdd diagnosing-bugs code-review retro handoff \
  codebase-design writing-for-agents grilling \
  -a claude-code codex zcode github-copilot -y --copy

# 或从本地克隆安装
npx skills@latest add . --list          # 先看识别到哪些技能（应为 14 个）
npx skills@latest add . -a codex -y --copy
```

- `-s` / `-a` **不接受逗号分隔**，必须用空格分隔（逗号会被当成一个名字 ⇒ `No matching skills found`）。
- `--copy` 避免 Windows 符号链接问题。
- `codex` 与 `github-copilot` 共用 `.agents/skills/`；`zcode` 用 `.zcode/skills/`。
- WorkBuddy 不在 installer 支持的 agent 列表中；其项目级技能目录是 `{workspace}/.workbuddy-ai/skills/`，需手动投放。

首次安装后，在每个 repo 中运行一次 `setup-matt-pocock-skills` 来完成 issue tracker、labels 和 docs 目录配置。

### 方式二：作为 Claude Code plugin

```
/plugin marketplace add Vin535LFJ/mattpocock-skills-zh-CN-custom
/plugin install mattpocock-skills@mattpocock
```

两种方式只选其一：两个都装会让每个 skill 被安装两次。

## 验证安装

```bash
# 1. 列出定制源识别到的技能：应为 14 个
npx skills@latest add . --list

# 2. 核对磁盘上的技能数：应为 14 个 SKILL.md
find skills -name SKILL.md | wc -l

# 3. 核对四处回填：应恰好命中 4 个 SKILL.md
grep -rl "本地回填" skills/

# 4. 核对发布索引与技能数一致
node -e "const p=require('./.claude-plugin/plugin.json'); console.log(p.skills.length)"
```

## 四处回填

相对中文版上游，本仓库保留了 4 处经验证的最小回填（`implement`、`code-review`、`diagnosing-bugs`、`grilling`）。每处都记录了上游基线提交、原始差异、调整理由与验证方法，见 [BACKPORTS.md](./BACKPORTS.md)。

> ⚠️ 从上游刷新内容时，这 4 处可能被覆盖，需对照 [BACKPORTS.md](./BACKPORTS.md) 重新应用。

## 上游与许可证

- 英文上游：[`mattpocock/skills`](https://github.com/mattpocock/skills)（作者 Matt Pocock）
- 中文版上游：[`vinvcn/mattpocock-skills-zh-CN`](https://github.com/vinvcn/mattpocock-skills-zh-CN)
- 基线提交、同步方式与远程配置见 [UPSTREAM.md](./UPSTREAM.md)

许可证为 MIT，见 [LICENSE](./LICENSE)（英文原文）与 [LICENSE.zh-CN.md](./LICENSE.zh-CN.md)（中文参考译文）。分发与二次发布前请先阅读许可证条款。

## 维护文档

| 文档 | 内容 |
|---|---|
| [README.md](./README.md) | 用途、14 个技能清单、安装与验证方法 |
| [UPSTREAM.md](./UPSTREAM.md) | 上游仓库、基线提交、远程配置与同步方式 |
| [BACKPORTS.md](./BACKPORTS.md) | 四处回填的原因、文件路径与重新应用方法 |
| [SYNC.md](./SYNC.md) | 更新、比较差异、验证与发布的操作流程 |
| [AGENTS.md](./AGENTS.md) / [CLAUDE.md](./CLAUDE.md) | 仓库结构与约定 |
| [CONTRIBUTING.md](./CONTRIBUTING.md) | 翻译请求与贡献流程 |
| [TRANSLATION-GLOSSARY.md](./TRANSLATION-GLOSSARY.md) | 翻译术语表 |
