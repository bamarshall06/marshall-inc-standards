# The software factory — how a hall builds software

> **Public edition.** A working standard from Marshall Inc, shared as it is used. Real names, paths and accounts are replaced with generic ones (floors such as *Business* and *Teaching*, `/Users/<you>`, `<your-github-user>`). References like `gm-district #372` point to private repositories and are kept only as history. Start with the [README](../README.md).

## Summary
> How any hall takes on a development project: one engine, pinned once in the GM and installed in the app's own repo; a milestone planned as specs whose questions are settled before the build; built on request, one milestone at a time, by laps that scripts judge before any model does; proven by evidence the builder cannot edit; and tested by Bryan once, as a whole, before it ships. Written from what Business runs.

**Status:** org-wide standard, written 2026-11-23 (gm-district #154, milestone *SF's factory
builds in every hall*). It replaces `docs/factory.md` (2026-10-24, a theory of controls, never
shipped) and the GM's own project loop (`bin/project`, with its planner, coder and tester
roles), which never ran for real and is retired. Bryan decided on 2026-11-14 that every hall
builds through Business's factory, and on 2026-11-23 that the organisation runs **one
system**. **Applies to:** every app any hall changes through pull requests. A hall with no
development project meets it with nothing in place. Entry point: [`sop.md`](sop.md).

| | |
|---|---|
| §1 | Two factories, and the line between them |
| §2 | One engine, pinned once, in the app's repo |
| §3 | What a factory app has |
| §4 | Planning: a milestone of specs |
| §5 | The tiers: how far the factory may go alone |
| §6 | Ready before the first lap |
| §7 | Building: on request, one milestone at a time |
| §8 | Proving: evidence the builder cannot edit |
| §9 | Questions are settled at intake |
| §10 | Bryan tests the whole, once, and then it ships |
| §11 | Rules that hold throughout |
| §12 | Not met yet |

---

## 1. Two factories, and the line between them

A hall that runs software runs two factories over one backlog.

| | Operations | Software |
|---|---|---|
| Job | runs the business | changes the app |
| Made of | scheduled jobs, skills, reporting agents | the factory's workflows |
| Writes | the app's data, and a ledger or channel only where a settled exception allows | code, through pull requests |
| Never | changes code | runs the business |

1. **The line is between running and changing.** *Running* a script is operations, on
   production. *Changing* it is software: a spec, an issue, a lap, a test, before the new
   version ever runs for real.
2. **Operations never goes through testing.** Its work is real; a run against test data is a
   rehearsal, not the action. The test environment belongs to the software factory alone.
3. **A job is not an issue.** It becomes one only when it fails, and the cause decides where it
   goes: the code is wrong → the software factory; a decision is missing → a person; it writes a
   ledger → a person runs it, and the factory only proposes.
4. **A reporting agent never acts** ([`doe-framework.md`](doe-framework.md) §14). It files what it
   finds as an issue; the factory picks issues up from there.

## 2. One engine, pinned once, in the app's repo

1. **The engine is `ai-software-factory` over Archon's shared SDLC workflows**, at the one
   revision recorded in the GM's `references/factory.json`, with Business's pack (the
   `sf-milestone-lap` and `sf-milestone-review` workflows and the consumer that drives them).
   **A hall never pins its own revision.** An upgrade is a change to `references/factory.json`,
   proven in one app before any other installs it.
2. **It is installed in the app's repo** (`apps/<app>/`, its `<prefix>-<app>` repository), never
   in the hall's `<prefix>-district` repo. The factory builds the app, so it travels with the app.
3. **One orchestrator per repo.** No second planner, builder or tester beside it: no hall
   `planner`, `coder` or `tester` agents, no `bin/project`, and no `archon-backlog` (two things
   filing planned work onto one board is the two-writer failure). Business deleted its own
   set on 2026-10-25 for this reason; the GM retired its set on 2026-11-23.
4. **The hall writes none of the AI steps.** The workflows and their agents are the engine's;
   the hall writes its context (§3), its specs (§4) and its rules as data.

## 3. What a factory app has

| Path | What it is |
|---|---|
| `MISSION.md` | The product: what it is, **what it is not** (each exclusion with why), the hard boundaries, and the decisions only a person makes. Every backlog and build decision is read against it. |
| `FACTORY.md` | The evidence record: the tested source SHA, run IDs, coverage, and the limits still open. |
| `FACTORY_RULES.md` | The project's constraints. **Irreversible actions stop the factory**: changing identity or who may act as whom, and migrating or deleting stored data, are held for a person. |
| `factory/pack.json` | The pinned engine, matching `references/factory.json`. |
| `harness/END-TO-END.md` | **At least two journeys** through the running app, each naming the exact value it expects. |
| `harness/harness.config.json` | The ordinary gate, and `protected_paths` (§8). |
| `.factory/holdout/HOLDOUT.md` | Holdout scenarios, behind a fence hook that refuses every agent except the holdout writer and verifier. |
| `.kiro/specs/_template/` | The spec template (§4). |
| `rules/` | The gates, the question routing, the run limits and the model per tier, **as data** (convention 3). |
| `~/.local/state/<prefix>-factory/` | The readiness evidence (§6). **Outside the repo**, so a lap cannot write what admits it. |

