# The security standard — secrets, keys, the traps, and the production baseline

> **Public edition.** A working standard from Marshall Inc, shared as it is used. Real names, paths and accounts are replaced with generic ones (floors such as *Business* and *Teaching*, `/Users/<you>`, `<your-github-user>`). References like `gm-district #372` point to private repositories and are kept only as history. Start with the [README](../README.md).

## Summary
> Secrets live in 1Password, in the vault named after the hall's volume, and nowhere else; the shared server keys are the Security district's, offered by the 1Password SSH agent and never on disk; the traps that fail silently are named with their reasons; and every live server meets the production baseline, which Security writes, maintains and measures.

**Status:** org-wide standard, written by the **Security district** (`sec`, `/Volumes/SECURITY`)
on 2026-11-19 (security-district #47, for gm-district #376). Security owns it: a change starts
in Security and reaches the GM by message. **Applies to:** every hall and the GM. Entry point:
[`sop.md`](sop.md). The short rules are also in the *Hard restrictions* of every `CLAUDE.md`,
because an agent's contract must carry its rules; this document carries the reasons.

| | |
|---|---|
| §1 | Secrets: 1Password, one vault per volume |
| §2 | Keys: who holds the server keys |
| §3 | Where a secret must never be |
| §4 | The traps that fail silently |
| §5 | The production baseline |
| §6 | How Security checks it |

---

## 1. Secrets: 1Password, one vault per volume

**Every secret lives in 1Password, in the vault named after the hall's volume.** The vault name
equals the volume name, so a hall code maps to a vault with no lookup table.

| Vault | Whose |
|---|---|
| `BUSINESS` · `MONEY` · `FINANCES` · `TEACHING` · `PROJECTS` · `FAMILY` | that district's |
| `SECURITY` | the Security district's, **including the shared server keys** (§2) |
| `INFRASTRUCTURE` | the GM's: what spans halls and is not a server key |
| `MARSHALL-INC` | Marshall Inc, the building |
| `Personal` | Bryan's own logins; no automation reaches it |

- **A hall writes only into its own vault**, the same boundary as its Reminders list and Notes
  folder. Rentals's vault left with Rentals.
- **The boundary is enforced by the credential.** Each hall has a 1Password **service account**
  scoped to its own vault. `~/.zshenv` picks the token from `$PWD`: under `/Volumes/<VOLUME>` a
  shell gets that vault and nothing else, and a directory that maps to no vault gets **no**
  token, so `op` prompts rather than quietly using a broader one. The token is chosen at shell
  start: after `cd` between halls, `op-token-reload`; `op-token-status` says which you hold.
- **Machine-local automation reads the login keychain** through the GM's `bin/_secret.py`
  (Keychain first, 1Password as the interactive fallback). The login keychain does **not** sync
  between Macs, so a Keychain copy is never the only copy.
- **Store a secret in the keychain with `bin/keychain-put`**, never
  `security add-generic-password`: `-w` with no value truncates at 128 bytes, `-w <value>` puts
  the secret in `ps` and in a Claude Code allow-rule, and `-w -U` stores the literal `-U`.
- **A scheduled job never reaches for `op`.** With no token in its environment, `op` falls back
  to the desktop app and raises a dialog nobody can click; the job blocks, then fails. A launchd
  job reads the keychain instead. Wrapping it in `zsh -c` to get a token makes the shell the
  TCC-responsible process, which has no Reminders or Calendar grant.
- **AWS credentials come from 1Password** with `op read` into the environment, never from a
  file.

## 2. Keys: who holds the server keys

**The shared server keys are the Security district's** (gm-district #309, 2026-11-19): `server-key-a`
and `server-key-b` live in the `SECURITY` vault, and Security runs everything that reaches the
servers. **A district may hold its own server key, in its own vault, and no other district's**:
`money-app` (`MONEY`), `fam-money-app` (`FINANCES`), Business's own key
(`BUSINESS`) and Teaching's test identity (`TEACHING`). `INFRASTRUCTURE`
holds no server key.

- **A key never touches disk.** The 1Password SSH agent offers it (`~/.config/1Password/ssh/agent.toml`
  lists which items, in order); `ssh -i keys/<name>.pub` names the **public** half and the agent
  supplies the private one. A hall's `keys/` holds public halves only. Revocation is one action,
  in one place.
- **Order in `agent.toml` matters.** `ssh` offers keys in order and a server allows about six
  attempts; a host that needs a later key is pinned with `IdentitiesOnly` in `~/.ssh/config`.
  Too many keys reads as a dead host, not as an auth failure.
- **A service account cannot drive the SSH agent.** The agent needs the desktop app; no token of
  any scope stops `ssh` asking for approval. Fixing SSH prompts with service accounts is a wrong
  turn that has been taken once already.
- **One `ssh` connection per run, not N.** `~/.ssh/config` sets `ControlMaster auto`,
  `ControlPath ~/.ssh/cm/%C`, `ControlPersist 30m`, so a scheduled job costs one approval.
  `ControlPath=none`, not `ControlMaster no`, opts out.
- **A key that has been displayed is rotated, not moved.** A private key printed in a session
  survives in that session's transcript.

## 3. Where a secret must never be

Not in `.env`; not in a config file; not in a git remote URL; not in `.git/config`; not in
`.claude/settings.local.json` (running a command once with an inline secret saves the whole
command there as an allow-rule); not in a workflow file or a repository secret; not in a commit;
not in an archive. **Every script that zips, copies or syncs excludes `*.pem`, `*.key`,
`*id_rsa*`, `.env` and `*credentials*`, and is checked, not assumed** — then reads the archive
back and scans it by **content**, because a live token once sat in a file named `service`, which
no name pattern matched. "Could not check" is a refusal, never "clean".

## 4. The traps that fail silently

Each of these produced a confident, wrong answer on this Mac. The rule is in every `CLAUDE.md`;
the reason is here.

| Trap | What happens | The rule |
|---|---|---|
| **`grep -r` skips gitignored paths** | In an interactive shell `grep` is ugrep honouring `.gitignore`; on 2026-09-05 a search for a known password across a volume found nothing, while the same search in its `inbox/` found the credential file. | Sweep with `find … -print0 \| xargs -0 grep -lF "$SECRET"`, with a **positive control** that proves the sweep can find a planted copy. |
| **`--no-ignore-files` is not a fix** | Outside an interactive shell `grep` is BSD grep, which rejects the flag on stderr; with `2>/dev/null` the error reads as "no matches". | The same: `find \| xargs grep`, and a positive control. |
| **`log` fails emptily** | `log` is a shell function too; `log show --predicate …` errors on stderr and prints nothing. | Call `/usr/bin/log`, and treat an empty result as "could not check" until a control returns a row. |
| **The tmux token leak** | A tmux server copies the environment of the shell that started it into every pane, so one started from the GM hands its wide token to every hall's session. | Start sessions with `bin/dm`, which strips the token and unsets it on the server; `bin/dm doctor` proves it. |
| **Online-only placeholders** | A Dropbox or OneDrive file can report a real size with zero blocks; two empty reads checksum equal. | Refuse `st_blocks == 0 and st_size > 0`; materialise first. |
| **An unmounted volume** | Writing to `/Volumes/<NAME>` while it is unmounted creates a folder on the boot disk, and the data vanishes when the volume returns. | `os.path.ismount`, never `exists`. |
| **Deleting inside a sync root** | A deletion propagates to the cloud and every device. | Copy, verify, then act. |
| **A check that exits 0 having done nothing** | A remote command fed no input, a skill that could not reach a server, a sweep that skipped a directory: each reported clean. | Count what was checked. An empty result is a failure until proved otherwise. |

## 5. The production baseline

**What every live server must look like.** Security writes it, maintains it and measures it
(§6). A server that does not meet a line has an issue on `security-district`, or an exception
recorded there with its reason.
Exceptions are data, in `.claude/skills/shared/references/baseline-exceptions.json`, each with
its reason and its issue.

| | The rule |
|---|---|
| **Bind** | An application binds **loopback**, never `*:port`; nginx is the only public listener (80, 443) beside SSH (22). UFW is the second layer, not the only one. |
| **Backup** | Every database is dumped on a schedule, with a **size floor**, a stated **retention**, and an **off-box copy**. A dump is written as `.partial` and renamed on success, so a truncated run never looks like a restore point. |
| **Restore** | **A dump that has never been restored is not a backup.** Read one back into a scratch database and count the tables. |
| **Rollback** | The outgoing build is kept, so a deploy can be undone. A documented rollback that nothing implements is worse than none. |
| **nginx** | HSTS; `location ~ /\. { deny all; }` (a dotfile is refused with 403 or 404, not served and not answered by the app's catch-all); and the security headers `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, `Content-Security-Policy`, `Permissions-Policy`. |
| **certbot** | **No dead renewal configs.** A domain that no longer points at the server has its renewal removed; a renewal that fails twice a day trains everyone to ignore it. |
| **Migrations** | One migration engine per application, chosen deliberately, with a numbering preflight in front of it. |
| **Access** | SSH by key only, from the 1Password agent (§2); no password login; fail2ban on. |
| **Inbound** | **Every inbound webhook proves who sent it.** A request from another company (Plaid, Shopify, Stripe…) is accepted only when the sender's signature verifies, such as Plaid's `Plaid-Verification` JWT or Shopify's HMAC, and is refused otherwise; an unverified webhook is an open write into the app. Checked per app, in its code. |
| **Stack** | Every server runs **the standard stack** below, at or above each minimum. A departure is a choice written down, never drift. |

### The standard stack

**What every live server and app is built from**, so a new app starts from it and an old one
cannot fall behind unnoticed (security-district #99). It is **data**:
`.claude/skills/shared/references/standard-stack.json` holds each part and its **minimum
version**, and `baseline-check` measures every server against it on every run. **Keeping it
current is an edit to that file:** raise a minimum and change `updated`, and the next run names
every server and service still below it. The minimums are in the file and nowhere else, so this
page never goes stale when one is raised.

| Layer | The standard | Measured |
|---|---|---|
| Server | Ubuntu LTS on a cloud VPS, security updates automatic | on the server |
| Web | nginx, the only public listener; certbot for certificates | on the server |
| Database | MySQL, loopback only, on the same server | on the server |
| Access | OpenSSH, key only; fail2ban; UFW | on the server |
| Language | Go, built with a **supported** release | each running service's binary |
| Backend | Gin; SQL through sqlc | each app's repo |
| Migrations | goose, run by the deploy, numbering preflight, dump first | each app's repo |
| Frontend | React with Vite, a static build served by nginx | each app's repo |
| Process | a systemd unit per service, on loopback, behind nginx | the Bind line |
| Deploy | GitHub Actions on the M4's runner; no repository secrets; the key from the 1Password agent | `docs/standards/deploy-standard.md` |

**A departure is a choice written down, never drift:** a row in the file's `known_departures`
with its reason, or a `Stack` exception in `baseline-exceptions.json` with its issue.

## 6. How Security checks it

- **The server checks** are this district's skills: `server-health`, `ssl-monitor`,
  `backup-verify`, `db-monitor`, `weekly-maintenance`, `incident-response` — all read-only.
  The baseline gets its own read-only check, one SSH session per server, which reports each line
  as `ok`, `not-ready` or **`could not check`, which never counts as passing.**
- **Secrets are swept** by the GM's `bin/credential-sweep`, which walks the tree itself rather
  than trusting `grep`, and reaches into `.git/` for the files named in its references.
- **Every role proposes; none acts.** A finding goes to Bryan with the exact `--confirm` command
  that would fix it ([`doe-framework.md`](doe-framework.md) §14).
- **Changing this standard:** edit it in Security's `docs/security-standard.md`, then send it to
  the GM with `message-district --to gm`; the GM copies it into `docs/standards/` and ships it.
