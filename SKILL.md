---
name: pm-feedback-analysis
description: "Multi-platform user feedback analysis for PMs. Crawl and analyze Discord, Reddit, and GitHub data to extract user pain points, generate structured reports with exact user quotes, and produce phased product roadmaps. Supports competitive analysis across products. Requires: Discord token + DiscordChatExporter CLI (for Discord); no auth needed for Reddit/GitHub."
---

# PM 多平台用户反馈分析 → 产品 Roadmap

从 Discord / Reddit / GitHub 三平台采集用户反馈，提取痛点，生成带用户原话的结构化分析报告和分阶段产品 Roadmap。面向 PM 的端到端工作流。

## 平台覆盖

| 平台 | 数据价值 | 抓取难度 | 需要认证 |
|------|---------|---------|---------|
| **Discord** | 极高（深度讨论、实时反馈） | 中 | 需要 User/Bot Token |
| **Reddit** | 高（长文讨论、竞品对比） | 低 | 不需要 |
| **GitHub** | 高（结构化 bug/feature request）+ 安全洞察 | 极低 | 不需要（公开 API） |
| **Twitter/X** | 中（碎片化、噪声多） | 极高（API 收费、反爬） | 需要付费工具 |

## Prerequisites

- **DiscordChatExporter CLI**（Discord 数据导出）
  - Mac ARM: `curl -L "https://github.com/Tyrrrz/DiscordChatExporter/releases/latest/download/DiscordChatExporter.Cli.osx-arm64.zip"`
  - 解压后 `chmod +x DiscordChatExporter.Cli`
- **Discord Token**（User token 或 Bot token）— 获取方式：浏览器 DevTools → Network → 找 Authorization header
- **Python 3**（数据预处理）
- **gh CLI**（GitHub 数据，通常已安装）
- 用户需提供: Discord Token + 目标 Server 名称 + Reddit Subreddit URL（可选）+ GitHub 搜索关键词（可选）

## Workflow（7 个阶段）

---

### Phase 0: 确认分析范围

向用户确认：
1. **目标产品名称**
2. **要分析的平台**（Discord / Reddit / GitHub / 全部）
3. **Discord Token**（如分析 Discord）
4. **Reddit Subreddit URL**（如分析 Reddit）
5. **GitHub 搜索关键词**（如分析 GitHub — 默认用产品名搜索）
6. **分析目的**（竞品分析 / 自家产品改进 / 投资调研）
7. **时间范围**（默认全量）

---

### Phase 1: 数据采集

#### 1a. Discord 数据采集

1. 验证 token：`DiscordChatExporter.Cli guilds -t "TOKEN"`
2. 列出频道：`DiscordChatExporter.Cli channels -t "TOKEN" -g GUILD_ID`
3. 识别高价值频道（按优先级）：
   - **必导**: support、help、bug-report、feedback、feature-request
   - **重要**: general、announcement、changelog、getting-started
   - **补充**: 多语言频道（chinese、portuguese 等）、showcase
4. 导出整个 server 为 JSON：
   ```bash
   DiscordChatExporter.Cli exportguild -t "TOKEN" -g GUILD_ID -f Json -o ./discord_data/
   ```
5. Forum 频道无法直接导出，会报错提示 — 记录但不阻塞
6. 检查导出结果：`ls -lh ./discord_data/`，排除空文件和无权限频道

**大社区模式**（>10K 消息）：加 `--after "YYYY-MM-DD"` 限制时间范围

**注意**：User Token 技术上违反 Discord TOS，有小概率封号风险。用完建议改密码使旧 token 失效。

#### 1b. Reddit 数据采集

无需认证，使用 Reddit 的 JSON API（任意 URL 后加 `.json`）：

