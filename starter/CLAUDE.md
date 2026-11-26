# CLAUDE.md

<!-- Copy this file to the root of your project. Claude Code reads it at the start of every
     session, so the rules hold even after you clear the context. Edit the first section;
     leave the standards section pointing at docs/standards/. -->

## This project

<One or two sentences: what this project is for, and who it is for.>

- Code lives in: <folder>
- Notes I write live in: <folder>
- Documents I receive (PDFs, scans) live in: <folder>, never in git

## The standards

This project follows the standards in `docs/standards/`. **Start with `docs/standards/sop.md`**,
the front page; it points to the other nine. Read the one that applies before you act.

Where this project works differently, the difference is written in `docs/sop-local.md`, and
that file wins for this project. Never edit a file in `docs/standards/`; write the difference
in `sop-local.md` instead.

## Rules that are never broken

These are copied here on purpose: an agent that has to follow a link to learn a rule will
sometimes not follow it.

1. **Nothing gets built without an issue**, and nothing is finished until something other than
   you has checked it (a test, a matching total, a page that loads).
2. **Dry run first.** Show what a change would do, and act only when I say so.
3. **Never delete without asking.** Copy, check the copy, then act.
4. **No secrets in files.** Passwords and keys live in a password manager, never in the repo.
5. **Commit only the files you changed.** Never `git add -A`.
