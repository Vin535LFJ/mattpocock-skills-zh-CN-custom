# BACKPORTS.md — 四处最小回填

本仓库相对**中文版上游** `vinvcn/mattpocock-skills-zh-CN` 保留了 **4 处经验证的最小回填**。这 4 处对应**英文上游**已修、但中文版尚未同步的改动，属于跨工具兼容与防自欺的护栏。

> ⚠️ **这是一处有意的本地差异。** 从上游刷新内容时，这些改动**可能被覆盖**；覆盖后请对照本文件重新应用。
>
> 识别方式：`grep -rl "本地回填" skills/` ⇒ 期望恰好命中 4 个 `SKILL.md`（`implement` / `code-review` / `diagnosing-bugs` / `grilling`）。

## 版本基线

| 项 | 值 |
|---|---|
| 英文上游 | `mattpocock/skills` |
| 中文版上游同步基线 | `mattpocock/skills@24fe0ef`（v1.3.1，2026-10-08） |
| 回填依据的上游提交区间 | `24fe0ef..b0618bc`（v1.3.1 → 上游当前 HEAD） |
| 回填时上游 HEAD | `mattpocock/skills@b0618bc` |
| 中文版上游 HEAD | `vinvcn/mattpocock-skills-zh-CN@bf98e53` |
| 本仓库基线 | `bf98e53`（沿用中文版上游 Git 历史） |

## 总览

| # | 技能 | 文件 | 上游提交 | 改动要点 |
|---|---|---|---|---|
| 1 | `implement` | `skills/engineering/implement/SKILL.md` | `e48341a` + `04320ee` + `6d6a5b9` | 斜杠命令 → Skill 工具调用；ticket 标题前置 |
| 2 | `code-review` | `skills/engineering/code-review/SKILL.md` | `3da8c01` | standards 来源扩大；子代理前台并行；tracker doc 措辞泛化 |
| 3 | `diagnosing-bugs` | `skills/engineering/diagnosing-bugs/SKILL.md` | `f3fc563` | 强制造红须 diff 未改动副本自证 |
| 4 | `grilling` | `skills/productivity/grilling/SKILL.md` | `95249b0` | 问题措辞须让「是」等于接受推荐答案 |

---

## 1. `implement` — 工具调用方式 + ticket 标题前置

**文件**：`skills/engineering/implement/SKILL.md`

**为什么**：中文版仍写「使用 `/tdd`」「使用 `/code-review`」的**斜杠命令**风格；上游已改为**调用 Skill 工具**。本仓库同时用于 Claude Code / Codex / ZCode / WorkBuddy，**斜杠命令在这些工具里不成立** ⇒ 属跨工具兼容性修复。

**上游基线（英文原文）**

```diff
 Implement the work described by the user in the spec or tickets.
 
-Use /tdd where possible, at pre-agreed seams.
+If the user passes a ticket reference, fetch it from the issue tracker and state its title before starting. If the reference is ambiguous, ask.
+
+Call the Skill tool with "tdd" where possible, at pre-agreed seams.
 
 Run typechecking regularly, single test files regularly, and the full test suite once at the end.
 
-Once done, use /code-review to review the work.
+Once done, call the Skill tool with "code-review" to review the work.
 
 Commit your work to the current branch.
```

**本仓库应用的中文差异**

```diff
 实现用户在 spec 或 tickets 中描述的工作。
 
-尽可能在预先约定好的 seams 上使用 `/tdd`。
+如果用户传入了 ticket 引用，先从 issue tracker 取回它，并在开始前说明它的标题。如果引用有歧义，先询问。
+
+尽可能在预先约定好的 seams 上，调用 Skill 工具执行 `tdd`。
 
 ...
 
-完成后，使用 `/code-review` 审查这次工作。
+完成后，调用 Skill 工具执行 `code-review` 审查这次工作。
```

**重新应用方法**：从上游 `mattpocock/skills` 取 `skills/engineering/implement/SKILL.md` 在 `b0618bc` 的版本，按上述对应关系把中文版的三处措辞改回。

**验证方法**：`grep -n "Skill 工具执行" skills/engineering/implement/SKILL.md` 应命中 2 处；`grep -n "ticket 引用" ...` 应命中 1 处；文件中不应再出现 `/tdd` 或 `/code-review` 斜杠调用。

---

## 2. `code-review` — standards 来源扩大 + 子代理前台并行

**文件**：`skills/engineering/code-review/SKILL.md`

**为什么**：上游修正了「standards 来源只提两个文件名」与「子代理发出方式含糊」两个问题。

**上游基线（英文原文）**