```python
import json, urllib.request, time

all_posts = []
after = None
headers = {"User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)"}

for page in range(10):  # max ~1000 posts
    url = f"https://www.reddit.com/r/{SUBREDDIT}/new.json?limit=100"
    if after:
        url += f"&after={after}"
    req = urllib.request.Request(url, headers=headers)
    resp = urllib.request.urlopen(req)
    data = json.loads(resp.read())
    posts = data.get('data',{}).get('children',[])
    if not posts: break
    all_posts.extend(posts)
    after = data.get('data',{}).get('after')
    if not after: break
    time.sleep(2)  # 遵守速率限制

# 保存帖子
with open('reddit_posts.json', 'w') as f:
    json.dump(all_posts, f, ensure_ascii=False)

# 拉取热门帖的评论
top_posts = sorted(all_posts, key=lambda p: p['data'].get('num_comments',0), reverse=True)[:30]
all_comments = {}
for p in top_posts:
    permalink = p['data']['permalink']
    url = f"https://www.reddit.com{permalink}.json?limit=500"
    try:
        req = urllib.request.Request(url, headers=headers)
        resp = urllib.request.urlopen(req)
        all_comments[p['data']['id']] = json.loads(resp.read())
        time.sleep(2)
    except: pass

with open('reddit_comments.json', 'w') as f:
    json.dump(all_comments, f, ensure_ascii=False)
```

**关键注意**：Reddit JSON 导出为单行大 JSON。文件超过 256KB 时 Read 工具打不开，必须用 Bash + Python 解析后提取结构化数据。

#### 1c. GitHub 数据采集

两个维度：

**官方 Repo Issues**（如果有官方 repo）：
```bash
gh issue list -R owner/repo --state all --limit 1000 --json title,body,comments,labels,createdAt
```

**第三方灰产工具生态**（安全洞察 — 在 GitHub 搜索产品名）：
寻找以下模式的 repo：
- `{product}-account-manager` — 多账号管理
- `{product}-reset-machine-code` — 机器码/设备ID重置
- `{product}-token-mng` — Token 管理
- `{product}2api` — 将 IDE 能力转为 API 的逆向代理

这些工具的**存在本身**就是重要信号。star 数反映问题严重程度。用 WebFetch/WebSearch 获取 repo README。

**重要**：GitHub 搜索结果可能包含已删除/改名的 repo。**务必逐一用 WebFetch 验证 repo 是否真实存在**，不要对不存在的 repo 编造分析。

---

### Phase 1.5: 数据预处理（大社区模式）

当单频道消息量 > 500 条时，生成 Python 脚本 `preprocess.py` 预处理：

```python
# 输入: 原始 JSON 导出
# 输出: filtered_{channel}.json + stats_{channel}.json + highlights_{channel}.json

# Step 1: 噪声过滤
# - 删除纯 emoji / bot / 系统消息
# - 删除低于 10 字符的打招呼消息

# Step 2: 统计摘要
# - 总消息数 / 过滤后 / 活跃用户 top 20 / 按月趋势 / 语言分布

# Step 3: 高价值消息标记
# - question / bug_report / feature_request / high_engagement(3+ reactions) / discussion_starter(2+ replies)
```

分析时**优先读 highlights 文件**，需要上下文时回溯 filtered 文件。

---

### Phase 2: 数据分析

**并行启动多个 Agent 分析不同数据源/频道**，最大化效率。

#### 分析维度

##### 2a. 痛点提取（核心）

**聚焦任务完成度而非表层抱怨**。用户说"太贵"通常意味着"花了钱活没干完"。深挖表面投诉背后的根因。

对每个痛点记录：
- **用户原话**（逐字摘录，保留原文语言，中英文都保留）
- **用户名 + 日期 + 来源**（频道/帖子/repo）
- **问题根因**（技术/产品层面，不是用户描述的现象）
- **影响范围**（个例 vs 系统性）
- **严重程度**: P0 / P1 / P2
- **跨平台验证**（同一问题在 Discord + Reddit + GitHub 都出现 = 高置信度）

##### 2b. 竞品洞察
- 用户从哪个竞品来 / 要去哪个竞品
- 具体功能对比评价（原话）
- 用户流动趋势（是净流入还是净流出）

##### 2c. 安全/反滥用洞察（GitHub 特有）
- 薅羊毛工具功能、star 数
- 暴露的反作弊弱点
- 同类产品灰产生态成熟度对比

##### 2d. 社区健康度
- 官方 vs 用户发帖比例
- Reddit 中官方营销帖 vs 真实用户讨论比例
- Discord 新用户试用→付费转化断裂点
- 语言分布（市场信号）

---

### Phase 3: 输出分析报告

生成 **三份递进报告**：

#### 3a. 宏观分析报告 `{product}_analysis.md`
```markdown
# {Product} 多平台用户反馈分析
## 数据来源概览（平台 | 数据量 | 时间跨度）
## 产品概况
## 用户痛点排名（P0/P1/P2 表格）
## 竞品格局
## 安全/反滥用洞察
## 关键发现总结（5 条以内）
```

