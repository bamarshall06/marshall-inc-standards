# The brain standard — what a hall knows, and what it received

> **Public edition.** A working standard from Marshall Inc, shared as it is used. Real names, paths and accounts are replaced with generic ones (floors such as *Business* and *Teaching*, `/Users/<you>`, `<your-github-user>`). References like `gm-district #372` point to private repositories and are kept only as history. Start with the [README](../README.md).

## Summary
> What you wrote is markdown in `brain/`, in its own repo; what you received is a PDF and `.json` sidecar in `documents/`, never in git. The brain kit runs it the same way in every hall, the index is a disposable cache, and a document that matters gets a note. A research session's findings are saved as a `type: research` note carrying a `track:` that says where they point (§2, *Research notes*).

**Status:** org-wide standard as of 2026-11-19 (gm-district #372, milestone #24). It began as a
portable guide, first adopted 2026-11-06/07 (§10); on 2026-11-19 it became the organisation's
standard, absorbed `tech-stack.md` (§5) and gained the brain kit (§4). **Applies to:** every
hall. Entry point: [`sop.md`](sop.md).

A knowledge system made of plain files, so you, Claude Code and Cursor all read the same
thing: **three folders and one index**, no app and no service. **Every hall runs it the same
way, through the brain kit** (§4); what differs by hall is data, in its `rules/brain.json`.

Sources: *Build a Second Brain with Markdown, JSON, and Claude Code* (the notes-plus-sidecars
shape, whose documents folder it called `docs/`); the Claude Code second-brain system guide
(naming, frontmatter, summaries, query rules); and the skill standard,
[`doe-framework.md`](doe-framework.md) (how the scripts are built). Examples that name a
specific project are marked **`[example]`**. Numbers given as *measured* were measured on a
real corpus; see **Appendix: what was measured**.

---

## 1. The shape

```
/Volumes/<VOLUME>/
├── CLAUDE.md        loaded automatically on startup. The directive.
├── brain/           EVERYTHING YOU WROTE — markdown, flat. Its own repo, <prefix>-brain.
│   ├── README.md    the index / map of content. GENERATED, never edited by hand.
│   ├── <note>.md    ...flat. `type:` classifies, not the path.
│   └── private/     OPTIONAL, and the one exception to flat: never committed.
├── documents/       EVERY DOCUMENT RECEIVED — flat, PDF + .json sidecar. A link into the cloud.
├── messages/        notices and messages, read and answered; archive/. A link into the cloud.
├── intake/          files that have arrived and are not filed yet. A link into the cloud.
└── var/brain/       the search index. Derived, disposable, gitignored.
```

**Two rules, and that is the whole system:**

> **If you wrote it, it is markdown in `brain/`, committed to the brain's own repo** — except
> `brain/private/`, if the hall keeps one, which is never committed.
> **If you received it, it is a PDF in `documents/` with a `.json` beside it, and it is never
> in git.**

Everything below is the consequence of those two sentences.

**Why `documents/` and not `docs/`:** in every hall `docs/` is the standards and developer
documentation ([`hall-standard.md`](hall-standard.md)). **Why a cloud:** documents are not in
git, so they need another copy; `documents/`, `messages/` and `intake/` are links into
`<cloud>/DISTRICTS/<VOLUME>/` — the hall's cloud, Dropbox unless `districts.json` → `cloud`
assigns another — which versions them and puts them on every device
([`communication-standard.md`](communication-standard.md) §2 has the rules for `messages/` and `intake/`).

**Knowledge vs evidence.** A scan of a tax bill is *evidence*: it goes to `documents/`. A note
about what the bill means is *knowledge*: it goes in `brain/`, with `authority:` naming the
document. **One person, one note:** a person who matters to two halls has one note, in the hall
that owns the relationship, never a copy in each.

### Why flat

Nesting does **not** make retrieval faster — `ripgrep` walks the tree
regardless, and a shorter path saves a couple of tokens. Flat buys two real
things:

- **No wrong guesses.** With folders, a session must decide whether a note is
  under `standards/` or `process/`, and a wrong guess is a failed read — a real
  round trip. Flat, there is one answer.
- **No duplicate classification.** A folder and a `type:` field say the same
  thing and can disagree. Keep the field, drop the folder.

**`private/` is the one exception, and the reason is not organisation.**
`.gitignore` protects it with a *directory* rule, which is why a private note
cannot be committed by accident. Flatten it and protection becomes per-filename.
Directory rules are the mechanism; keep the directory.

**`private/` is a choice, not a requirement.** In this organisation **every brain is its own
repository**, `<prefix>-brain` ([`github-standard.md`](github-standard.md) §1): nested at
`brain/`, ignored whole by the district repo, its only remote a **private** GitHub repo listed
in `rules/brain.json` → `containment.allowed_remotes`. The history is then the protection, and
a hall may decide sensitive facts live in ordinary notes. The validator enforces whichever the
hall decided. See §6 and §9.

---

## 2. `brain/` — what you wrote

### Filenames

Lowercase, kebab-case, descriptive, **self-contained**. The filename carries the
classification, because there is no folder to carry it.

- Good: `redis-distributed-rate-limiting.md`, `sop-b1-scope-of-work-creation.md`
- Bad: `redis.md` (vague), `work/client/2026/q1/spec.md` (nested)

### Frontmatter

```yaml
---
id: sop-b1-scope-of-work-creation   # the wikilink target. ALWAYS the filename stem.
title: "SOP-B1 — Scope of Work creation"
type: process                        # the only classifier. See below.
status: current                      # current | draft | superseded
updated: 2026-11-06
track: future-direction             # type: research only. See Research notes.
tags: [process, sop, budget]
authority: documents/sop-b1-scope-of-work-creation.pdf   # the document this came from
related:
  - "[[sop-a1-initial-walkthrough]]"
---
```

- **`id` must equal the filename stem.** The commonest bug in a vault like this
  is one note where it doesn't, and every wikilink to it silently misses.
- **`type:` is the only classifier**, so a wrong value is a note that cannot be
  found rather than an untidy one. Each hall keeps a small **closed** vocabulary in
  `rules/brain.json` → `contract.vocab.type`, and the validator refuses anything else.
  **`[example]`** — the GM's: `standard · decision · incident · reference · concept ·
  source · status`.
- **A `type: research` note carries `track:`**, one value from a closed list in
  `rules/brain.json` → `contract.vocab.track`, required through
  `contract.required_by_type` → `research: [track]`, so the validator refuses a research
  note without one. **`[example]`** — the GM's: `feature-request · future-direction ·
  research · strategy`. A hall that keeps research notes lists `research` among its types.
- **`authority:`** names the document in `documents/` the note was written from. This
  is what ties a note to evidence, and with the sidecar's `sha256` it is how you
  learn a source document changed under a note.
- **`related:`** is derivable from the `[[wikilinks]]` already in the body. Do
  not hand-maintain both.

### Every note opens with a Summary

```markdown
# SOP-B1 — Scope of Work creation

## Summary
> One or two sentences. The core takeaway, not a topic label.

## ...
```

**This is the single biggest speed win in the system, and the only one that is
not scriptable.** The query contract is *grep the frontmatter, read the Summary,
only then load the file*. Picking among five candidate notes costs ~1 KB of
summaries instead of ~20 KB of notes. Skip this and the grep-first rule is a
rule nothing obeys.

### `README.md` is the index, not a charter

Root entry point: every note, grouped, each with its one-line summary, plus
known gaps (broken wikilinks, orphan notes). **It is generated** by the kit's `readme.py
--confirm` from the notes and `rules/brain.json` → `readme`; never edit it by hand. If the project already has a
provenance or charter document, give it its own file and link to it — one job
per file.

### Links across brains

A note may link to a note **or a document** in another brain — another hall's, or a room's —
by naming the brain before the slug (gm-district #563, kit 1.8.0):

```
[[slug]]                  this brain
[[sf:slug]]               hall sf: a note id there, or a document stem in its documents/
[[family/school:slug]]    a room of another hall
[[school:slug]]           a room of this hall (from the hall, or from a sibling room)
[[family:slug]]           from a room: its own hall
```

- **A prefix of that shape is always a brain.** One that names no brain is an error
  (`validate.py`, exit 1), never a quiet fall back to this brain. So is a target that is both
  a note and a document in the other brain.
- **Link, never copy.** One person, one note still holds: the people note stays where it
  lives, and the other brain points at it.
- **Where the brains are is generated, not written.** Each hall and room reads
  `.claude/skills/shared/brains.json`, which the GM's `bin/ship-brain` writes from
  `districts.json` and `routing.json` (still the only place a path is defined) and
  `bin/ship-brain --check` reports stale. A hall never edits it.
- **The only door into another brain is its index**, `var/brain/index.db`, opened
  read-only. The kit never reads another hall's trees or runs its code.
- **Found, missing, or could not check — three answers, never two.** A brain whose volume
  is not mounted on this machine (the university's laptop is the obvious case) or that has
  no index is *could not check*, named in `rebuild.py`'s report; it is not counted as found,
  and not as missing. A missing target marks something worth writing, as a local one does.
- **`README.md` lists them, and does not resolve them**: whether another brain can be read
  depends on the machine, and the README must not. `rebuild.py` and `bin/brain-check` say
  which resolved.
- **Search follows the links** (§5): a question also searches every brain this brain links
  to, each ranked by its own weights and shown as its own group. `--local` searches this
  brain alone; `--brain sf,family/school` names the brains.

### Research notes, and research mode

**A research note is what a research session found, saved once the research is finished and
only when Bryan asks** (gm-district #587). Research mode (`/research`, the `research-mode`
mod) is for research and nothing else. The session **makes no changes and proposes no
plans**: no fix and no implementation steps, even when asked how to fix something. Acting on
research is a separate task, an issue that cites the note. Bryan leads the research stream;
the agent's part is to find, cite, and ask him sharper questions that move the research
forward.

The note is `type: research`, and its **`track:`** says where it points. That makes *"what
is our next step, what is our strategy"* a filter on the brain rather than a memory:

| `track:` | The research points to |
|---|---|
| `feature-request` | something to build: a feature in an app or a tool |
| `future-direction` | a direction worth taking later, not yet a feature |
| `research` | understanding only: what is true, with no direction yet |
| `strategy` | how to position or grow: strategy, including market strategy |

```
rg -l "^track: (future-direction|strategy)" brain/       what's next, and our strategy
rg -l "^track: feature-request" brain/                   what to build
```

Its sections, in order: **Summary · Question** (what Bryan set out to learn) **· Found**
(every fact cited: a file path, a note id or a URL) **· Inferred** (labelled as inference)
**· Open questions · Where this points** (directions to look at next, never implementation
steps) **· Relevant to**. *Relevant to* holds a cross-brain link (*Links across brains*,
above) to each other hall or room the research bears on: link, never copy. Search follows
links outward, so the other brain finds the note when one of its own notes links back.

Research mode writes the note into the brain of the hall or room the session is in, which
the mod finds by matching the session's folder against `districts.json`. Its instructions
are data, in the GM's `references/research.json` (`method`, `report`, `store`).

### Templates

Keep two, in `brain/README.md`, and the research note above:

- **Concept note** — Summary · Core concepts · Implementation / details ·
  Trade-offs & decisions · References
- **Project note** — Objective · Current status & next actions · Key decisions ·
  Linked resources

### Query rules — put these in `CLAUDE.md`, not here

`CLAUDE.md` loads on startup, so it carries the rules, briefly:

1. Filter on frontmatter before reading files (`rg "^type: process" brain/`).
2. Read the `## Summary` before loading a whole note.
3. Resolve `[[slug]]` to the file whose `id` is `slug`; `[[code:slug]]` is in brain
   `code` (§2, *Links across brains*).
4. Cite filenames in answers.
5. **When the notes don't cover something, say so** rather than answering from
   general knowledge as if they did.

---

## 3. `documents/` — what you received

**Every document has a JSON sidecar of the same name beside it**, holding the document's own
words, page by page. The sidecar **extracts; it does not interpret.** It holds the text and
where it came from, never "the amount due" or "what this means" — that is a note's job (below).

```
documents/sop-b1-scope-of-work-creation.docx     the original, never touched
documents/sop-b1-scope-of-work-creation.pdf      rendered from it
documents/sop-b1-scope-of-work-creation.json     the sidecar: its text, page by page
```

How the kit's `add_document.py` reads each kind (which extension is which kind is data, in
`rules/document_naming.json` → `extensions`):

