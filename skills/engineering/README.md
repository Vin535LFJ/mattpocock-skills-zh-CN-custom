# Engineering

我每天用于代码工作的 skills。

> 本仓库是精选发布集合，仅包含下列 11 个 engineering 技能。其余上游技能（`ask-matt`、`triage`、`improve-codebase-architecture`、`implement-spec`、`wayfinder`、`pr`、`prototype`、`research`、`wizard`）未纳入本仓库，见 [README](../../README.md)。

## User-invoked

只有在你显式输入名称时才能调用（Claude Code：`disable-model-invocation: true`；Codex：`agents/openai.yaml` 中的 `policy.allow_implicit_invocation: false`）。

- **[grill-with-docs](./grill-with-docs/SKILL.md)** - 追问式访谈，同时构建项目的 domain model、打磨术语，并内联更新 `GLOSSARY.md` 与 ADRs。
- **[setup-matt-pocock-skills](./setup-matt-pocock-skills/SKILL.md)** - 为 engineering skills 配置本仓库需要的 issue tracker、triage labels 与 domain docs 布局。每个 repo 运行一次。
- **[to-spec](./to-spec/SKILL.md)** - 把当前对话整理成 spec 并发布到 issue tracker。
- **[to-tickets](./to-tickets/SKILL.md)** - 把 plan、spec 或 conversation 拆成 tracer-bullet tickets，每个 ticket 声明 blocking edges：本地文件中用文本表示，真实 tracker 上用 native blocking links。
- **[implement](./implement/SKILL.md)** - 基于 spec 或 ticket 集合实现一段工作，在预先认可的 seams 上调用 Skill 工具执行 `tdd`，并在提交前调用 Skill 工具执行 `code-review` 收尾。
- **[retro](./retro/SKILL.md)** - 一次 session 结束后，针对 coding agent 的环境（navigation、automated checks、coding standards、steering files、tooling）提出改进建议，最严重的排最前。

## Model-invoked

模型或用户都可以调用（description 包含足够丰富的触发措辞，方便模型自动找到它们）。

- **[diagnosing-bugs](./diagnosing-bugs/SKILL.md)** - 面向棘手 bug 和性能回退的纪律化诊断循环：reproduce -> minimise -> hypothesise -> instrument -> fix -> regression-test。
- **[tdd](./tdd/SKILL.md)** - 使用 red-green-refactor 循环做 test-driven development；一次一个 vertical slice 地构建功能或修复 bug。
- **[domain-modeling](./domain-modeling/SKILL.md)** - 主动构建和打磨项目的 domain model：挑战术语、用场景做压力测试，并内联更新 `GLOSSARY.md` 与 ADRs。
- **[codebase-design](./codebase-design/SKILL.md)** - 用于设计 deep modules 的共享纪律和词汇：小 interface、清晰 seam、通过 interface 测试。
- **[code-review](./code-review/SKILL.md)** - 对固定点之后的 diff 做双轴 review：**Standards**（是否遵循 repo 的编码规范，外加 Fowler smell baseline？）和 **Spec**（是否忠实实现了源头的 issue/PRD？），作为并行 sub-agents 运行。
