# pm-feedback-analysis

[English](#english) | [中文](#中文)

---

<a id="english"></a>

A [Claude Code](https://claude.ai/code) skill that crawls Discord, Reddit, and GitHub to extract user pain points, generate structured reports with verbatim user quotes, and produce phased product roadmaps.

Built for PMs who want to turn community noise into actionable product decisions.

## What it does

```
/pm-feedback-analysis
```

End-to-end workflow:

1. **Crawl** — Discord (via DiscordChatExporter), Reddit (JSON API), GitHub (gh CLI + grey-market tool detection)
2. **Analyze** — Extract pain points with exact user quotes, cross-platform validation, root cause analysis
3. **Report** — Three-tier output: macro analysis, detailed quotes, task completion deep-dive
4. **Roadmap** — 4-phase prioritized roadmap tied to specific user feedback
5. **Compare** — Optional competitive analysis when multiple products are analyzed
6. **Present** — Optional HTML presentation generation via `/frontend-slides`

## Key features

- **Multi-platform**: Discord + Reddit + GitHub in one workflow
- **Verbatim quotes**: Every insight traceable to a real user's exact words
- **Cross-platform validation**: Same issue on 3 platforms = highest confidence
- **Task completion focus**: Digs past "it's expensive" to find "I paid but the task didn't finish"
- **Security insights**: Detects grey-market tools (account managers, machine code resetters) that reveal anti-abuse weaknesses
- **Multi-language**: Preserves original language (Chinese, Portuguese, etc.) — doesn't translate away context
- **Large community support**: Auto-sampling and preprocessing for 10K+ message servers

## Prerequisites

| Tool | Required for | Install |
|------|-------------|---------|
| [DiscordChatExporter](https://github.com/Tyrrrz/DiscordChatExporter) | Discord data | Download from releases |
| Discord Token | Discord access | Browser DevTools → Network → Authorization header |
| Python 3 | Data preprocessing | Usually pre-installed |
| gh CLI | GitHub data | `brew install gh` |

Reddit requires no authentication.

## Install

Copy the skill to your Claude Code skills directory:

```bash
# Clone
git clone https://github.com/yyilin66-cmyk/pm-feedback-analysis.git

# Copy to Claude Code skills
cp -r pm-feedback-analysis ~/.claude/skills/pm-feedback-analysis
```

Or manually copy `SKILL.md` to `~/.claude/skills/pm-feedback-analysis/SKILL.md`.

Then use it in Claude Code:

```
/pm-feedback-analysis
```

## Example output

When analyzing a product's Discord (900+ members), Reddit (165 posts), and GitHub:

| Report | Content |
|--------|---------|
| `{product}_analysis.md` | Pain point ranking, competitor landscape, key insights |
| `{product}_detailed_quotes.md` | 50+ verbatim user quotes organized by category |
| `{product}_task_completion.md` | Root cause analysis of "paid but task didn't finish" |
| `{product}_roadmap.md` | 4-phase roadmap with priorities tied to feedback |
| `{A}_vs_{B}_comparison.md` | Competitive analysis when multiple products analyzed |

## How it works

### Discord
Uses [DiscordChatExporter](https://github.com/Tyrrrz/DiscordChatExporter) CLI to export server channels as JSON. Parallel agents analyze different channels simultaneously.

### Reddit
Uses Reddit's public JSON API (append `.json` to any URL). No authentication needed. Fetches posts + comments for top discussions.

### GitHub
Two dimensions:
- **Official repo issues** via `gh` CLI — structured bug reports and feature requests
- **Third-party tool ecosystem** — searches for grey-market tools (`{product}-account-manager`, `{product}-reset-machine-code`) that reveal security weaknesses

## Design principles

1. **Pain-point driven** — Every roadmap item traces back to a real user quote
2. **Fix first, build later** — Phase 1 is always fixing existing issues, not adding features
3. **Task completion > pricing** — "The task didn't finish" matters more than "it's expensive"
4. **Cross-platform validation** — Discord + Reddit + GitHub all pointing to the same issue = ship it now
5. **Frequency-weighted** — 10 users mentioning a P1 > 1 user mentioning a P0

## License

MIT

---

<a id="中文"></a>

# pm-feedback-analysis (中文说明)

一个 [Claude Code](https://claude.ai/code) 技能，自动爬取 Discord、Reddit、GitHub 三平台用户反馈，提取痛点并生成带用户原话的结构化分析报告和分阶段产品 Roadmap。

**为 PM 打造**：把社区噪声变成可执行的产品决策。

## 它能做什么

```
/pm-feedback-analysis
```

端到端工作流：

1. **爬取** — Discord（DiscordChatExporter）、Reddit（JSON API，无需认证）、GitHub（gh CLI + 灰产工具检测）
2. **分析** — 提取用户痛点，保留原话，跨平台交叉验证，根因分析
3. **报告** — 三层输出：宏观分析、用户原话详细报告、任务完成度深度报告
4. **Roadmap** — 4 阶段优先级路线图，每项可追溯到具体用户反馈
5. **竞品对比** — 分析多个产品时自动生成对比报告
6. **演示** — 可选调用 `/frontend-slides` 生成浏览器内 Presentation

## 核心特性

- **三平台覆盖**：Discord + Reddit + GitHub 一个工作流搞定
- **用户原话逐字引用**：每个洞察都可追溯到真实用户的原话
- **跨平台交叉验证**：同一问题在 3 个平台都出现 = 最高置信度
- **聚焦任务完成度**：深挖"太贵"背后的真正问题——"花了钱活没干完"
- **安全洞察**：检测灰产工具（多账号管理器、机器码重置器），发现反作弊弱点
- **多语言支持**：保留中文、葡语等原始语言，不翻译丢失上下文
- **大社区适配**：10K+ 消息的 server 自动采样和预处理

## 前置要求

| 工具 | 用途 | 安装方式 |
|------|------|---------|
| [DiscordChatExporter](https://github.com/Tyrrrz/DiscordChatExporter) | 导出 Discord 数据 | 从 releases 下载 |
| Discord Token | 访问 Discord | 浏览器开发者工具 → Network → Authorization |
| Python 3 | 数据预处理 | 通常已预装 |
| gh CLI | GitHub 数据 | `brew install gh` |

Reddit 无需认证。

## 安装

```bash
# 克隆
git clone https://github.com/yyilin66-cmyk/pm-feedback-analysis.git

# 复制到 Claude Code skills 目录
cp -r pm-feedback-analysis ~/.claude/skills/pm-feedback-analysis
```

或手动复制 `SKILL.md` 到 `~/.claude/skills/pm-feedback-analysis/SKILL.md`。

然后在 Claude Code 中使用：

```
/pm-feedback-analysis
```

## 输出示例

分析某产品的 Discord（900+ 成员）、Reddit（165 帖）和 GitHub 后：

| 报告 | 内容 |
|------|------|
| `{product}_analysis.md` | 痛点排名、竞品格局、关键发现 |
| `{product}_detailed_quotes.md` | 50+ 条用户原话，按痛点分类 |
| `{product}_task_completion.md` | "花了钱活没干完"的根因分析 |
| `{product}_roadmap.md` | 4 阶段 Roadmap，每项追溯到具体反馈 |
| `{A}_vs_{B}_comparison.md` | 多产品竞品对比分析 |

## 工作原理

### Discord
使用 [DiscordChatExporter](https://github.com/Tyrrrz/DiscordChatExporter) CLI 将频道导出为 JSON。多个 Agent 并行分析不同频道。

### Reddit
使用 Reddit 公开 JSON API（任何 URL 后加 `.json`）。无需认证。自动拉取帖子和热门帖的评论。

### GitHub
两个维度：
- **官方 Repo Issues**：通过 `gh` CLI 获取结构化 bug 报告和功能请求
- **第三方灰产工具生态**：搜索 `{product}-account-manager`、`{product}-reset-machine-code` 等工具，发现安全弱点

## 设计原则

1. **痛点驱动** — 每个 Roadmap 条目可追溯到真实用户原话
2. **先修后建** — Phase 1 永远是修复现有问题，不是加新功能
3. **完成度 > 价格** — "活没干完"比"太贵"更重要
4. **跨平台验证** — 三平台都指向同一问题 = 立即修
5. **频次加权** — 10 人提到的 P1 > 1 人提到的 P0

## License

MIT