#### 3b. 用户原话详细报告 `{product}_detailed_quotes.md`
```markdown
# 按痛点分类的用户原话引用
## 每个痛点类别：5-10 条原话
## 每条包含：用户名、原文（逐字）、日期、来源、上下文
```

#### 3c. 任务完成度深度报告 `{product}_task_completion.md`
```markdown
# 聚焦"花了钱活干没干完"
## 任务半途而废案例（含原话）
## Agent 行为异常案例
## 上下文丢失案例
## 模型间完成度对比
## 根因分析表
```

---

### Phase 4: 输出产品 Roadmap

生成 `{product}_roadmap.md`，4 阶段递进：

```markdown
# Phase 1: 止血（0-4 周）— 修复信任与转化
# Phase 2: 增长（4-8 周）— 拓宽用户群
# Phase 3: 深化（8-16 周）— 巩固核心优势
# Phase 4: 扩展（16+ 周）— 构建生态壁垒

每个 Phase：优先级 | 任务 | 痛点来源 | 预期效果
附：关键指标表 + 竞争策略建议
```

---

### Phase 5: 竞品对比报告（可选）

当分析了多个产品时，生成 `{A}_vs_{B}_comparison.md`：

```markdown
## 产品定位对比
## 共同痛点（两家都有的问题 + 双方用户原话对比）
## A 独有优势（B 没有的）
## B 独有优势（A 没有的）
## 对同一问题的不同处理方式（表格）
## 用户群体差异
## 战略启示（如果做竞品该学什么）
```

---

### Phase 6: Presentation 生成（可选）

PM 需要展示时，调用 `/frontend-slides` skill 将报告转为浏览器内 Presentation。建议 25+ 页详细版，包含用户原话引用。

---

## Roadmap 设计原则

1. **痛点驱动** — 每个条目可追溯到具体用户反馈原话
2. **先稳后快** — Phase 1 永远是修复，不是加新功能
3. **任务完成度优先** — "活干没干完"比"价格贵不贵"更重要
4. **跨平台交叉验证** — Discord + Reddit + GitHub 都指向同一问题 = 最高置信度
5. **频次加权** — 被 10 个用户提到的 P1 > 被 1 个用户提到的 P0
6. **4 阶段递进**: 止血 → 建信任 → 做增长 → 筑壁垒

## 平台特性注意事项

### Discord
- `exportguild` 命令可一次导出整个 server，但 Forum 频道会失败（正常）
- 导出可能耗时数分钟到十几分钟，welcome 频道通常最大
- Reactions 是沉默多数的投票 — 50 个 👍 比 50 条消息更有说服力
- 多语言频道（chinese/portuguese）包含独特的本地化洞察
- 关注 reply 线程（`reference` 字段），它们包含问题-解答对

### Reddit
- JSON 为单行格式，大文件 Read 工具打不开，**必须用 Python 解析**
- 区分官方营销帖 vs 真实用户帖（检查 author 是否为官方账号）
- 评论中的竞品对比价值极高
- `.json` API 有速率限制，每次请求间隔 2 秒
- 很多 subreddit 以行业讨论为主而非产品反馈 — 需要过滤产品相关内容

### GitHub
- 搜索结果可能包含已删除/改名的 repo，**务必逐一验证真实性**
- 薅羊毛工具的 star 数直接反映问题严重程度
- 同类产品（Cursor/Windsurf/Kiro/Augment）的灰产工具可作为参考基线
- `gh` CLI 拉 Issues 是最干净的结构化数据源
- 即使没有官方 repo，第三方工具生态本身就是重要信号

### Twitter/X（最难，可跳过）
- 官方 API 几乎不可用（免费 1500 条/月）
- 可行方案：Apify Twitter Scraper（~$5-10）或手动搜索
- 如果 Discord + Reddit + GitHub 数据充足，X 可以跳过
- 量小时直接在 X 搜索 `{product name}` 手动评估是否值得深入

## 多语言处理

- Discord/Reddit 社区经常多语言混杂
- 非英语内容同样重要（甚至更有价值 — 本地化痛点）
- 保留用户原话的原始语言，不要翻译替换
- 语言分布本身是市场信号（大量葡语 = 拉美市场；大量中文 = 中国市场潜力）
