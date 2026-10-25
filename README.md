<div align="center">

# ⚓ Shipwright

### Hand it a ticket. Get back a verified pull request.

**A Claude Code plugin that takes a ticket from your tracker to a reviewed-ready PR,<br>reproducing the bug in a real browser first and proving the fix after,<br>while you stay at the helm for every step.**

```
/shipwright:ticket PROJ-123
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
| **0** | **Preflight** | Checks which tools you have connected (tracker, browser, git host, Figma) and whether your working tree is clean. Tells you exactly how to fix anything missing. |
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

Start a new Claude Code session, open your repo, and run `/shipwright:ticket <KEY>`.

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

Every team ships differently. The first run in a repo offers to create **`.claude/shipwright.md`** from [the template](templates/shipwright.md), pre-filled with what it can detect:

```markdown
## Branching
- Base branch rule: the latest `release-*` branch on origin
- Branch name: `<KEY>`

## Commands
- Dev server: `npm run dev` (http://localhost:3000)
- Type-check: `npm run type-check`

## Commit
- Format: `<KEY>: <summary>` (one line)

## House rules
- Never run prettier --write on ts files; use eslint --fix.
```

Commit it and your whole team gets the same workflow. Personal preferences (your browser, extra pauses) go in `.claude/shipwright.local.md`, which you gitignore.

**Keep internal hostnames, ticket sites and project specifics in that file, inside your own repo.** The plugin itself stays generic and public.

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
One config file per repo, and each person connects their own tracker, browser and `gh`.

## Roadmap

- [ ] Optional auto-approve for local-only steps (read, reproduce, plan)
- [ ] Batch mode: triage a sprint's tickets in one pass
- [ ] Shared run logs for handing a ticket over to a teammate
- [ ] More trackers: Shortcut, Azure Boards

Ideas and pull requests welcome. Open an issue with the ticket flow that Shipwright got wrong. That's the most useful bug report there is.

## License

[MIT](LICENSE)