The app also has a way to boot a fresh copy on its own port against a throwaway database, a
`/health` path, and a `/build-id` path that answers the deployed commit's 40-character SHA.

## 4. Planning: a milestone of specs

1. **The milestone is an outcome** ([`github-standard.md`](github-standard.md) §2). Each piece
   of it is a **spec**, `.kiro/specs/<slug>/`, with `requirements.md`, `design.md` and `tasks.md`,
   copied from the template **before any code is written**. A small fix gets a short spec, never
   none.
2. **`requirements.md` carries:** the story (As … I want … so that …); *Why this exists*, measured;
   numbered requirements a reviewer can check; **Decisions** (§9); **the test** (§10), one check
   per `### C<n>` with **Who**, **Where**, **Do**, **Before** and **Now**; **On screen** when a page
   changes, every label quoted as it will read; and *Out of scope*.
3. **Every task names an acceptance command, and that command is run and seen to fail before the
   build starts.** A check that has never been seen to fail proves nothing: the estate's receipts
   are a type check over zero files that exited 0 for a day, and a backup check satisfied by any
   directory for nine days.
4. **A spec checker refuses an incomplete spec**: a missing section, an open decision, a missing
   test or an On screen section a page change needs.
5. **A script files it**: the spec becomes one parent issue with a sub-issue per task, its
   dependencies as real blocked-by links, in the milestone. Each task issue carries a tier label
   (§5), or no lap may build it.

## 5. The tiers: how far the factory may go alone

| Tier | Work | Built by | Reaches production |
|---|---|---|---|
| `tier/1` | bug fixes, pages, reports: anything `git revert` undoes | the factory, on the standard model | after Bryan's test |
| `tier/2` | migrations, stock, orders, prices, money, channel pushes, and code that writes a ledger | the factory, on the strongest model | after Bryan's test, and a staged run where the hall has one |
| `tier/3` | **running** anything against a ledger, a live channel write, taxes, the brain | **a person** | the factory only proposes |

Ledger-writing **code** is tier 2; **running** it against the real books is tier 3. A tested
script is still a write with no undo. An issue with no tier label is out of every lap's scope.

## 6. Ready before the first lap

A readiness gate runs before an app's first lap and after any change to the harness. It exits
**0** ready, **2** a gate fails, **75** a gate could not be evaluated, and 75 never counts as
passing. It checks at least:

- `protected_paths` is set and covers §8's list.
- The journeys are present and are not the template's.
- **The runtime is calibrated**: the baseline passes, a planted fault fails, and a deliberately
  wrong candidate comes back inconclusive.
- **The mutation set is caught**, and each rung (static, unit, runtime, holdout) has caught a defect
  no earlier rung caught. A rung that has never failed is named as such.
- The lap's policy hook has been seen to refuse a live call.
- `/build-id` reads back a 40-character SHA. A 200 is not a pass: a web server answers an unknown
  path with the app's index page.
- Evidence goes stale when the files it was made from change, and the gate names the stale file.

**The hall scorecard reads it** ([`hall-standard.md`](hall-standard.md) §9): `bin/district-ready`
finds every factory app in a hall and checks that it is in an app repo (§2), has what §3 lists, is at
the pinned revision, and has its readiness evidence on file with the calibration passed. It never
reads the holdout and never re-runs the app's own gate. A hall with no factory app meets every row.

## 7. Building: on request, one milestone at a time

1. **Nothing runs on a timer.** Bryan names a milestone; the factory builds it and stops. A
   scheduled loop over the whole board spent 1.83B tokens in the week of 2026-10-26, over 256
   laps, 138 of them re-reviews.
2. **Dry run, then `--confirm`.** The dry run lists every issue in build order, each with why it is
   or is not workable, and changes nothing. One run at a time, behind a lock.
3. **One lap per task, in its own worktree**, made and removed by the engine. A lap worktree is
   for code only: evidence work refuses to run in one.
4. **Scripts judge a head before any model reads it.** The ordinary gate and the task's
   acceptance commands run first; a failing head is sent back with no model reading it.
