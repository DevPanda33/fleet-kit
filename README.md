# fleet-kit

Single source of truth for the **semantic-git conventions kit** used across the
DevPanda33 Conductor fleet (~24 repos): one reusable CI workflow, one validator script,
one set of local git hooks, one installer. Each fleet repo carries a ~8-line caller stub;
the policy itself lives here and updates by tag.

> **Status — pre-work (2026-08-16).** This repo is intentionally empty apart from this
> README. Nothing is released yet: **no `v1` tag exists.** The kit files land at task
> **T-3** of the approved *Semantic Git Conventions — Fleet Pattern v0.1* plan. This
> README exists first so T-3 has an address to ship to and the other 23 repos have a
> stable `uses:` ref format from day one.

---

## The release contract

This is the contract. It is normative from today, before any code exists, because every
caller stub written against fleet-kit depends on it.

### 1. Tag scheme

Two kinds of tag, meaning two different things:

| Tag | Mutability | Purpose |
|---|---|---|
| `v1` | **floating** — re-pointed at each `v1.x.y` release | what caller stubs pin; the fleet's deployment channel |
| `v1.x.y` | **immutable** — never moved, never deleted | pinning and rollback |

`v1` is re-pointed at a `v1.x.y` release **only after both** of these hold:

1. **fleet-kit CI is green** on that release, **AND**
2. **a canary PR on `DevPanda33/devpanda` passes** against it.

Caller stubs across ~24 repos pin `@v1`, so one retag updates fleet CI at once. Read that
plainly: `@v1` is **a mutable deployment channel by design, not "pinning."** The immutable
`v1.x.y` tags shipped alongside every release are what pinning and rollback actually use.

Release procedure that follows from the above:

1. Cut the immutable `v1.x.y` tag. The release script bakes fleet-kit's explicit `ref:`
   into the reusable workflow's own checkout step at tag time, so a run reached through
   `@v1` fetches the validator from the same immutable commit it was released as.
2. Wait for fleet-kit CI to go green on that tag.
3. Open the canary PR on `DevPanda33/devpanda` against that immutable tag; it must pass.
4. Only then force-move `v1` to that commit. The fleet picks it up on its next PR.

Rollback has two levers: move `v1` back to the last-good `v1.x.y`, which reverts the whole
fleet; or pin one repo's stub to a `v1.x.y` tag directly, which takes that repo off the
channel without touching anyone else.

### 2. The stable `uses:` ref

```yaml
uses: DevPanda33/fleet-kit/.github/workflows/semantics.yml@v1
```

That address is fixed as of this README — repo, path, and channel tag. Repos may
substitute a `v1.x.y` tag for `@v1` to pin. A `v2` would be a new channel tag and a
deliberate per-repo stub edit, never a `v1` re-point.

### 3. Upgrade invariant

> Within major `v1`, CI validation stays backward-compatible with the previous local hook
> minor, **OR** `v1` is re-pointed only after the batch hook upgrade lands.

Local hooks are per-repo *copies* carrying a `# semantic-git-kit vX.Y` stamp; CI always
fetches this repo at the stub's tag. Re-pointing `v1` at a stricter validator before repos
refresh their hooks produces silent green-local/red-CI skew — precisely the failure class
the kit exists to prevent. Breaking that compatibility is a **major** bump, not a `v1`
release.

---

## What lands here at T-3

**Do not build these yet.** Listed so the shape and the addresses are known in advance:

| Artifact | Role |
|---|---|
| `.github/workflows/semantics.yml` | reusable `workflow_call` workflow — the CI gate: branch name, PR commits, PR title |
| `check-git-semantics.sh` | the one validator, POSIX sh + ERE; modes `message` / `branch` / `pr-title` / `range` |
| `.githooks/commit-msg`, `.githooks/pre-push` | local hooks — zero dependencies, same validator |
| `.gitmessage` | commit template wired by `task setup` |
| `apply.sh` | per-repo installer: `install`, `verify`, `upgrade`, `rollback`, `ruleset enable\|disable` |
| `test.sh` | matrix runner — covers the validator matrix once per kit change |
| `CONVENTIONS.md`, `AGENTS.md` templates | shipped as kit templates, copied into target repos by `apply.sh` |

## What this README becomes at T-5

This file is destined to become **the fleet's canonical operator guide**, versioned with
the releases: prerequisites and permissions, the golden-path quickstart, the expected
`semantic-git: ready` summary output, verify / upgrade / rollback, a troubleshooting
runbook, and bot-config plus Conductor workspace-naming notes. The machine-local
`~/.claude/references/semantic-git-pattern.md` becomes a pointer stub to it. Until T-5,
this README is the release contract and nothing more.

---

## Conventions in this repo

fleet-kit follows the conventions it ships.

**Commits** — `type(scope)?: description`, subject ≤72 chars, lowercase type, imperative,
no trailing period. Types: `feat` `fix` `docs` `style` `refactor` `perf` `test` `build`
`ci` `chore` `revert`. **No AI attribution trailers** (`Co-Authored-By:` naming an AI,
`Generated with …`).

**Branches** — `type/kebab-description`. Exempt: `main` and Conductor workspace branches
(`DevPanda33/*`), which never open a PR — cut a semantic branch first.

**PR titles** — must parse as a conventional commit subject; ≤64 chars recommended, since
squash-merge appends ` (#N)`.

## License

MIT — see [LICENSE](LICENSE).
