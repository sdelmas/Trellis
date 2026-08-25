# Resplit upstream PR #534 into five reviewable slices

## Source requirement

`mindfold-ai/Trellis#534` ("Runtime hardening, task lifecycle convergence, and OpenCode mem
reader restore") is 91 commits / 299 files / +22,638 −7,799 from `sdelmas:chore/task-backlog-2026-08`
into `mindfold-ai:main`. It is `mergeable: true` but `mergeable_state: blocked`, with no approving
review (14 review events, all COMMENTED).

The maintainer (`taosu0216`, 2026-08-20) asked for it to be closed and resplit. Their stated
blockers:

1. Scope — three product lines plus a month of task archives in one review.
2. It archives tasks that are still `in_progress` on `main` and assigned to other people:
   `08-05-session-identity-*`, `08-06-converge-platform-templates`, `08-06-mem-full-recall`.
   Quote: "Those must not land."
3. CI red on `packages/cli/test/scripts/add-session.integration.test.ts:635`
   (`@typescript-eslint/no-non-null-assertion`), which belongs with the `add_session` rewrite.

## Scrub rules — every slice

Each slice branches off **current** `upstream/main` and must contain none of:

- `.trellis/workspace/sven/`
- task archives for work still `in_progress` on `main`
- `0.6.16-sd.N` or any other fork identity (version bumps, dogfood receipts)
- the marketplace gitlink move (`518f1b6d` and friends)
- pinning `trellis-implement` / `trellis-research` to Opus (separate product decision)
- the optional pre-start gate (`e5a563de` / `d39dd446`) — added then reverted on the branch

Dogfood copies under `.claude/`, `.cursor/`, `.omp/` are optional. If a slice changes a template,
the packaged template and the vendored `.trellis/scripts` copy must stay byte-identical.

## Task map

| Slice | Child task | Content |
|-------|-----------|---------|
| A | `08-25-pr534-a-opencode-mem` | OpenCode mem reader restore; upstream still ships a 34-line no-op, fork has the 592-line reader |
| B | `08-25-pr534-b-receipt-repair` | `trellis update` receipt repair (`5a92d584`) |
| C | `08-25-pr534-c-runtime-hardening` | Fix commits only: `5a1d59e0`, `c0d7cb7f`, `1cf22b51`, `cf8cb25c`, `9c88fec5`, `e1a17984`, `a95e7483`, `c9489ce8` |
| D | `08-25-pr534-d-add-session` | `76c53c5a` + `3a0a5f6b` + integration test, lint fixed |
| E | `08-25-pr534-e-lifecycle` | `f8d5de5f`, `00ae5af4`, `0740d1d6`, `0b6577d3` — after C; product pass wanted |

Ordering is a review constraint, not a dependency system: A first (maintainer reviews it
immediately), E only after C.

## Cross-slice acceptance criteria

- Each slice is a separate branch off `upstream/main` at `64e66369` or later, pushed to
  `sdelmas/Trellis`, with a PR into `mindfold-ai:main`.
- Every slice: `pnpm build` succeeds, `pnpm test` green, lint clean.
- Every slice passes the scrub check: `git diff --name-only upstream/main...<branch>` contains no
  path matching the exclusion list above, and no `-sd.` string appears in any changed
  `package.json`.
- `mindfold-ai/Trellis#534` closed with a comment linking the five replacement PRs.

## Non-goals

- Changing the fork's own `main` or its `-sd.N` release line.
- Landing the pre-start gate.
- Rewriting `chore/task-backlog-2026-08` (it is #534's head; leave it at `2749d3b4`).

## Result

`mindfold-ai/Trellis#534` closed 2026-08-25. Five replacement PRs open:

| Slice | PR | Branch | Base |
|---|---|---|---|
| A — OpenCode mem reader | [#574](https://github.com/mindfold-ai/Trellis/pull/574) | `upstream-pr/opencode-mem-reader` | `upstream/main` |
| B — update receipt repair | [#575](https://github.com/mindfold-ai/Trellis/pull/575) | `upstream-pr/receipt-repair` | `upstream/main` |
| C — runtime hardening | [#576](https://github.com/mindfold-ai/Trellis/pull/576) | `upstream-pr/runtime-hardening` | `upstream/main` |
| D — `add_session` state machine | [#577](https://github.com/mindfold-ai/Trellis/pull/577) | `upstream-pr/add-session-state-machine` | `upstream/main` |
| E — task lifecycle commands | [#578](https://github.com/mindfold-ai/Trellis/pull/578) | `upstream-pr/task-lifecycle` | slice C branch |

Plus one cross-repo companion: [`mindfold-ai/marketplace#17`](https://github.com/mindfold-ai/marketplace/pull/17),
mirroring the `workflow.md` hunks that slice E had to defer.

### Deviations from the maintainer's spec

1. **`2d06433b` dropped from slice A.** The docstring fix targets a template that
   still carries the `@opencode-ai/plugin` dependency upstream; the fix is
   inapplicable until that dependency is gone. Surfaced in #574 rather than
   applied blind.
2. **Slice E is stacked on slice C, not on `upstream/main`.** `00ae5af4` imports
   six helpers that land in C and do not exist on `main`. Consistent with the
   maintainer's "only after C".
3. **Slice E defers its `workflow.md` hunks.** The marketplace mirror assertion
   cannot be satisfied: the pinned commit `7310a50c` is an orphan off `d286b2c`
   and is not an ancestor of `mindfold-ai/marketplace@main`. Carried by
   marketplace#17 instead; disclosed in #578.
4. **Slice C writes `.trellis/.developer` in one test's `setupRepo`.** The
   maintainer assigned `0740d1d6` (`TRELLIS_DEVELOPER`) to slice E, but
   `task-children-normalization.integration.test.ts` in slice C needs a developer
   identity. Writing the file avoids pulling E forward into C.