| In | How the text reaches the sidecar |
|---|---|
| a PDF with text | `pdftotext -layout` |
| a scanned PDF (no text layer) | `ocrmypdf` on a **copy**; the original PDF is never touched and the OCR'd copy is discarded |
| `.docx`, `.doc`, `.pptx`, `.odt` | `soffice` renders a PDF beside the original, then as a PDF |
| a spreadsheet | `soffice --convert-to csv`, one page |
| a picture (`.png`, `.jpg`, `.heic` …) | kept **as the picture**, no PDF: `tesseract` reads any words, and `description` holds a sentence a person wrote after looking. A wordless logo is findable only by it. |
| `.csv`, `.txt` | its own lines, one page |
| markdown | not a document: what you wrote belongs in `brain/` |

### The note that explains a document

**Every document filed gets a note** saying why it is kept (Bryan, 2026-11-23; the daily
intake loop's step 5, [`hall-standard.md`](hall-standard.md) §11). A hall may list the types
that need none, as data in `rules/document_naming.json`, so it changes the list without a code
change. Documents filed before 2026-11-23 are exempt, so no hall's existing tree starts failing.
Until the kit reads the list of exemptions, it still asks for a note only for the types in
`note_required`, and the intake agent writes the rest.

The note is markdown in `brain/`, written to §2: a `## Summary` saying what the document is and
why it matters, what it obliges or decides, the dates and amounts that matter, and
**`authority:` naming the document's file** (`documents/<name>.pdf`). One note may cite several
documents (a contract and its amendments); a document is cited by the note that explains it,
never copied into it. With the sidecar's `sha256`, `authority:` is also how a note learns its
document changed under it.

**`intake` asks for the note** when it files a document, and the gate (§7) reports a document
no note cites wherever the hall's `gate` asks it to.

**Keep the original beside the PDF.** *The original is never touched* is the
rule the whole design rests on. If the `.docx` is the only copy you hold,
deleting it after conversion destroys evidence to save a few hundred KB.

Converting to PDF rather than extracting the `.docx` XML directly is the better
trade: one extraction path instead of two, and the renderer preserves headers,
footers and list numbering, which a naive XML walk loses. *Measured: the render
is faithful to within blank-line handling.*

**Flat means filename collisions are real.** Superseded and current copies of
the same document share a stem. Suffix them — `<stem>-superseded.pdf` — rather
than reintroducing folders.

### Name documents to a standard — the name is the only index a flat folder has

**The rule itself — the form, the date, one shared list of issuers and types for every hall — is
[`hall-standard.md`](hall-standard.md) §10.** This section is why it is shaped as it is.

With no folder to say *bank statement, 2026*, the filename must. A free-form
title per document does not survive contact with a real backlog: *measured on
1,100 filed documents*, three habits had coexisted, 187 names carried the old
filing cabinet's folder path (`2016-03-01-old-taxes-expenses-best-buy-…`) and
~305 followed nothing (`409749-Invoice_INV409749_1789569794499.pdf`,
`941 Q3.pdf`). Adopt a pattern and check it:

```
YYYY-MM-DD-<issuer>-<type>[-<ref>].<ext>        2026-09-21-moda-invoice-6111155.pdf
<the same stem>.json                            2026-09-21-moda-invoice-6111155.json
```

- **date** is the document's *own* date (invoice date, period end, notice
  date), only as precise as the document says — never the scan date.
