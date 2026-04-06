# pm-feedback-analysis

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
git clone https://github.com/YOUR_USERNAME/pm-feedback-analysis.git

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
