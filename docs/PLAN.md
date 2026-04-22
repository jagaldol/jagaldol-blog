# Obsidian Series Plan

Last updated: 2026-04-22

## North Star
- Build an `obsidian` category as a focused series, not as a loose subset of `development`.
- Keep part 1 usage-first: show why Obsidian became viable again once local-file AI agents could maintain it on the user's behalf.
- Keep later parts structure-first: explain the vault design, instructions, and maintenance workflow only after the value proposition is clear.

## Part 1
- Status: draft created, local build verified, screenshots organized and placeholders resolved
- Post path: `_posts/obsidian/2026-04-14-obsidian-ai-assistant-daily-note.md`
- Working title: `AI Agent로 Obsidian 개인 비서 만들기 - 일일기록부터 Day Planner까지`
- One-line message: `일일기록을 쓰면, Obsidian 위의 AI 개인 비서가 하루를 운영 가능한 형태로 정리해준다.`

### Narrative spine
- 2 years ago, Obsidian failed because maintaining structure manually cost more than the value returned.
- The turning point is that modern local-file AI agents can now read and write markdown vaults directly.
- The goal is not to build a personal assistant from scratch, but to stand on top of Obsidian's UI/UX, plugin ecosystem, and agent-friendly file model.
- The main user-facing workflow is `daily note -> agent follow-up -> schedule / TODO / long-term memory -> Day Planner view`.

### Content guardrails
- Write in blog prose, not presentation bullets.
- Open part 1 with a short hook, then move into a section titled `들어가기에 앞서: AI Agent가 필요한 이유`.
- Mention Claude Code, Codex, and OpenClaw explicitly in the hook as representative local-file agents, but keep the article centered on the workflow rather than tool comparison.
- Use a clearer heading hierarchy: a few large `##` sections with `###` subtopics instead of a flat sequence of same-level sections.
- Treat `daily-note-followup`, `13_my`, and `Day Planner` as core concepts for part 1.
- Mention scraping, kanban, and automatic folder maintenance only as optional extensions.

### Image plan
- Final assets in use:
  - `/assets/images/2026/04/14/obsidian-daily-note-raw.png`
  - `/assets/images/2026/04/14/obsidian-agent-followup-summary.png`
  - `/assets/images/2026/04/14/obsidian-daily-note-with-schedule.png`
  - `/assets/images/2026/04/14/obsidian-day-planner-timeline.png`
  - `/assets/images/2026/04/14/obsidian-graph-view.png`
  - `/assets/images/2026/04/14/obsidian-agents-md.png`
- Keep image filenames descriptive and store only the canonical versions under the post date path.

## Part 2
- Status: draft created, local build verified
- Post path: `_posts/obsidian/2026-04-15-obsidian-ai-agent-vault-structure.md`
- Working title: `AI Agent가 관리하는 Obsidian 구조 만들기 - AGENTS.md, 허브 노트, 00_agent_docs 설계`
- One-line message: `좋은 Obsidian 개인 비서는 좋은 모델만으로 유지되지 않고, 읽히는 구조와 지침 계층 위에서 유지된다.`

### Narrative spine
- Part 1 showed the user-facing automation flow; part 2 explains the structure that makes it sustainable.
- The main point is not aesthetics but operating cost: readable structure prevents the cleanup burden from falling back to the user.
- The article should show the real vault layout, then explain how `AGENTS.md`, hubs, `00_agent_docs`, and local skills split responsibilities.

### Content guardrails
- Keep the article partially standalone for readers who did not open part 1.
- Prefer code-block excerpts over screenshots for text documents such as `AGENTS.md` and hub note bodies.
- Use screenshots only where structure is visually useful: hub graph, vault graph, folder navigation.
- Treat `daily-note-followup` as one example of document routing, not the main subject.

### Image plan
- New hero asset:
  - `/assets/images/2026/04/15/obsidian-vault-graph-overview.png`
- Reused structural asset:
  - `/assets/images/2026/04/15/obsidian-graph-view.png`

## Part 3
- Status: draft created, generated hero added, local build verified
- Post path: `_posts/obsidian/2026-04-20-obsidian-ai-agent-maintenance.md`
- Working title: `AI Agent로 Obsidian 개인 비서를 오래 쓰는 법 - 지침과 장기 메모리 운영 실전편`
- One-line message: `좋은 구조를 만든 뒤에는, 새 정보와 새 규칙을 어디에 반영할지 계속 분류하고 수선하는 운영 규칙이 필요하다.`

