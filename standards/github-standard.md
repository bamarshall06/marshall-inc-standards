# The GitHub standard — repos, milestones and issues, and how work lands

> **Public edition.** A working standard from Marshall Inc, shared as it is used. Real names, paths and accounts are replaced with generic ones (floors such as *Business* and *Teaching*, `/Users/<you>`, `<your-github-user>`). References like `gm-district #372` point to private repositories and are kept only as history. Start with the [README](../README.md).

## Summary
> How a hall's work lands on GitHub: three repos per hall, every piece of work an issue in a milestone, and a commit shipped from one of many sessions on one volume with `bin/session-files` and `bin/ship` — never `git add -A` — through a pull request merged when its checks pass. Worktrees, labels, CI and branch protection.

**Status:** org-wide standard, written 2026-11-19 (gm-district #371, milestone #24). It folds in
three earlier standards, which are in git history: `repo-standard.md` (2026-11-19, #141),
`tracking-system.md` (2026-09-20) and `multi-session-ship.md` (2026-09-20). Their lessons are
kept here wherever they still hold. **Applies to:** every hall. **Teaching** keeps its
own tracking and shipping methods (§7), but not its own repo rules.

Entry point: [`sop.md`](sop.md), whose ten steps this document carries in depth. Every
remote is GitHub (`origin`, under `<your-github-user>`), read and written with `gh`, which holds the
credential: no token is read, stored or passed by anything here.

| | |
|---|---|
| §1 | Three repos per hall |
| §2 | Milestones and issues |
| §3 | Labels and the board |
| §4 | Where work happens: the main checkout, branches and worktrees |
| §5 | Shipping: `bin/session-files` and `bin/ship` |
| §6 | CI and branch protection |
| §7 | Who this does not bend for |
| §8 | Traps, and measuring it |

---

## 1. Three repos per hall

Bryan's rulings of 2026-11-14, -16 and -17 (the brain note `three-repo-rule`). Every hall has
**three kinds of repo**, each its own private GitHub repository under `<your-github-user>`:

| Repo | Named | Holds |
|---|---|---|
| **District** | `<prefix>-district` | The hall itself: `CLAUDE.md`, `.claude/`, `bin/`, `docs/`, `rules/`, `references/`, `staff.json`. Its machinery, never a product. |
| **Brain** | `<prefix>-brain` | What the hall wrote: the notes in `brain/` ([`brain-standard.md`](brain-standard.md)). Never received documents, never code. |
| **App** | `<prefix>-<app>`, one per app | One site, web app or tool, with its own deploy. **Two apps never share a repo, and an app never lives inside the district repo.** |

A hall that builds nothing has two repos, district and brain. That is complete, not a gap.
**Why three:** knowledge, machinery and each product have different lifecycles, deploy paths
and audiences. Folding them together is what made Business's district the app repo.

**The prefix** is the short name the hall's repos already carry, not always the district code.
The names are data, in the GM's `districts.json` → each district's `repos` and
`general_manager.repos`; nothing else writes them down. For reading:

| `gm` | `biz` | `money` | `fin` | `teach` | `proj` | `family` | `sec` |
|---|---|---|---|---|---|---|---|
| `gm` | `biz` | `diy-money` | `mf` | `teach` | `jcxc` | `fam` | `security` |

**The layout on the volume** ("Option A", Bryan 2026-11-16):

```
/Volumes/<VOLUME>/              the <prefix>-district checkout: the volume root IS the district repo
/Volumes/<VOLUME>/brain/        the <prefix>-brain checkout, ignored by the district repo
/Volumes/<VOLUME>/apps/<app>/   each app's checkout, ignored by the district repo
```

- **Every nested checkout is ignored by the root** (`/brain/`, `/apps/`), so no repo holds
  another. Commit and push each from its own folder: `git -C brain …`, `git -C apps/<app> …`.
- **An app's skills belong to the district repo**, at the root's `.claude/skills/`, named
  `<app>-<skill>` when the plain name would collide ([`doe-framework.md`](doe-framework.md)).
- **Shared trees are not repos.** `documents/`, `messages/` and `intake/` are links into the hall's cloud; `data/` and
  `var/` are ignored. None is ever committed to any of the three.

**Subdistricts** (Bryan, 2026-11-21, gm-district #459). A hall may have **rooms**: one mandate and
one agent each, at `<hall root>/rooms/<id>/`, declared in `districts.json` → `<hall>.residents.<id>`.
**Each room is a subdistrict, with its own brain and its own district repo:**

| Repo | Named | Checked out at | Holds |
|---|---|---|---|
| **District** | `<prefix>-<id>-district` | `rooms/<id>/` | the room's `CLAUDE.md` and `.claude/`: its machinery |
| **Brain** | `<prefix>-<id>-brain` | `rooms/<id>/brain/` | the room's notes, to [`brain-standard.md`](brain-standard.md) |

- **A room is a small hall** (Bryan, 2026-11-22, gm-district #547, milestone 38). It has its own
  `documents/`, `messages/` and `intake/`, links into the hall's cloud at `<cloud root>/DISTRICTS/<VOLUME>/ROOMS/<id>/`
  (`documents`, `messages`, `intake`), which the room's district repo ignores as `/documents`, `/messages` and `/intake`. A
  message or a file reaches it addressed `<hall>/<id>`
  ([`communication-standard.md`](communication-standard.md) §2). Every rule for a room is
  [`room-standard.md`](room-standard.md).
- **Everything else is the hall's.** A subdistrict lives on its hall's volume and shares its vault
  and apps: an app stays `<prefix>-<app>` at `apps/<app>/`. The room's `CLAUDE.md`
  follows the hall's, and nothing in it relaxes the hall's rules.
- **No repo holds another.** The hall's root ignores `/rooms/`, as it ignores `/brain/` and
  `/apps/`, and each room's district repo ignores its own `/brain/`.
- **One person, one note.** A note lives in the room's brain or the hall's, never both; what is the
  room's moves into its brain and out of the hall's.
- **The names are data**, in `districts.json` → `<hall>.residents.<id>.repos` (`district`,
  `brain`). The GM names them; the hall creates them, by notice.

**Changing a hall's repos.** A new app gets `<prefix>-<app>` at `apps/<app>/` and a line in
`districts.json` → `repos.apps`. A rename is GitHub's (it redirects), then `git remote set-url`
in every checkout, then `districts.json` — after checking every deploy workflow and runner that
names the repo, because a rename can stop a live deploy. Moving a checkout on disk needs
`rekey-claude <old> <new> --commit`.

---

## 2. Milestones and issues

**Nothing is built without an issue.** Every piece of work is an issue, every issue is in a
milestone, and both are written with the **district-planner** skill, linked into every hall:
`plan.py context` first, every write as a dry run, then `--commit`.

**Both are written in plain English, for Bryan first.** He reads every one; an agent builds
from them. `plan.py` checks a new one against the skill's `references/writing.json` and refuses
it until it passes. An existing one is rewritten to this shape only when Bryan names it.

**A milestone is an outcome**, named for what will be true ("Every inbox lives in Dropbox"),
and its description has three parts: **Why this matters** (the problem today, and who it
hurts), **What we're trying to do** (one sentence), and **Done when** (three to six checkable
bullets, in plain words). A due date only when there is a real one. An issue holds **one
milestone only**, so a sprint can never be a milestone; work flows continuously against a WIP
limit instead (§3).

**An issue is one user story, in two parts** with a `---` rule between them:

- **Part 1, for Bryan:** *The story* (**As** … **I want** … **so that** …), *Why this matters*,
  and *Done when* as checkable results. No code, paths or bare issue numbers here.
- **Part 2, for the agent:** *Technical spec (for the agent)*, with **Where**, **Approach**,
  **Constraints** and **How to check**, then the **`acceptance:` line naming a command** that
  proves it, last: the line a reviewer or the software factory can run. An issue with
  no command to run is a wish.

**Issues live with the code they change:** the hall's own work on `<prefix>-district`, an app's
on `<prefix>-<app>`. A hall's tools read every repo it owns, not only its district repo —
reading only the district repo once hid 181 of Rentals's 187 open issues. A hall may keep its
tracker on one repo when Bryan has ruled so (Business's stays on `sf-listing-mirror`,
2026-11-17), recorded in its `districts.json` notes.

**Nothing is deleted.** A close always leaves a comment saying why (`completed`,
`not_planned`, `duplicate`), and is reversible. Work set aside is **parked**, not closed:
labelled `state/future`, as the GM parked 99 issues on 2026-11-19 when its plan started over.

**Plans become issues, and the executor decides the shape:**

- **A person or a session executes it → one issue**, the steps as a checklist in its body. A
  plan's steps are sequential; separate cards would show parallel work that does not exist.
- **The software factory builds it → a milestone of specs**, each spec filed as one parent
  issue with a sub-issue per task and real blocked-by links, every task tiered and naming its
  acceptance command ([`software-factory.md`](software-factory.md) §4).

**`type/task` is not engineering work** and does not belong on an app repo. "Decide Vacation
for 2027" is something a person does; it goes on the district repo.

**The done check is advisory.** `scripts/done_check.py --sweep` (money's) finds open issues
whose number appears in code on a line claiming the work landed. It never closes an issue and
never fails a ship; narrowed, it flagged 1 of 42 and that one was real, where the loose version
flagged 23.

---

## 3. Labels and the board

**The labels are data**: `references/labels.json` in each hall, created on every repo from that
file and never by hand. Hand copies are how the halls drifted into five shapes.

| Namespace | Values |
|---|---|
| `type/` | `bug`, `feature`, `chore`, `task`, `test`, `epic` |
| `state/` | `future`, `backlog`, `next`, `doing`, `review` |
| `priority/` | `critical`, `major`, `normal`, `minor` |
| `area/` | the hall's own |
| `source/` | `obligation`, and whatever else files work without a person (Business's) |
| `needs/` | `bryan` — waiting on a decision only Bryan can make |

**One state per issue, enforced by the tools.** GitHub has no exclusive label scopes, so
nothing on GitHub stops an issue carrying two `state/` labels. `bin/board` and `bin/ship`
enforce it: a move replaces every `state/*` label with the one target, and the board flags an
issue holding two. **Closing marks it done**; there is no `state/done`. The states and the
**WIP limit of 3** are data in `references/workflow.json` (Security's is the reference,
security-district#29); `bin/board` refuses a fourth `doing` and names what is blocking.

**The board is labels, not a GitHub Projects board,** so the tools read and move it with `gh`
and nothing lives only in a web UI.

**`today` is not a state** (corrected 2026-09-21). A flag that crosses states never belongs in
`state/`; it is its own namespace (`focus/today`). The phone mirror it once fed is retired.

**A label that is not in the hall's `labels.json` is drift** — GitHub's nine default labels
(`bug`, `enhancement`, `question` …) included. Delete it, or add it to the file.

**A move between repos is a transfer, and labels do not travel.** `gh issue transfer` drops any
label the destination lacks: create them there first, or it is a retype, not a move. Measure
cross-references before moving (money broke 0). Count label **definitions**, not applications:
SF's "36 unscoped labels" were four definitions applied 36 times, and four renames preserved
every one where delete-and-recreate would have stripped them all, silently.

**The board in an app is optional** — for a hall whose application has an internal or admin
audience that wants to see the work; a public site has none, and needs no board (xc's
correction, 2026-11-19). Where a hall builds one: four pages — Current
Features, Feature Requests, Current Bugs, Bug Fixes — mirrored **one way**: the app never writes
to the tracker, and the production box holds no GitHub credential, so a script on the Mac
pushes a full snapshot to a localhost-only endpoint over SSH. **Current Features is curated**
(`var/features.json`), each entry citing the issues that built it; one citing none renders
"unlinked". Every tool that changes issue state pushes the board — never the deploy, which once
refreshed nothing for six ships — and a nightly Action catches closes made in the web UI.

---

## 4. Where work happens: the main checkout, branches and worktrees

**One checkout is one HEAD and one staging area.** A hall runs a District Manager and two
workers, and the GM adds its residents; several share each volume, and summoned roles and
the factory's laps write into a checkout while a session works. A
`git add -A` in any of them commits the others' half-finished work. Everything in §4 and §5
exists to make "stage only your own work" something a tool computes, not something a model
remembers.

**The main checkout** (the volume root) **stays on the default branch.** Marshall Inc
fast-forwards it within five minutes of a merge, but only while it sits there. Nobody commits,
stashes or discards another session's uncommitted change in it. The shared trees (`brain/`,
`data/`, `documents/`) are written here and only here.

| Who | Works in | Branch |
|---|---|---|
| District Manager, a small change | the main checkout | a short branch from `origin/<default>`, then back |
| District Manager, a milestone | `.claude/worktrees/m<N>-<slug>` | `m<N>-<slug>` |
| A worker | `.claude/worktrees/squad-<N>` (detached from `origin/<default>`) | `issue-<number>-<a-few-words>` per issue of a milestone, `<a-few-words>` for a task in words, one per task (as Marshall Inc's orders name them) |
| Computer Manager | the app's own main checkout (Building's `apps/building-app`) | a short branch per task |

**CORRECTED 2026-11-19: worktrees are no longer the escalation, they are the default** for a
milestone and for a worker. The 2026-09-20 standard made them the exception; the Marshall Inc
orders made them the rule, and that is what is measured.

- **`.worktreeinclude` carries the gitignored files a checkout needs** — a worktree holds
  tracked files only. Include `.claude/settings.local.json`, or the worktree re-prompts for
  every approved command.
- **Remove a worktree only when nothing in it is uncommitted or unpushed, and never with
  `--force`.** Then `git branch -d` and `git push origin --delete` the branch.
- **The trap every hall hits:** the evidence trees — `data/`, `brain/`, `intake/`, `taxes/` —
  are gitignored, live at the volume root, and there is exactly one of each. A skill run from a
  worktree writes the same `data/` as everyone else. **Those skills must refuse from a
  worktree**, with the honest reason: money's
  `.claude/skills/shared/district_root.py` → `require_main_checkout()` says *you are in the
  wrong place, the volume is fine* rather than "your volume is unmounted".
- **Unattended actors are not hooked sessions.** The software factory holds a pull request
  that touches a protected path for a person ([`software-factory.md`](software-factory.md) §8),
  and `bin/guard-scope` (a `PreToolUse` hook that fails closed) refuses a write outside an
  active lane before it happens. Two mechanisms, neither trusted alone.

---

## 5. Shipping: `bin/session-files` and `bin/ship`

Built and proven on Money on 2026-09-20 (its PRs #149-#154), adopted by the GM on
2026-09-26, and ported to GitHub on 2026-11-19 (security-district#29). **Every hall ships
with them**, from the one copy the GM keeps.

### Attribution: `bin/session-files`

A `PostToolUse` and `SessionStart` hook that records which files **this session** changed.

- **It diffs `git status --porcelain` between tool calls**, rather than reading the `Edit`
  tool's file path. Most edits go through Bash (`sed`, heredocs, python); a hook watching only
  `Edit|Write` would see almost nothing.
- **A session's first sight is a baseline, not a claim.** Otherwise a session opening on a
  dirty tree claims everyone else's work on its first call.
- **State lives in `<git rev-parse --absolute-git-dir>/claude-sessions/<id>`**: never
  committed, and per checkout, because the same path in two worktrees is two different files.

```
record      after each tool call: claim newly changed paths (the hook)
list        this session's uncontested paths — safe to stage
contested   paths another session also claims — a person must look
others      every dirty path this session may not stage
json        all three
```

Registered in `.claude/settings.json`, resolving the root from the caller's own cwd (never
`CLAUDE_PROJECT_DIR`, which is unset in practice):

```json
"PostToolUse": [{ "matcher": "Edit|Write|NotebookEdit|Bash",
  "hooks": [{ "type": "command", "timeout": 10,
    "command": "cd \"$(git rev-parse --show-toplevel 2>/dev/null || pwd)\" && bin/session-files record" }]}]
```

`SessionStart` runs the same command with `< /dev/null`. **Its stated limits:** two sessions
editing one file both claim it (recorded, not prevented — that is when a worktree is the
answer); an edit made outside Claude Code belongs to no session and is never staged; a file
changed then reverted stays claimed until clean.

### `bin/ship`

```
bin/ship                                    dry run: what would be committed
bin/ship --commit --issue N -m "…"          commit, branch, push, open the pull request
bin/ship --commit --no-issue "<why>"        ship with no issue; the reason goes into the commit
bin/ship --merge --commit --issue N         …and merge it when the checks pass
bin/ship --with-branch --issue N --commit   accept a finished branch as it stands
```

It **stages exactly the paths `session-files` lists** — it never decides what to stage — cuts
the branch **before** committing (so the default branch never runs ahead of `origin`), commits
with `Closes #N`, pushes, and opens the pull request. With `--merge` it waits for the checks
**on that exact commit**, merges, and cleans up. The repo is read from the `origin` remote;
nothing in it names a hall.

| It refuses | Why |
|---|---|
| **no issue** | `references/workflow.json` → `require_issue`. `--no-issue "<why>"` overrides, and the reason is written into the commit: an override that leaves no trace is a hole. |
| **contested paths** | claimed by two sessions: reported, never staged |
| **nothing claimed** | rather than committing a borrowed set |
| **checks not green** | *never ran* is reported separately from *failed* — it is also what a dead runner looks like. A repo with no workflows has nothing to wait for, and `--merge` says so out loud rather than reading as guarded. |

Being in the wrong board column **warns and proceeds**: a gate that blocks an opportunistic fix
is a gate people stop using. Only a missing issue refuses.

**Five defects found by running it**, each waiting for the next hall that copies it:

1. **Checkout is not volume.** Run from a worktree, a hardcoded path committed the main
   checkout's index. `CHECKOUT` is `git rev-parse --show-toplevel`; `VOLUME` is the first two
   path components (`/Volumes/<NAME>`), confirmed with `os.path.ismount`. The GM, whose root is
   a volume root only since 2026-11-19, follows the same rule.
2. **Attribution state under `.git/`** fails in a worktree, where `.git` is a file. Use
   `--absolute-git-dir`, never `--git-common-dir`.
3. **`CLAUDE_PROJECT_DIR` is unset in practice.** Resolve the root from the caller's cwd.
4. **Read checks on the commit's SHA** (`gh pr checks`, or `/commits/<sha>/check-runs`), never
   the list of runs filtered by name — which once reported "missing" for fifteen minutes while
   CI had been green in four seconds. It is also exactly what branch protection gates on.
5. **Mergeability is computed asynchronously.** Poll `mergeable` until it is known, then retry
   the merge.

---

## 6. CI and branch protection

The order is not a preference: each part is worthless or misleading without the one before.

1. **Build the commit, not the volume.** A green status on a directory other sessions are
   writing is a statement about nothing in particular. A test job uses `actions/checkout` on
   the self-hosted runner (PROJECTS's `test.yml` does); a job that must build what is on the volume
   materialises a detached worktree of the exact commit, with the mount guard first and an
   `if: always()` teardown. The pattern is in [`deploy-standard.md`](deploy-standard.md).
2. **A test workflow on `pull_request`.** A status exists only if a workflow runs on that
   event. **Never `pull_request_target`**: the M4's runners run on the host with the 1Password
   SSH agent available, which makes every workflow file security-relevant code.
3. **Branch protection on the default branch:** a rule or ruleset requiring a pull request and
   a required status check. **Read the default branch, never assume `main`** (Rentals's app
   defaulted to `production`, and every "protect main" protected the wrong one). **Pin the
   exact check name** as the run reports it; a renamed job leaves a required check that never
   reports, and the pull request waits forever reading like a slow runner.

**A repo with no CI names its local checks** in the hall's `sop-local.md`, and every pull
request body says which ran. A merge gated on a check that does not exist is worse than an
unguarded one, because it reads as guarded.

**A pull request body shows its evidence, not its adjectives** (#648). Each check that ran is
named **with what it printed**: the exit code or its last line (`make check → exit 0`,
`Ran 20 tests … OK`), and a positive control's refusal where the change is a guard. A section
headed `## Fresh review` holds the review by a fresh context that never saw the work
([`sop.md`](sop.md) step 7): each finding, and what was done about it, or *none*. "Tests pass"
with no output is a claim, and the reviewer cannot recheck it.

**Every repo with a pull-request test job protects its default branch** with that job as the
required check. A repo without one names its local checks instead (above).

**A check that needs what is not in git is run locally, never faked in CI** (Family, #562). A
hall repo's gate often reads trees a CI checkout of the commit does not have: `brain/` (its
own repo), `documents/`, `messages/` and `intake/` (cloud links) and `var/` (local state). A job
that passes without them reads as guarded and is not, so such a check is not a CI job. The repo
names it in the hall's `sop-local.md`, under *github-standard.md §6*: the command, and that it
runs from the **main checkout** (a worktree skips those trees, loudly). It runs before the
merge and again after it, and the pull request body says it ran. Whatever part of the check
does run on a clean checkout (the test suites, the self-tests) may still be a CI job and the
required check, as long as it is named for what it covers and not presented as the whole gate.
The general manager works this way: `make check` before, `make brain links` after.

---

## 7. Who this does not bend for

- **Teaching** keeps its own tracking and shipping methods (§2–§6): it is the one hall
  with a second machine in play, the university's MacBook, reached only through git, with
  `data/` one-way. It is **not** exempt from §1: its repos follow the three-repo rule, and its
  app repos take the `gc-` prefix when they are next renamed (#146).
- **Any other exception is the hall's to record**, in its `sop-local.md`, with the reason.

---

## 8. Traps, and measuring it

- **Issues and pull requests share one number sequence,** so `(#147)` in a merge subject may
  be a pull request. Eight false positives from this alone.
- **The REST `/issues` endpoint returns pull requests too.** Filter on `pull_request`, or use
  `gh issue list`.
- **`state=all` is required,** or every closed issue silently vanishes.
- **Milestone titles are not unique.** Key on the number.
- **A subprocess `grep` is not the shell's `grep`** — the false clean described in
  [`sop.md`](sop.md).

**Measuring it.** `bin/repo-check` compares every hall's `districts.json` → `repos` with its
checkouts and with GitHub (read-only; a hall it cannot reach is "could not check", never
clean). `gh label list -R <repo>` against `references/labels.json` for the taxonomy;
`gh api repos/<repo>/branches/<default>/protection` for protection; `bin/district-ready` for
the hook and `bin/ship`.
