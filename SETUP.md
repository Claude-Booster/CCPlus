# CCPlus — Setup & Operations

This guide covers CCPlus operations: **(A)** the weekly Routine (if/when enabled), **(B)** email
notifications via GitHub Watch, **(C)** on-demand triggers, and **(D)** the pre-commit PII guard.

> **Prerequisite — Claude Code on the web (hard gate for the Routine).** Routines require
> "Claude Code on the web" to be enabled for your account. This is an Anthropic org-side setting
> that an admin requests from your Anthropic account team — it can't be toggled by a user or
> automated. Check at <https://claude.ai/code>; if it says *"Not available for the selected
> organization,"* it's disabled — request enablement, or use the **Fallback** below.

---

## A. Weekly Routine (cloud, native)

1. Go to <https://claude.ai/code/routines> and create a new routine.
2. **Connect this repo** (`Claude-Booster/CCPlus`) via the GitHub connector (authorize once).
3. **Schedule:** weekly, cron `0 13 * * 1` (Mondays 13:00). Set the timezone to **Asia/Manila**
   (or your local zone). If the UI only accepts UTC, use `0 5 * * 1` for Mon 13:00 PHT.
4. **Model:** Opus. The reviewer subagent runs on Sonnet automatically (see
   `prompts/generate-ccplus.md` → Step 5).
