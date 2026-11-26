# The hall standard — what every hall is and has

> **Public edition.** A working standard from Marshall Inc, shared as it is used. Real names, paths and accounts are replaced with generic ones (floors such as *Business* and *Teaching*, `/Users/<you>`, `<your-github-user>`). References like `gm-district #372` point to private repositories and are kept only as history. Start with the [README](../README.md).

## Summary
> What every hall is and has: one volume recorded in `districts.json`, the layout at its root, its `CLAUDE.md` contract, `docs/standards/` and `sop-local.md`, `.claude/`, its people, what it owns outside its volume, the timed jobs it runs, one way of naming documents, the daily intake loop its intake agent runs, and how its own app opens inside Marshall Inc — and how `bin/district-ready` measures it.

**Status:** org-wide standard, written 2026-11-19 (gm-district #374, milestone #24). Until then
no document said what a hall is made of; the pieces were in the district template, in
`readiness.json`, in the checks of `bin/district-ready`, and in the layout half of the old
`repo-standard.md` (now [`github-standard.md`](github-standard.md) §1). §10 (naming) and §11 (the
daily intake loop and the intake agent) were added 2026-11-23 (gm-district #592, milestone #39),
from Bryan's decisions on Teaching's request #590. §12 (a hall's app in the building) was
added 2026-11-23 (gm-district #608, milestone #40), from Finances' request, and made
native-only on 2026-11-25 (gm-district #652), from Building's request. **Applies to:** every
district — `biz`, `money`, `fin`, `teach`, `proj`, `family`, `sec` — and to the general manager, the
**Boardroom**, whose differences are in its own `docs/sop-local.md`. Entry point:
[`sop.md`](sop.md).

A **hall** is one part of the organisation, with its own volume, its own contract, its own
repos, its own secrets and its own people. The GM operates on every hall and lives in none;
everything that crosses a hall's boundary goes through the GM, except a message
([`communication-standard.md`](communication-standard.md)).

| | |
|---|---|
| §1 | A hall is a volume, recorded in `districts.json` |
| §2 | The layout: what must be at the root |
| §3 | The contract: `CLAUDE.md` |
| §4 | `docs/` and `sop-local.md` |
| §5 | `.claude/`: settings, hooks, skills, agents, worktrees |
| §6 | The people: seats, orders and the roster |
| §7 | What a hall owns outside its volume |
| §8 | What a hall runs on its own |
| §9 | Measuring a hall |
| §10 | Naming a document: one standard for every hall |
| §11 | Intake: the daily loop and the intake agent |
| §12 | A hall's app in the building: the native wall item |

---

## 1. A hall is a volume, recorded in `districts.json`

**Every hall is one APFS volume, `/Volumes/<VOLUME>`, with exactly one authoritative host**
(`districts.json` → `hosts`) and never a second writable copy. The GM's `districts.json` is
the only place a hall's facts are written down; nothing hardcodes a path.

| Field | Holds |
|---|---|
| `name`, `volume`, `root`, `host` | what the hall is called, its volume, and the machine it lives on |
| `mandate` | what the hall is for, in one paragraph — the test for whose a thing is |
| `repos` | the prefix and the three kinds of repo ([`github-standard.md`](github-standard.md) §1) |
| `status` | the shape of the volume (below) |
| `messages`, `intake`, `brain`, `primary`, `contents` | where the trees are, and what else is on the volume |
| `notes[]` | the hall's **corrections** to the GM, **retired 2026-11-21** and kept as history; a hall now sends requests ([`communication-standard.md`](communication-standard.md) §4) |

**`status` says how the volume is shaped** (`districts.json` → `_status_meaning`):

| Status | Means |
|---|---|
| `flattened` | the volume root **is** the one project: `CLAUDE.md` and `.claude/` at the root |
| `hub` | the volume root is the hall itself — `CLAUDE.md`, `brain/`, `messages/`, `intake/`, `data/` at the root — and each project is a named subfolder; none owns the root |

Since the three-repo rule (Option A, 2026-11-16) every hall's root is its `<prefix>-district`
repo and every app is under `apps/`, so the difference is narrowing to history: a `flattened`
hall's root once *was* its app. **Never write to `/Volumes/<NAME>` without confirming it is a
real mount point** (`os.path.ismount`, not `exists`): an unmounted volume lets macOS create an
empty folder of that name on the boot disk, and the data vanishes when the real volume returns.

---

## 2. The layout: what must be at the root

```
/Volumes/<VOLUME>/                <prefix>-district: the volume root IS the district repo
├── CLAUDE.md                     the contract (§3)
├── .claude/                      settings and hooks, skills, agents, worktrees (§5)
├── docs/                         the shipped standards, and this hall's sop-local.md (§4)
├── rules/                        the hall's rules as data: brain.json, document_naming.json, intake.json …
├── references/                   the hall's configuration as data: workflow.json, labels.json, ship.json, launchd/ …
├── bin/                          the hall's tools: ship, board, session-files, and its own
├── staff.json                    the roster of the hall's roles (§6)
├── .gitignore                    ignores every nested checkout and every shared tree
├── brain/                        <prefix>-brain, its own repo     ┐
├── apps/<app>/                   <prefix>-<app>, one per app      │ ignored by the
├── rooms/<id>/                   <prefix>-<id>-district, one room │
├── documents → the cloud         what the hall received           │ district repo
├── messages → the cloud          notices, messages; archive/      │
├── intake → the cloud            files arrived, not yet filed     │
└── var/                          generated: logs, caches, the search index, run state ┘
```

**Required** in every hall: `CLAUDE.md`, `.claude/settings.json`, `docs/`, `rules/brain.json`,
`brain/`, `documents`, `messages`, `intake`, `.gitignore`. **Required where they apply:** `references/` and
`bin/` with the shipping tools wherever the hall commits code (every hall but `teach`); `apps/`
wherever the hall builds something; `staff.json` wherever the hall has roles of its own; a `Makefile`
with a `test:` target, or a documented test command, wherever there is code to test.

**Optional, with a reason:** `rooms/` (the hall's subdistricts, each its own two repos;
[`github-standard.md`](github-standard.md) §1), `data/` (structured data a hall's skills keep, gitignored),
`infra/`, `scripts/`, `tests/`, `.github/workflows/`, `logs/`, `README.md`,
`.worktreeinclude` ([`github-standard.md`](github-standard.md) §4), `.mcp.json`, `.vscode/`,
and `keys/` (**public** key halves only — the private halves never touch disk;
[`security-standard.md`](security-standard.md)). The lists are data, in the GM's
`readiness.json` → `root_layout`, which `bin/district-ready` reads.

**Anything else at the root is a stray**, and a stray is either explained in the hall's
`sop-local.md` — in a section headed `hall-standard.md §2`, naming each in backticks — or removed (copy, verify, then remove; never inside Dropbox). An app belongs
under `apps/<app>/`, media and records in `documents/` or `data/`, and an old folder kept after a
move is removed once the move is verified.

### 2a. A room: a small hall

A hall's room (`rooms/<id>/`) is laid out and measured as a small hall. Every rule for it,
its layout included, is [`room-standard.md`](room-standard.md).

**Received is not generated.** What a hall's jobs write — logs, undo records, caches, reports —
goes under `var/` (or `logs/`), never into `documents/`, `messages/` or `intake/`
([`brain-standard.md`](brain-standard.md) §3).

---

## 3. The contract: `CLAUDE.md`

**Every hall's `CLAUDE.md` is built from the GM's `templates/district-CLAUDE.md`**, and it is the
directive: what to do, what not to do, and where things are. Claude Code loads it at the start of
every session, so **a rule a hall must follow has to be in its contract** — an agent that has to
follow a link to learn a restriction will sometimes not follow it.

**What loads at start is what every session needs, and nothing else.** Claude Code loads the
contract, everything it `@imports`, every `.claude/rules/*.md` without `paths:`, each skill's name
and description, and the memory index into every session and every subagent. Each is paid for on
every start, and the longer it is, the less reliably it is followed
([memory](https://code.claude.com/docs/en/memory.md#claude-md-files)).

- **Rules, pitfalls and pointers only.** The reason behind a rule is a brain note, cited with
  `Why: [[note-id]]`; a "corrected on …" paragraph is history; and what the tree or the code
  already shows — layouts, dependency lists, cron and launchd tables, environment variables —
  is not repeated.
- **Point, don't copy.** A rule written in a standard, a skill or a doc is named from the
  contract, never restated in it.
- **Detail loads when the work needs it.** Topic detail goes in a `CLAUDE.md` inside the folder
  it is about, or in `.claude/rules/` with `paths:`, both of which load only when an agent
  touches matching files; or in `docs/`, named from the contract. **`@import` saves nothing**:
  an import loads at start with the file that imports it.
- **Skills and tools a worker never uses stay out of its session.** A skill used only on
  request sets `disable-model-invocation: true`, so its description costs nothing until it is
  called. A worker starts without the MCP servers it does not use.
- **The memory index stays under 200 lines:** Claude Code loads only the first 200 and drops
  the rest without a word.

**Size is measured, never gated.** `bin/context-budget` reports each hall's lines and bytes, so
the effect of a change can be seen; no count is a limit.

**The hall's own words come first.**

| Order | Section | Whose words |
|---|---|---|
| 1 | Mandate · Layout · Messages and intake · Apple data · Working with the brain · Working style · the hall's own rules | **the hall's**, written for its own volume |
| 2 | The standards | the hall's, naming the standards in its `docs/` |
| 3 | **How the GM and districts talk** | **the GM's, byte-identical in every hall** |
| 4 | **Hard restrictions** (the four conventions, the security rules, the seven rules for agents that act) | **the GM's, byte-identical in every hall** |

**The GM's blocks are written short:** each rule in one or two lines, pointing to its reason in
[`security-standard.md`](security-standard.md) §4 or [`doe-framework.md`](doe-framework.md)
§13–14, so most of what loads is the hall's. **They are never edited in a hall.** A change is
made in the GM's `CLAUDE.md` and the template together, and reaches the halls by notice. The
GM's `scripts/check-district-contracts.py` reads the rules under *Hard restrictions* from every
hall's contract and reports which are missing; a rule placed outside that heading is
documentation, and a rule inside it is checked.

---

## 4. `docs/` and `sop-local.md`

`docs/` holds **the standards the GM ships** (`standards.json` → `documents`) and **the hall's
own `sop-local.md`**. **A hall reads each standard through a link** to the GM's copy, made by
`bin/ship-standards` as `bin/hall-layers` links the org skills, so a merged change reaches it at
once. The links stay on this Mac: the hall does not track `docs/standards/` in its git (it
untracks its old copies itself, `git rm --cached`), and each link is in its `.git/info/exclude`.
A hall in `standards.json` → `standards_not_linked` keeps SHA-256-verified copies instead:
Teaching, whose repository travels to the university laptop. A standard is **never
edited in a hall**: `bin/guard-standards` refuses an edit through a link, and the next ship
overwrites a copy. A hall that works differently writes it in `sop-local.md`, one section per
change:

```markdown
## <the standard and section this changes, by its current name>

- **Change:** what this hall does instead.
- **Why:** the reason, so it can be judged later.
```

A change that would suit every hall belongs in the GM's copy: ask for it with a
`request/standard` issue on `gm-district` ([`organization-standard.md`](organization-standard.md) §4). **A copied standard the GM has retired is deleted by the hall** — `bin/ship-standards`
never deletes — and the GM's notice names which. A hall's own developer documentation sits
beside them in `docs/` under names that do not collide with a standard's.

---

## 5. `.claude/`: settings, hooks, skills, agents, worktrees

| Path | Holds | In git? |
|---|---|---|
| `settings.json` | permissions and **hooks** | yes |
| `settings.local.json` | this Mac's approved commands — **never a secret** (running a command once with an inline secret saves it here as an allow-rule) | no |
| `skills/` | the hall's own skills, the org skills linked in by `bin/hall-layers`, the kit's skills installed by `bin/ship-brain` ([`doe-framework.md`](doe-framework.md) §12) | the hall's own only |
| `agents/` | agent definitions, written against `templates/employee.md` | yes |
| `worktrees/` | `m<N>-<slug>` and `squad-<N>` ([`github-standard.md`](github-standard.md) §4) | no |
| `heartbeat/` | a timed job's heartbeat (§8), the folder ignoring itself | no |

**Hooks every hall registers in `settings.json`:**

- **`bin/session-files record`** on `PostToolUse` (matching `Edit|Write|NotebookEdit|MultiEdit|Bash`)
  and on `SessionStart`, resolving the root from the caller's own cwd — the attribution
  `bin/ship` stages from ([`github-standard.md`](github-standard.md) §5).
- **A `PreToolUse` guard** where the hall has a policy to enforce, failing closed
  ([`doe-framework.md`](doe-framework.md) §11).

`readiness.json` → `required_hooks` lists the hook events `bin/district-ready` looks for.

---

## 6. The people: seats, orders and the roster

**Seats are sessions.** `bin/dm` gives every hall a tmux session `dm-<code>` with three seats —
the **District Manager** (`hero`) and **workers 1 and 2** (`squad-1`, `squad-2`) — each started
by Marshall Inc with its orders from `~/.claude/marshall-inc/orders/<code>-<seat>.md`. Which body does
what across the organisation is [`organization-standard.md`](organization-standard.md). The GM adds its
residents (Computer Manager, Computer Worker). Always start a session with `bin/dm`:
a tmux server started any other way copies one shell's secret token into every pane.

**A hall has roles, and no officers or employees.** A role is an agent summoned by name when
someone wants it; nothing starts one on a schedule, so a hall has no shifts and no shift plist
(Bryan, 2026-11-21, gm-district #468). **One exception, by name: the hall's intake agent**
(§11), which one timed job starts once a day to run the intake loop (Bryan, 2026-11-23). It is
the only agent a schedule starts, and it is a role like any other: in the hall's `staff.json`,
with its `allowed_tools`. A hall's roles are in **its own** `staff.json`, which the
GM's `bin/_roster.py` merges with the GM's roster; **a name registered in both is a refusal, not a
precedence rule.** A role's `allowed_tools` is pre-approval *and* enforced policy — adding to it
grants real access.

**Agents only propose** until a lane climbs the ladder in [`doe-framework.md`](doe-framework.md) §14.

---

## 7. What a hall owns outside its volume

| | The hall's | Rule |
|---|---|---|
| **1Password** | the vault named after its volume (`PROJECTS` for `proj`) | a hall writes only into its own vault; `~/.zshenv` gives a session under `/Volumes/<VOLUME>` that vault's token and no other ([`security-standard.md`](security-standard.md)) |
| **The hall's cloud** (Dropbox, unless `districts.json` → `cloud` assigns another) | `DISTRICTS/<VOLUME>/` — `documents/`, `messages/` and `intake/` | linked into the volume; never moved or deleted inside without copy, verify, then act |
| **GitHub** | its `<prefix>-*` repos | [`github-standard.md`](github-standard.md) |
| **Reminders** | a list | the hall creates and completes its own; not the task list (GitHub is) |
| **Contacts** | a group | one person, one record — someone in two halls is one contact in two groups |
| **Notes** | a folder | a capture folder only: drained into `intake/`, then cleared |
| **Calendar** | none — **one calendar, `Home`, for every hall** | write only on Bryan's instruction, through `bin/cal-event` |
| **Bookmarks** | a Chrome folder | the GM keeps the one tree |

The containers are mapped in the GM's `routing.json` → each district's `containers`. **A hall
never writes into another hall's** volume, vault, list, group or folder; what belongs
elsewhere is sent to that hall with `message-district`, the one way into its `messages/` or `intake/`.

**Event, reminder or issue: choose by what has to happen.**

- **Someone has to be somewhere** → an **event** on `Home`, through `bin/cal-event`.
- **Someone has to do a small, often repeating thing** → a **reminder** in the hall's own list,
  through `bin/apple reminders`, **with a due time, so it rings** on the phone and the Watch. A
  reminder with no time is a note nobody is told about.
- **Something has to be built or delivered** → a **GitHub issue** ([`github-standard.md`](github-standard.md)
  §2), never a reminder.

Every write is a dry run first, then logged and read back. An agent may add a reminder; it never
completes them in bulk, and it proposes a completion rather than making it.

---

## 8. What a hall runs on its own

A **timed job** is anything the machine runs without being asked: a nightly check, a fetcher, a
watcher. It is never an agent's shift: a hall has no employees (§6). **The one exception is the
daily intake run (§11)**, which starts the hall's intake agent; every rule below applies to it.

- **launchd is the only scheduler.** A cloud routine (`/schedule`) cannot reach Apple data, and
  `/loop` dies with its session. **A declared cadence is not a schedule: the plist is the hire,
  and the JSON is the paperwork.**
- **The plist is the declaration.** Every job's plist is tracked in the hall's district repo under
  `references/launchd/`, named for its label, `com.example.<prefix>-<name>.plist`, where
  `<prefix>` is the hall's `repos.prefix` in `districts.json`. The copy in `~/Library/LaunchAgents`
  on the hall's host is an install of that file, byte for byte, and is changed by changing the
  tracked one and installing it again.
- **Declared, installed and loaded agree, and `launchctl list` is the evidence.** A job unloaded
  stays unloaded only until something reloads it; a job that runs and is not declared is a stray.
- **A failure reaches a person, not a log.** Every job writes a heartbeat (`bin/heartbeat`).
  Exiting non-zero into a log file is silent: an hourly job alerts after N failures in a row, once,
  and once on recovery; a daily job reports every run, so a missing report is the alarm. Keep the
  failure counter off the thing that might fail — a counter in the synced folder cannot count a
  sync-folder outage.
- **macOS grants are per binary.** A job reading Apple data goes through `bin/apple`, the one
  granted helper, and never runs `op`: a headless job cannot answer a prompt. A job that needs Full
  Disk Access needs it on the *exact* interpreter it runs (for Homebrew Python, the versioned file
  under `Cellar/…/bin/python3.x`), and an upgrade of that interpreter revokes it.
- **A job that acts climbs the ladder** ([`doe-framework.md`](doe-framework.md) §14). One with no
  way to undo it only proposes.
- **Every login a hall's work depends on is watched before it ends** (gm-district #661). A
  sign-in that lapses stops work with nothing in a log: Claude Code's has an end date, GitHub
  drops a self-hosted runner it stops hearing from, and a locked 1Password stops every SSH key.
  The host's logins are data in the GM's `references/logins.json`, read by `bin/logins`, and an
  hourly heartbeat fails when one is ending within its window, has ended, is signed out, or could
  not be read; *could not check* is never *fine*. A hall that depends on a login of its own (an
  API token with an expiry, a service's session) adds it there, by request. **Renewing is a
  person's job**, at the desk: the check says which login and how, and never stores or prints a
  token.

---

## 9. Measuring a hall

`bin/district-ready <code>` measures a hall against this standard. It is **read-only and
advisory** — it gates nothing — and it reports each check as `ok`, `not-ready` or
**`could not check`, which never counts as passing.** Its thresholds and exemptions are data in
`readiness.json` (`teach` is exempt from the tracking checks).

**What it should measure, one check per section of this standard:**

| Check | Section |
|---|---|
| the volume is mounted, and `districts.json` names it | §1 |
| no stray at the root that `sop-local.md` does not explain | §2 |
| `CLAUDE.md` carries the byte-identical blocks (`check-district-contracts.py`) | §3 |
| the hall's own words come first, and the GM's blocks match the template; lines and bytes reported, never gated (`bin/context-budget`) | §3 |
| the standards are linked or current, none is a broken link, and no retired one is left (`bin/ship-standards`) | §4 |
| `sop-local.md` is real, not the template, and cites current section names | §4 |
| `session-files` is registered; `bin/ship` is present and speaks GitHub | §5 |
| `references/workflow.json` exists where the hall tracks with labels | §5 |
| `messages/` and `intake/` are links into the hall's cloud that git ignores | §2, §7, [`communication-standard.md`](communication-standard.md) §2 |
| no notice or message open at the top of `messages/` | [`communication-standard.md`](communication-standard.md) §6 |
| every plist in `references/launchd/` is installed byte for byte and loaded, and no `<prefix>-` job runs undeclared | §8 |
| the repos match `districts.json` (`bin/repo-check`); the brain kit is current (`bin/ship-brain --check`); the skills pass `bin/skill-check` | [`github-standard.md`](github-standard.md), [`brain-standard.md`](brain-standard.md), [`doe-framework.md`](doe-framework.md) |
| the hall names from the shared list, and every document filed since the shared list reached it passes `naming.py check` against it | §10 |
| the intake agent is in `staff.json`, its daily job is declared, installed and loaded, and today's report exists | §11 |
| the oldest item at the top of `intake/` is younger than the threshold in `readiness.json` | §11 |
| every item on the hall's wall, once its native items are applied, is a complete native item (package, view, config; a server on the M4 has its `away` route) or a page (`https`, not this Mac) (`readiness.json` → `wall`) | §12 |
| each factory app is in an app repo, has what the factory standard lists, is at the GM's pin, and has its readiness evidence on file (`readiness.json` → `factory_app`) | [`software-factory.md`](software-factory.md) §2, §3, §6 |

---

## 10. Naming a document: one standard for every hall

**Every hall names a document the same way, from the same list.** A flat `documents/` has no
folders, so the name is its only index ([`brain-standard.md`](brain-standard.md) §3), and a name
is only an index if it means the same thing in every hall. Until 2026-11-23 each hall kept its
own list, and they had drifted: across seven halls, 13 issuers and 22 types were spelled two
ways (`biz` and `southern-fabric`, `ga-dor` and `georgia-dor`, `notice` and `letter`), so "every
state tax department document" could not be asked of the organisation.

```
<date>-<issuer>-<type>[-<ref>].<ext>        2026-10-31-jones-county-water-statement.pdf
<the same stem>.json                        its sidecar
```

| Part | Rule |
|---|---|
| **date** | **The document's own date** (an invoice's date, a statement's period end, a notice's date, a signature's date), **only as precise as the document says**: `YYYY-MM-DD`, `YYYY-MM` or `YYYY`, never padded with a guessed day, and `undated` for a document that carries none. **Never** the day it was scanned, received or filed. |
| **issuer** | **Who the document is from**, as one slug from the shared list. An institution, never a person's first name. One organisation has one slug in every hall. |
| **type** | **What the document is**, one word from the shared list, **with no dash in it**, so a name parses back into its parts one way. One word means one type in every hall. |
| **ref** | Optional: what tells two of the same issuer and type apart (an account's last digits, a period, a person). Words, at most a few. |
| **collision** | A name already taken takes `-2`; nothing is ever overwritten. A rename moves the document and its sidecar together, in one step (`rename.py`). |

**The shared list is one, and the Boardroom keeps it.** The issuers (with their aliases), the
types and the date rule are one list for the whole organisation, kept in the general manager
(`references/naming.json`, written by `bin/naming-merge` from every hall's rules and
`references/naming-resolutions.json`, where each conflict between halls is settled) and
installed in every hall by `bin/ship-brain` as `.claude/skills/shared/naming.json`. A hall on
the list says `"vocabulary": "shared"` in its `rules/document_naming.json` and keeps none of
those keys itself; the kit refuses a hall that says shared and still keeps them.

- **A hall keeps only what is truly its own**, in its `rules/document_naming.json`: which types
  need no note (§11, step 5), whether `bin/doc-check` fails a document no note cites (`gate`),
  and how it renders each kind of file (`extensions`: a hall may keep pictures as pictures, or
  refuse them until they are a PDF with text).
- **An issuer the list lacks is added in the Boardroom.** Because the Boardroom names almost
  everything it routes (§11), it meets a new issuer first and adds it there. A hall that receives
  a document directly and finds no issuer for it files it under the nearest true slug or
  `unknown`, flags the name in the report (§11), and the Boardroom adds the issuer; the hall's
  next run renames it.
- **Whoever reads a document first names it.** For a file that arrives through the Boardroom,
  that is the Boardroom, which sends it already named and with its sidecar (§11). The hall checks
  the name (`naming.py check`) and the sidecar's hash rather than naming it again. A hall that
  disagrees renames with `rename.py` and the shared list is corrected in the Boardroom, so the
  same mistake is not made twice.
- **A hall moves onto the list in its own session.** `bin/naming-merge --renames <hall>` lists,
  read-only, every filed document whose name uses a slug the list merged away, with its new
  name (79 across the organisation on 2026-11-23). The hall renames them with `rename.py`, a log
  line each, updates its own `issuer_fallback_only` to the list's slugs, removes its own
  vocabulary and sets `"vocabulary": "shared"`. Until then it names from its own list.

---

## 11. Intake: the daily loop and the intake agent

**Every hall empties its `intake/` every day, by one loop, run by its intake agent.** A file
that sits in `intake/` for months is a file nobody can find, and the week the Boardroom's own
backlog was cleared (2026-11-23) it held scans from May. `intake/` is a landing zone, not a
store ([`communication-standard.md`](communication-standard.md) §2).

### The two stages

A file passes through two intakes: the **Boardroom's**, where almost everything arrives (the
scanner, email, the backlog), and then the **owning hall's**. The Boardroom reads each file once,
so the hall does not read it again.

**Stage one — the Boardroom, before anything is sent:**

1. **Open it.** Extract its text; OCR a scan that has none (on a copy: the original is never
   touched).
2. **Decide whose it is.** One fact: whose it is, never what it is about
   ([`communication-standard.md`](communication-standard.md) §1). The rules that decide it are
   data in the Boardroom, and every district they name must exist in `routing.json`.
3. **Name it and write its sidecar**, to §10, with the shared list.
4. **Send the pair** (`bin/file-doc`), SHA-256 verified, into the owning hall's `intake/`.

A file the Boardroom cannot open, or cannot place, stays in its intake for Bryan.

**Stage two — the hall, the same six steps for every item, in this order:**

| Step | What | An item that fails it |
|---|---|---|
| 1. **It opens** | It can be read. A file that came named, with its sidecar, opens if the sidecar's `sha256` matches the file. Anything else is opened: a PDF has pages and text (or is OCR'd), an Office file is a valid archive and renders, a picture decodes. | **Proved unreadable**, then sent to the Boardroom (below). Never deleted. |
| 2. **It is worth keeping** | Measured against the hall's retention rules, not a feeling: a receipt that substantiates a tax claim or an HSA payment is worth keeping however small. | **Proposed for discard**, with the reason. Never discarded by the agent. |
| 3. **It is this hall's** | The owner test is the one the Boardroom used: whose it is. | **Sent straight to the hall it belongs to** (`message-district`'s `send-file.js --to <code>`), and the Boardroom told it was misrouted so it corrects the rule; the hall's copy is set aside. An owner the hall cannot name is left for Bryan, in the report. |
| 4. **It is filed** | Its §10 name passes `naming.py check` with no part guessed; it moves into `documents/` with its sidecar, copied, SHA-256 verified, then removed. | **Left in `intake/`**, with the part that needs a person named in the report. |
| 5. **It has a note** | **Every document filed gets a brain note** saying why it is kept, with `authority:` naming the filed file ([`brain-standard.md`](brain-standard.md) §3). A hall may list types that need none (§10). | The document is filed; the missing note is in the report. |
| 6. **The index is rebuilt once** | After the last item, not after each. Then the hall's search test (`search.py --eval`) runs, so a batch that pushes the right answers down is caught the day it lands. | The run reports *could not check*. |

**Proved unreadable** means three things, all recorded in the log line: the file is **not a
cloud placeholder** (`st_blocks > 0` after a read to hydrate it); **two different extractors
both failed** (for a PDF, text extraction and a render to an image; for an Office file, an
archive test and a render; a picture, a decode); and its **SHA-256, size and each extractor's
error** are written down. Then it goes to the Boardroom's `intake/` with a short message saying
what was tried; after the verified send the hall sets its own copy aside (below), because
`send-file.js` never removes the sender's original; and the Boardroom keeps
every unreadable file from every hall for Bryan to look at one by one.

**Nothing in `intake/` is deleted.** `intake/` is in the hall's cloud, where a delete reaches
every device. An item is filed, sent, or **set aside**: copied into the hall's logs tree
(`retired/`), SHA-256 verified, then removed, with a log line that `--undo` can walk back.

### The intake agent

**Each hall has one intake agent**, a role named **`<prefix>-intake`**, `<prefix>`
being the hall's `repos.prefix` in `districts.json` (`sf-intake`, `mf-intake`, `jcxc-intake`), defined in the hall's `.claude/agents/` and registered in its own `staff.json` with
its `allowed_tools`. The prefix keeps the name from colliding with another hall's when the
rosters are merged (§6). The hall's District Manager can summon it at any time; **one timed job
starts it once a day.**

- **The job** is `com.example.<prefix>-intake-daily`, a plist in the hall's
  `references/launchd/`, on the hall's authoritative host, under every rule of §8: a heartbeat,
  a failure that reaches a person, grants on the exact binary, and never `op`. The Boardroom's
  run comes first each morning, so the halls' runs see what it sent; the halls' runs are
  staggered, so they do not compete for the same machine.
- **Its guards:** it refuses to run when the volume is not a mount point; it refuses an item
  that is an online-only placeholder that will not hydrate; it holds a lock, so two runs never
  overlap; and it exits 75, *could not check*, rather than report an empty `intake/` it could not
  read.
- **What it may do alone, and what it only proposes** — autonomy is granted by verifiability
  ([`doe-framework.md`](doe-framework.md) §13–14):

| Action | Alone? | Why |
|---|---|---|
| Open, survey, check the owner | **yes** | read-only |
| Send a proved-unreadable or not-ours item to the Boardroom | **yes** | a verified copy into the one intake that routes; logged |
| File a document whose name passes `naming.py check` with no part guessed | **yes** | hash-verified, and `--undo` reverses it |
| Write a note that passes `validate.py` | **yes** | `--undo` removes it while it is unedited |
| Rebuild the index once, and run `search.py --eval` | **yes** | the index is a cache; the test is a measurement |
| File a name with a guessed part, or a low-confidence owner | **no — in the report** | the guess is the judgement |
| Discard an item with no value | **no — proposed** | it has no way back |
| Anything that looks like a credential | **no — left in place, flagged** | a credential goes to the hall's vault by Bryan's hand, never into a document or a note |

- **The report.** Every run writes one dated file, `var/intake/reports/<YYYY-MM-DD>.md`
  (generated, so under `var/`, §2a): what was filed and the note each got, what was sent to the
  Boardroom and why, what was left and the one thing each needs, what is proposed for discard,
  the search test's score, and the log lines written. **A daily job reports every run, so a
  missing report is the alarm** (§8). If anything was left for Bryan, the run also puts **one
  reminder, with a due time, in the hall's own Reminders list** (§7), naming the report.

---

## 12. A hall's app in the building: the native wall item

**Marshall Inc is one app, and every hall's app is native and travels inside it** (Bryan,
2026-11-25). A hall that wants an app does not ship one: it publishes its app as a **Swift
package**, and the building opens it from the hall's wall, inside the window. If each hall
shipped its own app, each would need its own certificate, notarizing and installer; the building
has them once, and they are the Computer Room's
([`organization-standard.md`](organization-standard.md) §1).

**Why native.** Marshall Inc updates itself on every Mac Bryan sets up, and a release carries the
app and every hall package pinned in it, and nothing else. An app a hall runs on the M4 and hangs
as a `http://127.0.0.1:<port>` page opens nothing on any other Mac, so it never reaches the
people using them.

### Two kinds of wall item

A hall's wall (the live Marshall Inc `config.json` → `districts.<ROOM>.wall`) holds items of one
shape, `{ id, shape, title, … }`, where `shape` is only the picture hung on the wall. **What the
item opens is the one field that follows:**

| Item | Carries | Opens | Is |
|---|---|---|---|
| **The native wall item** | `native`: `{ "package", "view", "config" }` | the package's one root view, inside the window | **a hall's app**: the only kind |
| **A page** | `url`: a claude.ai artifact link, or a public `https://` site | the page in the browser | a document or a page, never an app; it opens on any Mac |
| **A local web item** | `url`: `http://127.0.0.1:<port>` or `localhost` | a web view of a server on the M4 | **not hung.** A hall that has one ports it to a package |

```json
{ "id": "money", "shape": "picture", "title": "Household money",
  "native": { "package": "HouseholdMoneyKit", "view": "HouseholdMoneyRoot",
              "config": { "server": "https://…" } } }
```

An item carries `url` **or** `native`, never both; one with both, or with neither, is hung with
its error and opens nothing, so the mistake shows. A package the building does not list (below)
is the same error. A hall with no package hangs no app; it may hang pages.

### What the hall provides

- **The package, in the hall's own app repo** (`<prefix>-*`, [`github-standard.md`](github-standard.md)),
  with its code, its tests and its tags. **The building never edits it**; a change is the hall's,
  made in its repo and tagged.
- **One public root view**, a SwiftUI `View` built from one value the building hands it (below).
  That view is the package's whole surface: nothing else is called.
- **Tests the building can run.** The package's tests run in the building's `swift test`, so a
  release of Marshall Inc cannot include a package whose tests fail.
- **A platform no newer than the building's.** The package declares the building's macOS
  version or an older one.
- **A package that runs on any Mac Bryan sets up, not only the M4.** No `/Volumes/<NAME>` in its
  source (what it needs of the hall comes from its `config`), no secret built in, and no server
  it expects to find on the Mac it runs on. Every server it talks to is named in its `config`;
  one that lives on the M4 is reached from another Mac through the building's own server, which
  the Computer Room configures, never by the package guessing an address.

### What the building gives it

- **Its config**, from the wall item's `config`, read at build — a server's base URL above all.
  **Nothing that varies is hardcoded in the package**, and a package fails loudly when a value
  it needs is missing (convention 2), rather than reaching for a default.
- **The window chrome**: the header, Back, and Previous/Next. The package draws only its own
  content.
- **A Keychain access group for the hall's own login.** A token a hall's server issues is stored
  there and nowhere else, and never in a file.
- **Signing, notarizing and releases**, under the building's one identity
  (`apps/building-native/references/signing.json` in the Building hall). The building lists each
  package as a SwiftPM dependency **pinned to a tag** the hall published; the Computer Room bumps
  the pin when the hall tags, and `Package.resolved` is the evidence of what shipped.

### What a package may never do

- **Reach another hall.** No other hall's server, data, vault or Keychain group. A hall's server,
  keys and data stay its own (§7).
- **Run a shell or `op`.** No `Process`, no shell, no 1Password: a package runs inside the app
  every hall shares, and inherits nothing a session holds.
- **Write anywhere but its own server's API.** Its data lives on the hall's server; it writes no
  file on the Mac and no Apple data (that is `bin/apple`'s, §7).
- **Send anything off the Mac that Security has not reviewed.** What a package sends, and where,
  is read by Security ([`security-standard.md`](security-standard.md)) before the building first
  pins it, and again when it adds a new destination.

### Asking for it, and how a new version reaches every Mac

A hall asks the Computer Room to host its package by message (`message-district`, to `bldg`),
naming the repo, the folder, the view and the config it needs. After that, each new version
takes the same four steps:

1. **The hall tags its package** and messages `bldg` with the tag.
2. **The Computer Room pins it** (`Package.swift` and the project file), runs the hall's tests in
   its own `swift test`, and merges.
3. **The Computer Room asks Bryan "publish build N?"** and, on his yes, publishes the Marshall
   Inc release: notarized by Apple, then released to every Mac. The release is the one door; a
   hall never ships its own Mac app, certificate or installer.
4. **The Computer Room answers the hall's message** only when it can say
   *pinned at <tag>, released in build N*, or why it is waiting
   (*pinned, waiting for Bryan's go to publish build N*).

Nothing about it comes to the Boardroom unless the hall wants this standard changed
(`request/standard`, [`organization-standard.md`](organization-standard.md) §4).
`bin/district-ready` fails a hall whose wall hangs anything but a complete native item or a page (§9).
