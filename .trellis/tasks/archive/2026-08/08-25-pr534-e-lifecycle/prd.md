# PR E: task lifecycle commands and metadata

Fifth and last slice of the `mindfold-ai/Trellis#534` resplit. Parent task:
`.trellis/tasks/08-25-resplit-pr534`.

## Source commits

Cherry-picked in the maintainer's order, each kept as its own commit:

| Fork commit | Subject |
|---|---|
| `f8d5de5f` | `task.py rename` with atomic back-reference rewrite |
| `00ae5af4` | record branch at task start, validate metadata before archive |
| `0740d1d6` | resolve developer identity in linked worktrees |
| `0b6577d3` | align task-context validation with PR preflight |

## Base

Branched off **`upstream-pr/runtime-hardening`** (slice C, upstream PR #576), not
off `upstream/main`. This is not a stylistic choice: `00ae5af4` imports
`read_json_checked` / `describe_json_read_failure` from `common/io.py` and
`INDEX_LOCK_RETRY_ATTEMPTS` / `index_lock_path` / `run_git_retry_index_lock` /
`stderr_indicates_index_lock` from `common/git.py`. All six land in slice C and
none exist on `upstream/main`. Attempting E on `upstream/main` first produced
content conflicts in `task_store.py` and `task.py` that could only be resolved by
pulling C's helpers forward — i.e. by silently re-merging C.

This matches the maintainer's own sequencing: *"Only after C."* The upstream PR
must therefore be merged after #576.

## Scope

Four product-surface changes to the task lifecycle:

1. **`task.py rename <name> <new-slug> [--dry-run]`** — renames the task
   directory and rewrites every back-reference (task.json identity fields,
   parent/children links, jsonl manifests) in one pass, with each step
   idempotent so a failed rename can be re-run verbatim.
2. **Branch metadata** — `task.py start` records the current branch on the task;
   `archive` validates that metadata before the task leaves the active tree,
   where it becomes unrecoverable. A stale branch (merged and deleted) warns
   rather than blocks.
3. **Worktree developer identity** — developer resolution follows a linked
   worktree back to its main working tree, and `TRELLIS_DEVELOPER` overrides it.
   Without this, every command in a `git worktree` reports `No developer set`.
4. **JSONL manifest validation** — `implement.jsonl` / `check.jsonl` are created
   empty instead of seeded with an `_example` placeholder row, and
   `task.py validate` rejects that row exactly as PR preflight already does.
   Includes the migration that strips the placeholder from this repo's own 16
   task manifests.

## Exclusions

Shared scrub rules from the parent PRD apply — no `.trellis/workspace/sven/`, no
in-progress task archives, no `0.6.16-sd.N` fork identity, no marketplace
gitlink bump, no Opus pinning, no pre-start gate.

Four task-artifact directories carried by the source commits were dropped:
`08-08-task-rename`, `07-27-validate-task-branch-metadata-before-archive`,
`08-08-developer-worktree-provisioning`, `07-23-align-task-validation-preflight`.

### Deviation: `workflow.md` is not in this PR

`f8d5de5f` and `0b6577d3` also edit `.trellis/workflow.md` and its packaged twin
— one line documenting `rename`, three describing the empty-manifest gate. Both
were reverted out of this branch.

`packages/cli/test/templates/trellis.test.ts` asserts
`marketplace/workflows/native/workflow.md` is byte-identical to
`packages/cli/src/templates/trellis/workflow.md`. Any `workflow.md` edit
therefore fails CI until the marketplace submodule is bumped — and the pointer
cannot be bumped, because the pinned commit `7310a50c` is **not an ancestor of
`mindfold-ai/marketplace@main`**. It is an orphan off `d286b2c`; `main` went
`cfb2f38` → `a478b28` (DSH), and the `task_error` block that `7310a50c` added
never landed on `main`. Reconciling that is upstream's pre-existing sync debt and
is out of scope for this slice.

The doc text is carried instead by **mindfold-ai/marketplace#17**, which mirrors
the same four hunks onto marketplace `main`. Once the pin is reconciled, the
`workflow.md` edits can land as a one-file follow-up. The skill-level docs
(`brainstorm.md`, `continue.md`, the `trellis-meta` references) that describe the
same behavior *are* included here, since they carry no mirror assertion.

## Acceptance criteria

- Branch `upstream-pr/task-lifecycle` sits on top of `upstream-pr/runtime-hardening`
  and carries exactly four commits.
- `pnpm -r build` succeeds; `pnpm -r test` is fully green with no skipped-over
  failures.
- `.trellis/scripts` is byte-identical to
  `packages/cli/src/templates/trellis/scripts`.
- No change to `package.json` versions (stays `0.6.15`), the `marketplace` or
  `docs-site` gitlinks, or either `workflow.md`.
- No path under `.trellis/workspace/` or `.trellis/tasks/archive/` in the diff.
- A PR is open against `mindfold-ai:main` from `sdelmas:upstream-pr/task-lifecycle`,
  stating the #576 merge-order dependency and the deferred `workflow.md` hunks.

## Verification

```bash
git log --oneline upstream-pr/runtime-hardening..upstream-pr/task-lifecycle   # 4 commits
git diff --name-only upstream/main...HEAD | grep -E 'workflow\.md$|^marketplace$' # empty
pnpm -r build && pnpm -r test
```

Result: `core → 19 files / 346 passed, 1 skipped`; `cli → 77 files / 1783 passed`.
