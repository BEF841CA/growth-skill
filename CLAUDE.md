# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Growth-skill is a **Claude Code Skill** that tracks anyone's cognitive growth trajectory through daily observations. It is inspired by [nuwa-skill](https://github.com/alchaincyf/nuwa-skill) but fundamentally different: nuwa-skill does one-shot distillation of famous people from public data; growth-skill does **longitudinal cognitive tracking** from personal observations over time.

## Architecture

This is a Claude Code Skill project — `SKILL.md` is the core implementation, not traditional code. The skill is a set of instructions that Claude Code follows when triggered.

### File Relationships

- **SKILL.md** — The main skill file. Contains YAML frontmatter (name, description, trigger words) and a multi-phase execution pipeline. This is the single source of truth for all behavior.
- **references/growth-framework.md** — Methodology that SKILL.md's Phase 3 references during profile synthesis. Contains signal type definitions, signal data structure, Ebbinghaus forgetting curve parameters, adaptive verification thresholds, contradiction handling rules, and quality checklist.
- **references/profile-template.md** — Output template that Phase 3 uses to generate cognitive profiles. Defines the exact structure of `profile.md`.
- **data/** — Runtime data directory (created during use, not committed). Each tracked person gets an isolated subdirectory with observations, signals, archives, and profiles.

### Multi-Phase Pipeline (SKILL.md)

| Phase | Purpose | Trigger |
|-------|---------|---------|
| Phase 0 | Entry routing — classify user input | Every input |
| Phase 0A | Create new tracking object | "追踪 [名字]" |
| Phase 0B | Manage/delete/correct records | "删除记录"/"修正" |
| Phase 0C | Correct extracted signals (post-Phase 1 hook only) | Only after Phase 1 response |
| Phase 0D | Rollback profile to historical version | "回滚"/"恢复画像"/"回退到" |
| Phase 1 | Record observation + lightweight signal extraction | "记录"/"今天"/fuzzy match |
| Phase 2 | Periodic signal status check | Cron (deep: weekly, casual: monthly) |
| Phase 3 | Profile synthesis ("sleep consolidation") | "演化 [名字]" / manual trigger |
| Phase 4 | Growth review — compare profile snapshots | "成长报告"/"复盘" |
| Phase 5 | Tracking overview — all persons status | "看看状态" |

### Memory Hierarchy (signals.jsonl archiving)

Signals follow a 4-layer memory cycle simulating human brain. Each layer uses a dual-file scheme: raw file (full preservation, human-read only) + summary file (clustered/merged, used by synthesis):

1. **Working memory** (`signals.jsonl`) — Active signals, capacity ~200, append-only (updates append new lines, read by deduplicating on `id`)
2. **Consolidation** (`signals-archive/consolidating/`) — Monthly: `YYYY-MM-raw.json` + `YYYY-MM-summary.md`
3. **Long-term** (`signals-archive/long-term/`) — Yearly: `YYYY-raw/` + `YYYY-summary.md`
4. **Schemas** (`signals-archive/schemas/`) — Permanent pattern crystallization, never forgotten

Forgetting follows Ebbinghaus curve: `retention = e^(-t/stability)`. Different signal types have different stability (milestone=180d, emotional_pattern=30d). Repeated matching increases stability via spaced repetition effect.

## Installation & Testing

```bash
# Install as local skill
npx skills add .

# No build step, test suite, or dependencies — this is pure markdown
```

Verification is done by installing the skill and running through the phases interactively:
1. "追踪 Leo，我的儿子" → check `data/` directory creation
2. Input a few observations → check `observations/` and `signals.jsonl`
3. "演化 Leo" → check `profile.md` generation
4. "Leo的成长报告" → check growth narrative output
5. "看看状态" → check overview display

## Key Design Decisions

- **Zero-friction recording**: Users write free-form text, no structured input required
- **Organic output**: Responses read like a biographer's notes, not data reports ("一个模式正在浮现" not "信号置信度0.7")
- **Contradictions as growth**: Temporal contradictions are celebrated evidence of cognitive change, not resolved
- **Honesty first**: Always annotate limitations rather than over-interpret sparse data
- **Observer bias awareness**: All profiles include a section analyzing the observer's systematic recording biases
- **Age-aware analysis**: Birth date is required; the system adjusts cognitive analysis framework based on age group (0-3, 3-6, 6-12, 12-18, 18-30, 30-50, 50-65, 65+)
- **Signal correction**: Phase 0C allows users to correct system-extracted signals without modifying original observation records — addresses the gap between what was observed and what the system inferred
- **Soft profile rollback**: Phase 0D restores a historical profile snapshot without touching signal data; hard rollback (signals + profile) is available on explicit request with risk confirmation

## When Editing

- SKILL.md is the single source of truth — all behavior is defined here
- growth-framework.md is referenced by SKILL.md Phase 3; changes here affect synthesis logic
- profile-template.md defines output structure; changes here affect what profiles look like
- The three files form a tight contract — changes to one may require updates to others
