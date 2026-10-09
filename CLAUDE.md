# CLAUDE.md

本仓库是 `vinvcn/mattpocock-skills-zh-CN` 的**精选定制发布集合**：只发布 **14 个技能**（engineering 11 + productivity 3），并保留 **4 处经验证的最小回填**。仓库约定详见 [AGENTS.md](./AGENTS.md)。

Skills 按 bucket folder 组织在 `skills/` 下：

- `engineering/` - 日常代码工作
- `productivity/` - 日常非代码工作流工具

> 上游的 `misc/`、`in-progress/`、`deprecated/` 以及未纳入的技能已移除，不随本仓库发布。

`engineering/` 或 `productivity/` 中的每个 skill，都必须在顶层 `README.md` 中有引用，并在 `.claude-plugin/plugin.json` 中有条目。顶层 `README.md` 中的每个 skill 条目都必须把 skill 名称链接到对应的 `SKILL.md`。每个 bucket folder 的 `README.md` 按 **User-invoked** 与 **Model-invoked** 分组列出该 bucket 的 skills。

每个 `SKILL.md` 要么是 user-invoked（`disable-model-invocation: true`），要么是 model-invoked。定义见 [docs/invocation.md](./docs/invocation.md)。

本仓库也是单 plugin 的 Claude Code marketplace（`.claude-plugin/`）。修改 plugin manifest 后运行 `claude plugin validate . --strict`。

## 本地回填

相对中文版上游有 4 处有意回填（`implement` / `code-review` / `diagnosing-bugs` / `grilling`）。用 `grep -rl "本地回填" skills/` 识别（期望 4 个）。原因与重新应用方法见 [BACKPORTS.md](./BACKPORTS.md)。

## 翻译术语

翻译与本地化时参考仓库根目录的 [翻译术语表](./TRANSLATION-GLOSSARY.md)；术语的请求与决定流程见 [贡献指南](./CONTRIBUTING.md)。从 `mattpocock/skills` 刷新内容前先读 [`.skills/translate-skill/SKILL.md`](./.skills/translate-skill/SKILL.md)。

行文不使用破折号（中文「——」与英文 em-dash `—`），与上游 v1.3.1 的 em-dash 清理一致：改用逗号、冒号、分号、括号或拆句，不要在后续翻译中重新引入。领域文档约定沿用上游 v1.3.1 的重命名：文件名为 `GLOSSARY.md` / `GLOSSARY-MAP.md`（文件名保留英文，不本地化）。

## 同步与回填记录

同步见 [SYNC.md](./SYNC.md)，回填见 [BACKPORTS.md](./BACKPORTS.md)，上游来源见 [UPSTREAM.md](./UPSTREAM.md)。
