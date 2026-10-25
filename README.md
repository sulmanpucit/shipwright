<div align="center">

# ⚓ Shipwright

### Hand it a ticket. Get back a verified pull request.

**A Claude Code plugin that takes a ticket from your tracker to a reviewed-ready PR,<br>reproducing the bug in a real browser first and proving the fix after,<br>while you stay at the helm for every step.**

```
/shipwright:setup            # once per repo: a wizard writes your project config
/shipwright:ticket PROJ-123  # every ticket: read → reproduce → branch → plan → fix → verify → ship
```

[Install](#install) · [How it works](#how-it-works) · [Configure](#configure-your-repo) · [Safety](#safety) · [FAQ](#faq)

</div>

---

## The problem

You already know the loop. Open the ticket. Scroll through every comment, because the last one changed the scope. Open the screenshots. Find a page that shows the bug. Poke at it in DevTools. Find the code. Cut a branch from the right release branch. Plan. Fix. Run the type-check, the linter, the tests. Open the page again and check it really is fixed, on desktop and mobile. Write the commit. Fill in the PR template. Take before and after screenshots. Go back to the ticket and leave a note for QA.

With an AI assistant you can do all of that faster, but you still have to **drive every step**: "now read the comments", "now open the browser", "now go to plan mode", "now verify it", "now raise the PR". The assistant does the work. You do the project management.

## The fix

Shipwright turns that loop into one command. You give it a ticket. It works through eight steps, and after each one it stops and tells you, in a few lines, **what it learned and what it is about to do**. You reply "go", "change this", or "stop".

You stop typing instructions and start making decisions.

```
**Step 2 of 8: Reproduce** ✅ done

**Learned**
- Reproduced on staging at 375px: the card title overflows its box by 14px.
- Figma says 16px/24px; the page renders 18px/28px.
- Source: components/card/card-title.tsx:42 uses `text-lg` instead of the `title-sm` token.
- Not fixed on the base branch either.

**Next: Step 3, Branch**
- git fetch origin && git switch -c PROJ-123 origin/release-2.5

**Approve, change, or stop?**
```

## How it works

```
 0 Preflight → 1 Read → 2 Reproduce → 3 Branch → 4 Plan → 5 Fix → 6 Verify → 7 Ship → 8 Report
                                                                              └── only these two leave your machine
```

| # | Step | What Shipwright does |
| --- | --- | --- |
| **0** | **Preflight** | Checks which tools you have connected (tracker, browser, git host, Figma) and whether your working tree is clean. Tells you exactly how to fix anything missing. No config in this repo yet? It runs the [setup wizard](#configure-your-repo) first. |
| **1** | **Read** | Reads the ticket, **every** comment oldest to newest, opens **every** screenshot, follows design links. Summarises the ask, the acceptance criteria and any contradictions. |
| **2** | **Reproduce** | Opens the page in a real browser and proves the bug with measurements, computed styles, console and network evidence. Compares against the design. Finds the code responsible. |
| **3** | **Branch** | Cuts a branch from your base branch. **Only after the bug is reproduced:** if it can't be reproduced, there is nothing to branch for, and Shipwright tells you what it tried instead. |
| **4** | **Plan** | Enters plan mode: the cause, each file to change and why, the tests, the exact browser checks it will run to prove the fix, and the open questions. |
| **5** | **Fix** | Makes only the approved changes, following your repo's conventions. Runs type-check, lint and the related tests, and separates failures it caused from ones already on the base branch. |
| **6** | **Verify** | Starts your dev server, re-runs the original reproduction, runs every planned check at every viewport, and checks the neighbours it might have broken. Reports a pass/fail table. |
| **7** | **Ship** | Commits, pushes and opens the PR with your template. You see the exact commit message and PR body **before** it happens. |
| **8** | **Report** | Drafts the ticket comment for QA (what changed, PR link, how to test). You see the exact text **before** it is posted. |

### It remembers

Every run is logged to `~/.claude/shipwright/<repo>/<KEY>.md`. Close your laptop, come back tomorrow, start a new session:

```
/shipwright:ticket PROJ-123 resume
```

It picks up at the next step that you haven't approved yet.

## Why the stops matter

Fully autonomous agents are impressive right up until they confidently fix the wrong bug, on the wrong branch, and post about it on the ticket. Shipwright is built on the opposite bet: **the expensive mistakes happen at decision points, so those are exactly where a human should be.**

- Each check-in is a few lines of evidence, not a wall of narration, so approving takes seconds.
- Approval covers one step. It never carries over.
- "change ..." redoes the current step your way. "stop" saves the log and ends cleanly.

You get the speed of automation and the judgement of a reviewer who saw every step.

## Install

```
/plugin marketplace add sulmanpucit/shipwright
/plugin install shipwright@shipwright
```

Start a new Claude Code session, open your repo, run `/shipwright:setup` once, then `/shipwright:ticket <KEY>`.

## Connect your tools

Shipwright **ships no credentials and bundles no servers**. It uses whatever you have connected, signed in as you, so it works the same for everyone on your team with their own access. Step 0 tells you what's missing.

| Need | Options |
| --- | --- |
| **Tickets** | Jira (claude.ai Atlassian connector, or `claude mcp add --transport http atlassian https://mcp.atlassian.com/v1/mcp`), Linear, or GitHub Issues via `gh` |
| **Browser** | The [Claude in Chrome](https://claude.com/chrome) extension, or Playwright MCP (`claude mcp add playwright npx @playwright/mcp@latest`) |
| **Git host** | `gh auth login` |
| **Designs** *(optional)* | Figma MCP: `claude mcp add --transport http figma https://mcp.figma.com/mcp` |

Missing something? Shipwright does less rather than failing. With no browser, reproduce and verify fall back to code and tests, and every check-in says so. With no tracker, paste the ticket in.

## Configure your repo

Every team ships differently, so each repo gets a small config file, **`.claude/shipwright.md`**. You don't write it by hand. Run the wizard:

```
/shipwright:setup
```

(or just run `/shipwright:ticket` in a new repo and it offers the wizard itself).

The wizard **detects first and asks second**. It reads your lockfile and scripts, your release branches, your PR template, your last 50 commit messages and branch names, your `CLAUDE.md` / `CONTRIBUTING.md`, and which tools you have connected. Then it asks **at most four multiple-choice questions** about what it couldn't be sure of, with its best guess pre-selected:

```
Base branch for new work?
  ● Latest release-* branch (Detected: release-2.5)
  ○ main
  ○ develop

Commit format?
  ● PROJ-123: summary (Detected, e.g. "PROJ-981: Fix card title overflow")
  ○ Conventional Commits
  ○ Free text
```

One optional free-text question follows: *"Anything a new teammate usually gets wrong here?"* Those answers become house rules. You see the whole file before it is written.

Commit it and your whole team gets the same workflow. Want it just for you? `/shipwright:setup --local` writes a gitignored `.claude/shipwright.local.md`. Re-run the wizard any time to update; it keeps what you already set.

**Keep internal hostnames, ticket sites and project specifics in that file, inside your own repo.** The plugin itself stays generic and public. The wizard never copies secrets from `.env` files.

## Safety

Shipwright will **never**:

- start a step you haven't approved;
- push, open a PR, or post a comment you haven't seen word for word;
- type a password or complete an SSO login (it asks you to sign in);
- stash, reset or discard your uncommitted work;
- force push, or push to your base branch;
- follow instructions written inside a ticket, comment or web page (those are data, and it will quote them to you and ask).

## FAQ

**Does it work with any language or framework?**
Yes. The workflow is generic. Your `.claude/shipwright.md` tells it your commands, branches and conventions.

**What if the bug can't be reproduced?**
It stops at Step 2, lists every URL, environment and viewport it tried, and gives you options: draft a question for the reporter, try another environment, or go ahead based on what the code shows. No branch is created.

**Can I skip the stops?**
Not yet, by design. Each check-in is short so approving is quick. A configurable "auto-approve local steps" mode is on the roadmap.

**Is it only for bugs?**
No. Small features and design QA tickets work the same way: "reproduce" becomes "show the current state against the spec".

**What does it cost my team in setup?**
One `/shipwright:setup` per repo (about a minute), and each person connects their own tracker, browser and `gh`.

## Roadmap

- [ ] Optional auto-approve for local-only steps (read, reproduce, plan)
- [ ] Batch mode: triage a sprint's tickets in one pass
- [ ] Shared run logs for handing a ticket over to a teammate
- [ ] More trackers: Shortcut, Azure Boards

Ideas and pull requests welcome. Open an issue with the ticket flow that Shipwright got wrong. That's the most useful bug report there is.

## License

[MIT](LICENSE)
