# The SOP — how Marshall Inc gets work done

> **Public edition.** A working standard from Marshall Inc, shared as it is used. Real names, paths and accounts are replaced with generic ones (floors such as *Business* and *Teaching*, `/Users/<you>`, `<your-github-user>`). References like `gm-district #372` point to private repositories and are kept only as history. Start with the [README](../README.md).

## Summary
> How Marshall Inc gets work done: who does what (Bryan, the District Manager, two workers, the GM's residents), and one task from issue to merged pull request and deploy, shipped from one of many sessions with `bin/session-files` and `bin/ship`. The front page of the standards.

**Status:** rewritten 2026-11-19 (gm-district #370, milestone #24) from a measurement of every
hall, every repo and the Marshall Inc orders on that day. It replaces the SOP of 2026-09-21,
whose procedure (`bin/issue-seed`, `bin/board`, `bin/ship`) had stopped being used almost
everywhere; that version is in git history. **Applies to:** every hall — `gm`, `biz`, `money`,
`fin`, `teach`, `proj`, `family`, `sec`, `bldg`. A hall that works differently says so in its own
`docs/sop-local.md`.

This page is the front door. It says who does what and how one task goes from start to finish.
Everything else is in the ten standards, each the only place its subject is written down.

| Standard | The one thing to carry |
|---|---|
| [`organization-standard.md`](organization-standard.md) | **Who runs what.** The Boardroom sets every standard and runs intake; the Computer Room builds the building; Security does all security; a district runs its own domain. Standards flow down, requests come up, districts deal with each other directly. |
| [`github-standard.md`](github-standard.md) | Three repos per hall; every piece of work is an issue in a milestone; it lands through a pull request. |
| [`brain-standard.md`](brain-standard.md) | What you wrote is markdown in `brain/`; what you received is a PDF and sidecar in `documents/`. |
| [`doe-framework.md`](doe-framework.md) | **The skill standard.** The orchestrator does not compute and the scripts do not reason. The autonomy ladder and its D1/D2/D3 rungs are its §14. |
| [`hall-standard.md`](hall-standard.md) | What every hall is and has, and how `bin/district-ready` measures a hall against it (§8). |
| [`communication-standard.md`](communication-standard.md) | `messages/` and `intake/`, notices, requests, messages and files between halls, a notice closed by archiving it, a message by `actioned:`. |
| [`security-standard.md`](security-standard.md) | Secrets live in 1Password, in the vault named after the volume; never in a file. Every live server meets the **production baseline**. Written and maintained by the Security district. |
| [`deploy-standard.md`](deploy-standard.md) | A deploy runs on the M4's self-hosted runner and ships no secrets. |
| [`software-factory.md`](software-factory.md) | **How a hall builds software.** One engine, pinned once, in the app's repo; a milestone of specs, built on request; Bryan tests the whole once before it ships. |
| [`room-standard.md`](room-standard.md) | **Subrooms only.** A room is a small hall: every rule for a subroom, in one place. Linked into rooms, never into a hall. |

---

## Who does what

The bodies of the organisation — the Boardroom, the Computer Room, Security and the districts —
and the one job of each are [`organization-standard.md`](organization-standard.md). This section
is the seats inside a hall.

Marshall Inc (the app, once Claude Castle) starts every session through `bin/dm`, in tmux, and
hands each one its orders from `~/.claude/marshall-inc/orders/<code>-<seat>.md`. The seats are the
same in every hall.

| | Seat | Does |
|---|---|---|
| **Bryan** | — | Decides, assigns and approves. Anything outward-facing (a message sent, a calendar event, a deletion, a merge he asked to review) waits for him. |
| **District Manager** | `hero` (`dm-<code>:hero`) | The hall's planner and CEO, and a worker when there is something to do. Plans with the district-planner skill, works small changes and milestones, writes the shared trees, and proposes what the workers take next. |
| **Workers** | `1`, `2` (`dm-<code>:squad-N`) | Take one task from Bryan through to a merged pull request, each in a worktree of its own. Never touch the main checkout. |
| **Residents** | none today | A hall's extra seats beyond its hero and workers (`districts.json` → `residents`). The GM's Computer Manager and Computer Worker retired 2026-11-21 (#509): Marshall Inc's people are the Building hall's. Each hall's `intake/` is its District Manager's to work. |
| **Marshall Inc** | — | Fast-forwards each hall's main checkout within five minutes of a merge, while it sits on the default branch. |

**Every seat `bin/dm squad` starts runs on the model and effort the GM's
`references/models.json` names** (#650):
the latest Opus at medium effort, Opus 5.5's default and Anthropic's starting point, unless a
kind of seat is given its own there after testing. `bin/dm squad` passes both, and fails if
the file is missing, so a seat never silently runs on whatever a user default says.

**Bryan and Marshall Inc coordinate the seats; a Claude coordinator is a decision not taken**
(2026-11-25, #650). Claude Code's projects (public beta) run one conversation that starts,
watches and judges other sessions, and Anthropic's own team works that way. Here the orchestrator stays a script
([`doe-framework.md`](doe-framework.md) §13): Bryan assigns, Marshall Inc starts each seat
through `bin/dm`, and the handoff lives on the issue. It is written down so its absence reads
as a choice; revisit it when a coordinator's judging can be checked by a script.

---

## One task, start to finish

Written down from what every order in `~/.claude/marshall-inc/orders/` already says, and from how
the work of 2026-11-19 was actually done.

1. **Plan it as an issue in a milestone.** The District Manager uses the district-planner skill:
   `plan.py context` first, every write as a dry run, then `--commit`. Issues and milestones live
   on the hall's `<prefix>-district` repo, or on an app repo for app work. Nothing is deleted;
   a close always leaves a comment saying why.
2. **Bryan assigns it** from Marshall Inc, to the District Manager, a worker or a resident.
3. **Work in the right place.**
   - *A small change:* a short branch from `origin/<default>` in the main checkout.
   - *A milestone:* its own worktree, `.claude/worktrees/m<N>-<slug>`, on branch `m<N>-<slug>`.
   - *A worker:* its own worktree, `.claude/worktrees/squad-<N>`, a branch per task:
     `issue-<number>-<a-few-words>` for a milestone's issue, `<a-few-words>` for a task in words.

   The main checkout stays on the default branch. Nobody commits, stashes or discards another
   session's uncommitted change in it.
4. **Do the work.** `bin/session-files`, a hook in every hall, records which files *this*
   session changed, so nobody has to remember. **Never `git add -A`** — several sessions share
   one volume, and `-A` commits their half-finished work. The shared trees (`brain/`, `data/`,
   `documents/`) live once on the volume and are written from the main checkout, by each tree's
   own rules.
5. **Leave a handoff on the issue.** A session can end at any moment: a restart, a crash, a
   seat taken back. So before pausing, whenever the work waits on Bryan, and before a session
   ends with the work still open, post a comment on the issue in three lines: **Done** (what is
   finished, and where), **Waiting on** (who, for what) and **Next** (the next command or step).
   Name any worktree that holds work not yet committed, and keep `state/doing` true. Whoever
   picks the issue up, the same seat or another, starts from that comment. See *Three records
   of a session* below.
6. **Ship it with `bin/ship --issue N`** (dry run first, then `--commit`). It refuses without an
   issue (`--no-issue "<reason>"` writes the reason into the commit), stages exactly the files
   this session changed and nothing else, commits with `Closes #N`, pushes the branch and opens
   the pull request. Its checks are the repo's pull-request CI where it has one; where it has
   none, the hall's local checks named in its `sop-local.md`.
7. **Have it reviewed in a fresh context, then merge it.** The session that did the work has
   already talked itself into every choice, so a fresh context reviews the pull request
   against its issue: `/code-review`, or a subagent given only the issue and the diff. Its
   findings, and what was done about each, go under `## Fresh review` in the pull request body,
   beside each check's actual output ([`github-standard.md`](github-standard.md) §6). Then
   merge with `bin/ship --merge` when the checks pass and the review is in, **whatever the
   change touches: nothing waits for a person** (Bryan, 2026-11-25), unless Bryan asked to
   review that one first.

   `bin/ship` waits for CI on that exact commit, reports *never ran* separately from *failed*,
   and merges that commit and no later one. The GM's copy also refuses a merge with no
   `## Fresh review` line in the body, and checks the gate in a dry run. A hall takes the gate
   by taking the GM's `bin/ship` **together with** a `references/review.json` of its own,
   which `bin/ship` reads as the default branch has it, so a pull request cannot change the
   heading it is checked against. `bin/ship` itself runs from the branch being merged, so a pull
   request that edits it runs its own gate: the fresh review is what catches that. Until then a
   hall meets the rule by hand.
8. **Clean up.** Back to the default branch, `git pull --ff-only`, and delete the branch here and
   on GitHub. Remove a milestone's worktree only when nothing in it is uncommitted or unpushed,
   and never with `--force`. Then do what follows the merge — a skill with `--commit`, a brain
   note, `bin/ship-standards` — from the main checkout.
9. **Deploy, where the repo deploys,** by its own workflow ([`deploy-standard.md`](deploy-standard.md)),
   and verify the live site afterwards, on the box first.
10. **Report in a few lines:** what was done, the PR link, anything Bryan must decide, and one
   closing line — `Next: #<issue>`, `Next: <a few words>` or `Next: nothing` — so the next
   assignment is one press. When the work is left open, the same report goes on the issue as its
   handoff (step 5).

**Why a tool and not a habit.** Nothing built without an issue, never `git add -A`, and
nothing finished until it is checked are rules a model has to *remember*; `bin/ship` and
`bin/session-files` make them something a tool *computes* and refuses
([`github-standard.md`](github-standard.md)). Every hall ships with them; the GM keeps the
organisation's one copy.

### Three records of a session

Added 2026-11-25 (gm-district #640), after a restart stopped work in the middle and the new
session had to rebuild what the old one had been doing. Each record has one job:

| Record | Where it lives | What it is for |
|---|---|---|
| **The transcript**, the black box | Claude Code saves every session word for word in `~/.claude/projects/<the folder's path>/`, on the Mac that ran it. | Carrying on mid-thought after a crash: `bin/dm squad <hall> <seat> --resume` for a seat, which reopens that seat's own session; `claude --resume` (pick from the list) by hand. Not `claude --continue` where seats share a folder: it takes whichever spoke last. Looking up exactly what was run or said. |
| **The handoff**, on the issue | A comment on the issue (step 5): Done, Waiting on, Next. | Anyone, on any Mac, picking the work up in a minute. It survives a wiped Mac; the transcript does not. |
| **The lesson**, in the brain | A note in the hall's `brain/` ([`brain-standard.md`](brain-standard.md)). | What was learned and must be kept: a decision, a trap, a measurement. |

**A transcript is never copied into git, Dropbox or `documents/`.** It holds whatever the session
saw, a secret printed by mistake included, and the hard restrictions forbid archiving a secret.
It stays where Claude Code keeps it. What must outlive it goes on the issue or into the brain.

---

## Two layers: the organisation's, then the hall's

| | The organisation's (the GM's) | The hall's own |
|---|---|---|
| **Standards** | `docs/standards/<standard>.md`, linked into every hall by `bin/ship-standards` (`standards.json` lists them; Teaching keeps copies). Never edited in a hall. | `docs/sop-local.md`: one section per change, naming the section it changes, what the hall does instead, and why. |
| **Skills** | One copy in the GM's `.claude/skills/<name>/`, linked into each hall by `bin/hall-layers` (`standards.json` → `skills`). | Any real folder in `.claude/skills/`. A real folder with an org skill's name **is the override**. |

The hall's layer wins for that hall, except the hard restrictions in `CLAUDE.md`, which nothing
overrides. A change that would suit every hall belongs in the GM's copy: ask for it with a
`request/standard` or `request/feature` issue on `gm-district`
([`organization-standard.md`](organization-standard.md) §4).

---

## Two rules that cut across everything

### A check that cannot fail is worse than no check

It reports a clean bill of health nobody verified. Measured instances, all from this
organisation, all of which produced a confident and entirely false result:

- `tsc --noEmit` against a solution `tsconfig.json` type-checked **zero files** and exited 0 for
  a whole day — then a deploy failed on six syntax errors it had never looked at. Use `tsc -b`.
- `grep -r` here is **ugrep**, which honours `.gitignore` — precisely where credentials live. And
  `--no-ignore-files` is not a reliable fix: under BSD grep it is rejected outright, printing
  usage to stderr, so `2>/dev/null` turns a hard error into a clean bill of health.
- **A subprocess `grep` is not the shell's `grep`.** A sweep got `/usr/bin/grep`, exited 2,
  printed nothing, and reported "none" having proved nothing.
- `mapfile` under bash 3.2 killed a health check at its first line; the nine missing output files
  rendered as nine "NO DATA" servers and the skill reported a full-estate outage.
- `[ -d "$DEST" ]` is satisfied by any ordinary directory, which is how nine days of archives were
  written to a dead path while the script reported success.

**Every check needs a proven positive control.** Plant something it must catch, and confirm it
does, before believing a clean result.

### An absence must be distinguishable from a decision

`never ran` reported separately from `failed`. An empty board refused rather than mirrored. "Could
not check" instead of "CRITICAL". `UNCONFIGURED` instead of `0%`. *A silent empty result looks
identical to "there was nothing".*

And its corollary: **a staleness indicator is not a substitute for not being stale.** Money's
guard worked perfectly, printed "Synced 3 hours ago" in small grey text, and nobody read it.

---

## Conventions that make the rest hold

The four in every hall's `CLAUDE.md` come first and are not optional: **dry-run by default, no
silent fallbacks, rules that can change live in data, scripts print JSON or tables.** Two more:

- **Advisory checkers never block.** A checker that can block is one people route around, and a
  permanently-red check is one nobody reads. `bin/district-ready` measures and gates nothing.
- **Documents correct themselves in place and keep the record of having been wrong,** with the
  measurement that forced the correction. A correction with no trace is the documentation version
  of an untracked override. This page is an example: its 2026-09-21 predecessor is in git
  history and summarised in the brain note `standards-2026-11`.

---

## What is NOT enforced

Stated plainly, because a half-known gate is worse than a known one. These are gaps by design,
accepted for a single operator:

- **A session merges its own pull request.** No approval is required anywhere.
- **A repo with no pull-request CI** is checked by the local checks its hall names, which the
  pull request records but nothing re-runs.
- **The self-hosted runners on the M4 share one SSH agent.** Treat a workflow file as
  security-relevant code ([`deploy-standard.md`](deploy-standard.md)).
- **Nothing checks that a claim in a document is still true.** The SOP this page replaced was 63
  days stale when it was measured. Re-measure, and correct in place.
