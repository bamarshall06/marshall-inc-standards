# The deploy standard — getting an app to production, and proving it is live

> **Public edition.** A working standard from Marshall Inc, shared as it is used. Real names, paths and accounts are replaced with generic ones (floors such as *Business* and *Teaching*, `/Users/<you>`, `<your-github-user>`). References like `gm-district #372` point to private repositories and are kept only as history. Start with the [README](../README.md).

## Summary
> How an app reaches production and is proven live: the `ship` skill and each hall's `references/ship.json`, the M4's self-hosted GitHub Actions runners, the traps, and the eight parts of a safe automatic deploy that ships no secrets.

**Status:** proven 2026-09-08; trap 7 replaced 2026-09-21; rewritten for GitHub Actions
2026-11-19 (#147); made the organisation's deploy standard 2026-11-19 (gm-district #377,
milestone #24), renamed from `actions-deploys.md` and given how a deploy is started (below) and
the production baseline's owner. **Applies to:** every app any hall deploys. Entry point:
[`sop.md`](sop.md).

Written up from what Family built and the nine failures it cost, and corrected since from what
Business, Money and PROJECTS found. Copy the pattern; the traps below are not
hypothetical, and most of them fail in a way that blames the wrong thing.

---

## Starting a deploy: the `ship` skill and `references/ship.json`

**One app, the whole chain: what is committed goes to production and is proven live.** The org
`ship` skill, linked into every hall, does it with `scripts/ship.py` — dry run, then `--commit`
when Bryan has asked to ship: preflight, push (or open a pull request), wait for the test run on
that SHA, start or watch the deploy run, and **verify the health URLs**. *(Its name clashes with
`bin/ship`, which proposes and merges work ([`github-standard.md`](github-standard.md) §5);
renaming the skill `deploy` is open for Bryan.)*

**Each hall supplies `references/ship.json`**, one entry per app, so nothing about a repo, a
workflow or a URL is in the script:

| Key | Means |
|---|---|
| `path` | the app's checkout, from the hall root (`apps/<app>`) |
| `repo`, `remote`, `branch` | the GitHub repo, its remote (`origin`), the branch production builds from |
| `mode` | `push`: push `HEAD` to `branch`. `pr`: open a pull request and **stop** — the deploy is a later ship |
| `test_workflow`, `runner` | the run on the pushed SHA that must be green; a self-hosted runner that must be online |
| `deploy.trigger` | `push` (watch the run the push started), `dispatch` (run `workflow` with `inputs`) or `none` |
| `health` | every URL and the status it must answer. **Required when anything deploys** |

**Rules:** it ships only what is **committed** — it never stages and never commits (commit by
path first). **The verifier is the health check, not the run:** `SHIPPED` means production
answered as configured; a green run with a wrong answer from production is a failure.
`--allow-untested "<reason>"` ships past a test run that failed or never ran, and writes the
reason down. A hall whose deploy does more (a migration step, a proofread gate) overrides the
skill with its own folder and says why in `sop-local.md` — or, better, runs a separate skill
*before* it. **Every app a hall deploys has its entry in `references/ship.json`.**

**Marshall Inc is not shipped this way.** It is a Mac app, and its release is the Computer Room's
deploy: notarized by Apple, then published to every Mac, only on Bryan's word, carrying every
hall's pinned package ([`hall-standard.md`](hall-standard.md) §12). The org `ship` skill is for
the halls' servers and web apps and never ships Marshall Inc. A hall's Mac app is never a deploy
of its own: it is a package tag sent to `bldg`.

## The production baseline belongs to Security

What the server must look like — loopback bind, a daily dump with a size floor and an off-box
copy, the outgoing binary kept, nginx headers, no dead certificate renewals, one migration
engine — is the **production baseline**, written and maintained by the Security district in
[`security-standard.md`](security-standard.md). A deploy must leave the box meeting it; Security
measures it. Several parts of a safe deploy below (the pre-migration dump, `server.prev`) are the
deploy's share of it.

---

## What this replaces

A GitHub-hosted runner deploying to a cloud VPS with a **private key stored as a repository
secret**. The self-hosted version keeps the key in 1Password and never writes it anywhere.
That is the whole reason to bother.

| | GitHub-hosted (`ubuntu-latest`) | Self-hosted on the M4 |
|---|---|---|
| Where it runs | a disposable cloud VM | the M4, **Bryan's login session** |
| SSH key | `secrets.SSH_PRIVATE_KEY` — a copy of the key lives on GitHub | 1Password agent. **No copy anywhere.** |
| Architecture | linux/amd64 for free | **darwin/arm64** — cross-compile or ship a broken binary |
| Toolchain | `setup-go` / `setup-node` | already installed; PATH is the problem instead |
| Parallel jobs | separate machines + `upload-artifact` | one machine; artifact round-trips are pure overhead |

**The deploy ships no secrets.** Business went from 43 repository secrets to zero on
2026-09-21 and its workflow carries not one `secrets.*` or `vars.*` reference. A proposal to
add a repository secret to a deploy is that mistake coming back; refuse it. The server's
`.env` is authoritative and left alone: **when a key rotates, the server's `.env` is updated
directly from 1Password. Nothing regenerates it.**

---

## The runners

**Every deploy runs on a self-hosted runner on the M4, and ships no secrets.** No deploy runs
on a GitHub-hosted runner, and no repository holds a key, a server address or a login as a
secret.

**One runner per repository.** Each is a LaunchAgent on the M4,
`com.example.github-runner-<name>`, installed in `~/actions-runner-<name>`, logging to
`~/Library/Logs/github-runner-<name>.log`, and registered to its own repo with a label the
workflow names in `runs-on` (`[self-hosted, m4]`, or a label of its own). **The runners are the
GM's:** a hall asks the GM for one, and never edits one to suit one repo. A runner whose repo is
retired is unloaded and removed.

**The runner runs directly on the Mac, not in a container, and that is required.** The job
needs the real volume and Bryan's 1Password agent socket. Neither exists inside a container.

### One OS user, one SSH agent: an accepted exposure

The runners are separate processes but **one OS user and one SSH agent**. The 1Password
service-account token *is* stripped from each (`exec env -u OP_SERVICE_ACCOUNT_TOKEN … run.sh`
in every plist, verified 2026-11-19), so no district's vault scope leaks.

**The SSH agent is not stripped and cannot be** — it is what makes these jobs work. So any
workflow on any of these runners can ask the agent for any key it holds: `server-key-a`,
`money-app`, `server-key-b` and the rest. A malicious or careless workflow file is therefore an
estate-wide problem, not a district one. Runners under distinct OS users would close it.
Bryan chose this knowingly. **Treat a workflow file as security-relevant code and review it
as such** — and never trigger on `pull_request_target`, which runs a fork's workflow with
that access.

---

## The nine traps

Numbered because most of them fail while blaming something else.

### 1. PATH — and the two fixes that do not work

The runner is a LaunchAgent, so launchd hands it `/usr/bin:/bin:/usr/sbin:/sbin`. **No
`/opt/homebrew/bin`.** Every Homebrew tool is missing.

Two obvious fixes were tried and **neither reaches a job on a host-mode runner**: a job-level
`env: PATH:` (failed as trek run #4) and `$GITHUB_PATH` (failed as run #5).

The hook that works is the runner's own LaunchAgent, which every plist here carries:

```xml
<!-- ~/Library/LaunchAgents/com.example.github-runner-<name>.plist -->
<key>EnvironmentVariables</key>
<dict>
  <key>PATH</key>
  <string>/opt/homebrew/bin:/opt/homebrew/sbin:/usr/bin:/bin:/usr/sbin:/sbin</string>
</dict>
```

**The runners are GM-owned.** Do not edit one to suit one repo — ask the GM. Prove PATH
arrived in a step rather than trusting it:

```sh
for t in go node npm rsync git ssh; do printf '  %-6s %s\n' "$t" "$(command -v "$t" || echo MISSING)"; done
```

### 2. `/usr/bin/rsync` is openrsync, not GNU rsync

Modern macOS ships openrsync — a different implementation. Deploying with it by accident
is worse than not deploying. **Refuse rather than fall back:**

```sh
ver=$(rsync --version 2>&1 | sed -n '1p')
case "$ver" in
  *openrsync*) echo "REFUSING: openrsync, not GNU rsync. PATH is wrong."; exit 1 ;;
esac
```

### 3. `head -1` in a pipeline kills the step — use `sed -n 1p`

**This cost four red runs reporting a broken toolchain while the toolchain was fine.**

Steps run under `bash -eo pipefail`. `rsync --version | head -1` makes `head` close the
pipe, `rsync` dies on SIGPIPE with status 141, `pipefail` promotes that to the pipeline's
status, and `-e` kills the step. `sed -n '1p'` reads to EOF, so nothing is signalled.

This applies to **any** `long-output-command | head`, not just rsync.

### 4. The runner is darwin/arm64 — cross-compile explicitly

`go build` on `ubuntu-latest` produced linux/amd64 as a side effect of where it ran. Here
it produces a **Mac binary that cannot execute on the server** — and it fails at
`systemctl restart`, not at build, so the failure surfaces after the service is already
stopped.

Set `GOOS=linux GOARCH=amd64` explicitly, then **verify the artifact is ELF x86-64 before
shipping**.

### 5. Build outside the tracked tree

**A deploy must not write into anything git tracks.** SF's backend binary is tracked, and its
workflow used to `go build -o bin/server`: every successful deploy dirtied the tree, and the
*next* deploy failed its own "refuse anything not committed" gate. Run #13 succeeded and run
#14 died on it — two deploys in a row was not possible, and it only ever looked fine because
each new commit happened to carry the rebuilt binary.

Build into a cache path (`~/Library/Caches/<name>-deploy`), and keep that path **separate**
from the checkout path below, which is `rm -rf`'d on every run.

### 6. The 1Password agent must be checked, not assumed

```yaml
env:
  SSH_AUTH_SOCK: /Users/<you>/Library/Group Containers/2BUA8C4S2C.com.1password/t/agent.sock
```

```sh
test -S "$SSH_AUTH_SOCK" || { echo "agent socket absent -- is the desktop app running and unlocked?"; exit 1; }
ssh-add -l | grep -q server-key-a || { echo "agent is not offering 'server-key-a'"; exit 1; }
```

The agent needs the **desktop app**, unlocked. A service account cannot drive it — that
wrong turn was taken on 2026-09-02. This is why the runners are LaunchAgents and not
LaunchDaemons, and why a headless runner on the iMac could not do these jobs at all.

### 7. The trigger — automatic on push, guarded by checks that hold unattended

**This replaces what this section said until 2026-09-21.** It used to prescribe
`workflow_dispatch` with a required `confirm: Type DEPLOY to publish` input and said *"Do not
add `push:` without Bryan saying so."* **That gate did not exist.** Money measured it on
2026-09-21: its `bin/deploy` supplied `{"confirm": "DEPLOY"}` itself and its `bin/ship` ran
`bin/deploy --commit` as the last step of every ship. No human typed it. The deploy was already
automatic — the trigger was `bin/ship` rather than `push` — and this document told anyone
reading it that production was protected when it was not.

**A typed confirmation is only a gate where a human types it.** The moment a district builds a
dispatch tool — the natural next step, and one this organisation encourages — the confirmation
becomes a string one script passes to another.

**Business reached this first**, on 2026-09-14, and its reasoning is the standard now:

> *"the human tick was never actually checking the schema. It was a pause, not a test, and a
> pause that fires on every deploy stops being read. Two migration collisions in two days both
> got through it."*

SF did not remove the gate. It **replaced** it with a migration-numbering preflight that hard
refuses the exact two failures the tick let past. That is the transferable idea.

So: automatic is the standard, in eight parts. Each exists because of a named failure.

**1. `branches: [main]`** — not a bare `push:`, which deploys topic branches. A bare `push:` is
correct for a test workflow and wrong for a deploy.

```yaml
on:
  push:
    branches: [main]
  workflow_dispatch: ...
```

**2. Limit the paths that deploy** — otherwise a documentation commit rebuilds and restarts
production. Either an ignore list:

```yaml
    paths-ignore: ['docs/**', '**.md', '.claude/**', 'scripts/**', 'var/**']
```

or, better, Money's **allowlist** of the only trees that ship
(`paths: ['apps/backend/**', 'apps/frontend/**']`). An ignore list has to be extended every
time a new top-level directory appears, and the failure is silent.

**3. The migrations expression — the sharp edge.**

```yaml
      - name: Apply pending migrations
        if: ${{ github.event_name == 'push' || inputs.run_migrations }}
```

**A push trigger has no inputs.** So `inputs.run_migrations` is empty, which is falsy, which
means a naive `on: push` added to an existing dispatch workflow produces a deploy that
**silently never migrates** — while helpfully printing its "skip migrations" branch. The new
binary then meets the old schema. This is the single most likely way to get this wrong,
**and it fails in the direction that looks fine.**

**4. A migration-numbering preflight** — SF's, and the reason the tick could be retired.
Hard-refuse two migrations sharing a version number; **name** any pending file at or below the
applied high-water mark, because goose keys on the number and *skips* such a file while the
deploy reports success. SF's two real incidents: run #88 caught two unapplied files both
numbered 161 (loud, production stayed up), and the next day a second file numbered 162 when 162
was already applied — which is not an error, it is a skip, and the first request afterwards
takes the process down minutes after a green deploy.

**5. A `concurrency:` group.**

```yaml
concurrency:
  group: deploy-${{ github.repository }}
  cancel-in-progress: false
```

Two deploys of the same repo share one fixed build path, so without it the second run's
`git worktree remove --force` destroys the first run's build mid-flight. Under dispatch a
human paces it; under `on: push` it is two merges in a minute. `cancel-in-progress` is
**false**: cancelling a deploy mid-migration is how a schema ends half-applied.

**6. A pre-migration dump.** A nightly backup means a migration at 14:00 has an
eleven-hour-old copy behind it. Under dispatch a person chose that moment; under automation
nobody did.

```sh
sudo -n mysqldump --single-transaction "$DB" | gzip > "/home/ubuntu/backups/pre-deploy-$(date +%s).sql.gz"
```

**7. Preserve the outgoing binary.** Stage the new one beside the old and keep the old as
`server.prev` before the swap. **A deploy you cannot undo is not made safe by usually
working.**

**8. Keep the escape hatch, and fail loudly.** Keep `workflow_dispatch` beside `push` — it is how
you deploy a rollback or re-run after fixing the box. On failure, fail the run and file **one**
tracking issue, updated rather than duplicated; a job that files a fresh issue every morning
trains you to close them unread. **Do not roll back a migration automatically:** a rollback
cannot undo a migration, and restoring an old binary against a new schema is worse than a
visible outage.

#### The readiness checklist

Every deploy answers **yes** to each of these before it runs on `push:`. The point is that the
decision is made against facts rather than against how careful the workflow file reads.

- Does CI prove the build — a `pull_request` test job — not just vet it?
- Does the deploy build the *pushed commit*, from a detached worktree?
- Is the outgoing binary preserved (`server.prev`)?
- Is there a migration-numbering preflight?
- Is there a `concurrency:` group?
- Is there a pre-migration backup?
- Is there a post-deploy health check, on the box first, then public?

A hall that deploys by hand instead (`workflow_dispatch` with a `confirm`) records why in its
`sop-local.md`.

#### The one original objection that has not expired

The runners are LaunchAgents in a login session. **A push while the Mac is asleep, or while
1Password is locked, fails at the SSH-key check** — the agent cannot serve a key from a locked
vault. Under dispatch this is invisible, because a human dispatching a deploy is by definition
at an awake machine. Under `on: push` it is a real failure mode, and the honest mitigation is
that the run goes red and someone notices. Money's workflow recorded three objections —
*"asleep, locked, or mid-edit"* — and `mid-edit` was solved by the detached-commit build below.

### 8. Declare a `type:` on every dispatch input, and confirm with a checkbox

GitHub accepts an untyped input as a string, so an untyped `confirm` renders as a text box and
works. Declare the type anyway: it says what the gate is, and the form then renders what you
meant rather than whatever the default is.

Where a human does confirm, a boolean checkbox is preferred over typed text — *the gate exists
so that a deploy is a decision rather than a stray click; it does not need to be a spelling
test.*

```yaml
      confirm:
        description: 'Yes — publish to <site>'
        type: boolean
        required: true
        default: false
```

A push carries no inputs, so a `confirm` step must be guarded by
`if: ${{ github.event_name == 'workflow_dispatch' }}`, as Money's is — or every push fails
its own gate.

### 9. A tar built on macOS ships AppleDouble `._` files

A checkout on the M4's runner carries extended attributes (`com.apple.provenance`), and macOS
`tar` writes each one as a `._<name>` companion file. Nothing on the runner shows them, and
`tar -t` on macOS hides some, so the archive looks clean and is not. Teaching's first
LMS deploy to the department server put 876 of them on the server, and its content importer failed reading them;
two other deploys shipped 55 more without failing (teaching-district #78–80, gm-district #526).

**Every deploy that builds a tar, or rsyncs, from a runner on macOS does both:**

1. Build with `COPYFILE_DISABLE=1` in the environment of the `tar` (or the whole step).
2. Before it ships, list the archive with Python's `tarfile`, not `tar -t`, and refuse any
   member whose last path part starts with `._`:

```bash
COPYFILE_DISABLE=1 tar -czf "$OUT" -C "$BUILD" .
python3 - "$OUT" <<'PY'
import sys, tarfile, posixpath
bad = [m.name for m in tarfile.open(sys.argv[1]) if posixpath.basename(m.name).startswith("._")]
if bad:
    sys.exit(f"refusing {sys.argv[1]}: {len(bad)} AppleDouble member(s), e.g. {bad[:3]}")
PY
```

An rsync from the runner excludes them instead: `--exclude '._*'`.

---

## Build a checked-out commit, not the working tree

**This inverts what this document said until 2026-09-20.** A working-tree build was the
norm here and a checkout was the exception. It is now the other way round, because the
guards that made the working-tree build defensible **do not hold**. Full reasoning, the
measurements, and the wider methodology this belongs to:
[`github-standard.md`](github-standard.md) §4–§6.

The guards previously recommended here were these:

```sh
[ -z "$(git status --porcelain)" ] || { echo "working tree dirty"; exit 1; }
git fetch origin main --quiet
[ "$(git rev-parse HEAD)" = "$(git rev-parse origin/main)" ] || { echo "push first"; exit 1; }
```

They are **point-in-time**. They run in step 2; the build, the migration loop and the asset
pipeline run in steps 6-9 against the same mutable directory. Any edit landing in that
window ships to production with the gate already passed, and the run still reports the
pre-edit SHA. The migration loop globs the live directory, so a migration file appearing
mid-run is applied to production. And the deploy is itself a **writer** — `npm ci` deletes
and reinstalls `node_modules`, `npm run build` rewrites `dist/` — so a dispatched deploy
pulls the floor out from under anyone using the tree. With a dozen live Claude Code sessions
across the districts, that is the normal case, not bad luck.

**So: materialise a detached worktree of the exact commit, and build that.**

```sh
# 1. The volume must be a mount point. An unmounted volume is an empty directory on the
#    boot disk that every later check passes vacuously against. Business's guard.
mount | grep -q "on $VOLUME " || { echo "REFUSING: $VOLUME is not a mount point."; exit 1; }
test -f "$REPO/<a file that must exist>" || { echo "REFUSING: not the repo"; exit 1; }

# 2. Materialise the exact commit. Everything after this builds $WORK, never $REPO.
cd "$REPO"
git fetch origin main --quiet
git worktree remove --force "$WORK" 2>/dev/null || true
rm -rf "$WORK"
git worktree add --detach "$WORK" origin/main      # or "$SHA" for a test job
echo "building $(git -C "$WORK" rev-parse HEAD)"

# 3. ALWAYS tear down (`if: always()`). `git worktree add` refuses a path already
#    registered, so one orphan from a cancelled run breaks every subsequent run.
git -C "$REPO" worktree remove --force "$WORK" 2>/dev/null || true
rm -rf "$WORK"
git -C "$REPO" worktree prune
```

The `rm -rf` between remove and add is **not** redundant: `worktree remove` fails on a path
that exists but is not registered, so `|| true` alone leaves a directory that `worktree add`
then refuses. Keep the defensive remove in step 2 **and** the `if: always()` teardown: a killed
runner process skips even an `always()` step.

`WORK` goes in `~/Library/Caches/<name>` — Rentals and SF converged on that independently.
Use `git worktree`, not `git archive`: a worktree keeps `.git` available inside the
checkout, which a test job needs to diff a commit range.

**On a `pull_request` run, `github.sha` is the synthetic merge ref** (`refs/pull/N/merge`),
which exists only on GitHub. Fetch it by name, or build
`github.event.pull_request.head.sha` — Money's `test.yml` refuses by name rather than let
`worktree add` fail with "invalid reference".

**Once the build comes from `origin/main`, turn the dirty-tree refusal into a report.**
Uncommitted work is excluded structurally, so refusing on it only means one session cannot
deploy because another has unsaved edits — the exact problem being fixed. Keep printing what
will ship and what will not: the SHA of `origin/main`, any local commits ahead of it, and any
uncommitted changes, each marked *does not ship*.

**`actions/checkout` would work, and the deploys use a worktree anyway**, for two reasons
Money's `deploy.yml` records: the build reads the volume's own object store, so it needs no
network and no credential; and `$WORK` is a detached worktree of one exact commit, a build that
cannot pick up another session's uncommitted edits.

**The one exception: a repo whose real artifact is not in git** — for example images that
are regenerable only through a paid, non-deterministic API, where a rebuild would reshuffle what
is already published. Such a deploy builds the *working tree*, behind the dirty-tree and
`HEAD == origin/main` guards above, which are load-bearing there. Say so in a comment in the
workflow and in the hall's `sop-local.md`, rather than letting the next reader assume it was
sloppy.

**Verify the live site after deploying, on the box first** — a deploy that returns 0 is not a
site that works, a public 200 can come from nginx while the service behind it is dead, and a
public failure can be DNS or TLS rather than you:

```sh
ssh -o BatchMode=yes "$TARGET" 'curl -sf http://localhost:<port>/health' || exit 1
for p in / /lessons/; do
  code=$(curl -s -o /dev/null -w '%{http_code}' --max-time 20 "https://example.com$p")
  [ "$code" = "200" ] || exit 1
done
```

---

## Worked examples

All are heavily commented and are better read than summarised:

- `/Volumes/BUSINESS/.github/workflows/deploy.yml` — the full pattern: automatic on
  push, zero secrets, migration preflight, pre-migration dump, detached worktree of
  `origin/main`, `server.prev` and rollback, on-box then public health check.
- `/Volumes/MONEY/apps/diy-money-app/.github/workflows/deploy.yml` and `test.yml` — the
  path allowlist, a deploy of a fixed SHA, and CI that builds the pull request's head commit.
- `/Volumes/FAMILY/apps/fam-trek-with-us/.github/workflows/deploy.yml` — a straight **port of
  a GitHub-hosted workflow** to the self-hosted runner, with the forced differences documented
  inline. Read this one if you are moving a workflow off `ubuntu-latest`.
- `/Volumes/FAMILY/apps/fam-lds-seminary/.github/workflows/deploy-site.yml` — a working-tree
  deploy, asset gate and live verification.
