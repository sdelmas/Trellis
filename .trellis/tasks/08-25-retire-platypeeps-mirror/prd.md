# Retire the platypeeps Trellis mirror onto the sdelmas fork

## Context

Local `ai/Trellis` pushed to `platypeeps/Trellis` — a private, standalone (non-fork) mirror
owned by the Platypeeps org. All real work already happened on `sdelmas/Trellis`, the actual
GitHub fork of `mindfold-ai/Trellis`: PRs #1–#6 live there, it carries 132 tags (the mirror
carries 0), and every branch SHA matched across the two repos (`main` = `0665d319` in both).
The mirror held exactly one thing the fork did not: branch
`dependabot/npm_and_yarn/undici-6.28.0` and its open PR #1 (undici 6.23.0 → 6.28.0).

`marketplace/` (submodule) already pointed at `sdelmas/marketplace` with upstream push disabled.

## Requirements

1. All Trellis work targets `sdelmas/Trellis` and `sdelmas/marketplace`.
2. Nothing pushes automatically to `mindfold-ai/*` — `origin` push stays DISABLED.
3. The dependabot undici bump moves from the mirror to `sdelmas/Trellis` as a PR.
4. `platypeeps/Trellis` is archived (read-only) so nothing lands there by accident.

## Non-goals

- Changing `.gitmodules` (tracked upstream file; its url stays `mindfold-ai/marketplace`).
- Deleting `platypeeps/Trellis` — archive only, reversible.
- Touching upstream `mindfold-ai` state, including open PR #534.

## Acceptance criteria

- `git remote -v` in `ai/Trellis` shows `fork` = `sdelmas/Trellis`, no `platypeeps` remote,
  `origin` push DISABLED.
- `git rev-parse HEAD fork/main` reports the same SHA.
- `sdelmas/Trellis` has branch `dependabot/npm_and_yarn/undici-6.28.0` at the mirror's SHA
  `d782b34e`, with an open PR into `sdelmas:main`.
- `gh api repos/platypeeps/Trellis --jq .archived` returns `true`.
- No live config under `~/repos` or `~/.claude` references `platypeeps/Trellis`
  (archived task docs and session transcripts excluded).

## Verification

```bash
git remote -v
git rev-parse HEAD fork/main
gh api repos/sdelmas/Trellis/branches/dependabot/npm_and_yarn/undici-6.28.0 --jq .commit.sha
gh api repos/platypeeps/Trellis --jq .archived
```
