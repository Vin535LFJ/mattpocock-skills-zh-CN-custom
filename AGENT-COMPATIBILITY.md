# AGENT-COMPATIBILITY.md — 跨 Agent 调用兼容方案

> 本文档**只做设计**，不修改任何技能文件，也不新增第五处回填。它描述 37 个技能（含 4 处回填）在不同宿主（harness）中的调用方式、能力边界与回退机制。
>
> 相关文档：[README](./README.md)（用途与安装）、[UPSTREAM](./UPSTREAM.md)（上游与远程）、[BACKPORTS](./BACKPORTS.md)（四处回填）、[SYNC](./SYNC.md)（同步流程）、[docs/invocation.md](./docs/invocation.md)（仓库自身的 user-invoked / model-invoked 约定）。

## 0. 结论摘要

- **技能源是纯 Markdown 文件**，不依赖任何宿主专有二进制；安装器 `skills`（vercel-labs/skills v1.7.1）已验证可识别 37 个技能并投放文件。
- **真实硬依赖只有 4 条**（都写成「调用 Skill 工具」）：`grill-with-docs → grilling, domain-modeling`、`implement → tdd, code-review`、`tdd → codebase-design`、`retro → writing-for-agents`。
- **回填 #1（implement）是跨工具兼容的关键**：它把 `/tdd`、`/code-review` 斜杠式改成通用「Skill 工具」调用，正是为了让斜杠命令不成立的宿主（Codex / ZCode / WorkBuddy 等）也能工作。
- **本机只验证到「安装器识别 + 文件投放」这一层**；「Agent 运行时发现技能」与「Agent 实际调用成功」**未验证**，不得标记为已验证。
- 所有**跨技能调用都必须遵守 invocation 语义**：`disable-model-invocation: true` 的 7 个 user-invoked 技能只能由人显式触发，**不得用「直接读取 SKILL.md」绕过**。

## 1. 范围与术语

| 术语              | 含义                                                                       |
| --------------- | ------------------------------------------------------------------------ |
| 宿主 / harness    | 运行 agent 的客户端，如 Claude Code、Codex、ZCode、WorkBuddy                        |
| Skill 工具        | 宿主提供的「按名字加载并执行某个 skill」的原生机制（Claude Code 的 Skill tool、Codex 的 skill 机制等） |
| 斜杠命令            | `/skill-name` 形式的人工触发；**并非所有宿主都支持**                                      |
| sub-agent / 子代理 | 宿主提供的并行子任务机制（如 Claude Code 的 Task/subagent）                              |
| user-invoked    | frontmatter 含 `disable-model-invocation: true`，只能由人显式触发（本仓库 22 个）         |
| model-invoked   | 省略 `disable-model-invocation`，模型与人都可触发（本仓库 15 个）                          |

**本仓库 37 个技能的 invocation 分类**（由 frontmatter 与 `agents/openai.yaml` 实测确认）：

| 分类            | 数量 | 技能                                                                                                        |
| ------------- | -- | --------------------------------------------------------------------------------------------------------- |
| user-invoked  | 22 | 逐技能以各 `SKILL.md` 的 frontmatter 为准 |
| model-invoked | 15 | 逐技能以各 `SKILL.md` 的 frontmatter 为准 |

> 只有 model-invoked 的 15 个会参与模型的自动选择；user-invoked 的 22 个只能由人显式触发。

## 2. 调用分层

跨技能引用与调用必须区分以下 4 层，**不得混为一谈**：

| 层  | 名称                | 判定依据                                    | 例子                                               | 约束                                                     |
| -- | ----------------- | --------------------------------------- | ------------------------------------------------ | ------------------------------------------------------ |
| L1 | **宿主原生 Skill 调用** | 文本明确写「调用 Skill 工具并指定 `X`」               | `implement` 调用 `tdd`                             | 目标必须是 model-invoked；宿主须提供 Skill 工具                     |
| L2 | **用户显式触发**        | 文本要求「让用户运行 `/X`」                        | `code-review` 提示用户运行 `/setup-matt-pocock-skills` | 只能由人触发，agent 不得代劳                                      |
| L3 | **模型自动调用**        | 由 model-invoked 技能的 `description` 触发词命中 | 用户说「帮我 review」→ 自动命中 `code-review`               | 仅限 model-invoked                                       |
| L4 | **直接读取技能文件的兼容回退** | 宿主无 Skill 工具时，读取 `SKILL.md` 并按指令执行      | 见 §5                                             | **仅对 model-invoked 允许**；对 user-invoked 禁止（会绕过用户显式触发约束） |

