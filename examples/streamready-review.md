# AFE Review: StreamReady

- **Repository:** [dev-oleksandrv/streamready-vega-app](https://github.com/dev-oleksandrv/streamready-vega-app) @ `514a4b1`
- **Date:** 2026-10-30
- **Checks run:** yes
- **Guide version:** AFE 1.0 (this was the first project reviewed; the review was redone in the 1.0 format)
- **Context:** an open-source Fire TV app for Vega OS (React Native + a C++ module), published on the Amazon Appstore, with one maintainer. Real users depend on it, so "growing" is a sensible aim for most pillars.

## At a glance

| Pillar | Repo | Harness |
|---|---|---|
| Verification | At-scale checks; all green in about a minute | Nothing runs the checks during a session |
| Legibility | Boundaries enforced by lint; some written rules aren't | Path-scoped rules per area |
| Traversable Knowledge | Strong ADRs and architecture doc; index kept in two places | Good rules; referenced skills aren't in the repo |
| Freshness | Updated by habit, not checked; drift already present | No drift checks |
| Operability & Context Economy | Setup and checks run anywhere; device needs a human | Short entry file, rules load per path |
| Bounded Autonomy | Clear zones and git rules | Only `/ship` is protected technically; no permission rules |

**In short:** the repository is well above what a project like this usually has. The gap is the harness: the project explains its rules very well, but almost nothing applies them while the agent works. A committed permissions file, two hooks, and a small drift fix (about 1.5 hours together) would close most of it.

## Symptoms already visible

- The `history` module (ADR 0012) is missing from the architecture summary in `CLAUDE.md` and from the commit-scope lists in `CLAUDE.md` and `CONTRIBUTING.md`, but commit `826903e` already uses `feat(history)`. → Freshness (practice drift)
- `CLAUDE.md` says "No `any`", but no active lint rule checks it. → Legibility
- The rules say `test/` mirrors `src/` one-to-one, but 42 of 138 source files (excluding `index.ts` and `content.ts`) have no mirrored test file. → Legibility, Freshness
- `CLAUDE.md` tells the agent to use `amazon-devices-vega-*` skills that aren't in the repository. → Traversable Knowledge
- "Never push" is written down, but nothing technically stops a push. → Bounded Autonomy

## Verification
> How does the agent know its change is correct?

**In place**
- `pnpm run check` runs lint, typecheck, Jest, and native C++ tests (`package.json`).
- All ran green on Linux without the Vega SDK:

  | Command | Result | Time |
  |---|---|---|
  | `pnpm run lint` | no errors or warnings | 9 s |
  | `pnpm run typecheck` | pass | 5 s |
  | `pnpm test` | 85 suites, 784 tests, all pass | 26 s |
  | `pnpm run test:native` | 56 tests under ASan + UBSan, all pass | 26 s |
  | `pnpm run test:coverage` | 98.5 % lines, 96.8 % branches | — |

- CI (`.github/workflows/ci.yml`) runs the same steps on every pull request and on `main`, plus a changeset check.
- `.claude/rules/testing.md`: tests first for logic and fixes; no real network, timers, or devices; fakes in `test/support/`; table tests for thresholds.
- `/ship` runs the full check before pushing and merges only the commit CI tested (`--match-head-commit`).

**Worth considering**
- No hooks. "Run `pnpm run check` before every commit" is an instruction only, and there's no `.claude/settings.json` in the repository. Nothing checks types after an edit or stops a turn that ends with failing tests.
- Targeted runs (`pnpm test <path>`) work but aren't mentioned in `CLAUDE.md`. The full check takes about 65 s, so a documented fast path would help inside the loop.
- On-device behavior (D-pad, focus) is checked by a human with a local, gitignored checklist. That's reasonable here; saying so in `CLAUDE.md` would tell the agent where its verification ends.

## Legibility
> Is there one obvious way to do each thing here?

**In place**
- `.eslintrc` enforces the layering `app → modules → shared`: no deep cross-module imports, pure TypeScript in `domain/` and `verdict/`, and `shared/` can't import `modules/` or `app/`. The lint messages explain the rule.
- One interface per extension point (`SpeedTestEngine`, `GeoIpProvider`), injected side effects, one store per module, user-facing copy in `content.ts`.
- Old approaches get removed: the TypeScript WebSocket engine was deleted when ADR 0006 replaced it.

**Worth considering**
- "No `any`, no non-null `!` without a comment" isn't checked: `@typescript-eslint/no-explicit-any` and `no-non-null-assertion` aren't active for `src/`.
- `.claude/rules/architecture.md` admits that sibling relative imports (`../../verdict/...`) can't be caught by the current lint setup. `eslint-plugin-boundaries` or `dependency-cruiser` can close that.
- The "mirrors 1:1" rule doesn't match the tree (see symptoms). An agent that follows it literally will create unneeded test files. Narrowing the rule to what's practiced (domain, stores, engines, providers) would fix it.

## Traversable Knowledge
> Can the agent find why things are the way they are, and what's in progress?

**In place**
- `CLAUDE.md` (79 lines): purpose, commands, working agreement, git rules, a one-screen architecture, style, Vega specifics, and links to `docs/ARCHITECTURE.md` and `docs/adr/`.
- Path-scoped rules in `.claude/rules/` (`architecture`, `speedtest`, `testing`, `ui-tv`).
- `docs/ARCHITECTURE.md` (409 lines) and 12 ADRs with context, decision, and consequences, plus written ADR rules. Supersede links are reciprocal (0002 ↔ 0006).
- Rules point at the relevant ADRs.

**Worth considering**
- No `AGENTS.md`, so other agents won't find the guidance. A link `AGENTS.md → CLAUDE.md` covers most of it.
- The ADR index exists twice (`docs/adr/README.md` and `ARCHITECTURE.md` § 12). They match today.
- ADRs don't state which paths they govern. With 12 that's fine; at 40 it won't be.
- The `amazon-devices-vega-*` skills are referenced but not in the repository, so a fresh clone doesn't get them.
- Unfinished work lives in `docs/superpowers/`, which is gitignored, so it doesn't survive a fresh clone or another machine.
- The architecture, testing, and UI conventions live only in `.claude/rules/`, so they reach Claude Code and nobody else: another agent or a contributor reading `docs/` never sees them. Moving the content to `docs/conventions/` and keeping the rule files as short pointers would make it available to everyone.

## Freshness
> Is what the agent finds still true?

**In place**
- No broken references: every path mentioned in `CLAUDE.md`, the rules, `ARCHITECTURE.md`, `CONTRIBUTING.md`, and `README.md` exists, or is a shorthand, an example, or a deliberately gitignored file.
- Structural changes usually update docs in the same branch (e.g. `b5bfa70`, `826903e`).

**Worth considering**
- No drift check in hooks or CI; freshness depends on memory.
- Practice drift is already present (the `history` scope and module, see symptoms). Commit `826903e` updated `ARCHITECTURE.md`, the ADR, and the ADR index, but not the two short summaries the agent reads first.
- The duplicated ADR index is where the next drift will likely appear.

## Operability & Context Economy
> Can the agent run and observe the project alone, without drowning in noise?

**In place**
- `pnpm install --frozen-lockfile` takes 12 s from a clean clone; all checks run without the Vega SDK (native tests need only a C++17 compiler).
- Pinned toolchain: `packageManager: pnpm@12.6.0`, `engines.node >=22`, a committed lockfile, Node 22 in CI. The tool server is pinned (`amazon-devices-buildertools-mcp@1.0.16`).
- `scripts/dev.sh` is a one-command, re-runnable device session.
- Code generation is scripted and the output is labeled "never edit by hand".
- Context economy is good: a short entry file, rules loaded per path, minimal native logs.

**Worth considering**
- The agent can't observe the running app. It needs a device or the Vega Virtual Device and a person watching the TV. The buildertools tool server might cover part of this; documenting what the agent *can* see on the virtual device would help.
- `pnpm test` prints 85 PASS lines in color on success, a little noise per run.

## Bounded Autonomy
> What may the agent do alone, what needs a human, and what can be undone?

**In place**
- A clear working agreement: propose before non-trivial work; no dependency or build-config changes without asking; stop when scope grows.
- Git limits: the agent may branch and commit, never push, rewrite history, or merge, except through `/ship`.
- `/ship` has `disable-model-invocation: true`, so only the maintainer can start it. It stops on any failure and never retries with different flags.
- Open-source hygiene: no secrets, no real IPs, documentation IPs in fixtures, no logging of signed URLs.
- CI has `contents: read`; the release workflow uses a narrowly scoped token.
- Releases are versioned with changesets and tagged, which gives a clear point to roll back to.

**Worth considering**
- Except for `/ship`, every limit is instruction-only. Nothing technically blocks `git push`, `pnpm add`, or edits to `manifest.toml` or `kepler/turbo-modules/generated/**`.
- No secret scanning in CI, although "no secrets" is the top open-source rule.
- Whether the agent's environment holds secrets (signing material, store credentials) depends on the maintainer's machine and isn't visible here.

## Recommended next steps

| # | Step | Addresses | Effort |
|---|---|---|---|
| 1 | Commit `.claude/settings.json` with deny rules for `git push`, `pnpm add` / `pnpm up`, and edits to `kepler/turbo-modules/generated/**` and `manifest.toml`. `/ship` keeps its path. | Bounded Autonomy | 15 min |
| 2 | Add a post-edit hook (typecheck and lint of changed files) and an end-of-turn hook (related Jest tests; send the agent back once on failure). | Verification | 1 h |
| 3 | Add `history` to the `CLAUDE.md` architecture block and to the scope lists in `CLAUDE.md` and `CONTRIBUTING.md`. Keep one ADR index and link to it from `ARCHITECTURE.md`. | Freshness, Knowledge | 10 min |
| 4 | A small `scripts/docs-drift.mjs` in `check`: doc paths exist, `CLAUDE.md` commands exist in `package.json`, every module and scope is listed, the ADR index matches the files, and hook commands and rule globs resolve. | Freshness | 1–2 h |
| 5 | Turn on `no-explicit-any` and `no-non-null-assertion` (warnings for existing code), and close the sibling-import gap with `eslint-plugin-boundaries`. | Legibility | 1 h |
| 6 | Add secret scanning (e.g. gitleaks) to CI. | Bounded Autonomy | 15 min |
| 7 | Narrow the "mirrors 1:1" rule to what's practiced. | Legibility | 10 min |
| 8 | Link `AGENTS.md → CLAUDE.md`; document where the Vega skills come from; consider a versioned place for plans of unfinished work. | Knowledge | 15 min |
| 9 | Move the content of `.claude/rules/` into `docs/conventions/` and keep the rules as pointers. | Knowledge, Freshness | 30 min |

## Not visible from the repository

- Branch protection on `main`: check the repository settings.
- What the agent's environment can reach (signing keys, store credentials): check the machine or sandbox the agent runs in.

## Appendix: commands run

```bash
git clone --depth 200 https://github.com/dev-oleksandrv/streamready-vega-app
pnpm install --frozen-lockfile
pnpm run lint && pnpm run typecheck && pnpm test && pnpm run test:native
pnpm run test:coverage
npx eslint --print-config src/App.tsx   # effective lint rules
# plus: src→test mirror scan, doc path-reference scan, doc co-change scan over git log
```