5. **It is bounded.** At most 10 laps a run; 2 laps and 1 send-back per task, then the task is
   labelled `needs-owner` with one line saying why, and the run moves on. A `STOP` file ends the
   run; a usage limit ends it at once, and it never waits for the next allowance window.
   Rerunning carries on where it stopped.
6. **Models are bound per run, in the app's rules**: a small model routes, the standard model builds
   `tier/1` and reviews a task, the strongest builds `tier/2` and runs the one full review. Never
   in a user-wide config.
7. **Tasks land on the milestone's branch**, not on `main`; a task's issue closes when it lands,
   so the tasks that depend on it can start, built from that branch.

## 8. Proving: evidence the builder cannot edit

1. **The writer cannot edit the evidence.** `MISSION.md`, `CLAUDE.md`, `harness/**`,
   `factory/**`, `.kiro/**`, `rules/**`, `.github/workflows/**`, the database schema and `bin/**`
   are protected paths: a pull request that touches one is held for a person. Otherwise a loop
   with no limit finds that changing the test is the cheapest way to green.
2. **The holdout is read only by its two agents.** The fence refuses everyone else, including a
   session that merely names the folder.
3. **Review is independent of the builder**, names the head it reviewed, and states a verdict.
   Stale, ambiguous or self-attested evidence holds the work.
4. **One full review per milestone**, on the milestone's pull request: the ordinary gate, the app
   check with the holdout, the evidence check, and the strongest model. A milestone it sends back
   stops for Bryan rather than buying a second.

## 9. Questions are settled at intake

1. **Every question a spec raises is answered before it is filed**, in its `## Decisions` table
   (question, answer, by whom), never left open, TBD or unknown: migration numbers, what it
   depends on, the one read-only live call that proves an API's shape, tests it may change and
   why, and anything only Bryan can say.
2. **A question the build still raises is routed by rules as data.** **Product** questions go to
   Bryan: what customers see or are told, money and the ledger, scope, anything that cannot be
   undone. **Technical** ones go to the factory's manager: a migration collision, a branch
   dependency, a re-pinned test with its reason. **Unclassified goes to Bryan**: the rule fails
   towards asking.
3. **Only product questions reach Bryan.** A settled technical stop is a line in the digest and a
   comment on the pull request.

## 10. Bryan tests the whole, once, and then it ships

1. **The factory builds a task at a time; Bryan tests a milestone at a time.** When every task is
   in, one pull request goes from the milestone's branch into `main`. Its body is the test script:
   each spec's checks, verbatim, and how to start the app locally.
2. **The checks describe the whole system**, what a person does with it end to end, never one
   task's piece. A spec too big to test in one sitting is two specs. Data a check needs is added to
   the test seed as a task of the spec; a check never points at a production record.
3. **Ship refuses unless the head is still the commit handed over for testing.** It merges with a
   merge commit, deploys by [`deploy-standard.md`](deploy-standard.md), and reads `/build-id` back.
4. **Hand work on the branch still gets the review.** Skipping the review is Bryan's call, said
   for that ship, and a review issue is filed straight after, so the gap is closed later.

## 11. Rules that hold throughout

These are the hard restrictions, where the factory is most likely to test them.

- **No agent declares its own work complete.** A script computing a result closes a loop.
- **Bounded, not endless.** Every repair loop has a cap, and then asks a person.
- **No reversal path, no delegation.** Tier 3 is proposed, never executed, and that is an end
  state, not a phase.
- **No secrets in GitHub or in the repo.** The factory's runtime configuration lives outside the
  repo, mode 600; credentials come from the hall's vault ([`security-standard.md`](security-standard.md)).

## 12. Not met yet

A standard states what must be true; these are where the organisation's one running factory
falls short of it today.

- **§4.3, the failing acceptance command.** Business's lap runs a task's acceptance
  commands *after* the build, never before it. The GM's retired `bin/project` had this check
  (its red gate); it is the one thing taken from it.
- **§2.2, the app's repo.** Business's factory is tracked in its hall repo
  (`business-district`, at the volume root) and builds `sf-listing-mirror`, from when the root was the
  app. A new install goes in the app's repo; moving SF's is SF's to plan.
- **§6, the holdout rung** has never been shown to catch a defect no earlier rung caught.
- **§10.3 depends on a test environment.** Business's testing server was deleted on
  2026-11-04, and Bryan tests on the local stack.

---

## Further reading

- Business's `docs/OPERATIONS_FACTORY.md`: the worked example, with its history and
  measurements.
- [`github-standard.md`](github-standard.md) §2: milestones and issues.
- [`doe-framework.md`](doe-framework.md) §13–§14: verification seams and the autonomy ladder.
- [`deploy-standard.md`](deploy-standard.md): how the shipped commit reaches production.