> ⚠️ **L4 的红线**：`implement`、`to-spec`、`to-tickets`、`grill-with-docs`、`retro`、`setup-matt-pocock-skills`、`handoff` 这 7 个是 user-invoked。即使宿主不支持斜杠命令，**也不得**让模型自动读取并执行它们来「替代」用户触发。

## 3. 依赖图：真实调用 vs 纯文本提及

### 3.1 硬依赖（L1，真实 Skill 工具调用）

| 调用方               | 被调用                          | 证据（文件:行）                                        | 原文                                                   |
| ----------------- | ---------------------------- | ----------------------------------------------- | ---------------------------------------------------- |
| `grill-with-docs` | `grilling`、`domain-modeling` | `skills/engineering/grill-with-docs/SKILL.md:7` | 「调用两次 Skill 工具，分别指定 `grilling` 和 `domain-modeling`。」 |
| `implement`       | `tdd`                        | `skills/engineering/implement/SKILL.md:11`      | 「尽可能在预先约定好的 seams 上，调用 Skill 工具执行 `tdd`。」**（回填 #1）** |
| `implement`       | `code-review`                | `skills/engineering/implement/SKILL.md:15`      | 「完成后，调用 Skill 工具执行 `code-review` 审查这次工作。」**（回填 #1）** |
| `tdd`             | `codebase-design`            | `skills/engineering/tdd/SKILL.md:26`            | 「…调用 Skill 工具并指定 `codebase-design` 获取词汇。」            |
| `retro`           | `writing-for-agents`         | `skills/engineering/retro/SKILL.md:11`          | 「调用 Skill 工具并指定 `writing-for-agents`，获取写作风格指南。」      |

依赖图（箭头 = 硬调用）：

```
grill-with-docs ──▶ grilling
                └─▶ domain-modeling

implement ──────▶ tdd ──▶ codebase-design
           └────▶ code-review

retro ──────────▶ writing-for-agents

handoff ────────▶ （"suggested skills" 占位，无具体目标，见 §3.2）
```

- 所有**被调用方都是 model-invoked**，符合 `docs/invocation.md` 的约定（user-invoked 可以调用 model-invoked，反之不可）。
- `implement` 是唯一的 user-invoked 调用方；它调用两个 model-invoked 技能，合法。

### 3.2 软引用（不是技能调用）

| 调用方                        | 目标                                  | 证据                                                                    | 性质                     | 是否算依赖           |
| -------------------------- | ----------------------------------- | --------------------------------------------------------------------- | ---------------------- | --------------- |
| `tdd`                      | `code-review`                       | `tdd/SKILL.md:38`「Refactoring 属于 review stage（见 `code-review` skill）」 | **文本指针**               | ❌ 否             |
| `retro`                    | `writing-for-agents`                | `retro/SKILL.md:44`「遵循 `writing-for-agents` skill 中的建议」               | 文本建议（另有 §3.1 的硬调用）     | 硬调用另有依据         |
| `handoff`                  | （泛化）                                | `handoff/SKILL.md:10`「写明下一个 agent 应调用 Skill 工具并指定哪些 skills」           | **占位说明**，不点名任何技能       | ❌ 否             |
| `code-review`              | `setup-matt-pocock-skills`          | `code-review/SKILL.md:13`「请让用户运行 `/setup-matt-pocock-skills`」         | **L2 用户指令**            | ❌ 否（非 agent 调用） |
| `to-spec`                  | `setup-matt-pocock-skills`          | `to-spec/SKILL.md:9`                                                  | L2 用户指令                | ❌ 否             |
| `to-tickets`               | `setup-matt-pocock-skills`          | `to-tickets/SKILL.md:11,60`                                           | L2 用户指令                | ❌ 否             |
| `setup-matt-pocock-skills` | `grill-with-docs`、`domain-modeling` | `setup-matt-pocock-skills/domain.md:11,45`                            | 文档说明（懒创建 glossary 的来源） | ❌ 否             |
| `setup-matt-pocock-skills` | `to-spec`、`to-tickets`              | `setup-matt-pocock-skills/SKILL.md:40`                                | 文档说明（explainer 文本）     | ❌ 否             |
| `setup-matt-pocock-skills` | `triage`、`wayfinder`                | `setup-matt-pocock-skills/issue-tracker-*.md`                         | 文档说明，且**这两个技能均已在本仓库**  | ❌ 否             |

