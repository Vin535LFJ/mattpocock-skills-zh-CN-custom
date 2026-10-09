# UPSTREAM.md — 上游来源与同步方式

## 上游仓库

| 角色 | 仓库 | 说明 |
|---|---|---|
| 英文上游 | [`mattpocock/skills`](https://github.com/mattpocock/skills) | Matt Pocock 原始技能仓库（英文） |
| 中文版上游 | [`vinvcn/mattpocock-skills-zh-CN`](https://github.com/vinvcn/mattpocock-skills-zh-CN) | 简体中文本地化版，本仓库的**直接基线** |
| 本仓库 | 本目录（`mattpocock-skills-zh-CN-custom`） | 中文版**全量发行副本**（37 个技能）+ **4 处**回填 |

本仓库**以已经验证的中文版为基础**建立：克隆中文版上游（保留其 Git 历史），保留其**全部技能**，并额外应用 4 处已对照英文上游验证的修复。**未**从任何消费方安装镜像反向拼装源码。

## 基线提交（SHA）

| 项 | SHA / 版本 | 日期 |
|---|---|---|
| 英文上游 HEAD（建立本仓库时） | `b0618bc436ad893b3c5e84e55fba86586d34a404` | 2026-10-08 |
| 英文上游 `v1.3.1` tag（peeled） | `24fe0ef7737efae15c87225755e9f6f5965e4888` | 2026-10-08 |
| 中文版上游同步基线 | `mattpocock/skills@24fe0ef`（v1.3.1） | 2026-10-08 |
| 中文版上游 HEAD = 本仓库基线 | `bf98e53f92089fec9b4885f128a565d7eac0337f` | 2026-10-08 |

> 四处回填依据的上游提交区间为 `24fe0ef..b0618bc`，逐项对照见 [BACKPORTS.md](./BACKPORTS.md)。

## 远程（remote）配置

本仓库沿用中文版上游的 Git 历史，remote 规划如下：

| remote | 地址 | 用途 |
|---|---|---|
| `upstream-zh` | `https://github.com/vinvcn/mattpocock-skills-zh-CN.git` | 可追溯的中文版上游（比较中文版变化、拉取翻译刷新） |
| `upstream-en` | `https://github.com/mattpocock/skills.git` | 英文上游（重新推导回填差异） |
| `origin` | `https://github.com/Vin535LFJ/mattpocock-skills-zh-CN-custom.git` | 本仓库自己的**公开**远程仓库，用于发布与日常推送 |

> `origin` 已创建为 **GitHub 公开仓库**（账号 `Vin535LFJ`），默认分支为 `main`，当前可见性为 `public`。
>
> **推送认证（推荐做法）**：优先使用 SSH 或凭据助手，**不要**在命令行内联 token。
>
> - 首选：配置 GitHub SSH key，并使用 `git@github.com:Vin535LFJ/mattpocock-skills-zh-CN-custom.git`；
> - 或使用 `gh auth login`，由 `gh` 托管凭据；
> - 或让 `git-credential-manager` 在首次推送时交互式保存 HTTPS 凭据。
>
> ⚠️ **不要**用 `-c http.extraHeader="Authorization: ..."` 或把 token 拼进 remote URL 的方式推送：token 会进入 shell 历史、进程列表或 `.git/config`，可能泄露。本仓库的 `.git/config` 与全部跟踪文件均不含任何凭据。
>
> ```bash
> git push origin main
> ```
>
> 注：更早的提交 `186e725`（信息为「docs: 记录 origin **私有**远程仓库地址与安装命令」）在**提交信息**中沿用了创建时的「私有」措辞；仓库此后已改为 **public**。**以本节为准**，提交信息的历史措辞不再代表当前可见性。

## 同步方式

本仓库采用 **skill-guided content localization**（与中文版上游一致），不做 Git fork-sync：

1. 以英文上游 `mattpocock/skills` 为**内容来源**，以中文版上游 `vinvcn/mattpocock-skills-zh-CN` 为**翻译来源**。
2. 只翻译自然语言说明；**保留**目录名、skill name、frontmatter key、命令、代码块、路径、URL、package/tool/API identifiers 与行为关键 labels。
3. 刷新前先读 [`.skills/translate-skill/SKILL.md`](./.skills/translate-skill/SKILL.md)；术语以 [TRANSLATION-GLOSSARY.md](./TRANSLATION-GLOSSARY.md) 为准。
4. 刷新后**保持全量技能集（37 个）**，并对照 [BACKPORTS.md](./BACKPORTS.md) 重新应用 4 处回填。

详细操作流程见 [SYNC.md](./SYNC.md)。

## 许可证与署名

- 许可证：**MIT**，见 [LICENSE](./LICENSE)（英文原文）与 [LICENSE.zh-CN.md](./LICENSE.zh-CN.md)（中文参考译文）。
- 原始作者：**Matt Pocock**（<https://www.aihero.dev>），英文原仓库 `mattpocock/skills`。
- 简体中文本地化：`vinvcn/mattpocock-skills-zh-CN`。
- 本仓库为上述两者的**衍生全量发行副本**（37 个技能 + 4 处回填），保留原始许可证与署名。

**分发方式**：MIT 允许再分发与修改，但要求保留版权声明与许可证文本。本仓库现为**公开仓库**（`Vin535LFJ/mattpocock-skills-zh-CN-custom`），已保留 `LICENSE` / `LICENSE.zh-CN.md` 与署名，符合 MIT 的分发要求。发布前请再确认目标平台的许可与合规要求。

## 上游与 Mgtv-Pet-SDK 的关系

本仓库与 `Mgtv-Pet-SDK` **无关**：不从其三个安装镜像反向拼装源码，也不包含其 `AGENTS.md`、`bugList`、`docs/kb/`、架构候选报告、`MEMORY.md` 或 `mgtv-pet-sdk-*` 项目技能。Mgtv-Pet-SDK 仅作为**回填已通过验证**的旁证来源（用于交叉核对，不作为源码来源）。
