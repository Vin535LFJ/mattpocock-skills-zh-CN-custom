# 仓库指南

本仓库是 `vinvcn/mattpocock-skills-zh-CN`（Matt Pocock Skills 简体中文本地化版）的**精选定制发布集合**：只发布 **14 个技能**（engineering 11 + productivity 3），并保留 **4 处经验证的最小回填**。

Skills 按 bucket folder 组织在 `skills/` 下：

- `engineering/` - 日常代码工作（11 个）
- `productivity/` - 日常非代码工作流工具（3 个）

> 上游的 `misc/`、`in-progress/`、`deprecated/` 三个 bucket 以及未纳入的 engineering/productivity 技能，**已从本仓库移除**，不随本仓库发布。上游仍保留它们，见 [UPSTREAM.md](./UPSTREAM.md)。

`engineering/` 或 `productivity/` 中的每个 skill，都必须在顶层 `README.md` 中有引用，并在 `.claude-plugin/plugin.json` 中有条目。

顶层 `README.md` 中的每个 skill 条目都必须把 skill 名称链接到对应的 `SKILL.md`。

每个 bucket folder 都有一个 `README.md`，列出该 bucket 中的所有 skills，并给出一行描述；skill 名称需要链接到对应的 `SKILL.md`。Bucket `README.md` 和顶层 `README.md` 都按 **User-invoked** 与 **Model-invoked** 分组。

每个 `SKILL.md` 要么是 user-invoked（frontmatter 中设置 `disable-model-invocation: true`，并在 `agents/openai.yaml` 中设置 `policy.allow_implicit_invocation: false`，只能由人类显式调用），要么是 model-invoked（模型和用户都可以调用）。完整定义、description 约定，以及为什么 user-invoked skill 可以调用 model-invoked skills 但不能调用另一个 user-invoked skill，见 [docs/invocation.md](./docs/invocation.md)。

本仓库也是一个单 plugin 的 Claude Code marketplace：`.claude-plugin/marketplace.json` 列出唯一的 `mattpocock-skills` plugin。修改 `.claude-plugin/plugin.json` 或 marketplace manifest 后，运行 `claude plugin validate . --strict`。Plugin 的公开 skill 集合继续遵循本仓库 bucket 规则。

## 本地回填（不要被同步覆盖）

本仓库相对中文版上游有 **4 处**有意的最小回填，集中在 `implement`、`code-review`、`diagnosing-bugs`、`grilling` 四个技能。识别方式：

```bash
grep -rl "本地回填" skills/
# 期望恰好 4 个 SKILL.md
```

原因、上游基线、原始差异与重新应用方法见 [BACKPORTS.md](./BACKPORTS.md)。**从上游刷新内容后，务必对照 BACKPORTS.md 重新应用这 4 处回填。**

## 翻译刷新

从 `mattpocock/skills` 刷新上游内容时，改文件前先使用 [`.skills/translate-skill/SKILL.md`](./.skills/translate-skill/SKILL.md)。本仓库采用 skill-guided content localization，不做 Git fork-sync：保留简体中文本地化身份，安装命令指向本仓库地址，不要导入上游 repository-management state。翻译术语以 [翻译术语表](./TRANSLATION-GLOSSARY.md) 为准：刷新与本地化时优先采用已决定的译法；未决定的术语先按 [贡献指南](./CONTRIBUTING.md) 的流程提出请求，决定后再落地。

行文不使用破折号（中文「——」与英文 em-dash `—`），与上游 v1.3.1 的 em-dash 清理一致：改用逗号、冒号、分号、括号或拆句，不要在后续翻译中重新引入。领域文档约定沿用上游 v1.3.1 的重命名：文件名为 `GLOSSARY.md` / `GLOSSARY-MAP.md`（文件名保留英文，不本地化）。

## 同步与回填记录

- 同步记录写在 [SYNC.md](./SYNC.md)。
- 回填记录写在 [BACKPORTS.md](./BACKPORTS.md)。
- 上游仓库、基线提交与同步方式见 [UPSTREAM.md](./UPSTREAM.md)。
- 跨 Agent 调用兼容方案见 [AGENT-COMPATIBILITY.md](./AGENT-COMPATIBILITY.md)。