> 🔴 **重要修正**：任务清单里「`tdd → codebase-design、code-review`」中的 **`code-review` 不是硬依赖**，而是 `tdd/SKILL.md:38` 的一句文本指针（「见 `code-review` skill」），全文没有「调用 Skill 工具并指定 `code-review`」。真实硬依赖只有 `tdd → codebase-design`。

### 3.3 已排除的伪依赖

| 命中模式                                                          | 为什么不是依赖                                                  |
| ------------------------------------------------------------- | -------------------------------------------------------- |
| `implement` / `implementation`                                | 英文普通名词（如「implementation details」），非技能名；产生大量假命中           |
| `gh pr` / `gh issue` / `glab`                                 | **CLI 命令名**，不是技能                                         |
| `wayfinder:<type>` 的 `research`/`prototype`/`grilling`/`task` | **工单标签值**，不是技能调用                                         |
| `triage`                                                      | 在 `setup-matt-pocock-skills` 中是**条件跳过**（未安装时整节省略） |
| `pr`                                                          | `gh pr` 的 CLI 名（见上）                                      |

## 4. 能力矩阵

### 4.1 能力分类（五类，严格区分，不得越级声称）

| 类别           | 判定依据                                             |
| ------------ | ------------------------------------------------ |
| ① **官方文档支持** | `skills` CLI README 的 `agent-list` **明确点名**该宿主   |
| ② **仅安装器声称** | 安装器的 `-a` 列表包含该宿主，但官方 README 未点名                 |
| ③ **实际测试成功** | 本机实测完成**文件投放**：`--copy` 写入目标目录且与源逐字节一致（**仅此一层**） |
| ④ **仅文件存在**  | 文件已落盘，宿主运行时**尚未消费**                              |
| ⑤ **未验证**    | 宿主运行时的「技能发现」与「实际调用」；或整体未测                        |

### 4.2 逐宿主归属

| 宿主                  | ① 官方文档支持 | ② 仅安装器声称 | ③ 实际测试成功（文件投放）            | ④ 仅文件存在 | ⑤ 未验证             |
| ------------------- | -------- | -------- | ------------------------- | ------- | ----------------- |
| **Claude Code**     | ✅        | —        | ✅ `.claude/skills/`（14 个） | ✅       | 运行时发现 / 调用        |
| **Codex**           | ✅        | —        | ✅ `.agents/skills/`（14 个） | ✅       | 运行时发现 / 调用        |
| **ZCode**           | —        | ✅        | ✅ `.zcode/skills/`（14 个）  | ✅       | 运行时发现 / 调用        |
| **OpenCode**        | ✅        | —        | ✅ `.agents/skills/`       | ✅       | 运行时发现 / 调用        |
| **Cursor**          | ✅        | —        | ✅ `.agents/skills/`       | ✅       | 运行时发现 / 调用        |
| **VS Code Copilot** | —        | ✅        | ✅ `.agents/skills/`       | ✅       | 运行时发现 / 调用        |
| **WorkBuddy**       | —        | —        | ❌ 不在 installer agent 列表   | —       | 全流程（投放 + 发现 + 调用） |

说明：

