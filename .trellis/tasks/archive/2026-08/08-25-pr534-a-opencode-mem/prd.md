# PR A: restore the OpenCode mem reader

Slice A of the `mindfold-ai/Trellis#534` resplit. Parent: `08-25-resplit-pr534`.

## Requirement

Upstream `main` still ships `packages/core/src/mem/adapters/opencode.ts` as a 34-line silent
no-op (the `better-sqlite3` revert). Restore the zero-dependency read-only SQLite reader that
exists on the fork (592 lines), so OpenCode 1.2+ sessions can be listed, searched, and recalled.

Maintainer: "Restoring a zero-dependency SQLite reader is independently useful and does not touch
the Python runtime... I will review A as soon as it is open."

## Scope

Branch `upstream-pr/opencode-mem-reader`, off `upstream/main` at `64e66369`.

Included (6 files):

- `packages/core/src/mem/adapters/opencode.ts` — the reader
- `packages/core/src/mem/internal/paths.ts` — `opencodeDbPath`
- `packages/core/src/mem/sessions.ts`
- `packages/core/test/mem/adapters.test.ts`, `packages/core/test/mem/api.test.ts`
- `packages/cli/src/commands/mem.ts` — required: it removes the "OpenCode reader unavailable"
  notice, which becomes a false statement once the reader works

`packages/core/src/mem/internal/sqlite-readonly.ts` needs no change — it already exists upstream
and is byte-identical to the fork's copy, which is what makes this slice self-contained.

## Excluded, and why

- `packages/cli/src/configurators/opencode.ts` docstring fix (`2d06433b`). The maintainer listed
  it as keep-able, but its premise does not hold on upstream `main`: upstream's opencode template
  `package.json` still declares `@opencode-ai/plugin`, so the existing docstring is accurate
  there. The fork's "correction" describes the bare `{"type": "module"}` template produced by the
  pre-start-gate work, which is explicitly excluded from every slice. Applying it would make
  upstream's comment wrong. Flagged in the PR body for the maintainer to overrule.
- No `package.json` changes — no `-sd.N` version, no OpenCode dependency removal (the maintainer
  wants that as its own one-file PR if wanted at all).
- No marketplace gitlink move, no `.trellis/workspace/sven/`, no task archives, no Opus pinning.

## Acceptance criteria

- `git diff --cached --name-only` matches the 6 files above exactly.
- `pnpm build` succeeds; `pnpm test` green in both packages.
- Versions on the branch stay `0.6.15` (upstream's), no `-sd.` string anywhere.
- PR opened from `sdelmas:upstream-pr/opencode-mem-reader` into `mindfold-ai:main`.

## Verification

```bash
pnpm build && pnpm test
git diff --cached --name-only | grep -E "\.trellis/workspace/|\.trellis/tasks/archive/|^marketplace$"   # expect no hits
grep '"version"' packages/*/package.json                                                                # expect 0.6.15
```

Note: the marketplace submodule must be checked out at the branch's recorded gitlink
(`7310a50c`) or `trellis.test.ts > marketplace native workflow mirror` fails spuriously.
