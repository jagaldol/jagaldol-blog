# Agents Guide

## Purpose
This file defines practical operating rules for AI/code agents working in this repository.

## Repository Facts
- Stack: Jekyll blog based on Minimal Mistakes.
- Deploy: GitHub Pages workflow on push to `main`.
- Timezone: `Asia/Seoul` (KST).
- Permalink policy: category-based URL (`/:categories/:title/`).

## Post Authoring Rules
1. Keep front matter `date` aligned with intended publish time in KST.
2. Keep post filename date aligned with front matter date:
   - `_posts/<category>/YYYY-MM-DD-title.md`
3. Store post images under date-based paths:
   - `/assets/images/YYYY/MM/DD/...`
4. If post date changes, update all related paths together:
   - post filename
   - image directory/date path references inside the post
5. Commit a post with all referenced assets in the same commit.

## Category Rules
- Prefer existing categories first.
- Current active categories include:
  - `ai`
  - `development`
  - `obsidian`
  - `papers`
  - `daily`
  - `experience`
  - `google-ml-bootcamp`
  - `naver-boostcamp`
- Avoid vague new categories (for example, `blog`).

## Commit Rules
- Keep each commit scoped to one topic/post when possible.
- Do not include unrelated files in the same commit.
- Recommended message prefixes:
  - `ai: ...`
  - `development: ...`
  - `chore: ...`

## Pre-Commit Checklist
1. Links and image paths resolve correctly.
2. No duplicate image files for the same section.
3. Scheduled future posts are expected to appear only after a new build/deploy runs after publish time.
4. `git status` is clean except intended files.

## Docs Directory Policy
- `docs/` is for internal working notes only (for example, `TODO`, planning notes, migration memos).
- Do not place publishable blog posts in `docs/`.
- Do not store post images in `docs/`; keep post assets under `/assets/images/YYYY/MM/DD/`.
- Keep docs lightweight and text-first unless binary attachments are strictly necessary.
- Maintain a simple index of docs files in `docs/README.md`.
- Record user feedback that affects workflow/content rules in `docs/FEEDBACK.md`.
- If docs files are added, removed, or renamed, update `docs/README.md` in the same change.
- During ongoing work, keep docs up to date continuously instead of batching changes later.

## Notes
- `docs/` is excluded from Jekyll build output.