- **① 官方文档支持**（`skills` CLI README 的 `agent-list` 点名）：**OpenCode、Claude Code、Codex、Cursor**。
- **② 仅安装器声称**（`-a` 列表含，README 未点名）：**ZCode、VS Code Copilot**。
- **③ 实际测试成功**（仅到「文件投放」层）：**Claude Code**（`.claude/skills/`）、**Codex / OpenCode / Cursor / VS Code Copilot**（`.agents/skills/`）、**ZCode**（`.zcode/skills/`）。
- **④ 仅文件存在**：上述投放目录中的文件已落盘，宿主运行时尚未消费。
- **⑤ 未验证**：所有宿主的运行时「技能发现」与「实际调用」；**WorkBuddy** 全流程（不在 installer agent 列表，需手动投放至 `{workspace}/.workbuddy-ai/skills/`，且历史上存在工作区 scope 未解析导致技能不被注册的问题）。
- **`.agents/skills/` 是共享目录**：Codex / VS Code Copilot / OpenCode / Cursor 在 `skills@1.7.1` 中均投放于此；只有 Claude Code（`.claude/skills/`）与 ZCode（`.zcode/skills/`）使用独立目录。
- ⚠️ **③ 只代表「安装器写了文件」，不等于「宿主读到了」（④）或「跑通了」（⑤）**。⑤ 在本轮**一律为未验证**。

## 5. 建议的回退机制

当宿主缺少某种能力时，按下表回退。**回退不得违反 invocation 语义。**

| 缺失能力               | 影响技能                                                                     | 建议回退                                                               | 约束                                                      |
| ------------------ | ------------------------------------------------------------------------ | ------------------------------------------------------------------ | ------------------------------------------------------- |
| 无 Skill 工具         | `grill-with-docs`、`implement`、`tdd`、`retro`                              | 若目标为 **model-invoked**：直接读取该技能的 `SKILL.md` 并按指令执行（L4）；或由人显式点名      | **禁止**对 user-invoked 目标做 L4 回退                          |
| 无斜杠命令              | 全部 7 个 user-invoked                                                      | 由人在对话中**显式点名**技能（如「用 `implement` 技能」），agent 再读取其 `SKILL.md` 执行     | 必须由人发起，agent 不得自行触发                                     |
| 无 sub-agent        | `code-review`（双轴）、`codebase-design`（DESIGN-IT-TWICE）、`grilling`（fact 查找） | 退化为**串行 / inline** 执行：先完成 Standards 轴，再完成 Spec 轴，最后聚合；两个轴线的报告仍分开呈现 | 回填 #2 要求「前台并行」，无并行能力时降级为串行，但**不得合并两个轴线**                |
| 无后台任务              | `grilling`（fact 查找不阻塞）                                                   | 改为同步查找，或直接向用户说明无法并行                                                | 不阻塞其余 frontier 问题                                       |
| 无 issue tracker 集成 | `to-spec`、`to-tickets`、`code-review`                                     | 使用 local-file 形态（`.scratch/` 或 `docs/` 下的 markdown）                | 见 `setup-matt-pocock-skills` 的 `issue-tracker-local.md` |
| 宿主不注册项目级技能         | WorkBuddy 等                                                              | 以**该仓库为工作区**重开会话；或手动将技能文件放入宿主约定的项目级目录                              | 不得写入用户级目录（除非用户明确要求）                                     |

## 6. 技能源定位方法

供窗口 B（Mgtv-Pet-SDK）与其他消费者引用：

| 项                 | 值                                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------------- |
| 仓库                | `https://github.com/Vin535LFJ/mattpocock-skills-zh-CN-custom`（公开）                                             |
| 默认分支              | `main`                                                                                                        |
| engineering 技能路径  | `skills/engineering/<name>/SKILL.md`                                                                          |
| productivity 技能路径 | `skills/productivity/<name>/SKILL.md`                                                                         |
| 安装（多 agent）       | `npx skills@latest add Vin535LFJ/mattpocock-skills-zh-CN-custom -s <空格分隔的技能名> -a <agent...> -y --copy`        |
| 列表校验              | `npx skills@latest add Vin535LFJ/mattpocock-skills-zh-CN-custom --list` → 期望 `Found 14 skills`                |
| 消费者锁文件            | 安装后生成 `skills-lock.json`（记录 `source` / `sourceType` / `skillPath` / `computedHash`），**它是消费者侧的安装锁，不是技能源的发布清单** |
| 技能源自身的发布索引        | `.claude-plugin/plugin.json` 的 `skills` 数组（14 条）                                                              |

