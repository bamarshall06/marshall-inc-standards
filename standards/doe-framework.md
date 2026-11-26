# The skill standard — DOE: how skills, agents, rules and hooks are built

> **Public edition.** A working standard from Marshall Inc, shared as it is used. Real names, paths and accounts are replaced with generic ones (floors such as *Business* and *Teaching*, `/Users/<you>`, `<your-github-user>`). References like `gm-district #372` point to private repositories and are kept only as history. Start with the [README](../README.md).

## Summary
> The skill standard: the orchestrator does not compute and the scripts do not reason. How a skill, an agent, a rule and a hook are built and what each cannot do, where a skill lives, when to use several agents, and the autonomy ladder with the D1/D2/D3 readiness rungs.

**Status:** org-wide standard as of 2026-11-19 (gm-district #373, milestone #24). It keeps the
filename `doe-framework.md` because skills in every hall link to it, and its sections 1–10 keep
their numbers (the anatomy is still §3, the skeleton still §6a). On 2026-11-19 it absorbed
three documents, which are in git history: `claude-folder.md` (2026-09-25, §11–§12),
`doe-framework-multi-agent-plus-setup.md` (2026-09-26, §13–§14) and `district-readiness.md`
(2026-09-26, §14). **Applies to:** every skill, agent, rule and hook in every hall. Entry point:
[`sop.md`](sop.md).

**The one sentence:** *the orchestrator does not compute, and the scripts do not reason.*

| | |
|---|---|
| §1–§2 | What DOE is, and why rules live in data |
| §3–§6 | A skill: its anatomy, its conventions, what it costs to run, where the templates are |
| §7–§9 | Lessons that cost real time: sequencing, derived state, anti-patterns |
| §10 | Starting a skill in a hall |
| §11 | The four kinds of thing in `.claude/`, and what each cannot do |
| §12 | Where a skill lives, and the DOE contract `bin/skill-check` enforces |
| §13 | Several agents: when to split, the hand-off, what actually enforces a boundary |
| §14 | The autonomy ladder, and D1/D2/D3 |

---

## 1. What DOE is

DOE (Directive / Orchestration / Execution) is **Nick Saraev's** framework for building reliable
AI agents. LLMs are stochastic and compound errors across chained steps, so push the
deterministic work into scripts and keep the model narrowly scoped to routing and judgement.

| Layer | Artifact | Role |
|---|---|---|
| **Directive** | `SKILL.md` | The SOP: when to trigger, the domain rules, the script inventory, the safety constraints. Read by people and by Claude. |
| **Orchestration** | Claude Code | Reads the directive, interprets intent, picks the script and the flags, shows the person, handles retries. |
| **Execution** | `scripts/*.py` (or Go, shell, Swift) | Deterministic work: queries, arithmetic, API calls, file I/O, validation. **Never ask the model to "figure it out".** |

**Why the split:** if each step is about 90% accurate, five chained model-only steps land at
0.9⁵ = 59%. Pull the deterministic work into scripts and the model routes once, not five
times, and the scripts are reproducible.

## 2. Rules live in data

**If a rule could change without a code change, it is data — a JSON file in `rules/` or
`references/`, or a database row — never a constant in source.** That is convention 3 of every
hall's `CLAUDE.md`. In an application it gives three tiers: the **data** is the source of truth
for rules and configuration (prompts, mappings, thresholds); the **skills** manage that data,
orchestrate and validate; the **application** executes, stateless over the rules.

---

## 3. Anatomy of a skill

```
.claude/
  skills/
    shared/
      skill_db.py           # cross-skill utilities (DB access, colored output)
    <your-skill>/
      SKILL.md              # the directive
      scripts/
        list_things.py      # read-only inspection
        update_thing.py     # mutation, --confirm required
        validate.py         # assertions, non-zero exit on failure
        pipeline.py         # orchestration across several scripts
```

### `SKILL.md` contents

**Frontmatter** (YAML — required):

```yaml
---
name: <your-skill>
description: >
  One sentence describing what this skill does and when to trigger it.
  Used by the orchestrator to decide when this skill is relevant.
---
```

The `description` is a trigger contract, not a tagline. Write it so the orchestrator can match user intent confidently. Good: *"Trigger when the user says 'X', 'Y', or explicit `/your-skill`."* Bad: *"Handles various X tasks."*

**Body** — the directive itself. Typical sections:

- **Key concepts** — domain rules the orchestrator must respect
- **Script inventory** — table of scripts with one-line descriptions
- **Cost / safety constraints** — dry-run policy, credentials, rate limits
- **Self-annealing notes** — lessons learned, so failures improve the directive over time

**Write every instruction plainly, with its reason** (2026-11-25, #649). This holds for a
`SKILL.md`, an agent definition and a hall's `CLAUDE.md` alike. One calm line and the *why*
behind it, never capitals, stacked bold or "IMPORTANT" to make it land. Anthropic's guidance for
current models: *"Older models sometimes needed the emphasis. Current ones follow a plainly
worded line, and piling on emphasis mostly makes Claude over-apply the rule"*
(code.claude.com/docs/en/best-practices). Bold marks the one phrase a reader must not miss.
Something that must never happen is not made louder: it becomes a deny rule or a hook (§11). Keep in a contract what Claude cannot find in the repo; leave out what it
can read for itself. Claude Code's `/doctor prompt-audit` finds instructions written for older
models, files and commands that no longer exist, and files that contradict each other. It
changes nothing until asked; a hall runs it after changing its contract, skills or agents.
Existing wording, the hard restrictions above all, changes only on Bryan's yes to a finding
(gm-district #656).

Section 12 is the part of this anatomy a script checks: `bin/skill-check` enforces five rules
on every skill in every tier.

---

## 4. Conventions that make it work

These are the small habits that keep DOE from decaying.

- **Dry-run by default; `--confirm` or `--commit` to mutate.** Every script that writes prints what it *would* do and exits 0 when called without the flag. Both spellings are in use in this organisation; a script's `--help` says which.

  **Who supplies the flag is a separate decision from whether the flag exists.** The guard's job is to stop a half-formed or mistaken invocation from defaulting to a write. Whether the *human* must approve each diff depends on the blast radius: staging every write for approval is right for irreversible or outward-facing work, and pure friction for reversible writes to the user's own data. **`[example]`** — in this project the orchestrator supplies `--confirm` directly and reports afterwards, because approving every calendar entry cost more than it protected. Verification moved from before the write to after it, which is where it has actually caught bugs: a timestamp the store silently normalised, a batch that half-failed.
- **Shared DB / I/O helpers** live in `.claude/skills/shared/` — not duplicated per skill. When you fix a bug in one skill's DB code, you don't want to hunt for it in seven others.
- **No silent fallbacks.** If the live source of truth is missing (a required DB row, an env var, a file), **fail loudly** with a message naming exactly what's missing. Never substitute a stale cache, hardcoded constant, or JSON fallback. Silent fallbacks are how bugs get deployed.
- **Content-hash your AI output.** When you cache a polished/translated string, also cache `SHA256(input)`. On the next request, recompute the hash — if it matches, serve the cache; if not, regenerate. Free re-reads, correct-by-construction invalidation.
- **Prompts in DB rows, not source.** Prompts are rules that change as you iterate on quality. Put them in a table (e.g. `ai_prompts (prompt_key, model, max_tokens, template)`), load them at startup, and `--confirm`-edit them via a skill. **`[example]`** — in this repo, `backend/internal/services/prompt_store.go` loads the `ai_prompts` table at boot and fails loudly when a key is missing.
- **Scripts print JSON or tables, not prose.** Orchestrators can read both; users can read both. Logs aren't chatbots — stay structured.

---

## 5. What running a skill costs

Everything runs on the M4, orchestrated by Claude Code on a Max subscription. **A Max
subscription covers Claude Code and claude.ai. It does not include API credits.** The moment a
*script* calls the Anthropic API directly, that is separate, per-token billing on an API key.

| Approach | Covered by Max? | Good for |
|---|---|---|
| **Claude Code does the work**, driven by a skill | Yes | interactive work, small batches, anything you want to watch |
| **A script calls the API** with a key | No — per token | unattended batches, folder watchers, structured output at volume |

Pick per skill, and **write which one you chose into the skill's safety section**: the cost
profile is the most surprising thing for whoever reads it next. With orchestration on a flat
subscription, usage limits, not cost, are the scarce resource — a skill that loops over 400
documents will hit them. Running locally is a **transport** choice, not an architecture: a
well-written skill ports to a hosted agent without a rewrite.

## 6. Starter templates

**The templates live with the `skill-author` skill**, linked into every hall, so they cannot
drift from this document: `.claude/skills/skill-author/references/SKILL.md.tmpl` (**6a**, the
`SKILL.md` skeleton: frontmatter, Key Concepts, Scripts with a `Mutates?` column, Safety,
Self-Annealing Notes) and `script.py.tmpl` (**6b**, a dry-run-first Python script with JSON
output and safe exits). `skill-author` fills them in and checks the result against §12.

**6c, shared helpers:** one module in `.claude/skills/shared/` per concern (a database
connection, colours, the hall's root), imported by every skill and never copied. Fail loudly
when its source is missing rather than creating an empty one: an empty index looks exactly
like "no results".

---

## 7. Sequencing and measured output

Some deterministic outputs are not computable from the source at all — they
depend on a measurement only an external engine can make. Page numbers are the
canonical case, and they surface a set of rules worth stating once.

**`[example]`** — the IIS journal pipeline paginates each issue. Page numbering
restarts at 1 for every issue, articles are numbered in the order the folder
lists them, and each article's page range is written into three places: the
running header (`pp. 12-23`), the DOI (`.../1_iis_2026_12-23`), and the footer's
starting page number.

### The rules this pattern forces

- **The sequence is a DB row, not a sort call.** "The order the folder lists
  them in" is a *default*, not the rule. Emit it as a config artifact
  (`running_order.json`) with an explicit position per item, so the editor can
  reorder without touching code and the next run preserves the change. A
  hardcoded `sorted()` gives you no way to say "this one goes first."
- **Match the human's sort, not your language's.** The Finder sorts naturally,
  so `82` precedes `143`. A plain string sort puts `143` first and silently
  produces a different running order than the one on the user's screen. If a
  human will read the ordering as authoritative, reproduce *their* ordering.
- **Measurement is Execution, and it may need a real engine.** Page count is
  the result of laying text out against font metrics; no amount of XML parsing
  yields it. Render for real, and never estimate — an approximate page range is
  a wrong DOI.
- **Reach for a headless engine, never a GUI app.** The obvious move is to
  drive the application the recipient uses — it is authoritative, after all.
  Resist it for anything batch-sized. A GUI app opens a window per item, steals
  focus, and on macOS runs sandboxed, so it raises modal permission dialogs that
  block the run with errors that look like bugs in your script. **`[example]`**
  — the IIS paginator first drove Microsoft Word by AppleScript and spent an
  afternoon on `-1708 doesn't understand the "close" message`, which turned out
  to be a *Grant File Access* dialog waiting behind the app. `soffice --headless
  --convert-to pdf` does the same job for a whole batch in one invocation, with
  no GUI and no dialogs. Keep the GUI path only for spot-checking a handful of
  items against the headless one.
- **Keep the render.** If measuring produced a PDF, do not throw it away — it
  is a free page proof for whoever has to check the work.
- **Fail loudly when the engine is unavailable.** A missing renderer is exactly
  the "silent fallback" the framework forbids. Do not fall back to a cached
  page count, a word-count heuristic, or the `<Pages>` value the application
  last wrote — say the engine is unreachable and stop.
- **Assignment runs after formatting, never before.** Formatting changes
  pagination. Ordering the passes wrongly produces ranges that were true of a
  document that no longer exists.
- **Converge, then verify.** Writing the range into the header can itself change
  the page count. Re-measure after writing, report any item whose count moved,
  and make the pass re-runnable until it settles.
- **Restart per unit.** When the numbering restarts per issue, per volume, per
  batch, every path in the project is resolved per unit too — inputs, outputs,
  reports, and the editor's decision files. One shared output folder is how two
  issues end up with one running order.

### Anti-patterns specific to sequencing

| Don't | Do | Why |
|---|---|---|
| Derive order from a `sorted()` inside the script | Emit `running_order.json`; the script consumes it | The order is an editorial decision, so it belongs in Tier 1 |
| String-sort filenames a human will also sort | Natural sort, matching the file browser | `143` before `82` is a silently different issue |
| Estimate page counts from word counts | Measure in the real renderer, or fail | An approximate page range is a wrong, permanent DOI |
| Assign numbers before formatting | Format, then measure, then assign | Formatting moves the page breaks |
| Let excluded items consume a position | Only shipping items get a number | Otherwise the printed sequence skips |

---

## 8. Derived state, and how it goes stale

Most of the bugs that survive a careful review are not wrong logic. They are a
value that was correct when it was computed and was never told that the thing it
was computed from had changed.

**The rule: anything derived from something else must either be regenerated when
that source changes, or be visibly marked as stale. Never silently kept.**

Five specific forms, each of which has actually shipped.

### A derived artifact names its source, and moves when the source moves

**`[example]`** — an LLM grader writes a comment arguing for the verdict it
chose: *"which is what keeps it off the top band."* The instructor moves the
band. The sentence is now false, but it sits under the new score looking
considered, and a wrong comment that reads as deliberate is worse than no
comment. The same defect appeared one level up in the same system: the overall
feedback and the student's email are derived from the per-criterion scores, so
after an override the summary still quoted a total that was arithmetically
wrong.

The fix is not "remember to update it." It is that changing the source either
regenerates the derived text or clears it and says why.

### A derived view must not read from a source that can disappear

**`[example]`** — a grading page populated its email box from the draft record.
Posting a grade deletes the draft. So the instant a grade was posted, the page
showed an empty email field for a message that had already been sent, and the
instructor reasonably reported that the email had been lost. Nothing was lost;
the only copy the *view* knew about had been consumed.

If a view reads from a transient source, it needs a durable one for the state
after the transient one is gone.

### One rule, one implementation

Two implementations of the same rule are two rules, and they will disagree.

**`[example]`** — "greet the student by the name they asked to be called" was
expressed in three places: a template email path, a composed email path, and a
CLI flag. In one afternoon the same policy produced three behaviours — the
preferred name, the roster name, and no greeting at all. Six students were
addressed by names they do not use, on the message telling them their grade.

Share the implementation or make one authoritative and have the others call it.
Do not restate a rule in a second language and trust both copies to age
together.

### An absence must be distinguishable from a decision

Silence is ambiguous. It can mean "nobody did this", "this failed", or "someone
deliberately chose not to" — and months later nothing recovers which.

**`[example]`** — posting grades without notifying students writes an audit row
marked `suppressed` rather than writing nothing at all, so *"was this student
told?"* answers *"no, deliberately, at this timestamp"* instead of leaving a gap
for a reader to interpret. Likewise a criterion the grader could not assess is
recorded as undecided, never as zero: a column of zeros asserts a judgment that
was never made.

This is the same instinct as **no silent fallbacks** (§4), pointed at stored
state rather than at runtime.

### Never bundle an irreversible action with a reversible one

**`[example]`** — committing a grade and emailing the student were a single
operation. A posted grade is editable; a sent email is not. Bundled, the
irreversible half inherits none of the safety of the reversible half, and there
is no moment between *"I think this is right"* and *"the student has been
told"*. Splitting them into two deliberate acts costs one extra click and buys
back the ability to be wrong.

### How to catch this before it ships

When you add a value computed from another value, **write down what invalidates
it** — in the migration comment if it is a column, in the docstring if it is a
function. If you cannot state the invalidation rule, you do not yet know whether
the field should exist.

Content-hashing cached AI output (§4) is the mechanical version of this: cache
`SHA256(input)` alongside the output and the invalidation is automatic. The
general case rarely has such a clean hash, but it always has an answer to *"what
would make this wrong?"* — and that answer belongs next to the field, in
writing.

---

## 9. Anti-patterns

| Don't | Do | Why |
|---|---|---|
| Hardcode business rules in app code (switch statements, maps, const dicts) | Put them in a DB table; app reads at startup or per-request | Rules change. Code deploys are slow, DB updates are fast. |
| Bake AI prompts into source | Store in an `ai_prompts` table; load via a prompt-store service | Prompts iterate constantly. Source-baked prompts require a deploy per tweak. |
| Script that writes without a `--confirm` flag | Dry-run by default, `--confirm` to execute | The orchestrator will occasionally call the wrong script on the wrong row. Default should be harmless. |
| Silent fallback when live data is missing (return `[]`, substitute JSON, use stale cache) | Raise loudly with the name of what's missing | Silent fallbacks ship broken features that look fine. |
| Per-skill DB connection code | One `shared/shared_db.py`, all skills import it | You will fix a connection bug once, not eleven times. |
| Scripts that print free-form prose | Print JSON or markdown tables | Both humans and orchestrators parse structured output reliably. |
| LLM-only pipelines (five chained Claude calls, no scripts) | Push the deterministic work into scripts, keep the LLM routing | Compound error probability. |
| Keep derived text after its source changed | Regenerate it, or clear it and say why | A stale sentence under a new number reads as considered, not as forgotten. |
| Populate a view from a record that gets deleted | Read the durable copy for the after state | An empty field looks like data loss even when nothing was lost. |
| Express one rule in two languages | Share it, or make one authoritative | Two implementations are two rules, and they diverge silently. |
| Record nothing when someone deliberately chose not to act | Record the decision | Later, "nobody did it" and "we chose not to" are indistinguishable. |
| Bundle an irreversible action into a reversible one | Two deliberate acts | The irreversible half inherits none of the safety of the other. |

---

## 10. Starting a skill in a hall

1. **Ask `skill-author`** ("make a skill"). It scaffolds `.claude/skills/<name>/` from §6 and
   checks it against §12. A hall's own skill lives in its district repo; an org skill is the
   GM's to write (§12).
2. **Start small and read-only** — an audit or an inventory. Mutations come later, behind
   `--confirm` or `--commit`.
3. **Run the dry run until it prints everything the real run would do,** then run it for real.
4. **When you are about to hardcode a rule or a prompt, stop:** put it in `rules/` or
   `references/` (§2).
5. **When two skills share a helper, move it to `shared/`.**
6. **Ship it like any change** ([`github-standard.md`](github-standard.md)), and write what you
   learned into its Self-Annealing Notes.

---

## 11. The four kinds of thing in `.claude/`, and what each cannot do

They are routinely confused, including in published guidance, so each is defined by **what it
cannot do** — the half that gets assumed, and the half that costs you.

| | What it is | What it **cannot** do |
|---|---|---|
| **Skill** | A **verb** — a procedure that does one thing | Hold standing context, or form an opinion |
| **Agent** | A **noun** — a role with standing context that chooses among the hall's skills and writes back what it learned | Route work to another hall |
| **Rule** | A statement in a contract | **Enforce itself** |
| **Hook** | `PreToolUse` / `PostToolUse` in `.claude/settings.json` — the only preventive mechanism | See a write made through a tool it does not match |

> **Agents never replace skills; they decide which to run and why.** (`templates/employee.md`)

- **A skill cannot hold standing context.** Invoked twice, it knows nothing new the second time
  — which is what makes it testable, and why a recurring judgement ("what needs attention
  today") is not a skill. A skill that says *"decide whether this is worth doing"* has become an
  agent with a misleading filename.
- **An agent** is `.claude/agents/<slug>.md`, written against `templates/employee.md`. **Its
  frontmatter is documentation, not configuration:** the runner reads the roster
  (`staff.json`), not the file. Three registries must agree by hand — the definition, the
  roster entry, and whatever schedules it (launchd) — and nothing checks that they do. One
  hall's roster entry outlived its deleted definition by eighteen days.
- **A rule cannot enforce itself.** *"Do NOT edit files under `tests/`"* is followed most of the
  time, which is worse than never, because it produces the belief that it is enforced. A rule
  becomes real only when something computes it: a hook, a deny list, or a git seam.
- **A hook** sees the call before it happens; exit 0 allows, exit 2 blocks. **A hook matching
  `Edit|Write` does not see `sh -c 'echo x > file'`** — match `Bash` too if the agent holds it.
  A hook guarding against an accident may fail open; **a policy gate must fail closed**, or
  *enforced* silently becomes *requested*.
- **`.claude/settings.json` is read from the session's working directory.** A runner that
  starts an agent in a hall loads that hall's settings, not the GM's; policy that must travel
  with an invocation has to be passed with it.

**Three traps, all of which read as configured:**

1. **Tool names must be real** — `Read`, `Write`, `Edit`, `Bash`, `Grep`, `Glob`, `WebFetch`,
   `WebSearch`, `NotebookEdit`. Published examples use `ReadFile` and `RunTerminal`; a
   misspelled name does not fail, it silently restricts nothing. Hand an agent a wrong name and
   confirm it is *rejected*.
2. **`tools:` does not restrict paths, and `--allowedTools` does not restrict at all.** Measured
   2026-09-26: an agent given `--allowedTools "Bash(test:*)"` ran `python3 -c 'print(42)'` and
   reported 42. `--allowedTools` is a pre-approval list; `--disallowedTools` is what restricts.
   Keep `allowed_tools` so an unattended run does not stall on a prompt — never add to it
   expecting to narrow anything.
3. **Any interpreter collapses an allowlist.** `Bash(python3:*)` is bare `Bash`, as are `sh`,
   `node`, `perl`. Match a **token prefix** (`Bash(python3 scripts/scan.py:*)`) and refuse shell
   metacharacters outright, or `ls; rm -rf x` passes a head check.

**Research mode** is a hook that travels by mod: `/research` puts any session in any hall into
read-only fact finding, enforced against the GM's `references/research.json`, failing closed;
only Bryan can turn it off (gm-district#354, `apps/gm-research-mode`). It proposes no plans, and
what it finds is saved, at Bryan's word, as a `type: research` note with a `track:`
([`brain-standard.md`](brain-standard.md) §2, #587).

---

## 12. Where a skill lives, and the DOE contract

**Every skill has one home.** A second copy is a fork nobody maintains. Four tiers, each
reaching a hall in exactly one way:

| Tier | Home | How a hall gets it | For |
|---|---|---|---|
| **Org** | the GM's `.claude/skills/<name>/`, named in `standards.json` → `skills` | a **symlink** made by `bin/hall-layers`, listed in the hall's `.git/info/exclude` | a verb every hall needs the same way: planning, printing, the calendar, messages |
| **Kit** | the GM's `kit/brain/` | a **copy** installed by `bin/ship-brain`, SHA-256 checked | the brain, documents and intake ([`brain-standard.md`](brain-standard.md) §4) |
| **Hall** | the district repo's `.claude/skills/<name>/` | it is already there | the hall's own work, **including its apps' skills** |
| **Local** | the GM's `.claude/skills/<name>/`, named in `standards.json` → `local_skills` | it does not | the Boardroom's own work: the desk (`fix-monitors`, `display-reset`, `jump-connect`) |

- **A hall overrides an org skill by keeping a real folder of the same name,** and says why in
  its `docs/sop-local.md`. `bin/hall-layers` never touches a real folder. A kit skill is not
  overridden: a hall changes it through its `rules/`, or omits it with a reason.
- **App skills live in the district repo** (Bryan, 2026-11-17), named `<app>-<skill>` on a
  collision — a session opened at the hall never sees `apps/<app>/.claude/`.
- **Retire, don't archive.** Claude Code loads whatever sits in `.claude/skills/`, including
  `_archive/` and `old-*`. Git history is the archive.
- **A hall's skill never sits in `~/.claude/skills`.** Personal skills rank above project
  skills, so one there loads everywhere and nothing can override it.

**Every skill, in every tier, is written in DOE**, and `bin/skill-check` checks five rules
(#321); the thresholds and accepted headings are data in the GM's
`references/skill-check.json` → `doe`:

| # | The rule | Finding |
|---|---|---|
| 1 | **The frontmatter is a trigger contract.** `name` is the folder's name; `description` says what it does and names at least one phrase the user says, in quotes, or a `/slash-command`. | `doe_frontmatter` |
| 2 | **The directive has its sections:** `## Scripts`, and `## Safety`, `## Rules` or `## Hard limits`. With scripts, the table has a `Mutates?` column. | `doe_directive` |
| 3 | **Directive and scripts name each other.** Every script is named in `SKILL.md`, and every one it names exists. Helpers (`_*`) and tests (`test_*`) excepted. No scripts: say **None** under `## Scripts`. | `doe_scripts` |
| 4 | **Dry run by default.** A script the table marks as mutating carries `--confirm` or `--commit`. | `doe_dryrun` |
| 5 | **No hardcoded place.** No script holds a hall's root as a literal path: find a hall from `districts.json` with `os.path.ismount`, the GM from the script's own `Path(__file__).resolve()`. | `doe_path` |

A hall checks its own with `skill-author`, or `bin/skill-check --hall <code> --kind doe` from
the GM; from a worktree, add `--root .` to check that checkout rather than the hall's main one. Fix a finding in the skill, never by loosening the rule for one skill.

---

## 13. Several agents

**Multiple agents do not repeal the arithmetic. They multiply it.** A second agent is another
probabilistic step, and one that re-derives context the first already had is a second chance to
get it wrong at twice the cost. The case for splitting must be stronger than "it would be
tidier", and **the split must fall on a seam where the hand-off can be made deterministic.**

| Reason to split | The test that settles it |
|---|---|
| **Context saturation** | one agent must hold too much at once, and its window is visibly degrading |
| **Adversarial separation** | the work and the check must not be done by the same actor |
| **Machine placement** | the work must run where the data is |
| **Credential boundary** | two pieces of work must not hold the same secret |

Not reasons: it feels more organised; the diagram looks better; a role name exists. **Only
adversarial separation buys correctness:** *an actor that can touch both the code and the check
can always make the check pass* — not from dishonesty, from optimising for the signal it was
given.

**The hand-off is the fourth artifact**, and it has four required fields (in a software
factory, a spec's task carries them: [`software-factory.md`](software-factory.md) §4):

| Field | What it must be |
|---|---|
| `acceptance` | **one command** that exits non-zero now and zero when the task is done — run it before any work exists and require it to fail |
| `file_scope` | what may be touched, split into source and tests |
| `stop_after` | what finishing looks like, in one line |
| `undo` | how to reverse it |

Hand over the contract, not the conversation: an agent given another's whole transcript gains
its cost and none of its certainty.

**The orchestrator is a script, not an agent.** "Halt on first failure" and "strictly
sequential" are properties of a `for` loop with a `break`, not of a prompt. *The runner owns
the artifact* (`bin/shift`): depending on an agent to save its own output is a deliverable that
goes missing on the night nobody is watching.

**What actually enforces a boundary,** in descending order of trust: **a git seam** (commit the
test in its own commit, then require `git diff --name-only <pin> -- <tests>` to be empty — it
cannot fail open); **a `PreToolUse` hook that fails closed**; **a deny list**
(`permissions.deny`, or `--disallowedTools` for `claude -p`), real but coarse (§14). `tools:` and `--allowedTools` enforce nothing (§11).
Denying `Write` does not make an agent read-only if it holds `Bash`.

**Setting one up, in order:** write the rules down first; start with **one** agent at
`propose` and let it run; give it a definition file, not just a prompt; register it where the
runner looks (and remember nothing checks the three registries agree); only then consider a
second, and only for a reason in the table above.

**Anti-patterns:** splitting for tidiness · prose hand-offs · acceptance by adjective ("should be
clean and well-tested") · the writer marking its own homework · granting `Write` and
*describing* a boundary · forwarding a transcript as context · an unbounded unattended loop · a
lane with no undo · promotion by vibes · a check that has never been seen to refuse.

---

## 14. The autonomy ladder, and D1/D2/D3

**Two rungs, and the third is deliberately absent.**

| Rung | May do | Gate |
|---|---|---|
| `propose` | write a proposal, nothing else | the default for every new agent, indefinitely |
| `act-reversible` | act inside one named lane with a deterministic verifier and a reversal path; run the verifier; on failure undo, then propose | a task contract per lane |
| ~~`act`~~ | — | **not implemented: considered and refused.** No lane's failure is cheap enough to justify an unverified write, and a rung that exists will be reached for. |

**Autonomy is granted by verifiability, not by trust.** "Has this agent been reliable" decays
silently; "does this lane have a computable verifier" is a fact you can recheck. **Promotion is
a data edit** (the roster), never a code change, and demotion must be as cheap. **Anything whose
job is to report on the system never promotes:** the reporter cannot be the actor. These are the
seven rules at the end of the hard restrictions in the GM's `CLAUDE.md` and in
`templates/district-CLAUDE.md`, which every hall's contract is built from.

**D1/D2/D3 is how a lane climbs from `propose` to `act-reversible`** — written by the GM on
2026-09-26, and **not a schedule:** who goes first is an
operational decision. **One hall reaches D3 before the next starts D1**, so a failure has one
candidate cause.

| | |
|---|---|
| **D1 — prerequisites** | **A test suite that can fail** — not one that exists: one that has been *seen* to go red on a deliberately broken tree (`go test ./...` over zero test files prints `[no test files]` and exits 0). **The issue gate** ([`github-standard.md`](github-standard.md) §5). **One state per issue.** **A dev-test target with synthetic data** — a hall whose only environment is the live one cannot pass. |
| **D2 — the shadow run** | The lane stays `propose`; the verifier runs **anyway** on work a person did, and its verdict is recorded. What is measured is whether **the verifier** is right. **D2 ends at a disagreement where the verifier was right** — a shadow run that always agreed has not been tested. |
| **D3 — promotion, monitored** | Promotion by roster edit, and registration as a monitored subsystem: a **heartbeat** where only success updates `last_success`, a staleness threshold so silence ages into a warning, and somewhere that warning is read. A lane that reports nothing and one that reports "nothing needed doing" are different facts. |

### Approvals: auto mode, over a floor of deny rules

The ladder governs agents that work alone. A seat Bryan works with is interactive, and **its
approval mode is `auto`** (2026-11-25, #647): Claude Code's classifier approves the safe steps
and stops the risky ones, so an approval reaches Bryan only when something deserves his
decision. Anthropic measured it on 1,053 dangerous commands: people clicking through caught
13.6%, auto mode blocked 89%. So about one in ten still passes, and **the classifier is a
judgement, never the control for what must not happen.**

- **The reversibility test decides which is which.** If Claude got this wrong once, could you
  take it back? Reverting a commit can be; a secret printed, a forced push, uncommitted work
  thrown away or an email sent cannot. Reversible work is left to auto mode. **Irreversible
  work is a deny rule or a fail-closed hook, never only words** (§11: a rule cannot enforce
  itself).
- **Deny rules are committed, in the hall's `.claude/settings.json` → `permissions.deny`.**
  They are checked before anything else, in every mode, and no scope's allow rule can carve an
  exception out of one. A rule matches only the words it names, so a force has a
  middle-of-command form (`Bash(git push * --force*)`) and a `git -C *` twin, and a Read rule
  starts at the filesystem root (`Read(//**/.env)`) so it holds from any checkout. **No rule
  sees a script doing the same in a subprocess**: the hooks and the review (`sop.md`) stay. A floor, not a
  wall. Each rule is proved with a positive control before it merges: the call is refused,
  and the ordinary form beside it is not.
- **`autoMode` is the user's, never the repo's.** Claude Code does not read `autoMode` or a
  `defaultMode` of `auto` from a project's settings, so a repo cannot grant itself trust. The
  environment the classifier is told about is in Bryan's `~/.claude/settings.json` →
  `autoMode.environment`, keeping `"$defaults"`: our GitHub owner and repos, the M4's runners,
  the Dropbox root read from `~/.dropbox/info.json`, and the halls on their volumes.
- **A seat started by `bin/dm squad` gets its mode from data**, `references/seats.json` →
  `permission_mode`, when the caller names none; the Marshall Inc app passes its own.
- **An agent run unattended (`claude -p`) is not interactive and does not start in auto.** It
  keeps the ladder above: `--disallowedTools`, and `bin/guard-bash` against its roster entry.
  The committed deny rules bind it too, as they bind every session started in the hall.

Why: the GM's brain note `claude-code-best-practices-talk` (2026-11-25), Anthropic's advice checked claim by claim.

What `bin/district-ready` measures about a hall as a whole is the hall standard's
([`hall-standard.md`](hall-standard.md)); it reports `ok`, `not-ready` or **`could not check`,
which never counts as passing.**

---

## Further reading

- [`github-standard.md`](github-standard.md) — how a skill's changes ship
- [`brain-standard.md`](brain-standard.md) — the kit's skills
- [`templates/employee.md`](../../templates/employee.md) — the agent definition template

## When DOE isn't the right tool

DOE separates reasoning from execution. It does not make every execution appropriate for automation.

**`[example]`** — a previous project tried to scrape Google Scholar on a weekly timer, built exactly to the pattern: dedicated venv, systemd timer, loud-fail discipline. Google hard-blocked the production IP on the first request — no CAPTCHA, just a "We're sorry…" page. DOE gave full visibility into the failure (the skill did precisely what it was told) but could not fix the underlying problem: automated requests from cloud IPs are blocked regardless of stealth. The fix was a Chrome extension that posts to an authenticated endpoint when the user visits their own Scholar page — same Directive → Orchestration → Execution split, but the "browser" is a real human's browser, so there is nothing to detect.

The lesson transfers: Apple Notes cannot be driven reliably by AppleScript for anything structured (see `CLAUDE.md`). Don't build a skill that pretends otherwise. Treat it as an inbox a human fills and the agent drains, and put the structure in the vault where it belongs.