- **issuer** and **type** come from closed lists in a rules file. **No type
  contains a dash**, which is what lets a name parse back into its parts one
  way; refuse a name that parses two.
- **The sidecar shares the stem, always, and a rename moves both** in one step.
- Name collisions take `-2`; nothing is ever overwritten.

Proposing names for an existing backlog by script is worth doing, but **every
part must say where it came from, and a guess must be flagged** — then a person
approves the list before anything moves. Five traps cost real names here:

| Trap | Fix |
|---|---|
| A topic word (*"sales tax"*, *"941"*) outranked the agency actually named, so an IRS letter came out as the state's | split aliases into **strong** (the organisation's own name) and **weak** (a topic); any strong match wins |
| Your own company name is in the filenames of papers *others* sent you | make your own issuer **fallback-only** |
| A word on page 1 (*"invoice"* on a card statement) overruled the folder it was filed in | type order: filename → old folder → category → page text last |
| A page-1 date (*"date incorporated 2012"*) dated a 2024 return 2012 | a page-text date must agree with the year folder, and is always flagged |
| The old cabinet's folder phrases (*"business registration"*) are in hundreds of names | strip them before choosing the issuer — not before choosing the type |

**Received is not generated.** Run logs, undo records, caches, reports and API
exports are what *your* jobs wrote. They do not belong in the documents folder —
give them their own tree — or the flat folder fills with files no naming
standard fits.