### 6.1 安装器实测细节（`skills@1.7.1`）

- **`-s` / `--skill` 是变长参数，必须空格分隔**：`-s a b c`。**逗号会被当成一个名字** ⇒ `No matching skills found`。
- 安装须**显式**指定 `-a <agent>` + `--copy` + `-y`：`--copy` 避免 Windows 符号链接问题，`-y` 跳过确认。
- **Node 名义要求 vs 实测**：`skills@1.7.1` 的 `engines.node` = `>=22.20.0`；本机实测运行于 **v22.22.2**。
- **先 `--list` 断言**：`npx skills@latest add Vin535LFJ/mattpocock-skills-zh-CN-custom --list` → 期望**恰为 14**。

### 6.2 窗口 B 的可安装时机（技能包 vs 文档，分开陈述）

- **技能包**：GitHub `main` 当前发布的是 `186e725`（**14 技能** + 4 处回填）；**全量 37 技能的版本尚未推送**。从 GitHub 安装当前只能拿到 14 个技能，全量版本需在推送后才可用。
- **文档**：`README.md` / `UPSTREAM.md` / `AGENTS.md` 的修正与 `AGENT-COMPATIBILITY.md` 属**未发布的本地改动**（在 `186e725` 中不存在）。
- 因此：**安装以已发布的技能包为准**；**可见性与凭据说明不要采信 `186e725` 里的旧文档**（其仍写「私有仓库」并含内联 token 的推送建议）。

## 7. 四处回填对兼容性的影响

| # | 技能                | 回填内容                                                      | 对跨 Agent 兼容性的影响                                                                |
| - | ----------------- | --------------------------------------------------------- | ------------------------------------------------------------------------------ |
| 1 | `implement`       | `/tdd`、`/code-review` 斜杠式 → 通用「Skill 工具」调用；ticket 标题前置    | ✅ **正面且关键**：斜杠命令在 Codex / ZCode / WorkBuddy 等宿主不成立；改为 Skill 工具调用后，这些宿主才有可执行的语义 |
| 2 | `code-review`     | standards 来源改为「搜索全部 standards 文件」；两个 sub-agent **前台并行**发出 | ⚠️ **依赖宿主 sub-agent 能力**；无并行能力的宿主需按 §5 降级为串行，但**两轴报告仍须分开**                     |
| 3 | `diagnosing-bugs` | 强制造红时须 `diff` 未改动副本自证                                     | ✅ 工具无关：只需 `diff` 与文件副本，任何宿主都可执行                                                |
| 4 | `grilling`        | 问题措辞须让「是」等于接受推荐答案                                         | ✅ 工具无关：纯措辞约束                                                                   |

> 结论：回填 **#1 提升**跨工具兼容性；**#2 引入**对 sub-agent 能力的依赖（有回退）；**#3 / #4 中立**。

## 8. 风险

| 风险              | 说明           | 缓解                                                   |
| --------------- | ------------ | ---------------------------------------------------- |
| R1 宿主无 Skill 工具 | 4 条硬依赖无法直接执行 | 按 §5 对 model-invoked 目标做 L4 文件回退；user-invoked 目标由人触发 |


| R2 sub-agent 能力不一 | `code-review` 的双轴并行、`codebase-design` 的 DESIGN-IT-TWICE、`grilling` 的并行 fact 查找在部分宿主不可用 | 串行降级，保持两轴分离 |  
| R3 invocation 语义被绕过 | 用文件读取「替人触发」user-invoked 技能 | 明确红线（§2 L4）；不在技能文件中新增任何绕过机制 |  
| R4 WorkBuddy 技能注册 | 项目级技能可能不被注册（工作区 scope 问题） | 以该仓库为工作区重开会话；或手动投放 |  
| R5 文档与回填措辞不一致 | `docs/invocation.md` 说依赖用「`/skill` 风格 prose」，而回填 #1 及上游新版本用「调用 Skill 工具」 | 以技能文件为准；同步上游时统一措辞（**本轮不改**） |  
| R6 仅安装层验证 | ④「仅文件存在」与 ⑤「未验证」层未测，易被误认为「已可用」 | 本文档与报告明确标注未验证项 |  
| R7 上游同步覆盖回填 | 刷新上游会覆盖 4 处回填 | 按 [BACKPORTS.md](./BACKPORTS.md) 重新应用并 `grep -rl "本地回填" skills/` 校验 |

