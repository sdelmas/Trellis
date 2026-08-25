# PR B: trellis update receipt repair

Slice B of the `mindfold-ai/Trellis#534` resplit. Parent: `08-25-resplit-pr534`.

## Requirement

`.template-hashes.json` recorded hashes that disagreed with the files written, and omitted
entries for files that exist. Both make the receipt useless as a drift signal: a clean vendored
tree reports as locally modified, so real customizations cannot be told apart from noise. Found
during the 8-repo fleet rollout, where the receipt was the tool for deciding which local edits
were genuine, and it produced false positives on every repo.

Root cause: `analyzeChanges` classifies a file whose content already equals its template as
`unchanged`, and the write-back drew only from `newFiles`, `autoUpdateFiles`, and overwritten
`changedFiles`. `unchangedFiles` was never written back, so a wrong or absent entry beside an
already-correct file could not be repaired by any number of `trellis update` runs.

## Scope

Branch `upstream-pr/update-receipt-repair`, off `upstream/main` at `64e66369`.
Cherry-pick of `5a92d584`, code only:

- `packages/cli/src/commands/update.ts`
- `packages/cli/src/utils/template-hash.ts`
- `packages/cli/test/commands/update.integration.test.ts`

The cherry-pick also carried `.trellis/tasks/08-19-template-hashes-stale/` (prd.md, task.json).
Those conflicted against upstream and are excluded by the resplit scrub rules regardless, so they
were dropped from the index rather than resolved.

## Acceptance criteria

- Staged set is exactly the 3 files above; no `.trellis/` path, no marketplace gitlink.
- Version stays `0.6.15`; no `-sd.` string.
- `pnpm build` succeeds; `pnpm test` green in both packages.
- PR opened from `sdelmas:upstream-pr/update-receipt-repair` into `mindfold-ai:main`.

## Verification

```bash
pnpm build && pnpm test          # expect cli 1713 passed (+5 receipt tests), core 346 passed
git diff --cached --name-only | grep -E "\.trellis/|^marketplace$"   # expect no hits
```
