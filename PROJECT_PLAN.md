# CCPlus — Project Plan

*Automated pipeline that tracks and surfaces what's new in Claude Code — updates, tips,
techniques, and "hacks" — and publishes it as a versioned, cited, Anthropic-styled artifact.*

- **Repo:** https://github.com/Claude-Booster/CCPlus
- **Live site:** https://claude-booster.github.io/CCPlus/
- **Started:** June 4, 2026 · **Status:** v1.0 live · Active engine: GitHub Actions

---

## 1. Context

The owner (a Claude Code beginner on an enterprise plan) plans to rely heavily on
Claude Code for project delivery. Because the AI space changes weekly, they want an automated
pipeline that researches and surfaces what's new from Anthropic, the Claude community, and AI
experts, and publishes it as a versioned, citation-backed, shareable document — generated weekly
or on demand, and skipped (notification only) when nothing significant has changed.

## 2. Requirements

| # | Requirement | Status |
|---|-------------|--------|
| 1 | Artifact named `CCPlus_<YYYY-MM-DD_HHmm>` | ✅ |
| 2 | Anthropic/Claude documentation look & feel | ✅ |
| 3 | History log documenting updates from the prior version | ✅ |
| 4 | Proper citations + a References section at the bottom | ✅ |
| 5 | v1.0 covers all recommended default-config improvements to date | ✅ |
| 6 | Never overwrite — a new artifact per version | ✅ (timestamped files + Releases) |
| 7 | Accessible / shareable to others | ✅ (public GitHub Pages + Releases) |
| 8 | Weekly schedule **or** by request/trigger | ✅ GitHub Actions cron + `/ccplus` skill |
| 9 | No artifact if nothing significant — notify instead | ✅ issue comment + GitHub Watch email |
| 10 | Cross-model review (Opus ⇄ Sonnet) for accuracy | ✅ |

## 3. Active approach

- **Engine:** **GitHub Actions** (`.github/workflows/ccplus.yml`), credential `CLAUDE_CODE_OAUTH_TOKEN`
  set — uses Enterprise subscription, no PAYG billing. Claude Code on the web / Routines are
  disabled for this account; if enabled later, see `SETUP.md §A` to switch.
- **Schedule:** Mondays 05:00 UTC (= 13:00 Asia/Manila); on-demand via the `/ccplus` skill or
  Actions **Run workflow** button.
- **Format:** self-contained Anthropic-styled HTML.
- **Hosting / sharing:** public GitHub Pages from `docs/`, plus a GitHub Release per version.
- **Notifications:** a rolling **"📋 CCPlus run log"** GitHub issue receives one comment per run
  (both new-version and no-update). Repo watchers (Watch → All Activity) receive these by email.
  No Power Automate required.
- **Cross-model review:** Actions runs on Opus; spawns a Sonnet reviewer subagent to fact-check
  every claim before publish (roles swap if the base model is Sonnet).

## 4. Architecture

```
GitHub Actions cron (Mon 05:00 UTC)  ─┐
/ccplus (on demand)                   ├─►  generate-ccplus.md (Opus)
Actions "Run workflow"               ─┘         │
                                                ▼
        1. Gather   — WebFetch over config/sources.yaml → structured JSON
        2. Dedupe   — drop items already in state/state.json
        3. Significance — config/config.yaml rubric → generate OR notify-only
              │ notify-only → append runlog (no-update) → commit → STOP
              ▼ significant
        4. Generate — fill templates/template.html → docs/artifacts/CCPlus_<ts>.html
        5. Review   — Sonnet subagent (review-ccplus.md): pass / pass_with_edits / fail
        6. Publish  — update state + runlog, regenerate docs/index.html, commit & push,
                      create GitHub Release
                                                ▼
        Workflow posts issue comment on "📋 CCPlus run log" → GitHub Watch emails watchers
```

## 5. Repository layout

```
CCPlus/
  config/sources.yaml          # tracked sources (authoritative vs community)
  config/config.yaml           # significance rubric + model choices
  prompts/generate-ccplus.md   # the generator the pipeline runs
  prompts/review-ccplus.md     # cross-model reviewer instructions
  templates/template.html      # Anthropic-styled HTML shell ({{tokens}})
  state/state.json             # dedup ledger + version/changelog/hash tracking
  state/runlog.json            # per-run status log
  docs/index.html              # GitHub Pages landing page (all versions, newest first)
  docs/assets/                 # animated banner + pipeline diagram (README visuals)
  docs/artifacts/CCPlus_*.html # immutable artifact per version
  .githooks/pre-commit         # PII guard (committed, executable); reads .githooks/.blocked
  .githooks/.blocked           # blocked patterns (git-ignored, local only)
  .github/workflows/ccplus.yml # GitHub Actions workflow
  .claude/skills/ccplus/       # /ccplus on-demand trigger
  SETUP.md                     # setup & operations guide
  PROJECT_PLAN.md              # this file
```

## 6. Significance rubric (config/config.yaml)

**Generate** if any new item is: a Claude Code release changing features/defaults/flags/config
(not pure bugfix); new official Anthropic guidance on optimizing the default config; a new
default model or meaningful capability/limit change; or a community technique corroborated by
≥2 reputable sources that materially improves the default workflow.
**Notify-only** if items are merely cosmetic/typo/minor, single-source/unverified hype, or
already in the state ledger.

## 7. Build status

**Done**
- Repo scaffolded, pushed; GitHub Pages live (HTTP 200). Moved to **Claude-Booster** org.
- v1.0 artifact generated (Opus 4.8), reviewed (Sonnet 4.6 → `pass_with_edits`, two corrections
  applied), published to Pages and Release `v1.0`. 4 CI runs logged.
- GitHub Actions active — `CLAUDE_CODE_OAUTH_TOKEN` set as repo secret (subscription, no PAYG).
- Notifications: rolling GitHub issue ("📋 CCPlus run log"); watchers receive email per run.
- Animated banner + pipeline diagram added to README (`docs/assets/`).
- Pre-commit PII guard: `.githooks/pre-commit` (committed) + `.githooks/.blocked` (local, git-ignored).
- Git identity: `user.name=CCPlus`, `user.email=fredman08@users.noreply.github.com`.
- History: all commit identities and file content scrubbed of PII; force-pushed.

**Nothing remaining** — the pipeline is fully operational. Ongoing maintenance only:
- Tune `config/config.yaml` significance thresholds as needed.
- Add/update sources in `config/sources.yaml`.
- If Claude Code on the web is later enabled, switch to the native Routine (no API key).

## 8. Fallbacks (if GitHub Actions ever needs replacement)

- **Native Routine** — if Claude Code on the web / Routines are enabled for this account, switch
  to the Routine (no credential needed; uses subscription). See `SETUP.md §A`.
- **Local headless** — Task Scheduler / Power Automate Desktop runs `claude -p ...` weekly using
  the CLI login. No key required; machine must be on at run time.

## 9. Verification

- v1.0 renders with Anthropic styling; citations/References/History Log present; no stray
  template tokens; public Pages URL opens (HTTP 200).
- No-overwrite: each run writes a new timestamped file; `index.html` lists all; Releases archive
  each version.
- Notify path: a run with the state ledger already current yields `no-update` in `runlog.json`
  and a comment on the run-log issue (GitHub Watch emails watchers).
- Cross-model review: reviewer runs on the opposite model; verdict recorded in the artifact and
  `runlog.json`.

## 10. Maintenance

Tune `config/config.yaml` significance thresholds after the first runs; add sources in
`config/sources.yaml`; adjust shared styling in `templates/template.html`.