## 9. 验收案例

供窗口 B 或后续回归使用（**标注了每项的可验证层级**）：

| #   | 案例                                                                                                                                     | 期望                                                               | 层级           |
| --- | -------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ------------ |
| A1  | `npx skills@latest add Vin535LFJ/mattpocock-skills-zh-CN-custom --list`                                                                | 输出 `Found 14 skills`，且不含 `translate-skill`                       | ✅ 已测         |
| A2  | `find skills -name SKILL.md \| wc -l`                                                                                                  | `14`                                                             | ✅ 已测         |
| A3  | `grep -rl "本地回填" skills/ \| wc -l`                                                                                                     | `4`（implement / code-review / diagnosing-bugs / grilling）        | ✅ 已测         |
| A4  | 安装到 `-a codex -a claude-code`                                                                                                          | 两目录各 37 个技能，且与源逐字节一致                                             | ✅ 已测         |
| A5  | 安装后每个技能含 `agents/openai.yaml`；7 个 user-invoked 的 `SKILL.md` 含 `disable-model-invocation: true`                                         | 一致                                                               | ✅ 已测         |
| A6  | 在 Claude Code 中调用 `implement`，观察它是否按回填 #1 调用 `tdd` 与 `code-review`                                                                     | 命中                                                               | ⚠️ 未测        |
| A7  | 在 Codex 中确认 user-invoked 技能不会被模型自动触发                                                                                                   | 不自动触发                                                            | ⚠️ 未测        |
| A8  | 在无 sub-agent 的宿主中运行 `code-review`                                                                                                      | 串行降级，两轴报告分开                                                      | ⚠️ 未测        |
| A9  | WorkBuddy 以本仓库为工作区打开，技能被注册                                                                                                             | 可发现                                                              | ⚠️ 未测（历史曾失败） |
| A10 | **无凭据断言**：`git grep -nE "ghp_\|gho_\|github_pat_\|Authorization: (Basic\|Bearer)\|x-access-token" -- .`，并检查 `.git/config` 的 remote URL | 跟踪文件 **0 命中**；`.git/config` 不含任何凭据                               | ✅ 已测         |
| A11 | **远端 HEAD 比对**：`git rev-parse HEAD` vs GitHub `commits/main`                                                                           | 二者相同（`186e725a555de908e39902a84525c6be34be7786`）；注意工作树另有未推送的文档改动 | ✅ 已测         |

## 10. 上游同步方式

沿用 [SYNC.md](./SYNC.md) 的流程，针对本兼容方案特别注意：

1. 刷新上游后，**保持全量技能集（37 个）**，并**逐项重新应用 4 处回填**（`grep -rl "本地回填" skills/` 须为 4）。
2. 若上游把「调用 Skill 工具」的措辞进一步演进（例如统一 `docs/invocation.md`），需评估是否同步更新依赖图（§3）与回退机制（§5）。
3. 上游新增/删除本仓库技能的附属文件时，同步更新，保持相对路径不变。
4. 每次同步后重跑 §9 的 A1–A5（含 A10 / A11）。

## 11. 本轮验证边界（诚实声明）

- ✅ **已验证**：安装器识别技能源（37 个）、文件投放（Claude Code / Codex / ZCode / OpenCode / Cursor / VS Code Copilot）、安装内容与源逐字节一致、四处回填存在、frontmatter 元数据与 invocation 标志正确。
- ⚠️ **未验证**：任何宿主的**运行时技能发现**与**实际技能调用**（本机无可运行的对应 agent 运行时）。
- 因此：**不得**把「安装成功」等同于「Agent 可发现」，也**不得**把「Agent 可发现」等同于「Agent 实际调用成功」。
