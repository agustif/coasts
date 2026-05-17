# Agustif Coasts Fork Policy

This repository is the agustif-owned Coasts product line. It may intentionally
diverge from `coast-guard/coasts`.

## Remotes

- `origin`: `https://github.com/agustif/coasts.git`
- `upstream`: `https://github.com/coast-guard/coasts.git`
- `upstream` push URL is disabled locally.

Do not push directly to `coast-guard/coasts` from this checkout. Upstream PRs
must be explicit and should name `--repo coast-guard/coasts`.

## Branches

- `agustif/main`: default branch and maintained fork product line.
- `main`: optional upstream mirror or legacy branch. Do not land fork product
  work there.
- `agustif/legacy-v0.1.17`: preserved pre-divergence fork default branch.
- `import/upstream-main-YYYYMMDD`: reviewed upstream import branches.

Feature branches should start from `agustif/main` and open PRs back to
`agustif/main` unless the work is explicitly intended for upstream.

## Upstream Imports

Treat `upstream/main` as vendor input. Do not fast-forward `agustif/main`
automatically from upstream.

Use a reviewable import branch:

```bash
git fetch upstream main
git fetch origin agustif/main
git switch agustif/main
git pull --ff-only origin agustif/main
git switch -c import/upstream-main-$(date +%Y%m%d)
git merge --no-ff upstream/main
git push -u origin import/upstream-main-$(date +%Y%m%d)
gh pr create --repo agustif/coasts --base agustif/main --head import/upstream-main-$(date +%Y%m%d)
```

Prefer merge commits for upstream imports so audits can see exactly when vendor
state entered the fork.

## Local Account Binding

This checkout should resolve GitHub CLI operations as `agustif`.

```bash
gh auth switch --user agustif --scope cwd --selector /Users/af/coasts
```

The local `.ghaccount` file is globally ignored and should contain:

```text
agustif
```

## Proof Gates

For backend-only fork changes, avoid the Vite/UI build unless the change touches
the UI:

```bash
mkdir -p coast-guard/dist
touch coast-guard/dist/index.html
COAST_SKIP_UI_BUILD=1 cargo fmt --all -- --check
COAST_SKIP_UI_BUILD=1 cargo clippy --workspace -- -D warnings
COAST_SKIP_UI_BUILD=1 cargo test --workspace
```

For targeted daemon changes:

```bash
mkdir -p coast-guard/dist
touch coast-guard/dist/index.html
COAST_SKIP_UI_BUILD=1 cargo test -p coast-daemon <test_filter> --lib
COAST_SKIP_UI_BUILD=1 cargo clippy -p coast-daemon --lib --tests -- -D warnings
```

For release-like binary proof without rebuilding the UI:

```bash
mkdir -p coast-guard/dist
touch coast-guard/dist/index.html
COAST_SKIP_UI_BUILD=1 cargo build -p coast-daemon --bin coastd-dev -p coast-cli --bin coast-dev
```

Use full release/integration checks only when changing Docker orchestration,
ports, worktrees, secrets, packaging, or installer behavior.
