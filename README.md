# Baidu API Search Skill

简体中文 | [English](README.en.md)

给 AI Agent 用的百度 API 搜索 Skill。它把百度 AI Search API 和百度百科 API 整理成一个“先拿来源、再让模型分析”的搜索流程，适合需要中文网页资料的 agent。

这个项目不是爬虫，也不是生成答案接口。它的重点是保留原始搜索结果、去重、整理成 research pack，让 agent 在回答前有资料可看、有来源可引用。

## 适合什么场景

- 给 Codex、Claude Code、Cursor 等支持本地 Skills 的 agent 增加中文搜索能力
- 查询中文网页、产品口碑、近期热点、项目资料、平台讨论
- 简单实体问题先走百度百科，减少不必要的网页搜索
- 需要保留原始搜索结果，方便后续 AI 分析和人工复查

不适合的场景：

- 大规模采集网页正文
- 绕过搜索引擎限制
- 把百度作为海外官方资料的唯一来源
- 直接生成无来源的最终答案

## 功能

- 调用百度 AI Search `web_search`
- 调用百度百科做轻量实体查询
- 自动规划多个搜索词，覆盖产品、事件、项目、普通资料查询
- 根据覆盖情况和重复率自适应停止，避免无意义消耗
- URL 去重和近似重复结果合并
- 输出适合 agent 阅读的 `research_pack.md`
- 本地缓存重复查询
- 支持多个 API Key 做失败切换和额度隔离

## 环境要求

- Python 3.10+
- 百度 AI Search / AppBuilder API Key
- 能读取 `SKILL.md` 并运行本地 Python 脚本的 agent 环境

脚本只使用 Python 标准库，不需要额外安装依赖。

本项目需要百度 AI Search API Key 才能执行真实搜索。没有 Key 的用户请参考百度官方文档：

- [百度搜索 API 文档](https://cloud.baidu.com/doc/qianfan-api/s/Wmbq4z7e5)
- [API Key 创建与管理](https://cloud.baidu.com/doc/qianfan/s/wmh8l6tnf)
- [API Key 认证鉴权](https://cloud.baidu.com/doc/qianfan-api/s/ym9chdsy5)

## 安装

克隆仓库：

```bash
git clone https://github.com/valenovo/baidu-search-skill.git
cd baidu-search-skill
```

把 Skill 复制到你的 agent skills 目录。

项目级安装示例：

```bash
mkdir -p .codex/skills
cp -r skills/baidu-api-search .codex/skills/
```

Windows PowerShell：

```powershell
New-Item -ItemType Directory -Force .codex\skills | Out-Null
Copy-Item -Recurse skills\baidu-api-search .codex\skills\
```

如果你的 agent 使用用户级 skills 目录，把 `skills/baidu-api-search` 复制到对应目录即可。

## 配置 API Key

把百度 API Key 放到环境变量里：

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

多个 Key 只用于失败切换和额度隔离，不会把同一个关键词分页翻完，也不应该用来绕过官方限制。

不要把真实 Key 写进配置文件、README、Prompt、日志或 Git 提交里。

## 快速使用

进入 Skill 目录：

```bash
cd skills/baidu-api-search
```

普通中文网页搜索：

```bash
python scripts/search.py "新能源汽车 口碑" --mode normal
```

简单实体查询，优先走百度百科：

```bash
python scripts/search.py "量子计算" --mode lookup
```

查询最近或当前信息，建议关闭缓存：

```bash
python scripts/search.py "新能源汽车 口碑" --mode normal --freshness year --no-cache
```

只检查请求体，不真正调用 API：

```bash
python scripts/baidu_web_search.py --query "新能源汽车 口碑" --top-k 50 --dry-run
```

## 模式说明

| 模式 | 适合场景 | 消耗 |
| --- | --- | --- |
| `lookup` | “这是什么”“这个人是谁”等实体问题 | 通常 1-2 次 API 调用 |
| `fast` | 快速找一批来源 | 1 个搜索词，低延迟 |
| `normal` | 默认模式，适合普通问答和一轮资料检索 | 最多规划 12 个搜索词，覆盖够了会提前停 |
| `deep` | 用户明确要求深度、全面、多角度 | 更高预算，会做结果驱动扩展 |

建议默认用 `normal`。只有用户明确要求深度覆盖时再用 `deep`。

## 输出文件

每次运行会在 `runs/` 下生成一个目录：

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

一般让 agent 按这个顺序读：

1. `run_summary.json`
2. `research_pack.md`
3. 需要核查细节时再读 `raw_results.jsonl`

`research_pack.md` 会列出来源 ID、URL、域名、命中的搜索词、摘要、重复信息和覆盖缺口。

## 推荐给 Agent 的指令

如果你的 agent 同时有通用 `web_search` 工具，建议在 workspace 或系统指令里加一句：

```text
中文网页搜索、百度百科查询、最近/最新/热点类中文问题，以及用户说“查一下”“搜索”“联网查”时，优先使用 baidu-api-search skill，而不是 generic web_search。回答事实性结论时引用 research_pack.md 里的 source_id 或 URL。
```

这一步很重要。很多 agent 会优先调用框架自带的通用搜索工具，明确指令能提高自动触发率。

## 限制

- 当前流程里，百度 AI Search 单次查询 `top_k` 上限按 50 处理。
- 脚本主要收集搜索结果、摘要和结构化引用，不抓取完整网页正文。
- 百度更适合中文网页覆盖；海外官方资料建议配合其他搜索源。
- 搜索结果里可能有低质量页面、重复页面、推广内容或二手信息。Skill 会做基础标记，但最终判断仍需要 agent 或用户完成。

## 安全说明

- API Key 只从环境变量读取。
- Key 不会写入运行输出或缓存。
- `runs/`、`cache/`、`.env` 等本地文件已加入 `.gitignore`。
- 缓存命中时会标记为 `key_id: "cache"`。

## 项目结构

```text
skills/baidu-api-search/
├── SKILL.md
├── agents/openai.yaml
├── scripts/
└── references/
```

## License

MIT
