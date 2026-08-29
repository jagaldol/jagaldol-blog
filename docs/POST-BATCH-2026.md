# 2026 Knowledge Post Batch

## Goal

Turn the selected Lifebase and T3 notes into 23 standalone blog posts, dated between the previous latest post (`2026-04-30`) and `2026-08-28`.

## Writing contract

- Follow the current Minimal Mistakes frontmatter, heading, image, and category conventions.
- Use blog-native prose rather than copying presentation scaffolding.
- Remove private names, absolute local paths, account details, and unrelated personal records.
- Prefer official sources, dated claims, concrete commands, decision tables, and experiment evidence.
- Keep guides task-first, experiments evidence-first, and paper reviews claim-structure-limitations-first.
- Run an anti-slop pass across only the new posts before final verification.
- Do not commit or push until explicit user approval. Deliver a temporary remote preview after a successful local build and before publication.

## Post schedule

| Date | Topic |
| --- | --- |
| 2026-05-04 | GitHub HTTPS authentication, credential helpers, and `gh auth login` |
| 2026-05-10 | Claude Code terminal tips |
| 2026-05-19 | Ollama guide |
| 2026-05-23 | Ollama, LM Studio, and llama.cpp comparison |
| 2026-05-26 | LM Studio guide |
| 2026-05-30 | llama.cpp guide |
| 2026-06-04 | Quantization-aware training for local LLMs |
| 2026-06-09 | OpenCode and Oh My OpenAgent |
| 2026-06-13 | Ollama with Codex App experiment |
| 2026-06-18 | Local models as OpenCode agents |
| 2026-06-22 | Agentic Vision |
| 2026-06-28 | Sakana Fugu |
| 2026-07-05 | LLM-JEPA paper review |
| 2026-07-12 | Semantic Tube Prediction paper review |
| 2026-07-19 | SkillOpt paper review |
| 2026-07-24 | SkillOpt usage guide |
| 2026-07-29 | Docker command cheat sheet |
| 2026-08-03 | JSON-RPC 2.0 |
| 2026-08-08 | MCP communication and transports |
| 2026-08-14 | Muon optimizer review |
| 2026-08-25 | SkillOpt-Sleep experiment |
| 2026-08-26 | Multi-token Prediction and DeepSeek |
| 2026-08-28 | DeepSeek-V4 technical report review |

## Verification

- [x] 23 posts exist with aligned filename and KST frontmatter dates.
- [x] Every referenced asset exists under its post-date directory.
- [x] No Obsidian embeds, wikilinks, local absolute paths, or private names remain.
- [x] Internal links resolve to the intended category permalink.
- [x] Jekyll production build passes with future posts enabled.
- [x] Temporary remote preview is reachable from outside localhost.
- [x] Explicit user approval to commit and push was received after preview review.
