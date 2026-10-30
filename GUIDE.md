# The AFE Guidebook

> **Version:** 1.0 (October 2026)
> **For:** engineers who work with coding agents, and the agents themselves.
> **Covers:** any repository where coding agents read, change, and verify code. Examples use Node.js / TypeScript; the ideas don't depend on the stack.

This is a guide, not a standard. It collects practices that have worked, organizes them around the problems they solve, and explains why each one matters. Take what fits your project and leave the rest.

---

## Contents

1. [How the guide is organized](#1-how-the-guide-is-organized)
2. [Principles](#2-principles)
3. [Verification](#3-verification)
4. [Legibility](#4-legibility)
5. [Traversable Knowledge](#5-traversable-knowledge)
6. [Freshness](#6-freshness)
7. [Operability & Context Economy](#7-operability--context-economy)
8. [Bounded Autonomy](#8-bounded-autonomy)
9. [The harness](#9-the-harness)
10. [Putting it into practice](#10-putting-it-into-practice)
11. [Glossary](#11-glossary)
- [Appendix A: Harness mechanisms by pillar](#appendix-a-harness-mechanisms-by-pillar)
- [Appendix B: Symptom index](#appendix-b-symptom-index)
- [Appendix C: Sources](#appendix-c-sources)

---

## 1. How the guide is organized

### Six pillars

Every project where agents write code runs into the same six problems, whatever its size. Each pillar is one of those problems, phrased as a question.

| Pillar | The question |
|---|---|
| **Verification** | How does the agent know its change is correct? |
| **Legibility** | Is there one obvious way to do each thing here? |
| **Traversable Knowledge** | Can the agent find *why* things are the way they are, and what's in progress? |
| **Freshness** | Is what the agent finds still true? |
| **Operability & Context Economy** | Can the agent run and observe the project alone, without drowning in noise? |
| **Bounded Autonomy** | What may the agent do alone, what needs a human, and what can be undone? |

The questions stay the same for every project. The answers don't.

### Two layers

Every recommendation lives in one of two places:

- **Repo** — the code and everything around it that isn't agent-specific: structure, config, scripts, tests, docs, CI, and the hosting platform's settings for the repository (branch protection, secret scanning, code owners).
- **Harness** — everything that shapes the agent itself: instruction files, rules, skills, hooks, subagents, tool servers, permissions, and the environment the agent runs in (sandbox, container, network access).

The entry file (`AGENTS.md`) sits on the border. Its content is repo knowledge (principle 3), and it's also the first way the harness reaches the agent. This guide tags items about *telling the agent* something as [harness], and items about the knowledge itself as [repo].

A project can be excellent in one layer and empty in the other. The most common pattern is a well-built repo with a harness that *describes* the rules but does nothing to apply them while the agent works. Thinking in both layers makes that visible.

### Three stages

Recommendations are grouped by how far a project has come. Stages describe a project; they don't grade it.

| Stage | Typical project | What it needs |
|---|---|---|
| **Starting** | An experiment, a prototype, a weekend project. | The few things that pay off in the first hour. |
| **Growing** | Real users depend on it; one maintainer or a small team. | Rules that are checked, not only written; a harness that applies them. |
| **At scale** | Several teams, sensitive data, or a high cost of failure. | Complete coverage of the real risks, and a loop that improves the setup over time. |

A project can be at different stages in different pillars, and in different layers of the same pillar. That's normal. "Verification is at scale in the repo, starting in the harness" is a useful description.

### Recommendation tags

- **[repo]** or **[harness]** — which layer it belongs to.
- **Strongly recommended** — marks the few practices that prevent damage that is hard or impossible to undo. Everything else is advice to weigh against your project.

### How each pillar section is built

- **The question** and **why it matters**.
- **Symptoms** — how you notice the problem in practice. If you came here with a problem, start with [Appendix B](#appendix-b-symptom-index).
- **Recommendations** by stage.
- **Check yourself** — a few questions to see where you are.

---

## 2. Principles

1. **Outcome over artifact.** "The agent can confirm its change works" is the goal. "Have unit tests" is one way to reach it. Artifacts change with tools; outcomes don't.
2. **Problems, not mechanisms.** Hooks, skills, rules, and tool servers are mechanisms. They serve the pillars; they aren't pillars.
3. **Knowledge in neutral places, tools as thin adapters.** What the project knows (architecture, conventions, decisions, recipes, plans) lives where every agent and every person can read it: `AGENTS.md` and `docs/`. Tool folders (`.claude/`, `.cursor/`, …) hold only what is specific to that tool, such as hooks, permissions, and the wiring that points the tool at the right document. Switching or adding an agent then means rewriting a thin layer, not moving your knowledge.
4. **Written, then checked, then applied.** A rule that is only written is a suggestion. A rule that is only checked is unexplained friction. A rule that the harness also applies reaches the agent *while it works*, before it hits a failing CI run. Most recommendations follow this progression across the stages.
5. **Pace layers.** The pillars should hold for years. The recommendations get reviewed with each version of this guide. The tool details in [Appendix A](#appendix-a-harness-mechanisms-by-pillar) will change every few months; keep that part of your setup cheap to rewrite.
6. **Mistakes drive changes.** Change your setup when an agent actually makes a mistake, not when a new tool comes out. Every repeated mistake should become a permanent improvement to the repo or the harness. When you fix a defect, also name what keeps the next one out: a test, a check, a rule, or the reason none fits.
7. **Good for humans first.** Nearly everything here also helps a new engineer. If a practice only helps today's model work around today's limits, keep it small and expect to delete it.
8. **Right-sized.** Same pillars, different depth. Over-building a small project wastes as much as under-building a large one.
9. **A wrong document is worse than none.** A written rule that the codebase contradicts teaches the agent the wrong thing with full confidence. Fix it or delete it.

---

## 3. Verification

> **How does the agent know its change is correct?**

### Why it matters

This is the pillar everything else rests on. Without a way to check its work, an agent produces confident mistakes. Every other pillar only changes *which* mistakes.

### Symptoms

- The agent reports "done" and CI is red.
- You end up running the tests yourself after every agent session.
- Bugs come back that were fixed before.
- The agent "fixes" a failing test by changing the test.
- The agent waits minutes for checks, or skips them because they're slow.
- Tests pass, but bugs still get through.

### Recommendations

**Starting**

- **One command for all checks.** [repo] For example `<pm> run check` running lint, typecheck, and tests. Put it at the top of the instruction file.
- **Tests that run anywhere.** [repo] Core logic is covered by tests that need no network, no real clock, and no device. Use fakes, fixtures, and injected dependencies.
- **Tell the agent to run the checks.** [harness] The instruction file says which command to run and when.

**Growing**

- **CI runs the same checks, and blocks merging on failure.** [repo]
- **A fast, targeted check.** [repo] One file, one module, or the tests related to a change. Document it next to the full check. As a guideline: a targeted check under ~30 seconds, the full local check under ~5 minutes, CI under ~15 minutes.
- **Strict static checks.** [repo] A type checker in strict mode and a linter that fails with errors, not only warnings.
- **Deterministic tests.** [repo] Fix or quarantine flaky tests. Never retry them silently; a flaky test teaches the agent that red can be ignored.
- **Tests that prove behavior.** [repo] A passing test is only worth something if it would fail when the behavior breaks. Agents are good at writing tests that pass and check nothing, so write down what a test must prove:
  - It asserts an observable outcome: a returned value, the resulting state, or emitted output. "The mock was called" or "it didn't throw" isn't enough on its own.
  - The expected value is a literal or a fixture, never computed with the logic under test.
  - Fakes replace the unit's boundaries (network, storage, clock, randomness), never code inside the unit.
  - Failure paths get tests as deliberately as success paths, including the edge cases that apply.
  - A test is written first and seen to fail for the expected reason; a bug fix starts with a test that reproduces the bug.
  - Coverage shows which lines ran, not what was checked. Treat it as a floor, not as proof of quality.
- **Gates don't move to get a pass.** [repo + harness] Skipping or deleting a test, weakening an assertion, excluding code from coverage, or lowering a threshold to make a check pass is a decision for a human, not a fix. Put test-runner, coverage, and lint configuration in the "ask first" zone (see [Bounded Autonomy](#8-bounded-autonomy)).
- **Check after every edit.** [harness] A post-edit hook runs a fast typecheck or lint on the changed files and reports back in the same session.
- **Check before finishing.** [harness] An end-of-turn hook runs the related tests and sends the agent back once with the failure output if they fail.

**At scale**

- **Tests the agent can run end to end.** [repo] Browser automation, emulators, or local containers, so the agent can verify behavior, not only units.
- **Coverage where the risk is.** [repo] Measure coverage and watch the critical paths. A global percentage matters less than knowing the risky code is covered.
- **A second opinion.** [harness] A separate reviewer (a subagent, a second model, or a human) checks the change, so the agent doing the work isn't the only one judging it. Agents tend to rate their own work too highly.
- **Evidence with every change.** [repo + harness] Each change arrives with the commands that were run and their results, for example in a pull-request template. "I tested it" without output doesn't count.

### Check yourself

- Can a fresh clone run every check with one command, offline?
- How long does the agent wait to verify a one-line change?
- If the agent stops with failing tests, does anything notice before you do?
- Pick a recent test. Would it fail if the behavior it covers broke?

---

## 4. Legibility

> **Is there one obvious way to do each thing here?**

### Why it matters

Agents copy what they see. If the codebase has one clear pattern, the agent follows it. If it has three, the agent picks one at random or invents a fourth.

### Symptoms

- The agent puts new code in a different place each time.
- The agent recreates a pattern or library you removed months ago.
- Agent code works but looks foreign next to the existing code.
- The same concern is solved two different ways in two modules.

### Recommendations

**Starting**

- **A predictable structure.** [repo] A newcomer can say where a new feature, test, or config value goes.
- **Conventions written down.** [harness] The instruction file states the naming, structure, and style rules that aren't obvious from the code.
- **One way per concern.** [repo] One HTTP client, one validation library, one state approach. When you replace one, remove the old one.

**Growing**

- **Boundaries checked by tooling.** [repo] Module boundaries and dependency direction fail the build when broken: lint import rules, dependency-cruiser, boundary plugins, or build-system visibility. A lint message that explains the rule is better than one that only blocks.
- **Style rules that are written are also checked.** [repo] If the instruction file says "no `any`", a lint rule should say it too. A rule that only exists in prose will be broken.
- **Rules that match reality.** [repo] If the docs say "tests mirror `src/` one-to-one", either they do or the rule changes. Agents follow written rules literally.
- **Local conventions where they apply.** [harness] Path-scoped rules describe the conventions of the area being edited, instead of one long file covering everything.
- **Explicit extension points.** [repo] When something is meant to be pluggable (a storage backend, a provider), there's an interface for it, so new code has a template.

**At scale**

- **No dead code, no half-finished migrations.** [repo] If two patterns must coexist for a while, a decision record says which one is deprecated and how the migration ends.
- **A nudge on structural changes.** [harness] A hook asks "is there a decision record for this?" when a new dependency or a new top-level folder appears.

### Check yourself

- Pick three modules at random. Do they have the same internal shape?
- Is every style rule in your instruction file also enforced by a tool?
- Does your dependency list contain two libraries for the same job?

---

## 5. Traversable Knowledge

> **Can the agent find *why* things are the way they are, and what's in progress?**

### Why it matters

An agent can read the code, but the code doesn't say why. Without the reasons, the agent "fixes" deliberate decisions, re-opens settled questions, and repeats experiments that already failed. Without the current state, every session starts with amnesia.

### Symptoms

- The agent undoes a deliberate trade-off because it looked like a bug.
- You explain the same background in every session.
- A new session doesn't know what the previous one left half-done.
- The agent searches the whole repo to find out which rules apply to one file.
- The agent describes a plan or a proposal as if it were already built.
- Another agent, or a new contributor, misses rules your usual agent follows.

### Recommendations

**Starting**

- **A short entry file.** [repo] Aim for under ~100 lines: what the project is, the essential commands, an architecture summary, links to deeper docs, and what to ask about first. It's the one file every session reads.
- **Record big decisions.** [repo] When you choose a pattern, a boundary, or a tool, or replace one approach with another, write a short decision record (ADR): context, decision, consequences.

**Growing**

- **One home per fact.** [repo] Each kind of fact (purpose, architecture, decisions, conventions, operations) has one owner document, and a short map in `docs/` says which document owns what. Other documents link to the owner instead of repeating it. Copies are where drift starts.
- **A tool-neutral entry file.** [repo] Use `AGENTS.md`, the open standard most coding agents read. Tool-specific files (`CLAUDE.md` and others) become links or one-line pointers to it, never separate copies that drift apart.
- **Decision records that don't change.** [repo] Once accepted, a decision isn't edited. A new decision supersedes it, and both link to each other.
- **One index.** [repo] Keep a single list of decision records. If it appears in several places, all but one are links to it or are generated from it.
- **Conventions and recipes live in `docs/`.** [repo] Coding conventions, area rules, recipes, and plans are ordinary documents in the repository (for example `docs/conventions/`, `docs/recipes/`), not content inside one tool's folder. Tool-specific rule files and skills point to these documents instead of holding their own copy. Otherwise the knowledge reaches one agent and nobody else, and two copies drift apart.
- **Everything the entry file mentions exists.** [harness] If the instruction file says "use the X skill", that skill is in the repository, or the file says where it comes from. A fresh clone should get the same guidance as your laptop.
- **Guidance that loads when needed.** [harness] Area-specific guidance loads for matching paths only (path-scoped rules, skills), not all at once.
- **Mark what is decided, what is proposed, and what is built.** [repo] Agents readily describe a plan as if it already exists, and a later session then trusts it. Give statements a visible status: a *requirement* (what must be true), *proposed* (a suggestion, which agents may make), *accepted* (decided by a person), *implemented* (in merged code with a test). Write "X does Y" only for behavior that code and tests prove. Agents propose; people decide.
- **Work in progress is recoverable.** [repo] Unfinished work can be picked up from the repository: a plan or progress note for larger tasks, and commits that describe what changed and why. A new session shouldn't need you to explain where things stand.

**At scale**

- **Decisions declare their scope.** [repo] Decision records and module docs say which paths they govern, in a form tools can read (e.g. frontmatter). The agent can then filter decisions by the file it's editing instead of reading all of them.
- **Stable IDs as links.** [repo] IDs like `ADR-0007` appear where they apply: in docs, in lint messages, and in comments at non-obvious spots. One search finds everything related.
- **Recipes for repeated work.** [repo + harness] "Add a module", "add an endpoint", "cut a release" as short step-by-step recipes or skills.

### The three-reads test

Pick a random source file. Can the agent reach every decision that governs it in about three reads (entry file → index → decision record) without searching blindly? If not, the knowledge exists but isn't traversable.

### Check yourself

- Could a new agent session continue yesterday's unfinished task without your help?
- Does the instruction file reference anything that isn't in the repository?
- How many places list your decision records?

---

## 6. Freshness

> **Is what the agent finds still true?**

### Why it matters

Stale context is worse than missing context. An agent that can't find something will look further or ask. An agent that finds something wrong will act on it with full confidence. Freshness is separate from Traversable Knowledge because the failures differ: "I can't find it" versus "I found it and it's wrong."

### Four kinds of drift

| Kind | What it looks like | Example | Suggested handling |
|---|---|---|---|
| **Broken references** | Docs point at things that don't exist. | A doc names a deleted file; the entry file lists a command that's no longer in the manifest; a decision record is missing from the index. | Fail the check. These are objectively wrong. |
| **Stale content** | Code changed, the doc describing it didn't. | A new module exists, but the architecture summary doesn't mention it. | Nudge, don't fail. A doc can still be accurate after a code change. |
| **Practice drift** | Written rules disagree with what the team actually does. | The list of commit scopes misses a scope already used in history; "tests mirror `src/`" while many files have no mirror. | Review periodically. Fix the rule or the practice. |
| **Harness drift** | Agent configuration points at things that changed. | A hook calls a renamed script; a path-scoped rule's glob matches nothing; a skill uses an old command. | Check like broken references. Hooks fail silently, so this one hides well. |

### Symptoms

- The agent follows instructions that were true six months ago.
- A summary file lists five modules; the code has six.
- A hook or rule hasn't triggered in weeks, and nobody noticed.
- New contributors trust the README less than the code.
- Two documents say different things, and the agent picks one.

### Recommendations

**Starting**

- **Update docs in the same change.** [repo] When a change alters structure, its docs change in the same commit or pull request.
- **Say it in the instruction file.** [harness] "Structural changes update `ARCHITECTURE.md` and the decision index."

**Growing**

- **Decide which source wins.** [repo] Sources will disagree sometimes. Write down the order, so the agent knows which one to trust and which one to flag as wrong. A common order: the person's explicit instruction in the session; the project's process rules; accepted decisions; machine-readable contracts; requirements; code and tests (the only proof of what is *implemented*); then everything else (plans, notes, chat history, agent reports).
- **A drift check for broken references.** [repo] A small script that checks that paths named in docs exist, that commands in the entry file exist in the manifest, that every decision record is indexed, and that every top-level module appears in the architecture summary. Run it with the main check and in CI.
- **Check the harness too.** [repo] The same script checks that hook commands, rule globs, and skill commands still point at real things.
- **Generate lists that mirror the code.** [repo] Module lists, scope lists, and store tables are generated from the code, or checked against it.
- **A drift nudge before finishing.** [harness] At the end of a turn, check the docs whose scope covers the changed files and ask the agent once: update them, or confirm they're still accurate.

**At scale**

- **Docs declare their scope.** [repo] So stale-content checks can target the right document per changed path.
- **Stale-content comments on pull requests.** [repo] A bot names the doc and section to review. An explicit marker (e.g. a `Docs-Reviewed: no-change-needed` commit trailer) silences it. Avoid failing CI for stale content; it trains everyone to bypass the check.
- **Regular practice-drift reviews.** [repo] Compare the written rules against recent history every quarter.

### Check yourself

- When did a check last fail because a doc pointed at something missing?
- Does your architecture summary list every current module?
- Which of your hooks fired last week?

---

## 7. Operability & Context Economy

> **Can the agent run and observe the project alone, without drowning in noise?**

### Why it matters

An agent can only work with what it can run and see. If setup needs a human clicking through steps, or the only way to see the app is someone's screen, the agent works blind. And everything the agent reads costs context: noisy output pushes out the information it actually needs.

### Symptoms

- Setup works on your machine and nowhere else.
- The agent can't start the app, or can't see what the app is doing.
- Checks print thousands of lines when they pass.
- The always-loaded instruction file keeps growing, and the agent starts ignoring parts of it.

### Recommendations

**Starting**

- **Documented setup.** [repo] A short, fixed sequence from a clean clone to a working project. One command if you can.
- **Commands in the instruction file.** [harness] Setup, run, check, and build.
- **Pinned toolchain.** [repo] Runtime version, package manager, and a committed lockfile.

**Growing**

- **Safe default configuration.** [repo] Every required setting has a non-secret default (e.g. `.env.example`).
- **Checks that run without outside access.** [repo] No VPN, no production credentials, no physical device needed to run the checks.
- **Scripted, re-runnable dev tasks.** [repo] Dev servers, emulators, and code generation are scripts that are safe to run twice.
- **Generated code is labeled.** [repo] Generated files are marked, and the regenerate command is documented.
- **Quiet on success, specific on failure.** [repo] Summaries by default, verbose output behind a flag. Error and lint messages say what's wrong, where, and how to fix it.
- **A short always-loaded file.** [harness] Details load on demand, through links, path-scoped rules, and skills.
- **Pinned tool servers.** [harness] Tool servers the agent depends on are versioned in the repository with fixed versions.

**At scale**

- **The agent can observe the running system.** [harness] Logs, screenshots, metrics, or app state through a tool. The instruction file says what the agent can and can't observe, so it doesn't guess.
- **One instance per change.** [repo] The app can run per worktree or branch, so parallel agent sessions don't fight over ports and state.
- **Comments explain why.** [repo] Comments that restate the code cost context and add nothing.

### Check yourself

- How long from `git clone` to green checks on a new machine?
- What can the agent *see* of the running app?
- How much of a passing test run's output is useful?

---

## 8. Bounded Autonomy

> **What may the agent do alone, what needs a human, and what can be undone?**

### Why it matters

Agents are fast, and they make mistakes. The goal isn't to stop them from acting. It's to make sure irreversible or high-risk actions go through a human, that the agent meets an explanation before it meets a wall, and that whatever does go wrong can be undone.

### Symptoms

- You review every command the agent runs because nothing technically stops a risky one.
- The agent pushed, merged, published, or deleted something nobody approved.
- A dependency appeared that nobody chose.
- A bad change was hard to back out.
- The agent followed instructions it found in an issue, a web page, or a dependency's README.

### Strongly recommended, at every stage

These protect against damage that's hard or impossible to reverse.

- **No production secrets in the agent's environment.** [harness] Run the agent in a sandbox or container that holds only what the task needs. A prompt injection or a mistake can't leak what isn't there.
- **Publishing actions need a human.** [harness] Push, merge, release, deploy, and deleting data happen only on explicit human request.
- **Protected main branch.** [repo] Changes reach the main branch only through pull requests with passing CI.
- **Untrusted content is data, not instructions.** [harness] Text from issues, web pages, logs, and dependencies is read, never obeyed. Agents that read such content don't also hold permissions to push or publish.

### Recommendations

**Starting**

- **Write down the zones.** [repo] The entry file says what the agent may do alone, what it should ask about first, and what it must never do.
- **Small, separate changes.** [repo] One branch per change. Small changes are easier to review and to undo.

**Growing**

- **Apply the "never" list technically.** [harness] Use the agent's permission system or a pre-action hook for the things in the "never" list, not only prose. Blocks should explain themselves: the rule, the reason, and what to do instead.
- **Ask instead of block, for "ask first" items.** [harness] Not every risky action needs a wall. For things that are fine with a person's approval (editing a protected document, changing a gate's configuration), a pre-action hook can turn the action into a confirmation prompt for the person, and deny it only when nobody is there to answer (for example in CI or other unattended runs). Keep hard blocks for the "never" list.
- **Every change is revertible.** [repo] Atomic commits, a documented rollback for releases, and feature flags for risky changes.
- **New dependencies are checked.** [repo + harness] Before adding a package: does it exist (agents sometimes invent package names), is it maintained, does its license fit? Adding one goes through the "ask first" zone.
- **Secret scanning.** [repo] Scan every change for secrets in CI. CI tokens get the minimum permissions they need.

**At scale**

- **Owners for risky areas.** [repo] Code-owner rules require a human reviewer for CI, release, security, migrations, and infrastructure.
- **Parallel work is isolated.** [repo + harness] Worktrees or separate environments for parallel agent sessions.
- **Review stays possible.** [repo] Agents can produce more code than people can review. Watch pull-request size and review time, and change the work split before review turns into rubber-stamping.

### Check yourself

- Which items on your "never" list would actually be stopped if the agent tried them?
- What could the agent leak from its current environment?
- How long does it take to undo a bad merge?

---

## 9. The harness

The harness is everything that shapes the agent rather than the code. It isn't a pillar of its own. It's the layer through which every pillar reaches the agent during a session. These practices apply to the whole layer, whatever tool you use.

- **Keep it in the repository.** Configuration that affects how the project is built is versioned and reviewed like code. Personal, user-level configuration shouldn't carry project rules, because other people and other sessions won't have it.
- **Keep it thin.** The harness holds wiring, not knowledge: which document to load for which path, which commands to block, which checks to run when. The content it points to lives in `AGENTS.md` and `docs/` (principle 3).
- **Make it reproducible.** A fresh clone gives every person and every agent session the same guidance and limits.
- **Keep hooks fast and predictable.** Seconds, not minutes. Same input, same result.
- **Make every message actionable.** A hook or block says what's wrong, in which file, and what fixes it.
- **Add pieces only for real mistakes.** And remove pieces that no longer fire or no longer help.
- **Log what fires.** Record hook firings (e.g. to a git-ignored `.agent-state/log.jsonl`) and look at them now and then.
- **Fix the foundation first.** A nudge that fires in most sessions points at a gap in the repo (a missing recipe, an unclear decision, a slow test). Fix the gap instead of adding more hooks.

[Appendix A](#appendix-a-harness-mechanisms-by-pillar) maps the common mechanisms to the pillars they serve.

---

## 10. Putting it into practice

### Getting started

1. **Review the project** with [REVIEW.md](REVIEW.md), or walk through the "Check yourself" questions by hand.
2. **Decide what stage each pillar should be at.** A weekend project doesn't need at-scale verification. A product people rely on usually needs "growing" across the board.
3. **Start with what's cheap.** Applying a rule you already wrote down (a lint rule, a permission) usually beats building something new.
4. **Begin with the strongly recommended items** in Bounded Autonomy if any are missing.

### Keeping it alive

- **Keep a mistake log.** When an agent makes a mistake, write it down and ask which pillar and layer would have prevented it. That, not the news, decides what to change next.
- **Review every quarter.** Re-run the review, look for practice drift, read the harness log, and remove what no longer earns its place.
- **Review again after structural changes:** a new module, a new part of the stack, a new team.

### Signals worth watching

The guide describes the setup. These show whether it works. Watch them as trends, not targets; a target turns them into numbers to game.

- How often agent changes pass CI on the first try.
- How often agent changes get reverted.
- The most frequent mistake categories in your log, and whether they change after you act.
- How much of your review time goes to things a check could have caught.

### Monorepos

Review each deployable unit on its own, then the shared root (tooling, CI, the entry file). The weakest unit in a pillar tells you more than an average would.

---

## 11. Glossary

| Term | Meaning |
|---|---|
| **Agentic engineering** | Building software where coding agents write and run most of the code under human direction and review. This guide covers the codebase side of it. |
| **ADR** | Architecture Decision Record: one decision, its context, and its consequences, in one file that isn't edited after acceptance. |
| **Entry file** | The root file an agent reads first (`AGENTS.md`, `CLAUDE.md`, …). |
| **Harness** | Everything around the model that shapes how it works in a project: instructions, rules, skills, hooks, subagents, tools, permissions, and its execution environment. |
| **Hook** | A command the agent's tooling runs automatically at a point in the session (after an edit, before finishing, before a risky action). |
| **Owner document** | The one document that holds a given kind of fact; every other document links to it. |
| **Nudge** | A message from the harness that redirects the agent while it works, before a hard failure. |
| **Stage** | How far a project has taken a pillar: starting, growing, or at scale. |

---

## Appendix A: Harness mechanisms by pillar

> **This part changes fast.** It reflects tools as of October 2026. Names differ between agents; the purpose column is what matters.

| Mechanism | Purpose | Pillars |
|---|---|---|
| Instruction file (`AGENTS.md`, `CLAUDE.md`) | Always-loaded orientation | Knowledge, Legibility, Autonomy |
| Path-scoped rules | Load the right conventions doc for the area, on demand | Legibility, Knowledge, Context Economy |
| Skills / recipes | Repeatable procedures, loaded on demand | Knowledge, Operability |
| Post-edit hook | Fast typecheck or lint of changed files | Verification |
| End-of-turn hook | Related tests; docs drift nudge | Verification, Freshness |
| Pre-action hook | Block or explain risky commands and edits | Autonomy |
| Permission allow / deny lists | Hard limits on commands and paths | Autonomy |
| Session-start hook | Current state: branch, failing checks, work in progress | Knowledge, Operability |
| Reviewer subagent | Independent check of the change | Verification |
| Explorer subagent | Search without filling the main context | Context Economy |
| Tool servers (MCP) | Observe logs, UI, devices, databases | Operability, Verification |
| Sandbox / container | Limit what the agent can reach and leak | Autonomy |
| Manually invoked commands | Publish, merge, release only on request | Autonomy |

### Example: Claude Code (October 2026)

| Need | Where |
|---|---|
| Entry file | `AGENTS.md`, with `CLAUDE.md` as a link or pointer to it |
| Path-scoped rules | `.claude/rules/*.md` with `paths:` frontmatter; each rule points to the matching file in `docs/conventions/` |
| Recipes | `.claude/skills/<name>/SKILL.md`, pointing to `docs/recipes/` for the steps where possible; `disable-model-invocation: true` keeps a skill human-only |
| Hooks | `.claude/settings.json` → `hooks` (`PostToolUse`, `Stop`, `PreToolUse`, `SessionStart`) |
| Permissions | `.claude/settings.json` → `permissions.allow` / `permissions.deny` |
| Sandbox | Built-in sandboxing, or a dev container / cloud session |
| Tool servers | `.mcp.json`, with pinned versions |

---

## Appendix B: Symptom index

Start here if you came with a problem.

| You notice… | Look at |
|---|---|
| The agent says "done" and CI is red | [Verification](#3-verification): check before finishing |
| You run the tests yourself after every session | [Verification](#3-verification) |
| The agent changes tests to make them pass | [Verification](#3-verification): gates don't move to get a pass; a second opinion |
| Tests pass but bugs still get through | [Verification](#3-verification): tests that prove behavior |
| The agent describes a plan as if it were already built | [Traversable Knowledge](#5-traversable-knowledge): mark what is decided, proposed, and built |
| New code lands in a different place each time | [Legibility](#4-legibility) |
| The agent brings back a pattern you removed | [Legibility](#4-legibility): one way per concern; [Freshness](#6-freshness) |
| The agent "fixes" a deliberate trade-off | [Traversable Knowledge](#5-traversable-knowledge): record big decisions |
| You re-explain the background every session | [Traversable Knowledge](#5-traversable-knowledge) |
| Another agent or a new contributor misses rules your usual agent follows | [Traversable Knowledge](#5-traversable-knowledge): conventions live in `docs/` |
| A new session doesn't know what's half-done | [Traversable Knowledge](#5-traversable-knowledge): work in progress |
| The agent follows outdated instructions | [Freshness](#6-freshness) |
| Two documents say different things and the agent picks one | [Freshness](#6-freshness): decide which source wins; [Traversable Knowledge](#5-traversable-knowledge): one home per fact |
| A hook stopped working and nobody noticed | [Freshness](#6-freshness): harness drift |
| Setup only works on your machine | [Operability](#7-operability--context-economy) |
| The agent can't see what the app does | [Operability](#7-operability--context-economy): observe the running system |
| The agent ignores parts of the instruction file | [Context Economy](#7-operability--context-economy): a short always-loaded file |
| You approve every command by hand | [Bounded Autonomy](#8-bounded-autonomy): apply the "never" list technically |
| A dependency appeared that nobody chose | [Bounded Autonomy](#8-bounded-autonomy): new dependencies are checked |
| Reviews feel like rubber-stamping | [Bounded Autonomy](#8-bounded-autonomy): review stays possible |

---

## Appendix C: Sources

AFE is a synthesis of public writing and of experience on real projects. These are the main sources.

- Anthropic — [Claude Code: Best practices for agentic coding](https://www.anthropic.com/engineering/claude-code-best-practices)
- Anthropic — [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- Anthropic — [Effective harnesses for long-running agents](https://anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- Anthropic — [Beyond permission prompts: making Claude Code more secure and autonomous](https://anthropic.com/engineering/claude-code-sandboxing)
- Claude Code docs — [Hooks guide](https://code.claude.com/docs/en/hooks-guide), [Memory](https://docs.claude.com/en/docs/claude-code/memory), [Sandboxing](https://docs.claude.com/en/docs/claude-code/sandboxing)
- OpenAI — [Harness engineering: leveraging Codex in an agent-first world](https://openai.com/blog/harness-engineering)
- OpenAI / Linux Foundation — [AGENTS.md and the Agentic AI Foundation](https://openai.com/index/agentic-ai-foundation/)
- Google DORA — [AI Capabilities Model](https://dora.dev/research/ai/ai-capabilities-model)
- Simon Willison — [Agentic Engineering Patterns](https://simonwillison.net/guides/agentic-engineering-patterns/)
- HumanLayer — [12-Factor Agents](https://github.com/humanlayer/12-factor-agents); Heroku — [The Twelve-Factor App](https://12factor.net)
- Marmelab — [Agent Experience: Best Practices for Coding Agent Productivity](https://marmelab.com/blog/2026/01/21/agent-experience)
- Martin Richards — [Make Your Codebase Inevitable](https://www.martinrichards.me/post/make_your_codebase_inevitable/)
- Helio Medeiros — [It is time for Agent-Friendly Codebase](https://blog.heliomedeiros.com/posts/2025-08-07-agent-friendly-codebase/)
- maintainable.software — [How to Design a Maintainable Codebase for AI Coding Agents](https://maintainable.software/agentic-engineering-part-2-agentic-codebase-principles/)
