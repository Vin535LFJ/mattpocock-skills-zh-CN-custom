# SYNC.md — 更新、比较差异、验证与发布

本文件是本仓库的同步操作手册。上游来源与基线 SHA 见 [UPSTREAM.md](./UPSTREAM.md)；四处回填见 [BACKPORTS.md](./BACKPORTS.md)。

## 最新同步

- 本仓库基线：`vinvcn/mattpocock-skills-zh-CN@bf98e53`（同步英文上游 `mattpocock/skills@24fe0ef`，v1.3.1）。
- 上游当前 HEAD（建立本仓库时）：`mattpocock/skills@b0618bc`。**本仓库尚未整体同步到该 HEAD**，仅按需回填 4 处（见 BACKPORTS.md）。

## 操作流程

### 0. 准备

```bash
# 确认 remote 已就绪
git remote -v
# 期望：upstream-zh -> vinvcn/mattpocock-skills-zh-CN，upstream-en -> mattpocock/skills
# 拉取上游引用（不合并）
git fetch upstream-zh --prune
git fetch upstream-en --prune
```

### 1. 比较差异

```bash
# 中文版上游相对本仓库基线的变化（了解翻译刷新）
git log --oneline bf98e53..upstream-zh/main

# 英文上游相对中文版同步基线的变化（找出待回填项）
git log --oneline 24fe0ef..upstream-en/main

# 只看 4 处回填涉及技能的上游改动（其余技能的上游变化可整体跟进）
git log --oneline 24fe0ef..upstream-en/main -- \
  skills/engineering/implement skills/engineering/code-review \
  skills/engineering/diagnosing-bugs skills/productivity/grilling
```

### 2. 刷新内容（保持全量技能集）

1. 先读 [`.skills/translate-skill/SKILL.md`](./.skills/translate-skill/SKILL.md)。
2. 只翻译自然语言说明；保留目录名、skill name、frontmatter key、命令、代码块、路径、URL 与 tool/API identifiers。
3. 术语以 [TRANSLATION-GLOSSARY.md](./TRANSLATION-GLOSSARY.md) 为准。
4. 若上游新增/删除了技能或附属文件（如 `agents/openai.yaml`、参考文档、脚本），同步更新，保持相对路径不变。
5. **保持全量**：本仓库**不裁剪**技能集，应与中文版上游一致（engineering 20 + productivity 7 + misc 4 + in-progress 6 = **37**）。

```bash
find skills -name SKILL.md | wc -l   # 期望 37
```

### 3. 重新应用四处回填（刷新后必做）

刷新会覆盖本仓库相对上游的 4 处回填，因此**每次刷新后**都要对照 [BACKPORTS.md](./BACKPORTS.md) 逐项重新应用，并用其中的「验证方法」逐条核对。

```bash
grep -rl "本地回填" skills/    # 期望恰好 4 个 SKILL.md
```

### 4. 验证（每次改动后）

```bash
# a. 技能数：应为 37
find skills -name SKILL.md | wc -l

# b. 回填数：应为 4
grep -rl "本地回填" skills/ | wc -l

# c. 发布索引与技能数一致
node -e "const p=require('./.claude-plugin/plugin.json'); console.log('plugin.json skills:', p.skills.length)"

# d. installer 识别到的技能：应为 37（隔离目录内执行，勿污染其它仓库）
npx skills@latest add . --list

# e. 翻译/结构检查（仓库自带脚本）
node scripts/check-translation.mjs
node scripts/audit-english.mjs

# f. plugin manifest 校验（如本机有 claude CLI）
claude plugin validate . --strict
```

### 5. 提交与发布

```bash
# 一次同步一个主题，提交信息写清上游基线 SHA 与本次改动
git add -A
git commit -m "sync: 刷新自 <上游基线 SHA>，重新应用 4 处回填"

# 推送到本仓库自己的远程（origin 配置见 UPSTREAM.md）
git push origin main
```

> 发布前确认：`LICENSE` / `LICENSE.zh-CN.md` 与署名仍在；README / bucket README / `plugin.json` 三处技能清单一致。

## 同步记录（沿用中文版上游历史）

- 2026-10-08：中文版上游同步 `mattpocock/skills@24fe0ef`（v1.3.1）：`implement-spec`、`pr`、`retro` 从 in-progress 毕业至 engineering，移除 `resolving-merge-conflicts`，`CONTEXT.md`/`CONTEXT-MAP.md` 重命名为 `GLOSSARY.md`/`GLOSSARY-MAP.md`，并落地上游 em-dash 清理与跨 skill 调用措辞修正；维护 commit 为 `f659bb3`。
- 2026-09-24：中文版上游同步 `mattpocock/skills@c55ee46`，新增 beta `pr` 的简体中文翻译；维护 commit 为 `d71a03d`。
- 2026-09-17：中文版上游同步 `mattpocock/skills@959a8e9` 对 `retro` 的 deterministic checks 指引；维护 commit 为 `fb3bd36`。
- 2026-08-25：中文版上游同步 `mattpocock/skills@6654f6b` 的内容变化；维护 commit 为 `df042c8`，PR 为 [#29](https://github.com/vinvcn/mattpocock-skills-zh-CN/pull/29)。

## 本仓库记录

- 2026-10-09（收口）：确立本仓库为**中文版全量发行副本**（engineering 20 + productivity 7 + misc 4 + in-progress 6 = **37 个技能**）+ **4 处**已对照英文上游验证的回填；基线 `vinvcn/mattpocock-skills-zh-CN@bf98e53`。
- 2026-10-09（建库 → 调整为全量）：曾以「精选 14 技能」形态建立（commit `d182674`），后按「**全量发行 + 少量修复**」的方向调整：恢复全部技能与 `docs/`，移除 `dsh-plugin/`（非本仓库范围），保留 4 处回填与 `UPSTREAM.md` / `BACKPORTS.md` / `AGENT-COMPATIBILITY.md`。