### Narrative spine
- Part 2 explained the structure split; part 3 should show the operating moves that keep that structure useful over time.
- The article should stay practical: `문제 -> representative excerpt -> immediately usable rule`.
- The focus is not abstract maintenance philosophy but the actual routing decisions made when new facts, new rules, and document bloat appear.
- The strongest value is showing how `12_일일기록`, `13_my`, `AGENTS.md`, playbooks, checklists, and skills interact during routine updates.

### Content guardrails
- Do not re-explain the full folder structure or redefine `AGENTS.md`, hubs, `00_agent_docs`, and skills from scratch.
- Prefer generalized code-block excerpts over screenshots for internal documents.
- Keep excerpts short, safe, and non-identifying: around 5-12 lines per snippet.
- Use tables for the decision points: information routing, rule placement, and bloat signals.
- Keep the hierarchy at `H2-H4`; if a topic would need `H5`, flatten it into a table, bold lead-ins, or short bullets.

### Source anchors
- `/Users/hyejun/Documents/Obsidian/AGENTS.md`
- `/Users/hyejun/Documents/Obsidian/00_agent_docs/운영/에이전트 문서 운영.md`
- `/Users/hyejun/Documents/Obsidian/00_agent_docs/장기 메모리/장기 메모리 메모.md`
- `/Users/hyejun/Documents/Obsidian/00_agent_docs/플레이북/개인 정보 업데이트 플레이북.md`
- `/Users/hyejun/Documents/Obsidian/00_agent_docs/플레이북/13_my 업데이트 검증 체크리스트.md`
- `/Users/hyejun/Documents/Obsidian/.agents/skills/daily-note-followup/SKILL.md`

### Image plan
- Generated hero asset:
  - `/assets/images/2026/04/20/obsidian-agent-maintenance-hero.png`
- Keep the body screenshot-light and excerpt-heavy.

## Part 4
- Status: draft created, screenshot and hero gathered, local build verified
- Post path: `_posts/obsidian/2026-04-21-obsidian-workspace-plugin-stack.md`
- Working title: `Obsidian을 메모 앱이 아니라 작업 공간으로 만드는 핵심 플러그인 구성`
- One-line message: `핵심은 플러그인 수가 아니라, 기록·실행·정리 흐름이 에이전트와 함께 덜 틀리게 굴러가도록 만드는 조합이다.`

### Narrative spine
- Part 4 should answer why Obsidian works as a practical workspace without becoming a long VS Code comparison piece.
- The center of gravity is the plugin combinations that connect recording, execution, visualization, and upkeep.
- The article should move from `why this workspace works`, to `record/execute/organize flows`, to `why the agent makes fewer mistakes on top of them`.

### Content guardrails
- Avoid a flat plugin catalog. Group by workflow: `기록 흐름`, `실행 흐름`, `정리 흐름`.
- Keep the core plugin list narrow:
  - `obsidian-day-planner`
  - `periodic-notes`
  - `templater-obsidian`
  - `base-board`
  - `dataview`
  - `obsidian-custom-attachment-location`
- Mention `calendar` only as a helper around `periodic-notes`.
- Do not let `Image Toolkit`, `terminal`, or `wakatime` become main sections.
- Keep the article grounded in operation, including rough edges and current workarounds.
- Frame global Obsidian skills only briefly; save the full global-vs-local skill discussion for part 5.

### Source anchors
- Installed plugins under `/Users/hyejun/Documents/Obsidian/.obsidian/plugins/`
- Plugin notes under `/Users/hyejun/Documents/Obsidian/08_작업도구/Obsidian/Obsidian 플러그인/`
- Real config anchors:
  - `.obsidian/plugins/templater-obsidian/data.json`
  - `.obsidian/plugins/periodic-notes/data.json`
  - `.obsidian/plugins/obsidian-custom-attachment-location/data.json`
- Real template anchors:
  - `_Templates/일일기록 일간 노트.md`
  - `_Templates/TODO 노트.md`
- Real operating anchors:
  - `00_agent_docs/플레이북/Base Board 작업 지침.md`
  - `00_agent_docs/플레이북/이미지 첨부 운영 플레이북.md`
- Skill anchors:
  - global: `obsidian-cli`, `obsidian-markdown`, `obsidian-bases`
  - local: `.agents/skills/daily-note-followup/SKILL.md`

### Image plan
- Generated hero asset:
  - `/assets/images/2026/04/21/obsidian-plugin-stack-hero.png`
- New UI screenshot:
  - `/assets/images/2026/04/21/obsidian-base-board.png`
- Reused Day Planner screenshot:
  - `/assets/images/2026/04/14/obsidian-day-planner-timeline.png`