```diff
-The issue tracker should have been provided to you. If `docs/agents/issue-tracker.md` is missing, tell the user to run `/setup-matt-pocock-skills`.
+The issue tracker should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.
@@
-1. Issue references in the commit messages (…), fetched via the workflow in `docs/agents/issue-tracker.md`.
+1. Issue references in the commit messages (…), fetched via the workflow in the tracker doc.
@@
-Anything in the repo that documents how code should be written, such as `CODING_STANDARDS.md` or `CONTRIBUTING.md`.
+Search the repo for every file that documents how code should be written. When `CODING_STANDARDS.md` or `CONTRIBUTING.md` exists, it must be on the list.
@@
 ### 4. Spawn both sub-agents in parallel
 
+Issue both sub-agent calls together, in the foreground, and aggregate the reports they return.
+
 **Standards sub-agent prompt** should include:
```

**本仓库应用的中文差异**：对应四处

- `Issue tracker 应该已经提供给你；如果缺少 \`docs/agents/issue-tracker.md\`，请让用户运行 \`/setup-matt-pocock-skills\`。` → `…；如果没有，请让用户运行 \`/setup-matt-pocock-skills\`。`
- `1. Commit messages 中的 issue references（…），按 \`docs/agents/issue-tracker.md\` 中的 workflow 获取。` → `…按 tracker doc 中的 workflow 获取。`
- `Repo 中任何记录代码应该如何写的内容，例如 \`CODING_STANDARDS.md\` 或 \`CONTRIBUTING.md\`。` → `搜索 repo 中**每一个**记录「代码应该如何写」的文件。当 \`CODING_STANDARDS.md\` 或 \`CONTRIBUTING.md\` 存在时，它们必须在列表上。`
- 在 `### 4. 并行启动两个 sub-agents` 之后新增：`把两个 sub-agent 调用**一起发出、在前台执行**，然后聚合它们返回的报告。`

**重新应用方法**：从上游取 `skills/engineering/code-review/SKILL.md` 在 `b0618bc` 的版本，按上述四处对应关系回填中文版。

**验证方法**：`grep -n "tracker doc\|每一个\|一起发出" skills/engineering/code-review/SKILL.md` 应命中 3 处。

---

## 3. `diagnosing-bugs` — 强制造红需自证

**文件**：`skills/engineering/diagnosing-bugs/SKILL.md`

**为什么**：上游新增了一条防自欺的护栏。若「红」是人为改动代码/fixture 造出来的，必须先证明该改动确实生效，否则后续所有推理都建立在假信号上。

**上游基线（英文原文）**

```diff
 1. Turn the minimised repro into a failing test at that seam.
-2. Watch it fail.
+2. Watch it fail. If you forced the red by mutating code or a fixture, `diff` against a pristine copy to prove the mutation landed before you trust it.
 3. Apply the fix.
```

**本仓库应用的中文差异**

```diff
 1. 把 minimised repro 变成该 seam 上的 failing test。
-2. 看它 fail。
+2. 看它 fail。如果你是通过改动代码或 fixture 强行造出的红，先 `diff` 一份未改动的副本，证明这个改动确实生效了，再相信它。
 3. 应用 fix。
```

**重新应用方法**：从上游取 `skills/engineering/diagnosing-bugs/SKILL.md` 在 `b0618bc` 的版本，把 Phase 5 的第 2 步改回带护栏的版本。

**验证方法**：`grep -n "强行造出的红" skills/engineering/diagnosing-bugs/SKILL.md` 应命中 1 处。

---

## 4. `grilling` — 问题措辞须让「是」等于接受推荐答案

**文件**：`skills/productivity/grilling/SKILL.md`

**为什么**：上游新增一句约束，防止推荐答案与「是/否」的语义错位（`grill-with-docs` 依赖本技能）。回填时上游提交为 `95249b0`。

**上游基线（英文原文）**

```diff
 ➡️ <your recommended answer>
 ```
 
+Word each question so "yes" accepts your recommended answer.
+
 Each round the user answers reshapes the tree: …
```

**本仓库应用的中文差异**

```diff
 ➡️ <你的推荐答案>
 ```
 
+把每个问题都措辞成：回答「是」就等于接受你推荐的答案。
+
 每一轮用户给出的回答都会重塑这棵树：…
```

**重新应用方法**：从上游取 `skills/productivity/grilling/SKILL.md` 在 `b0618bc` 的版本，把该句回填到 round 格式说明之后。

**验证方法**：`grep -n "就等于接受你推荐的答案" skills/productivity/grilling/SKILL.md` 应命中 1 处。

---

## 上游尚未同步、但**未回填**的项（供后续评估）

以下为中文版落后于上游的其余改动，本仓库**未回填**（评估后认为对精选集合影响小，或属于未纳入的技能）：

- `setup-matt-pocock-skills`：`triage-labels` / `issue-tracker-github` 措辞（本集合不含 `triage`）
- `tdd`：为每个提议的 seam 补一行「能抓到什么 / 漏掉什么」（`3f59913`）
- `to-tickets`：sub-issue / 原生 blocking 关系（`9e2abf8`、`cffab50`）
- `handoff`：临时目录解析顺序（`4f4e943`、`0f5e033`）
- `ask-matt` / `wayfinder` / `wizard` / `teach` 等**未纳入本集合**技能的改动
- 上游新增的 `in-progress/chief-of-staff`（**未纳入，不安装**）
