# PR C: script runtime hardening

Slice C of the `mindfold-ai/Trellis#534` resplit. Parent: `08-25-resplit-pr534`.

## Scope

Branch `upstream-pr/runtime-hardening`, off `upstream/main` at `64e66369`. The eight fix commits
the maintainer named, replayed in topological order:

`5a1d59e0` path containment · `c0d7cb7f` create/archive/link collisions · `1cf22b51` JSON
read/write failures · `cf8cb25c` lifecycle hooks + config parsing · `e1a17984` empty
title/description rejection · `a95e7483` archive `index.lock` retry · `9c88fec5` hook process-tree
kill · `c9489ce8` non-list `children` + advisory `stat()` guards.

No `rename`, no `add_session` rewrite, no workflow.md — per the maintainer's constraint.

## Conflict resolutions

- `5a1d59e0` conflicted on 7 unique hunks (mirrored across the vendored and packaged copies).
  Upstream had independently landed a weaker containment check (resolve + `relative_to(repo_root)`).
  Took the fork side: it resolves both operands (covering upstream's symlink case), rejects the
  tasks directory itself, and requires containment inside `tasks_dir` rather than `repo_root` —
  a strict superset.
- `cf8cb25c`, `e1a17984`, `9c88fec5` each conflicted on `regression.test.ts` where the incoming
  side carried the "shipped markdown carries no trailing whitespace" block from `e77af366` — a
  commit in #534's "Also" section, not in slice C and not on upstream. Took the HEAD side each
  time so the block stays out.
- Task artifacts carried by the picks (`08-08-*`, `08-09-*`, `08-19-*`) were dropped, not resolved.
- `workflow.md` hunks from `e1a17984` reverted: the maintainer excluded workflow.md, since it
  drags in the marketplace mirror pair.

## Deviation: test made independent of slice E

`task-children-normalization.integration.test.ts` (from `c9489ce8`, so squarely in C) drives
`task.py` with `TRELLIS_DEVELOPER: "tester"`. That env var is implemented by `0740d1d6`, which the
maintainer assigned to **slice E** ("only after C"). As written, C's own test cannot pass on C.

Rather than pull E's feature forward, `setupRepo` now writes `.trellis/.developer` with
`name=tester`, which `get_developer()` on current `main` already reads. The `TRELLIS_DEVELOPER`
env in `runTask` is left in place, so the test keeps passing unchanged once E lands.

## Acceptance criteria

- No `.trellis/tasks/`, no workflow.md, no marketplace gitlink, no `.trellis/workspace/` in the diff.
- Vendored `.trellis/scripts` and packaged `templates/trellis/scripts` byte-identical.
- Version stays `0.6.15`.
- `pnpm test` green: cli 1750 passed (77 files), core 346 passed + 1 skipped.

## Verification

```bash
diff -rq -x __pycache__ .trellis/scripts packages/cli/src/templates/trellis/scripts   # identical
git diff --cached --name-only | grep -E "^\.trellis/tasks/|workflow\.md|^marketplace$"  # no hits
pnpm build && pnpm test
```
