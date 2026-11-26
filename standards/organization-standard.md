# The organisation standard — who runs what, and which way things flow

> **Public edition.** A working standard from Marshall Inc, shared as it is used. Real names, paths and accounts are replaced with generic ones (floors such as *Business* and *Teaching*, `/Users/<you>`, `<your-github-user>`). References like `gm-district #372` point to private repositories and are kept only as history. Start with the [README](../README.md).

## Summary
> Who Marshall Inc is made of and the one job of each: the Boardroom sets every standard and runs intake; the Computer Room builds the building; Security does all security; each district runs its own domain. Standards and tools flow down from the Boardroom, only requests come up, and districts talk and send files to each other directly.

**Status:** org-wide standard, written 2026-11-21 (gm-district #493, milestone #35) from Bryan's
words that day: *"the gm 'boardroom' is all powerful and their job is to set the standards. they
also do the intake of the files that come in. thats pretty much it. the computer room sets up the
building gui. the security team does all the security stuff."* Until then the organisation was
described in pieces: the GM's role in `districts.json`, the seats in [`sop.md`](sop.md) and
[`hall-standard.md`](hall-standard.md) §6, and the Computer Room only in its own repo.
**Applies to:** every hall and the Boardroom. Entry point: [`sop.md`](sop.md).

| | |
|---|---|
| §1 | The bodies, and the one job of each |
| §2 | Authority: who writes a standard |
| §3 | Down: what the Boardroom pushes |
| §4 | Up: requests, and nothing else |
| §5 | Sideways: any district to any district |

---

## 1. The bodies, and the one job of each

| Body | Where | Does | Does not |
|---|---|---|---|
| **Boardroom** (the general manager, `gm`) | the root of the `MARSHALL-INC` volume (`districts.json` → `general_manager`) | **sets every standard**, and **runs intake**: everything arriving from outside lands in its `intake/` and is filed to the hall it belongs to | business work; security; building the building |
| **Computer Room** | the Building hall (`bldg`), its own volume, with the Marshall Inc app at `apps/building-app` (`districts.json` → `districts.bldg`) | **builds and runs the building**: the GUI every session starts from, and **releases it to every Mac**. A Marshall Inc release is the Computer Room's deploy, carrying every hall's pinned package ([`hall-standard.md`](hall-standard.md) §12), published on Bryan's word because it goes to Apple. Its people are the Building hall's manager and worker | standards; intake; security |
| **Security** (`sec`) | its own volume (`districts.json` → `districts.sec`) | **all security**: secrets and vaults, the server keys and servers, the secret sweep, the archive gate, the safety of backups, the off-site witness, the tunnels, and the [`security-standard.md`](security-standard.md) | business work |
| **A district** | its own volume (`districts.json` → `districts`) | **its own domain**, with its own brain, documents, repos, vault and people ([`hall-standard.md`](hall-standard.md)) | another hall's domain |

A job that is not in the *Does* column of a body is not that body's. When the Boardroom finds
itself doing security work, or the Computer Room setting a rule, the work moves to its owner.

---

## 2. Authority: who writes a standard

**The Boardroom is all-powerful in one way: it alone sets the standards.** Every document in
`docs/standards/` is the Boardroom's, and a hall never edits one; a hall's own changes go in its
`docs/sop-local.md` ([`sop.md`](sop.md), *Two layers*).

The one exception is authorship, not authority: **Security writes the security standard**, in its
own tree, and the Boardroom copies it into `docs/standards/` and ships it like the others
(`standards.json` → `authors`).

---

## 3. Down: what the Boardroom pushes

Things flow **down**. The Boardroom pushes; a hall does not have to come and fetch.

| What | How |
|---|---|
| The standards | `bin/ship-standards`: a link into each hall, a copy where a hall cannot follow a link (`standards.json`) |
| The org skills | `bin/hall-layers` (`standards.json` → `skills`) |
| The brain kit | `bin/ship-brain` |
| A rule or a change a hall must carry out | a **notice** in the hall's `messages/` ([`communication-standard.md`](communication-standard.md) §3) |
| Files from outside | intake: `bin/file-doc` into the owning hall's `intake/` |

**A notice is binding.** The hall does what it asks and archives it.

---

## 4. Up: requests, and nothing else

**A district sends one thing up: a request.** When it wants a new feature from the Boardroom, or a
change to a standard, it opens an issue on the Boardroom's repository, `gm-district`, with one of
two labels:

| Label | For |
|---|---|
| `request/feature` | something the Boardroom should build or ship: a tool, an org skill, a change to the kit |
| `request/standard` | a change to a standard: what the standard says, what is actually true or needed, and why |

The request is written to [`github-standard.md`](github-standard.md) like any issue, and the
Boardroom triages it like any issue: it is planned, done or closed with a reason. Nothing else is
sent up for the Boardroom to read and answer.

**Corrections are retired** (2026-11-21). A hall no longer writes into `districts.json` →
`notes[]`; the notes written until then are kept as history, and nothing reads them. What a
correction used to say — what the Boardroom claimed, what is true, how it was verified, what to
change — is now the body of a `request/standard` issue.

---

## 5. Sideways: any district to any district

Districts deal with each other directly; nothing between two districts has to pass through the
Boardroom.

- **Any district may message any other district**, through the `message-district` skill
  ([`communication-standard.md`](communication-standard.md) §5).
- **Any district may send a file to any other district**, into that district's `intake/`, through
  the same skill: copied, SHA-256 verified, and never removed from the sender, because both
  folders are in a cloud sync root where a removal deletes on every device.

What stays fixed is ownership, not the route: **a hall writes only into another hall's
`messages/` and `intake/`**, and never into its volume, vault, Reminders list, Contacts group or
Notes folder. The receiving hall decides what happens to what arrives.
