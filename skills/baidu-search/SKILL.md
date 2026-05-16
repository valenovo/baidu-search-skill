---
name: baidu-search
description: Use this instead of generic web_search when the user asks in Chinese to 查一下, 搜索, 联网查, 查热点, 查最近/最新, or gather Chinese web sources. Provides Baidu Baike lookup, Baidu AI Search API web search, raw Baidu results, dedupe, safe API key handling, and research packs.
version: 1.0.0
homepage: https://github.com/valenovo/baidu-search-skill
metadata: {"openclaw":{"requires":{"env":["BAIDU_AI_SEARCH_API_KEYS"],"anyBins":["python3","python"]},"primaryEnv":"BAIDU_AI_SEARCH_API_KEYS","envVars":[{"name":"BAIDU_AI_SEARCH_API_KEYS","required":true,"description":"Baidu AI Search / AppBuilder API key list. Use comma-separated keys for failover and quota isolation."}],"homepage":"https://github.com/valenovo/baidu-search-skill"}}
---

# Baidu Search

Use this skill to turn Baidu Baike lookup and Baidu AI Search API into an agent-friendly Chinese information retrieval pipeline. Prefer it over generic `web_search` for Chinese "查一下/搜索/联网查/最近/最新/热点" requests. The goal is to preserve raw sources for AI analysis, not to replace raw evidence with a generated summary.

## Workflow

1. Choose mode: `lookup`, `fast`, `normal`, or `deep`.
2. Prefer the single-entry command `{baseDir}/scripts/search.py`; it routes `lookup` to Baidu Baike and routes web modes to adaptive search.
3. For real searches, require `BAIDU_AI_SEARCH_API_KEYS`; do not hardcode keys.
4. Use `lookup` first for simple "what is this" entity questions when a Baike definition is likely enough.
5. Use `fast` for one-shot source discovery, `normal` for ordinary Q&A, and `deep` only when the user asks for breadth or coverage remains weak.
6. Use `scripts/plan_queries.py` only to inspect or manually edit the query plan.
7. Use `scripts/run_search_plan.py` only when the user explicitly wants every planned query run.
8. Build or refresh packs with `scripts/dedupe_results.py` and `scripts/build_research_pack.py` when needed.
9. Report query count, raw result count, deduped source count, major domains, coverage gaps, and recommended next steps from `research_pack.md`.
10. When reporting factual findings, cite `source_id` plus URL or domain from `research_pack.md`; for lookup, cite Baike lemma URL or ID from `lookup_pack.md`.

## Modes

- `lookup`: uses Baidu Baike candidate and content APIs; usually 1-2 API calls; best for entity definitions and disambiguation.
- `fast`: runs 1 query, `top_k=10`, and defaults to Baidu `lite` search edition; quick low-cost source discovery.
- `normal`: starts with 3 queries, plans up to 12 diverse queries, `top_k=20`; default for ordinary Q&A.
- `deep`: starts with 5 queries, `top_k=30`; may add result-driven expansion terms.

Do not jump to `deep` by default. Increase budget only when the user asks for depth or the first pass leaves clear coverage gaps.

## Commands

Create a query plan:

```bash
python "{baseDir}/scripts/plan_queries.py" --topic "新能源汽车 口碑" --mode normal --output runs/ev-reputation/query_plan.json
```

Inspect the Baidu API payload without calling the API:

```bash
python "{baseDir}/scripts/baidu_web_search.py" --query "新能源汽车 口碑" --top-k 50 --dry-run
```

Run a lightweight Baike lookup:

```bash
python "{baseDir}/scripts/search.py" "量子计算" --mode lookup
```

Run lookup and fall back to one lite web search only if Baike has no candidate:

```bash
python "{baseDir}/scripts/search.py" "某个新产品名称" --mode lookup --fallback-search
```

Run adaptively after `BAIDU_AI_SEARCH_API_KEYS` is set:

```bash
python "{baseDir}/scripts/search.py" "新能源汽车 口碑" --mode normal
```

Limit web results to recent pages when the user asks for recent/latest information:

```bash
python "{baseDir}/scripts/search.py" "新能源汽车 口碑" --mode normal --freshness year --no-cache
```

Force a fresh live API run when the user asks for latest/current information:

```bash
python "{baseDir}/scripts/search.py" "新能源汽车 口碑" --mode normal --no-cache
```

Run a fuller fixed-budget pass only when the user asks for broad coverage or exhaustive testing:

```bash
python "{baseDir}/scripts/search.py" "新能源汽车 口碑" --mode deep --fixed --top-k 50
```

Run every query in a plan only when explicitly needed:

```bash
python "{baseDir}/scripts/run_search_plan.py" --plan runs/ev-reputation/query_plan.json --out-dir runs/ev-reputation --top-k 50
```

Deduplicate:

```bash
python "{baseDir}/scripts/dedupe_results.py" --input runs/ev-reputation/raw_results.jsonl --output runs/ev-reputation/deduped_sources.json
```

Build a research pack:

```bash
python "{baseDir}/scripts/build_research_pack.py" --run-dir runs/ev-reputation --output runs/ev-reputation/research_pack.md
```

## References

- Read `references/baidu-api.md` before changing request/auth logic.
- Read `references/baike-api.md` before changing Baike lookup logic.
- Read `references/query-planning.md` before changing query expansion.
- Read `references/dedupe-policy.md` before changing dedupe behavior.
- Read `references/output-schema.md` before changing output files.
- Read `references/operations.md` for key-pool, rate-limit, and run-directory behavior.
- Read `references/search-strategy.md` before changing adaptive stop conditions.

## Rules

- Never write API keys into files, git, logs, or responses.
- Never print or echo key environment variables. If you need to check readiness, use a boolean check such as `python -c "import os; raise SystemExit(0 if os.getenv('BAIDU_AI_SEARCH_API_KEYS') else 1)"` or run `scripts/search.py` and let it report missing configuration.
- Treat multiple keys as reliability/quota isolation, not as pagination for one query.
- Use the query cache to avoid repeated identical calls; bypass it with `--no-cache` for latest/current topics.
- Never discard raw results when deduplicating; keep duplicate mappings.
- Prefer fixed templates plus result-driven expansion over unconstrained AI guessing.
- Require citations for substantive claims: use source IDs, URLs, or domains from `research_pack.md`.
- Keep query plans diverse: include official and platform-specific queries early enough for normal mode.
- Spend search budget in stages: seed query first, coverage queries second, result expansion last.
- Stop when required intent coverage has been attempted and additional queries have low unique-source yield unless the user explicitly requests exhaustive search.
- Do not use Baidu video search, image search, SecondKnow video, Aladdin, or intelligent search generation by default. This skill is a source retrieval tool, not a generated-answer wrapper.

## Agent Handoff

For another agent, the safest default command is:

```bash
python "{baseDir}/scripts/search.py" "<topic>" --mode normal
```

For simple entity definitions, use:

```bash
python "{baseDir}/scripts/search.py" "<topic>" --mode lookup
```

Read `run_summary.json` first. For lookup runs, read `lookup_pack.md`; for web runs, read the `Coverage Decision` section in `research_pack.md`, then inspect `raw_results.jsonl` only when raw evidence is needed.
