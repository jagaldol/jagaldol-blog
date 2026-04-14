# Obsidian Series Plan

Last updated: 2026-04-14

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

## Later Parts
- Part 2 candidate: `AI Agent가 관리하는 Obsidian 구조 만들기`
  - focus on folder hubs, AGENTS or CLAUDE rules, instruction docs, and skills
- Part 3 candidate: `개인 비서를 오래 쓰기 위한 유지보수`
  - focus on long-term memory, instruction cleanup, and operational safeguards

## Repository Tasks
- Maintain `_pages/categories/obsidian.md`.
- Keep `docs/README.md` aligned whenever `docs/` files are added or renamed.
- Update `docs/FEEDBACK.md` when the user changes writing or workflow preferences.
