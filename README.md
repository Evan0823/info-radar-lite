# 📡 Info Radar Lite

> 轻量 AI 信息雷达 — 每日抓取 30 条 AI/科技资讯，LLM 智能打分，精选 Top 5 推荐。

源自 [agents-radar](https://github.com/duanyytop/agents-radar) 的核心管线，精简为一条命令、一个 Key、零服务依赖。

## ✨ 特性

- 🔍 **3 个免费数据源**：GitHub Trending · Hacker News · Dev.to，无需任何 API Key
- 🤖 **LLM 智能策展**：1 次调用批量评分，支持 Anthropic / OpenAI / DeepSeek
- 📊 **7 维评分 + 分类**：新颖性 · 实用价值 · 热度验证 · 信息密度，6 种分类标签
- 🔗 **去重持久化**：URL 标准化 + 标题相似度 + 历史记录三重去重
- 📱 **移动端网页**：暗色主题 GitHub Pages，列宽可拖拽，手机/桌面均适配
- ⏰ **一行定时**：Windows 计划任务，每天早 8 点自动运行
- 💰 **极低成本**：单次 LLM 调用 ~$0.05，月均 ~$1.5

## 🚀 快速开始

### 1. 克隆 & 安装

```bash
git clone https://github.com/evan0823/info-radar-lite.git
cd info-radar-lite
pnpm install
```

### 2. 配置环境变量

```bash
cp .env.example .env
```

编辑 `.env`，填入 LLM Key（二选一）：

```env
# 方式 A：Anthropic（推荐）
ANTHROPIC_API_KEY=sk-ant-xxxxx

# 方式 B：OpenAI
# OPENAI_API_KEY=sk-xxxxx

# 方式 C：DeepSeek（兼容 Anthropic 接口）
# ANTHROPIC_API_KEY=sk-xxxxx
# ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
# ANTHROPIC_MODEL=deepseek-v4-pro
```

### 3. 运行

```bash
pnpm start
```

30 秒后，终端显示彩色 Top 5 表格，`digests/` 目录生成当日 Markdown。

## 📖 命令说明

| 命令 | 作用 |
|------|------|
| `pnpm start` | 完整管线：抓取 → 去重 → 打分 → Markdown → manifest |
| `pnpm push` | 上传 digests/ + manifest.json + index.html 到 GitHub |
| `pnpm daily` | `pnpm start && pnpm push` 一键完成 |
| `pnpm manifest` | 单独重新生成 manifest.json |

## 🌐 发布到 GitHub Pages

### 一次性设置

```bash
# 1. 创建 GitHub 仓库（如 evan0823/info-radar-lite）
# 2. 生成 Token：Settings → Developer settings → Tokens (classic) → 勾选 repo
# 3. 填入 .env
```

```env
GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxx
GITHUB_REPO=evan0823/info-radar-lite
```

```bash
# 4. 启用 GitHub Pages（自动）
pnpm push
```

推送后访问 `https://你的用户名.github.io/info-radar-lite/` 即可。

### 每次更新

```bash
pnpm daily    # 等价于 pnpm start && pnpm push
```

## ⏰ 设置每日自动运行（Windows）

在 **PowerShell（管理员）** 中执行一次：

```powershell
schtasks /create `
  /tn "InfoRadarDaily" `
  /tr "D:\DMP\ClaudeCode\info-radar-lite\scripts\daily.bat" `
  /sc daily `
  /st 08:00
```

之后每天早 8:00 自动抓取 + 推送，手机上打开网页就是当天最新。

管理命令：

```powershell
schtasks /query /tn "InfoRadarDaily"   # 查看状态
schtasks /run   /tn "InfoRadarDaily"   # 手动触发一次
schtasks /delete /tn "InfoRadarDaily"  # 删除任务
```

## 📂 项目结构

```
info-radar-lite/
├── src/
│   ├── index.ts          # 核心管线：抓取 → 去重 → 打分 → 输出
│   ├── manifest.ts       # 扫描 digests/ → 生成 manifest.json
│   └── push.ts           # 通过 GitHub REST API 上传文件 + 触发 Pages 部署
├── scripts/
│   └── daily.bat         # Windows 计划任务入口
├── index.html            # GitHub Pages 网页（暗色主题，移动端优先）
├── digests/              # 每日推荐 Markdown + 去重表 seen.json
├── manifest.json         # 日期索引（网页侧边栏数据源）
├── .env.example          # 环境变量模板
└── package.json          # 3 个运行时依赖
```

## 🔧 数据管线

```
┌──────────────────────────────────────────────────┐
│  ① FETCH（并行抓取 3 个免费数据源）                    │
│     GitHub Trending (HTML)  →  10 repos           │
│     Hacker News  (Algolia)   →  15 stories        │
│     Dev.to       (Forem API) →  10 articles       │
├──────────────────────────────────────────────────┤
│  ② MERGE + DEDUP（三重去重）                         │
│     URL 标准化（去参数/去尾斜杠）                       │
│     精确 URL 匹配（同次 + 历史 seen.json）            │
│     标题相似度 > 85% 视为重复                         │
├──────────────────────────────────────────────────┤
│  ③ LLM SCORE（1 次调用，~12s）                      │
│     4 维度：新颖性(0-3) + 实用(0-3) + 热度(0-2) +     │
│              信息密度(0-2) = 总分 1-10              │
│     6 分类：工具·Agent·模型·行业·教程·产品             │
│     8-10 分不超过 25% 的条目                        │
├──────────────────────────────────────────────────┤
│  ④ OUTPUT（终端 + Markdown + 网页）                  │
│     终端：彩色表格（含评分/分类/理由）                   │
│     digests/YYYY-MM-DD.md：Markdown 完整报告        │
│     manifest.json：日期索引（网页侧边栏）              │
│     seen.json：持久化去重表                          │
└──────────────────────────────────────────────────┘
```

## 📊 网页界面

| 桌面 | 手机 |
|------|------|
| 左侧固定日期栏 | 顶部横滑日期栏 |
| 右侧内容区，列宽可拖拽 | 内容自适应，手指滑动 |
| 暗色/亮色主题切换 | 暗色/亮色主题切换 |

每列表头右边缘可**拖拽调整宽度**（桌面鼠标 + 手机触屏均支持）。

## ⚙️ 技术栈

| 层 | 技术 |
|----|------|
| 运行时 | Node.js ≥18 + TypeScript (tsx) |
| 包管理 | pnpm |
| LLM | Anthropic SDK / OpenAI SDK |
| 渲染 | marked.js (CDN) |
| 部署 | GitHub Pages + REST API 上传 |
| 定时 | Windows Task Scheduler |
| 依赖数 | **3 个运行时**（dotenv + 2 SDK） |

## ❓ FAQ

### 需要哪些 API Key？

最少 **1 个 LLM Key**。数据抓取完全免费，无需任何 Key。GitHub Pages 发布需要 GitHub Token。

### 支持哪些 LLM？

| Provider | 配置 |
|----------|------|
| Anthropic | `ANTHROPIC_API_KEY` |
| OpenAI | `OPENAI_API_KEY` |
| DeepSeek | `ANTHROPIC_API_KEY` + `ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic` |
| 其他兼容接口 | 设置 `ANTHROPIC_BASE_URL` 即可 |

### 数据源挂了怎么办？

每个源独立 catch 错误，单个失败不影响其他源。3 个源全挂才会报错退出。

### 如何增加数据源？

在 `src/index.ts` 的 Step 6（Fetchers 区域）添加新函数，返回 `LinkItem[]`，然后在 `main()` 的 `Promise.allSettled` 中追加即可。

### 如何改成英文输出？

修改 `SCORE_PROMPT` 中的 `lang` 参数为 `"en"`，以及 `saveMarkdown` 中的表头和文案。

### 每天只推荐 5 条，之前的链接会重复吗？

不会。`digests/seen.json` 记录所有已推荐 URL，下次运行自动跳过。最多保留 500 条历史。

## 📄 许可

MIT License — 详见 [agents-radar](https://github.com/duanyytop/agents-radar)