### The sidecar

```json
{
  "source": "documents/sop-b1-scope-of-work-creation.docx",
  "pdf": "documents/sop-b1-scope-of-work-creation.pdf",
  "sha256": "d5072465…",
  "bytes": 48213,
  "method": "soffice -> pdftotext",
  "tool": "LibreOffice 25.8 · poppler 26.09.0",
  "generator": "brain-kit/documents/add_document.py",
  "schema": 2,
  "extracted": "2026-11-06",
  "pages": [{"page": 1, "text": ["line", "line"]}]
}
```

**Schema 2** (2026-11-15, #150; the kit's `sidecar.py` is the definition): the ten fields
above are core, `generator` is one string in every hall, four optional fields have one
meaning everywhere — `description` (what a person wrote after looking; an image with no
words needs one), `title`, `purpose` (from the hall's closed list) and `retention`
(`permanent` · `until-superseded` · `review-YYYY` · `destroy-YYYY-MM-DD`) — and any other key
is the hall's own, kept verbatim by every rewrite.

Four fields come from the video — `source`, `method`, `extracted`, `pages`.
Five earn their place:

| | why |
|---|---|
| `sha256` of the **source** | the join key between a note's `authority:`, a provenance record and the bytes a session actually read. Also the whole staleness mechanism, and the rebuild key for the index. |
| `pdf` | when the source was not a PDF, says which file the text came out of. Without it the chain from note to text has a hole. |
| `bytes` | makes a zero-blocks cloud-storage read legible after the fact |
| `tool` | both the renderer and `pdftotext -layout` change output between releases. A sidecar that cannot be reproduced should at least say what made it. |
| `generator` + `schema` | how the tool knows a `.json` it is about to overwrite is one of its own, and which rules produced the text |

**Render it one page-line per output line** so the file reads like the page,
with `assert json.loads(text) == sidecar` as a self-check. Write UTF-8 with
`ensure_ascii=False` — the moment a document has curly quotes, an em dash or a
non-English page, ASCII escaping makes the file unreadable as a page, which is
the one thing the custom renderer exists for.

### Refusals

Dry-run by default; `--confirm` to write. Exit **1** the document · **2**
policy · **75** could not check.

| Refuses when | Exit |
|---|---|
| extension not handled; missing; not a regular file; 0 bytes | 1 |
| **`st_blocks == 0` and `st_size > 0`** — a cloud-storage placeholder, checked **before opening it**, because opening triggers a download | 1 |
| no words could be read (not by `pdftotext`, OCR or `tesseract`) **and** no `description` was given. **Never write an empty sidecar** | 1 |
| the converter, `pdftotext`, `ocrmypdf` or `tesseract` exits non-zero (quote its stderr) | 1 |
| a required tool is absent, or git is unavailable | 75 |
| the target resolves outside the repo root | 2 |
| **anything it would write under `documents/` is not gitignored** | 2 |
| a file already at the output path was not written by this tool | 2 |

Source unchanged (`sha256` matches) → skip, exit 0, file untouched. Without
this, `extracted:` churns a no-content diff on every run.

Two gotchas worth building in:

- **Headless conversion fails while the office suite's GUI holds the profile
  lock.** Pass an isolated profile (`-env:UserInstallation=file://<tmpdir>`) so
  a sweep never depends on whether an app happens to be open.
- **A file-provider cloud folder can report a real size with zero blocks on
  disk** — indistinguishable from an empty file to anything that checksums it.
  Refuse it; don't hash it. And `find <dir> -type f` returns nothing through a
  symlink, so use `find -L`.

---

## 4. The brain kit — the same code in every hall

The brain is software as well as files, and **every hall runs one copy of it: the brain kit**,
kept in the GM at `kit/brain/`, version-stamped (`VERSION`) and SHA-256
manifested (`.claude/skills/brain/KIT.json`). **Change the kit in `kit/brain/`, never an
installed copy.** `bin/brain-kit` tests it and installs it in the GM; `bin/ship-brain --hall
<code>` installs it in a hall (dry run, then `--commit`) and `bin/ship-brain --check` reports
every hall's version and drift. It never writes a hall's rules, notes or documents. **A room
runs its own copy** ([`room-standard.md`](room-standard.md) §6): `bin/ship-brain --hall <code> --room <id>` installs the kit
into `rooms/<id>/` once the room has written its own `rules/brain.json`, and `--check` lists each
room as `<hall>/<id>`. A note lives in one brain, the room's or the hall's, never both.
**Every install also writes the map of brains** (`.claude/skills/shared/brains.json`, §2),
generated from the org chart and not part of the kit's manifest, because it differs by hall;
`bin/ship-brain --map` rewrites it alone, so a new district reaches every hall without a kit
release.

| Skill | Scripts | Does |
|---|---|---|
| `brain` | `search.py`, `validate.py`, `rebuild.py`, `readme.py` | find what the hall knows; check every note against the contract; rebuild the index; regenerate `README.md` |
| `documents` | `add_document.py`, `sidecar.py`, `naming.py`, `rename.py`, `check.py` | a document in → PDF + `.json` sidecar out, under its standard name (§3); coverage and staleness |
| `intake` | `drain.py` | `intake/`, as the daily loop of [`hall-standard.md`](hall-standard.md) §11 — opens, worth keeping, whose, filed, a note, one re-index: evidence into `documents/` under its standard name; knowledge into a note with `drain.py note`, which retires the item it came from. A notice or message found here is refused: it belongs in `messages/`. |
| `shared/` | `brain_kit.py`, `frontmatter.py` | the one place the contract is parsed; imported by all, never copied |

`bin/brain-check` and `bin/doc-check` ship with it: the gate (§7).

**Everything that differs by hall is data**, never code: `rules/brain.json` (trees, closed
vocabularies, containment, the README's groups), `rules/document_naming.json` (issuers and
types), `rules/intake.json`, and `references/queries.json` (§5). A hall that needs a kit
skill to behave differently says so to the GM, which changes the kit; a hall that truly cannot
take a kit skill lists it in `rules/brain.json` → `omit`, with the reason, and keeps its own,
which must still do everything this standard asks of the kit's.

The kit's scripts follow the skill standard ([`doe-framework.md`](doe-framework.md)): the
`SKILL.md` says when and why, the scripts do the work, dry run by default, JSON out.

---

## 5. The index — contentless FTS5 with weighted bm25

### Why plain files, SQLite and a full rebuild

Three layers, and the discipline is in keeping them separate: **the sources of truth** (the
notes, the sidecars, and Apple's own data where a hall reads it) → **a derived cache** (SQLite
+ FTS5, disposable: deleting it loses nothing) → **access** (Claude reading the index, and
writing back only to the sources). **The one rule: never write anything to the index that
does not exist in a source.** That is what makes deletion honest — remove a record at the
source and the next rebuild simply does not find it.

- **Plain markdown with YAML frontmatter** is readable in twenty years, greppable, diffable,
  and belongs to no vendor.
- **SQLite, not JSON files,** because the real question — *what is going on with X* — is a
  join across notes, documents and people, not a search, and a join needs a database. It ships
  with the OS, is one file, and needs no server.
- **A full rebuild, never an incremental sync.** Incremental sync is where orphans and drift
  come from: a deleted record that never sends a deletion event lives in the cache forever. At
  this scale a rebuild takes seconds; it writes a temporary file and swaps it in atomically, so
  a failed run leaves the old index intact. **Test it by deleting the index and rebuilding;**
  if the counts differ, something lives in the cache with no source.
- **No dependencies, on purpose.** System Python, the OS's SQLite, a hand-written frontmatter
  parser that covers exactly the contract and fails loudly on anything else. A system you
  return to after six months should still run.
- **Semantic search is deferred, not refused.** Vectors can join the same SQLite file the day a
  real question defeats full-text search plus joins — and not before.

### When to build it

| Scale | What works |
|---|---|
| up to a few hundred files | grep + frontmatter + wikilinks + the README index |
| a few hundred to a few thousand | **SQLite FTS5 with weighted bm25** |
| semantic queries at volume | embeddings — and not before |

**The trigger is not a file count.** It is the day a grep returns more hits than
you want to read and you find yourself guessing which to open. Building it
earlier is defensible for one reason: a ranking system should be tuned while the
corpus is small enough to eyeball every result. At 50,000 files you cannot tell
a good weighting from a bad one.

### Schema — contentless, so the text is not stored a third time

An ordinary FTS5 table keeps a **verbatim copy** of every indexed column in a
shadow table, on top of the inverted index. That makes the text exist three
times: the original, the sidecar, and the index. *Measured on 546 KB of real
text: ordinary FTS5 **1.57×** the source; contentless **0.47×**.*

```sql
CREATE TABLE doc(                    -- plain table: returned, never scored
  rowid INTEGER PRIMARY KEY, path TEXT, id TEXT, kind TEXT,
  status TEXT, page INT, sha256 TEXT, updated TEXT);

CREATE VIRTUAL TABLE fts USING fts5( -- exactly four columns, exactly four weights
  title, tags, summary, body,
  content='', contentless_delete=1,
  tokenize='porter unicode61');
```

**One row per note; one row per sidecar *page*.** Page granularity is free — the
sidecar is already page-structured — and it lets an answer cite
*"`documents/sop-b1.pdf` p.3"*. To show the passage, read that page out of the
sidecar.

Keeping `path`/`kind`/`status`/`page` in a plain `doc` table rather than as
`UNINDEXED` FTS columns is deliberate: it keeps the weight vector four long,
with nothing inert to miscount.

### Weighting

The default — `ORDER BY rank` — is bm25 with every column weighted 1.0, and it
gives bad results. *Measured, searching `tiers`: a junk note repeating the word
in its body beat the actual standard note **-4.54 to -2.80**. Weighted 10/8/5/1,
the standard won **-5.29 to -4.54**.*

```sql
SELECT d.path, d.page, bm25(fts, 10.0, 8.0, 5.0, 1.0) AS score
FROM fts JOIN doc d ON d.rowid = fts.rowid
WHERE fts MATCH ? ORDER BY score;        -- ASCENDING. See trap 1.
--           title tags summary body
```

```json
{
  "_comment": "Positional, and MUST equal the FTS column count.",
  "columns": ["title", "tags", "summary", "body"],
  "scored": {
    "title":   {"w": 10.0, "why": "if the question names the note, nothing should outrank it"},
    "tags":    {"w":  8.0, "why": "a deliberately chosen vocabulary, never incidental"},
    "summary": {"w":  5.0, "why": "hand-written; a match means the note is ABOUT this"},
    "body":    {"w":  1.0, "why": "the baseline"}
  },
  "kind":   {"note": 1.0, "document": 0.6,
             "why": "a sidecar is raw extracted text; a note is distilled and current"},
  "status": {"current": 1.0, "draft": 0.7, "superseded": 0.2,
             "why": "a superseded document should never outrank the live one"}
}
```

`kind` and `status` are a second axis bm25 cannot express, and they matter more
than the column weights: **a hit in something you wrote should outrank a hit in
raw extracted text.** Apply them as multipliers after bm25 — and because bm25 is
negative, a multiplier below 1.0 moves the score toward zero, which demotes.
That works, but assert it in a test rather than reasoning it out once.

### Three traps, all of which fail *silently*

1. **`bm25()` returns a negative number; more negative is better.** Sort
   **ascending**. Sort it the intuitive way and you confidently get the worst
   results first.
2. **`bm25()` does not validate the weight count.** Too few weights and the rest
   default to 1.0 with no error — so adding a column shifts every weight one
   position and nothing complains. *Verified: 4 weights and 7 weights on the
   same table returned identical scores.* Assert
   `len(weights) == PRAGMA table_info(fts)` on every run.
3. **`snippet()` returns `NULL` on a contentless table — it does not raise.**
   Assume it works and you ship silently empty excerpts. Read the page from the
   sidecar instead; you get the real text rather than a truncated window.

Also: **`tokenize='porter unicode61'` is not optional.** With the default
tokenizer, searching `tiers` does not match a note titled `renovation-tier`.

A fourth, found in use: **bm25 all but ignores a word that is in about half the
corpus — even in a title weighted 10.** IDF is `log((N − n + 0.5)/(n + 0.5))`,
floored near zero, and the weight multiplies it. In a domain corpus the domain
word is that common: *measured, `tax` in 93 of 192 rows, IDF ≈ 0.06*, so
`Acme Bank Tax` lost to `Acme Bank Income` on *"acme bank tax account"* — the title rule
(*"if the question names the note, nothing should outrank it"*) silently failed.
Raising the title weight cannot fix it (10 × ~0 is ~0). What did: a **title
coverage** multiplier, `× (1 + w × share of the question's words in the title)`,
matched with the FTS tokenizer itself (`title : "word"`), with `w` in the rules
file. Sweep it against the queries rather than picking it: *measured, w = 0 →
8/10, 0.25–0.75 → 10/10, 1.0 → 9/10* (too high, a note merely *about* a term
beats the record that *holds* it). Take the middle of the passing band.

**Title coverage does not survive every word of a question being that common.** It multiplies
bm25, and FTS5 floors IDF to 1e-6 once a word is in half the rows or more, so when *all* the
question's words cross that line every score is ~0 and the order is whichever note repeats the
commonest word. *Measured 2026-11-21 (gm-district #462): `brain` in 58 of 70 rows, `standard`
in 35 of 70 — one note tipped it from 34 of 69 — and "what is the brain standard" went to the
wrong note.* The fix is a **smoothed IDF**, `log(1 + (N − n + 0.5)/(n + 0.5))`, which never
reaches zero: score each word on its own, divide out FTS5's floored IDF (computed from the same
counts) and multiply by the smoothed one. *Measured: 20/22 → 22/22 on the GM's questions; title
coverage then passes 22/22 from 0.5 to 1.5.* A `--raw` FTS5 query keeps FTS5's own bm25.

**A plain question is not FTS syntax.** `MATCH 'when is the 941 due'` requires
every word, including *the* and *is*. Drop stopwords (a list in the rules file)
and OR the content words; offer `--raw` for real FTS5 queries.

### Keep the weights falsifiable

Hand-tuned scores decay into numbers nobody can justify — a threshold of 0.85
that someone chose years ago and no one can defend. The cure is
`references/queries.json`: **~10 real questions, each with the file that should
come back first.**

```json
[{"q": "what are the renovation tiers", "top": "brain/renovation-tiers.md"},
 {"q": "change orders",                 "top": "documents/sop-b1-scope-of-work-creation.pdf"}]
```

The gate runs them and prints the score. A weight change then reads as *"8 of 10
top hits correct, was 9"* instead of a feeling. **The ten queries are the
feature, not the test.**

### The index is a cache, and it is as sensitive as its sources

- **Full rebuild every time**, until that is measurably too slow. *"Delete it
  and it reconstructs completely"* is the property worth having; `sha256` is
  already there when incremental is needed.
- **Fail loudly when the DB is missing** rather than creating an empty one — an
  empty index looks exactly like "no results" and hides the real problem.
- **Contentless is not the same as safe.** The inverted index still stores every
  distinct token; *`strings` on a contentless index still finds the terms and
  digit groups*. It loses word order and casing, nothing else. There is no
  sqlite setting that makes the index less sensitive than its sources, and stock
  `sqlite3` has no encryption.
- So **gitignore it**, and have the gate exit 2 if its path is ever trackable.
  Note `var/` is often *not* ignored wholesale, which makes it the one place
  where a sensitive file defaults to tracked — give it its own line.

- **Another brain's index is as sensitive here as there.** Linked search (§2) reads the
  indexes of the brains a note links to, read-only, and shows their passages. It writes
  nothing, and nothing it reads is copied into this hall's index: `record_links` keeps only
  the link, the brain it names (`brain`) and whether it resolved (`1` found, `0` missing,
  `-1` could not check).

**The rule that holds from day one: the files are authoritative and the index is
a cache. When they disagree, the files win.** Get this wrong and you have a
second source of truth.

---

## 6. What never enters git

The district repo's `.gitignore`, as every hall has it:

```gitignore
# THE BRAIN is its own repository (<prefix>-brain), nested here. The district repo
# never tracks it; commit it with `git -C brain ...`.
/brain/

# EVIDENCE, not knowledge, and the two landing zones. All are LINKS into the cloud, and to
# git a symlink is a file: no trailing slash, or the rule silently matches nothing.
# Never add a negation under them -- a `!*.json` would un-ignore the full text of
# every document while leaving the binaries ignored, which looks tidy and is the leak.
/documents
/messages
/intake

# THE SEARCH INDEX is derived, and holds everything the rules above protect, in
# one file. Note /var/ may not be ignored wholesale, so this is its own line.
/var/brain/
```

Inside the brain repo, its own `.gitignore` carries `private/` if the hall keeps one.

Three things to understand before adopting this:

- **A `.gitignore` written after the first sensitive file arrives is too late.**
  Excluding a path stops future commits only; what is in history stays in
  history. Write it first.
- **"The repo is private" is not the bar.** Private repos leak through access
  changes, forks and permanence. And if the documents belong to a different
  legal entity, privacy is irrelevant — that is an ownership question.
- **The cost is real and must be paid elsewhere.** `documents/` is not in git, so git is
  not its backup; the hall's cloud is its live copy. **A sync is not a backup** — a deletion on one
  device deletes everywhere, recoverable for about 30 days — which is why nothing ever moves or
  deletes inside it without copy, verify, then act.
- **So every hall's `documents/` has a second copy outside its cloud** (gm-district #662), and so
  does whatever a hall keeps in neither git nor its cloud (the GM's `filing/`, what `var/` keeps
  out of git). That copy is the organisation's cold-storage archive, which the Security district
  runs and owns (security-district #19): a monthly, encrypted copy that the uploading account can
  add to and never delete, held for six months. It meets four conditions, and a hall that cannot
  show them has no second copy:
  1. **It covers every cloud the halls use**, not only Dropbox: a hall whose `documents/` live in
     another cloud (`districts.json` → `cloud`) is in it too, and so is every room's.
  2. **It copies real bytes.** A cloud's online-only placeholder reports a size and copies as
     nothing, and two empty files have matching checksums; the archive materialises each file or
     refuses it, and names every one it refused.
  3. **It is verified at the far end**, by listing the archive and comparing checksums, never by
     the local copy having been written.
  4. **It has been restored.** One document, chosen at random, is brought back each quarter and
     compared byte for byte; the date of the last restore is on record.

  Between monthly copies, a document's only other copy is the cloud's own deleted-files store,
  about 30 days, so **a document is never removed from `documents/` in its first month**.
- **A brain that is its own repository is backed up by a private remote**, not
  by a nightly job, if you choose to give it one. Then the remote's privacy is
  load-bearing: check it with the host's API before the first push, list the
  one allowed URL in the rules file, and have the validator exit 2 on any other
  remote or a credential in the URL.

---

## 7. The gate

Enforcement belongs in one command the project already runs.

```make
check: test selftest documents brain

selftest:
	bin/doc-check --selftest
	bin/brain-check --selftest

documents:          # every document has a current sidecar; nothing leaky is trackable
	bin/doc-check

brain:              # index rebuilds; weight arity correct; test queries pass
	bin/brain-check
```

In every hall the gate is the kit's `bin/brain-check` (index, test questions, README current)
and `bin/doc-check` (sidecars, leaks). The GM runs its brain gate nightly at 04:15
(`bin/brain-nightly`); a hall that schedules its own does it with a LaunchAgent, never by
assuming a declared cadence runs.

Each gate: documented exit codes, JSON output, and a `--selftest` that proves it
refuses what it claims to refuse. Build the refusal table as **data** and apply
it to the real function — a guard proven by a re-implementation of itself is not
proven.

Two carve-outs, or the gate becomes noise:

- **A fresh clone has no documents** (they're gitignored). Print a SKIPPED
  banner and distinguish *no sources* from *sources present, sidecars missing* —
  a check that passes vacuously is worse than one that doesn't run.
- **A landing zone like `intake/` or `messages/` is exempt** from "every document needs a
  sidecar", or the gate fails the moment mail arrives and gets switched off.

Parity and leak checks (exit 2) are never exempt.

---

## 8. Day one, in order

For a new hall the GM does steps 2–4 with `bin/ship-brain --hall <code> --commit` and the hall
writes its own `rules/`. For any other project:

1. `CLAUDE.md` at root with the five query rules.
2. `brain/` (its own repo), `documents/`, `.gitignore` — **before** the first
   sensitive file.
3. `brain/README.md` as the index, with the two templates.
4. The naming standard and its checker (§3), then the `documents` skill. Prove
   it on one document before sweeping.
5. Sweep. For an existing backlog: a script proposes a name per document into a
   manifest, a person approves it, and only then does anything move — copy,
   verify the hash, then remove the original. Acceptance: `git status` shows
   nothing from `documents/`, and every name passes the checker.
6. Frontmatter and a `## Summary` on every existing note. Slowest step; highest
   payoff.
7. The index, with `references/queries.json` written **before** the weights are
   tuned.
8. Wire both gates into `make check`.

---

## 9. Running it unattended

Anything scheduled — a fetcher filing into the documents folder, a nightly rebuild — is a timed
job, and the rules for every timed job are [`hall-standard.md`](hall-standard.md) §8: the plist
tracked under `references/launchd/`, a heartbeat, a failure that reaches a person, and grants on
the exact binary. One trap is the brain's own:

- **A nested brain repository breaks tools that find "the repo" by
  `git rev-parse --show-toplevel`.** A session whose working directory is inside
  `brain/` sees the brain as the repo; hooks that look for their scripts there
  fail — and if they fail closed, every tool call is refused until the directory
  is reset. Work on the brain with `git -C brain …` and absolute paths; never
  `cd` into it.

---

## 10. One adoption, as a worked example — **`[example]`**

How one project (a small retail business, adopted 2026-11-06/07) applied this.
Its decisions are its own; the reasons are the reusable part.

| Decision | Why |
|---|---|
| `brain/` is its own git repo, gitignored by the project repo, whose `main` deploys to a public server | the project's history was the wrong place for company facts; the brain still gets history |
| No `private/`: account numbers live in ordinary notes. The brain's only remote is a **private** GitHub repo, listed in the rules file | the owner: *"this is a secure brain."* The containment (no public remote, outer repo never tracks it) is what protects it, and the validator enforces both |
| The documents folder is `documents/` in a cloud-sync folder, flat, named `YYYY-MM-DD-<issuer>-<type>[-<ref>]` | the project's `docs/` was already developer documentation; the sync service versions and copies it off-machine |
| Job output (logs, undo records, cache, reports) moved to its own `logs/` tree | received and generated files had shared one tree |
| Status vocabulary `active · blocked · done · archived` (multipliers 1.0 · 1.0 · 0.5 · 0.2) | tasks need *done* and *blocked*; *superseded* became *archived* |
| The rename of the existing backlog went through a reviewed manifest | the backlog's dates and senders could not be trusted to a script alone |

Results: 154 notes migrated with every link that resolved before still
resolving; 10/10 test questions correct; 1,100 documents proposed names, ~75%
needing no review.

---

## Appendix: what was measured

Verified on macOS, Python 3.15 / sqlite 3.54.1, poppler 26.09, LibreOffice 25.8.

| Claim | Measurement |
|---|---|
| `.docx` → PDF conversion is faithful | 3.5 s, 5 pages, **3,808 chars**; the publisher's own PDF export of the same document gave **4,107**. Difference is blank-line handling and a zero-width space. |
| Ordinary FTS5 stores the text verbatim | shadow tables include `fts_content`; `SELECT body` returns the exact string |
| Contentless FTS5 does not | no `fts_content`; `SELECT body` → `None` |
| …but still leaks vocabulary | `strings` on a contentless index finds the lowercased terms and digit groups |
| Index size, 121 rows / 546 KB of text | ordinary **860 KB (1.57×)**, contentless **256 KB (0.47×)** |
| Unweighted bm25 ranks badly | junk note **-4.54** beat the real standard **-2.80** on `tiers` |
| Weighted 10/8/5/1 fixes it | standard **-5.29** beat junk **-4.54** |
| bm25 accepts a wrong weight count | 4 weights and 7 weights on an 11-column table → identical scores, no error |
| `snippet()` on contentless | returns `NULL`, does not raise |
| `contentless_delete=1` | accepted on 3.54.1; `DELETE FROM fts WHERE rowid=?` works |
| Multipliers order correctly | note **-5.42**, live document page **-0.88**, superseded twin **-0.18** |
| Porter stemming is required | default tokenizer: `tiers` does not match `tier` |
| A cloud-storage placeholder is indistinguishable from an empty file | a real file reporting **1,692,900 bytes in 0 blocks** |
| A common word vanishes even from a weighted title | `tax` in **93 of 192** rows → IDF ≈ **0.06**; `Acme Bank Tax` lost to `Acme Bank Income` on *"acme bank tax account"* |
| Title coverage fixes it, and the weight must be swept | `w` = 0 → **8/10**; 0.25, 0.5, 0.75 → **10/10**; 1.0, 1.5 → **9/10** |
| Contentless index on a real brain | **3,132 KB → 1,644 KB** for 154 notes + 38 repo docs |
| Free-form document titles do not survive a backlog | of **1,100** filed documents, **187** carried old folder paths in the name and **~305** followed no pattern |
| A launchd job and Full Disk Access | same Python, `~/Library/Safari` and an external drive: **both refused** with the grant on the wrong binary, **both OK** with it on `Cellar/…/bin/python3.15` |