## Part 5
- Status: draft created, local build verified
- Post path: `_posts/obsidian/2026-04-22-obsidian-ai-assistant-skill-stack.md`
- Working title: `Obsidian AI 비서에게 어떤 스킬을 주면 좋을까 - 기본 스킬과 직접 만든 비서 운영 스킬까지`
- One-line message: `공식 기본 스킬 5개는 출발점이고, 실제 개인 비서 운영은 내 Vault 규칙을 담은 비서 운영 스킬이 붙어야 시작된다.`

### Narrative spine
- Part 5 should answer a more direct reader question: which skills are worth giving an Obsidian AI assistant?
- Keep the official Obsidian skill pack brief; it is the baseline, not the center of gravity.
- Make the center of gravity the directly built assistant-operation skills and why they exist.
- Show that a useful skill is usually a package of `SKILL.md`, `references`, and optional `agents/openai.yaml`, not just one prompt file.
- End with a light-weight “when to turn a workflow into your own skill” section rather than a full tutorial.

### Source anchors
- Official GitHub source of truth:
  - `https://github.com/kepano/obsidian-skills`
- Official baseline skills:
  - `obsidian-markdown`
  - `obsidian-bases`
  - `json-canvas`
  - `obsidian-cli`
  - `defuddle`
- Local assistant-operation skills:
  - `.agents/skills/daily-note-followup/`
  - `.agents/skills/organize-instructions/`

### Image plan
- Generated hero asset:
  - `/assets/images/2026/04/22/obsidian-skill-stack-hero.png`

## Part 6
- Status: planned
- Post path: `TBD`
- Working title: `Obsidian 스킬은 어디에 설치해야 할까 - 전역 기본 스킬과 Vault 전용 스킬을 나누는 기준`
- One-line message: `스킬을 잘 만드는 것만큼 중요한 건 설치 위치를 잘 나누는 일이다. 공용 능력은 전역에, Vault 규칙은 로컬에 둬야 재사용성과 안정성이 같이 산다.`

### Narrative spine
- Part 6 should answer the next practical question left after part 5: where should each skill live?
- Keep the article short and judgment-first: global versus local placement matters more than packaging details here.
- Show that installation location is an architectural choice because it changes context size, reuse, and failure rate across repositories.
- Use one realistic future example as the center: a global custom skill that captures work done in any repository and records it into today's Obsidian daily note.

### Content guardrails
- Keep the article compact rather than turning it into another full skill tutorial.
- Open with the common mistake: putting every useful skill into one local vault and then losing reuse everywhere else.
- Use `이 스킬이 특정 vault를 몰라도 되나?` as the main split criterion.
- Explain global placement with both official Obsidian baseline skills and reusable custom skills.
- Explain local placement with vault-specific rules, canonical documents, and section ownership.
- Do not re-explain `SKILL.md`, `references`, and `agents/openai.yaml` in depth; part 5 already covered their internal structure.

### Future global custom skill candidate
- Tentative name: `obsidian-worklog-capture`
- Role: capture work done in any repository and append it into today's Obsidian daily note as schedule sub-items and memo bullets.
- Why global:
  - it should be callable from arbitrary repositories
  - it should prevent the agent from rediscovering the vault structure every time
  - it should reduce context growth and routing mistakes when the main task lives outside the Obsidian repository
- Required references:
  - daily note path convention
  - `## 일정` insertion rules
  - `## 메모` phrasing rules
  - safeguards for time ranges and section ownership
- Future split of responsibility:
  - the global skill owns the cross-repo capture interface
  - the local vault documents still own the exact writing rules and note conventions

### Source anchors
- Official skills source of truth:
  - `https://github.com/kepano/obsidian-skills`
- Global skill directory:
  - `/Users/hyejun/.codex/skills/`
- Local vault skill directory:
  - `/Users/hyejun/Documents/Obsidian/.agents/skills/`
- Daily note and writing-rule anchors:
  - `/Users/hyejun/Documents/Obsidian/AGENTS.md`
  - `/Users/hyejun/Documents/Obsidian/00_agent_docs/플레이북/일일기록 일정 작성 플레이북.md`
  - `/Users/hyejun/Documents/Obsidian/12_일일기록/`

### Image plan
- Optional only.
- If needed later, prefer a simple diagram that contrasts `global skill -> arbitrary repos + Obsidian vault` with `local skill -> one specific vault`.

## Repository Tasks
- Maintain `_pages/categories/obsidian.md`.
- Keep `docs/README.md` aligned whenever `docs/` files are added or renamed.
- Update `docs/FEEDBACK.md` when the user changes writing or workflow preferences.
