# UPSTREAM.md — 上游来源与同步方式

## 上游仓库

| 角色 | 仓库 | 说明 |
|---|---|---|
| 英文上游 | [`mattpocock/skills`](https://github.com/mattpocock/skills) | Matt Pocock 原始技能仓库（英文） |
| 中文版上游 | [`vinvcn/mattpocock-skills-zh-CN`](https://github.com/vinvcn/mattpocock-skills-zh-CN) | 简体中文本地化版，本仓库的**直接基线** |
| 本仓库 | 本目录（`mattpocock-skills-zh-CN-custom`） | 精选 14 个技能 + 4 处回填的独立定制发布集合 |

本仓库**以已经验证的中文版为基础**建立：克隆中文版上游（保留其 Git 历史），在其上做「精选子集 + 四处回填」。**未**从任何消费方安装镜像反向拼装源码，也**未**重新复制上游全部技能。

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
| `origin` | **待配置**（本仓库自己的远程仓库） | 发布与日常推送 |

> ⚠️ **`origin` 地址尚未确定。** 本任务未猜测远程地址；待具备明确授权的命名空间后，再创建私有仓库并 `git remote add origin <url>`。具体所需信息见仓库根目录的交付报告。
>
> 若 `origin` 已配置，请在此处补记实际地址与可见性（私有/公开）。

## 同步方式

本仓库采用 **skill-guided content localization**（与中文版上游一致），不做 Git fork-sync：

1. 以英文上游 `mattpocock/skills` 为**内容来源**，以中文版上游 `vinvcn/mattpocock-skills-zh-CN` 为**翻译来源**。
2. 只翻译自然语言说明；**保留**目录名、skill name、frontmatter key、命令、代码块、路径、URL、package/tool/API identifiers 与行为关键 labels。
3. 刷新前先读 [`.skills/translate-skill/SKILL.md`](./.skills/translate-skill/SKILL.md)；术语以 [TRANSLATION-GLOSSARY.md](./TRANSLATION-GLOSSARY.md) 为准。
4. 刷新后**只保留本集合的 14 个技能**，并对照 [BACKPORTS.md](./BACKPORTS.md) 重新应用 4 处回填。

详细操作流程见 [SYNC.md](./SYNC.md)。

## 许可证与署名

- 许可证：**MIT**，见 [LICENSE](./LICENSE)（英文原文）与 [LICENSE.zh-CN.md](./LICENSE.zh-CN.md)（中文参考译文）。
- 原始作者：**Matt Pocock**（<https://www.aihero.dev>），英文原仓库 `mattpocock/skills`。
- 简体中文本地化：`vinvcn/mattpocock-skills-zh-CN`。
- 本仓库为上述两者的**衍生精选定制集合**，保留原始许可证与署名。

**分发方式**：MIT 允许再分发与修改，但要求保留版权声明与许可证文本。因此本仓库在**保留 `LICENSE` / `LICENSE.zh-CN.md` 与署名**的前提下可公开；若发布为私有仓库，同样建议保留这两份文件。发布前请再确认目标平台的许可与合规要求。

## 上游与 Mgtv-Pet-SDK 的关系

本仓库与 `Mgtv-Pet-SDK` **无关**：不从其三个安装镜像反向拼装源码，也不包含其 `AGENTS.md`、`bugList`、`docs/kb/`、架构候选报告、`MEMORY.md` 或 `mgtv-pet-sdk-*` 项目技能。Mgtv-Pet-SDK 仅作为**回填已通过验证**的旁证来源（用于交叉核对，不作为源码来源）。
