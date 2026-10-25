---
name: setup
description: Wizard that creates or updates the repo's .claude/shipwright.md config for the /ticket workflow, and reviews the Step rules Shipwright has learned. Detects everything it can from the repo, asks only what it cannot detect with the detected answer pre-selected, previews the file and writes it on approval. Use when the user runs /shipwright:setup, or when /shipwright:ticket finds no config.
argument-hint: "[--local]"
disable-model-invocation: true
---

# Shipwright setup wizard

The user invoked this with: `$ARGUMENTS`

Goal: a correct `.claude/shipwright.md` with the fewest possible questions. **Detect first, ask second, write last.** Nothing is written until the user approves the preview.

## 1. Detect

Run these from the repo root, in as few shell calls as possible. Each finding gets a confidence: **sure** (read directly from a file) or **guess** (inferred from history or naming).

| Field | How to detect |
| --- | --- |
| Package manager | Lockfile: `yarn.lock`, `pnpm-lock.yaml`, `package-lock.json`, `bun.lockb`, `poetry.lock`, `Gemfile.lock`, `go.mod`, `Cargo.toml`. Yarn version from `packageManager` in package.json; PnP if `.pnp.cjs` exists. Monorepo workspaces from package.json `workspaces` or `pnpm-workspace.yaml`. |
| Commands | Scripts in package.json (root and the main workspace): dev / start, test, lint, type-check / typecheck / tsc, format. Makefile targets. Prefer per-file forms for lint and test where the tool supports it. |
| Dev server URL | Port from the dev script, vite / next / webpack config, or `.env*` (`PORT=`). Default per framework (Vite 5173, Next 3000, CRA 3000). |
| Default branch | `git symbolic-ref refs/remotes/origin/HEAD` |
| Release branches | `git branch -r` grouped by prefix (`release-*`, `rc-*`, `release/*`, `develop`). If one prefix has several version-numbered branches, propose "latest `<prefix>*` by version". |
| Branch naming | Last ~50 branch names (`git for-each-ref --sort=-committerdate refs/remotes --count=50 --format='%(refname:short)'`): ticket key only, `type/KEY-slug`, or free. |
| Ticket key pattern | Keys like `[A-Z][A-Z0-9]+-\d+` in branch names and commit subjects; take the most common project prefix. |
| Commit format | Last ~50 commit subjects on the default branch (`git log --no-merges -50 --format=%s`): `KEY: summary`, Conventional Commits (`feat(scope): ...`), or free. Note whether bodies or `Co-Authored-By` trailers appear. |
| PR template | `.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE/`, `docs/pull_request_template.md`. |
| Tracker | Connected tools (Jira / Linear MCP, `gh`), plus ticket-site URLs in recent PR bodies or commits (`gh pr list --state merged -L 10 --json body`). |
| Browser tool | Connected browser automation tools (Claude in Chrome, Playwright, Chrome DevTools MCP). |
| Environments | URLs in README, `.env.example`, deploy configs and recent PR bodies that look like dev / staging hosts. Always a guess; list them for confirmation. |
| Existing rules | `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`: lift workflow rules that already exist (commit rules, forbidden commands) rather than asking for them again. |

If `.claude/shipwright.md` already exists, read it: this is an **update**. Keep every value the user set, every house rule and every Step rule, and only fill gaps or change what the user asks to change.

**Review rules.** On an update, if the config has Step rules (in either file), offer one extra option: "Review saved rules". Then list them grouped by step with their file (team / just me), flag duplicates, contradictions and rules that mention things no longer in the repo (scripts, branches, paths), and let the user keep, edit, move between team and personal, or delete each. Show the resulting diff before writing.

## 2. Ask

Use the AskUserQuestion tool if available (otherwise ask in chat as numbered multiple choice). **At most one round of up to four questions.** Only ask about fields that are a **guess** or undetected; never ask what you read for sure. Put the detected answer first, labelled "(Detected)". The user can always pick "Other".

Pick the four most uncertain from:
- **Base branch for new work**: latest `<prefix>*` (detected) / default branch / `develop` / `main`.
- **Commit format**: detected pattern with a real example from history / `KEY: summary` / Conventional Commits / free text.
- **AI attribution in commits and PRs**: none / allowed. (Detected from trailers in history.)
- **Screenshots in PRs for UI changes**: before/after table / Figma vs screenshot table / none.
- **Where to reproduce**: each detected environment as an option / localhost only.
- **Ticket comment after the PR**: draft for approval / skip.
- **Share with the team?**: commit `.claude/shipwright.md` (Recommended) / personal `.claude/shipwright.local.md` (gitignored). Skip this question if `$ARGUMENTS` has `--local`.

Then one free-text prompt, which the user may skip: **"Anything a new teammate usually gets wrong here?"** (forbidden commands, formatting rules, words to avoid). These become House rules.

## 3. Preview

Fill `${CLAUDE_PLUGIN_ROOT}/templates/shipwright.md` with the answers. Remove sections and placeholder lines that do not apply; no example values may survive. Always keep the `## Step rules` section with its comment (empty step headings may go), so learned rules have a home. Mark values that are still guesses with `<!-- guess: confirm -->` so they are easy to find later.

Show the complete file in a code block, with a short line above it: what was detected, what came from answers, how many guesses remain. Ask: **Write it, change something, or cancel?**

## 4. Write

- Write to `.claude/shipwright.md`, or `.claude/shipwright.local.md` for personal configs. For a local file, add `.claude/shipwright.local.md` to `.gitignore` if it isn't covered, and say so.
- Never commit. Tell the user the file is ready to commit if they chose to share it.
- If called from `/shipwright:ticket` Step 0, hand back to Step 0's check-in instead of ending.

Final message: file path, number of guesses left, and how to re-run (`/shipwright:setup`) to update it.

## Never

- Never put secrets, tokens or passwords in the config, even if found in `.env` files. Hostnames only.
- Never overwrite an existing config without showing the diff of what changes.
- Never ask more than one round of questions. Unknowns become guesses marked for later.
