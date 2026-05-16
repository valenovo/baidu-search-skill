# Baidu Search Skill

[简体中文](README.md) | English

A small Baidu search skill for AI agents that need Chinese web sources instead of a generated search summary.

It wraps Baidu AI Search and Baidu Baike into a source-first workflow: plan a few targeted queries, collect raw search references, deduplicate them, and write a compact research pack that an agent can cite.

## Why

Many agents can reason well once they have sources, but they do not always have a reliable way to search the Chinese web. This skill is meant for that gap.

It is not a crawler, a scraper, or an answer-generation service. It keeps the original search references so the model can inspect evidence before answering.

## What It Does

- Web search through Baidu AI Search `web_search`
- Lightweight entity lookup through Baidu Baike
- Query planning for product, event, project, and general research topics
- Adaptive stop conditions to avoid wasting API calls
- URL and near-duplicate cleanup
- Markdown research packs for downstream agents
- Local cache for repeated identical queries
- API key failover through environment variables

## Requirements

- Python 3.10+
- A Baidu AI Search / AppBuilder API key
- An agent or runtime that can read `SKILL.md` and run local Python scripts

The Python scripts use only the standard library.

This project requires a Baidu AI Search API key for live search. If you do not have one, start from the official Baidu docs:

- [Baidu Search API reference](https://cloud.baidu.com/doc/qianfan-api/s/Wmbq4z7e5)
- [API Key creation and management](https://cloud.baidu.com/doc/qianfan/s/wmh8l6tnf)
- [API Key authentication](https://cloud.baidu.com/doc/qianfan-api/s/ym9chdsy5)

## Install

Clone the repository:

```bash
git clone https://github.com/valenovo/baidu-search-skill.git
cd baidu-search-skill
```

Copy the skill folder into your agent's skills directory.

Project-level install:

```bash
mkdir -p .codex/skills
cp -r skills/baidu-search .codex/skills/
```

Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force .codex\skills | Out-Null
Copy-Item -Recurse skills\baidu-search .codex\skills\
```

For a user-level install, copy `skills/baidu-search` into the skills directory used by your agent.

## Configure

Set your Baidu API key in the environment:

```bash
export BAIDU_AI_SEARCH_API_KEYS="your-key"
```

Windows PowerShell:

```powershell
$env:BAIDU_AI_SEARCH_API_KEYS = "your-key"
```

Multiple keys can be provided as a comma-separated list:

```bash
export BAIDU_AI_SEARCH_API_KEYS="key1,key2,key3"
```

Multiple keys are used for failover and quota isolation. They are not used as pagination for one query.

Do not put real keys in config files, prompts, README examples, or committed logs.

## Quick Start

From the skill directory:

```bash
cd skills/baidu-search
python scripts/search.py "新能源汽车 口碑" --mode normal
```

For a simple entity question:

```bash
python scripts/search.py "量子计算" --mode lookup
```

For recent or current information:

```bash
python scripts/search.py "新能源汽车 口碑" --mode normal --freshness year --no-cache
```

To inspect the API request without calling Baidu:

```bash
python scripts/baidu_web_search.py --query "新能源汽车 口碑" --top-k 50 --dry-run
```

## Modes

| Mode | Use for | Budget |
| --- | --- | --- |
| `lookup` | Simple entity definitions and Baike disambiguation | Usually 1-2 API calls |
| `fast` | Quick source discovery | 1 query, low latency |
| `normal` | Ordinary agent Q&A and first-pass research | Up to 12 planned queries, stops early when coverage is enough |
| `deep` | Broad research when the user asks for depth | Larger query budget and result-driven expansion |

Use `normal` as the default. Use `deep` only when breadth matters.

## Output

Each run writes a directory under `runs/`:

```text
runs/<timestamp>-<mode>-<topic>/
├── query_plan.json
├── raw_results.jsonl
├── deduped_sources.json
├── research_pack.md
├── run_summary.json
├── adaptive_trace.json
└── errors.jsonl
```

Agents should normally read:

1. `run_summary.json`
2. `research_pack.md`
3. `raw_results.jsonl` only when detailed evidence is needed

`research_pack.md` includes source IDs, URLs, domains, matched queries, snippets, duplicate notes, and coverage gaps.

## Recommended Agent Instruction

Add a short instruction to your agent or workspace:

```text
For Chinese web search, recent Chinese topics, Baidu Baike lookup, or requests phrased as 查一下 / 搜索 / 最近 / 最新 / 热点, prefer the baidu-search skill over generic web_search. Cite source IDs or URLs from research_pack.md.
```

This matters when your agent runtime also exposes a generic `web_search` tool. Without an instruction, the model may choose the generic tool first.

## Limits

- Baidu AI Search currently has a per-query `top_k` ceiling of 50 in this workflow.
- The scripts collect search references and snippets; they do not fetch full page bodies.
- Baidu is strongest for Chinese web coverage. For overseas official sources, use this as a supplement rather than the only source.
- Search results can include low-quality pages, duplicate pages, and promotional content. The skill marks obvious quality flags, but final judgment still belongs to the agent.

## Safety

- API keys are read from environment variables only.
- API keys are not written to run outputs or cache files.
- Run directories and cache directories are ignored by git.
- Cached responses are marked with `key_id: "cache"`.

## Project Layout

```text
skills/baidu-search/
├── SKILL.md
├── agents/openai.yaml
├── scripts/
└── references/
```

## License

MIT
