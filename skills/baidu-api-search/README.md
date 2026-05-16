# Baidu API Search

简体中文 | English

面向 Agent 的百度 API 搜索 Skill。它调用百度 AI Search API 和百度百科 API，把中文搜索结果整理成可复查、可引用的 research pack，适合需要中文网页来源的 AI Agent 使用。

这个 Skill 不是爬虫，也不是答案生成接口。它只负责检索、去重、保留原始结果，并把来源整理给 Agent 后续分析。

## 功能

- 百度 AI Search `web_search`
- 百度百科轻量实体查询
- `lookup`、`fast`、`normal`、`deep` 四种模式
- 自动规划搜索词并根据覆盖情况提前停止
- URL 去重和近似重复合并
- 输出 `research_pack.md`、`raw_results.jsonl`、`run_summary.json`
- 支持本地缓存和多个 API Key 失败切换

## 环境要求

- Python 3.10+
- 百度 AI Search / AppBuilder API Key
- 能读取 `SKILL.md` 并运行本地 Python 脚本的 Agent 环境

脚本只使用 Python 标准库，不需要额外安装依赖。

API Key 请放在环境变量里：

```bash
export BAIDU_AI_SEARCH_API_KEYS="your-key"
```

Windows PowerShell：

```powershell
$env:BAIDU_AI_SEARCH_API_KEYS = "your-key"
```

多个 Key 用英文逗号分隔：

```bash
export BAIDU_AI_SEARCH_API_KEYS="key1,key2,key3"
```

不要把真实 Key 写进配置文件、README、Prompt、日志或 Git 提交里。

## 快速使用

```bash
python scripts/search.py "新能源汽车 口碑" --mode normal
```

简单实体查询：

```bash
python scripts/search.py "量子计算" --mode lookup
```

查询最近或当前信息：

```bash
python scripts/search.py "新能源汽车 口碑" --mode normal --freshness year --no-cache
```

## 给 Agent 的推荐指令

```text
中文网页搜索、百度百科查询、最近/最新/热点类中文问题，以及用户说“查一下”“搜索”“联网查”时，优先使用 baidu-api-search skill。回答事实性结论时引用 research_pack.md 里的 source_id 或 URL。
```

## English

Baidu API Search is a source-retrieval skill for AI agents that need Chinese web references.

It calls Baidu AI Search API and Baidu Baike API, then writes raw results, deduplicated sources, and a compact research pack for downstream analysis. It is not a crawler, scraper, or answer-generation service.

### Features

- Baidu AI Search `web_search`
- Lightweight Baidu Baike lookup
- Four modes: `lookup`, `fast`, `normal`, `deep`
- Query planning with adaptive stop conditions
- URL and near-duplicate cleanup
- `research_pack.md`, `raw_results.jsonl`, and `run_summary.json` outputs
- Local cache and multi-key failover

### Requirements

- Python 3.10+
- A Baidu AI Search / AppBuilder API key
- An agent runtime that can read `SKILL.md` and run local Python scripts

Set your API key in the environment:

```bash
export BAIDU_AI_SEARCH_API_KEYS="your-key"
```

Windows PowerShell:

```powershell
$env:BAIDU_AI_SEARCH_API_KEYS = "your-key"
```

### Quick Start

```bash
python scripts/search.py "新能源汽车 口碑" --mode normal
```

For a simple entity lookup:

```bash
python scripts/search.py "量子计算" --mode lookup
```

For recent or current information:

```bash
python scripts/search.py "新能源汽车 口碑" --mode normal --freshness year --no-cache
```