5. **Network access:** Trusted (or allow the source domains in `config/sources.yaml`).
6. **Branch pushes:** enable **"Allow unrestricted branch pushes"** so the routine can publish to
   `main` (otherwise it pushes to a `claude/` branch and you'll merge each run).
7. **Prompt / instructions:** point the routine at the repo and use:
   > *Follow `prompts/generate-ccplus.md` exactly, using `config/config.yaml` and
   > `config/sources.yaml`. Generate a new CCPlus version only if the significance rubric is met;
   > otherwise record a `no-update` run. Commit and push, and create a GitHub Release on generate.*
8. Click **Run once now** to confirm an end-to-end run works.

The routine draws on your Enterprise **subscription usage** (no API key), subject to a daily
routine-run cap. View run history on the routines page.

---

## B. Email notifications (GitHub Watch — no Power Automate / no premium)

The workflow posts one comment per run to a single rolling **"📋 CCPlus run log"** issue (it
creates the issue automatically on the first run). GitHub emails repo watchers on issue
comments, so you get an email for **every** run — both new-version and "no significant updates".
New versions additionally create a GitHub **Release** (also emailed). This needs no Power
Automate and no premium connector.

### B1. Turn on watching (one-time)

1. Open <https://github.com/Claude-Booster/CCPlus>.
2. Top-right **Watch** → **All Activity**. (This subscribes you to issues, comments, and releases.)
3. Confirm GitHub emails go where you want: <https://github.com/settings/notifications> →
   **Watching** → enable **Email**; and <https://github.com/settings/emails> → make sure the
   address you check is your primary/verified GitHub email.

### B2. Test

Actions tab → **CCPlus weekly** → **Run workflow**. Within a minute you should get an email for a
new comment on the run-log issue ("no significant updates" if nothing changed). To test the
"new version" path, run it with the **force** input checked (also creates a Release email).

> Notes: GitHub does **not** email you for activity *you* triggered, so a manual run you start
> yourself may not notify you — but the **scheduled** Monday runs will. Plain commits/pushes do
> not generate emails, which is why notifications go through the run-log **issue** (and Releases),
> not the commit. The old Power Automate `PA_WEBHOOK_URL` secret is no longer used and can be
> deleted: `gh secret delete PA_WEBHOOK_URL --repo Claude-Booster/CCPlus`.

---

## C. On-demand ("by request")

- **In any Claude Code session** (VSCode or CLI) opened on this repo: run **`/ccplus`** (force a
  build with `/ccplus force`). See `.claude/skills/ccplus/SKILL.md`.
- **GitHub Actions:** Actions tab → **CCPlus weekly** → **Run workflow** (check **force** to build
  even with no significant updates).

---

> **Note:** As of June 2026, Claude Code on the web is **disabled** for this account, so the
> Routine (section A) is not available yet. The active engine is **GitHub Actions** below. If
> web access is later enabled, switch to the Routine (it needs no API key).

---

## A2. GitHub Actions (active engine while Routines are unavailable)

The workflow is committed at `.github/workflows/ccplus.yml`. It runs weekly (Mon 05:00 UTC =
13:00 Asia/Manila) and via the **Run workflow** button (on-demand). **Active** — `CLAUDE_CODE_OAUTH_TOKEN`
is set as a repo secret (uses subscription, no PAYG billing).

### Activating

Add **one** credential as a repo secret at
`https://github.com/Claude-Booster/CCPlus/settings/secrets/actions`:

| Option | Secret | Notes |
|--------|--------|-------|
| **A. PAYG key** | `ANTHROPIC_API_KEY` | A Console pay-as-you-go key from console.anthropic.com. **Not** subscription credentials. ~$0.50–$2 per run. |
| **B. OAuth token** | `CLAUDE_CODE_OAUTH_TOKEN` | Generated via `claude setup-token`; uses a Claude subscription instead of PAYG. Availability depends on your Enterprise plan — confirm with your admin. |
| **C. Bedrock/Vertex** | (cloud creds) | If your organization already has Claude via AWS Bedrock / Google Vertex, authenticate the action with cloud credentials instead of an Anthropic key. Requires editing the workflow per the action's provider docs. |

Then: **Actions tab → CCPlus weekly → Run workflow** to test. New artifacts publish to GitHub
Pages automatically (push to `main` rebuilds Pages) and a GitHub Release is created.

> Verify the `anthropics/claude-code-action` version and input names against its current docs
> before enabling — the action evolves frequently.

---

## Other fallbacks

**Local headless** (no API key — uses your CLI login; PC must be on at run time):

1. Install the standalone CLI: `npm install -g @anthropic-ai/claude-code` (you currently have
   only the VS Code extension), then sign in once with `claude` and confirm `claude --version`.
2. Schedule weekly (Mon 13:00) via **Task Scheduler** / **Power Automate Desktop**, running from
   the repo directory:
   ```powershell
   claude -p "Follow prompts/generate-ccplus.md ..." --permission-mode acceptEdits
   ```
3. Keep the same repo and Pages; notifications come from GitHub Watch (no PA needed).

**Manual on-demand only** (zero infrastructure): run **`/ccplus`** in VS Code whenever you want
(e.g. a weekly calendar reminder). No CLI install, no key, no machine-on requirement.

---

---

## D. Pre-commit PII guard

Three hooks are committed (executable, mode `100755`):

| Hook | File | What it guards |
|------|------|----------------|
| `pre-commit` | `.githooks/pre-commit` | Staged file content · author identity · committer identity · env vars · OS fallback |
| `commit-msg` | `.githooks/commit-msg` | Commit message text |
| `pre-push` | `.githooks/pre-push` | Annotated tag message + tagger identity (before the tag reaches the remote) |

All three read patterns from `.githooks/.blocked`, which is **git-ignored and never committed** — you must create it locally.

### D1. Activate (one-time per clone)

```bash
# Tell git to use the .githooks/ directory
git config core.hooksPath .githooks

# Set safe commit identity (avoids OS fallback leaking your system username)
git config user.name  "CCPlus"
git config user.email "fredman08@users.noreply.github.com"

# Create the local patterns file — add your own restricted strings, one per line.
# Lines starting with # are ignored. Patterns use grep -iE syntax (case-insensitive).
# Example structure (replace with your actual restricted content):
#
#   your name variant 1
#   your name variant 2
#   your\.email\.localpart
#   your_handle
#   yourorg\.com
#   YourOrg
#
touch .githooks/.blocked   # or open in an editor and add your patterns
```

### D2. What each hook guards

**pre-commit** runs before every commit:
- Scans only the **added lines** of each staged file (`git diff --cached`).
- Checks **author and committer identity** (`git var GIT_AUTHOR_IDENT` and `GIT_COMMITTER_IDENT`) — this resolves env vars → local config → global → system → OS fallback in order, so a blocked name in `%USERNAME%` or `~/.gitconfig` is caught before the commit is written.
- Skips itself and the patterns file.

**commit-msg** runs after you type/save the commit message:
- Scans the full commit message text for blocked patterns.
- Catches PII typed into commit messages (`git commit -m "..."` or editor).

**`git tag -a` coverage** — git has no pre-tag hook, so a tag can be created locally with PII before any hook fires. The `pre-push` hook closes this: it inspects every annotated tag object (tagger identity + message) before it reaches the remote. PII in a tag message is caught at push time and the push is blocked.

**Common to both hooks:**
- If `.githooks/.blocked` is absent, both hooks exit silently — no false blocks on a fresh clone before the file is created.
- All pattern matching is case-insensitive.

---

## Maintenance

- **Tune significance** in `config/config.yaml` after the first few runs (too noisy → raise
  thresholds; missing things → lower them).
- **Add sources** in `config/sources.yaml`.
- **Adjust styling** in `templates/template.html` (used for every new version).
