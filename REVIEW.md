# Reviewing a project with AFE (for agents)

> Use this file together with [GUIDE.md](GUIDE.md). It describes how an agent reviews a repository against the guidebook and what the write-up looks like, so reviews by different agents are easy to compare.

A review doesn't grade a project. It describes where each pillar stands, what symptoms are already visible, and which next steps would help most for the effort.

**How to ask for a review:** give the agent both files and say something like:

> Review this repository with AFE (GUIDE.md + REVIEW.md). Run the checks: yes / no.

---

## 1. Ground rules

1. **Don't change the project.** Read and run, but don't modify, create, or delete files in the reviewed repository. Write the review outside it, or where the user asks. Installing dependencies into an ignored folder (e.g. `node_modules`) to run checks is fine.
2. **Evidence for every observation.** Cite a file path (and line, if useful), a config key, a command with its output, or a commit hash. If you can't point at evidence, don't claim it.
3. **Running checks.** If the user allows it, run the project's documented setup and checks. Never run anything that pushes, publishes, deploys, or needs production credentials. If you can't or may not run something, say "not run" rather than guessing the result.
4. **Unknown isn't missing.** Some things can't be seen from a repository: branch protection, organization settings, tools on the maintainer's machine. Say what you couldn't see and what would confirm it.
5. **Respect deliberate choices.** If a decision record or the entry file explains why a practice isn't followed, report it as a choice, not a gap.
6. **Count only what exists.** Plans in issues and TODO comments are intentions, not practices.
7. **Mind your own bias.** If you're reviewing the harness you run inside, say so. For high-stakes projects, suggest a review from a fresh session or a different agent.

## 2. Procedure

1. **Orient.** Read the entry file(s), README, contributing guide, top-level layout, package manifest(s), and CI config. Note the stack, the package manager, and the agent tooling in use.
2. **Understand the context.** What kind of project is it: an experiment, a product with users, a team system? Who maintains it? Use this to judge which stage makes sense per pillar (GUIDE § 1). Write the reasoning in one or two lines.
3. **Look at each pillar in both layers.** For each pillar, go through its recommendations in GUIDE §§ 3–8 and find evidence of each. At minimum:
   - **Verification:** check commands; CI steps; test count and runtime; fakes and fixtures; hooks that run checks; a sample of tests read for quality (observable assertions, literal expectations, failure paths); whether gate configuration is protected.
   - **Legibility:** lint config for import boundaries; compare *written* style rules with *active* lint rules (e.g. print the effective config); duplicate libraries per concern; the shape of 3–5 sampled modules.
   - **Traversable Knowledge:** entry file length and whether it's tool-neutral; decision records, their index(es), and supersede links; whether everything the entry file mentions exists in the repo; how unfinished work is recorded; whether each kind of fact has one owner document; whether statuses separate proposed from built; whether project knowledge lives in neutral places or only inside a tool's folder.
   - **Freshness:** whether a precedence order for conflicting sources is written down; scan for all four kinds of drift (GUIDE § 6): broken references, stale content, practice drift (written rules vs history and tree), and harness drift (hook commands, rule globs, skill commands).
   - **Operability & Context Economy:** follow the setup from a clean state, if allowed; version pinning; what needs a device, network, or credentials; how noisy passing checks are; what the agent can observe.
   - **Bounded Autonomy:** the written zones; which "never" items are applied technically; the strongly recommended items; secret scanning; dependency policy; how changes are undone.
4. **Run the checks** if allowed: setup, each part of the main check, and a targeted check if one is documented. Record exit codes, durations, and summary lines (test counts, coverage).
5. **Describe where each pillar stands,** per layer, in words: what's in place, what's missing, and which stage it resembles. No scores.
6. **List the symptoms you found:** concrete things in this repository that match the symptoms in the guide. These are the most persuasive part of a review.
7. **Recommend next steps,** ordered by value for the effort. Each step says what it addresses and roughly how long it takes (minutes, hours, days). Prefer applying a rule the project already wrote down over introducing a new one. Put any missing strongly recommended item first.
8. **Check your review before delivering it.** Every claim has evidence. Every number comes from a command you ran or a file you read. Every cited path exists.

## 3. Review template

Copy this structure. Keep observations short and let the evidence do the work.

````markdown
# AFE Review: <project name>

- **Repository:** <url or path> @ <short sha>
- **Date:** <YYYY-MM-DD>
- **Checks run:** yes | no
- **Guide version:** AFE <version>
- **Context:** <what kind of project this is, in one or two lines>

## At a glance

| Pillar | Repo | Harness |
|---|---|---|
| Verification | <a few words> | <a few words> |
| Legibility | | |
| Traversable Knowledge | | |
| Freshness | | |
| Operability & Context Economy | | |
| Bounded Autonomy | | |

**In short:** <two or three sentences: what's strongest, what would help most>

## Symptoms already visible
- <concrete finding in this repo> → <pillar>

## Verification
> How does the agent know its change is correct?

**In place**
- <observation with evidence>

**Worth considering**
- <gap, why it matters here>

<!-- repeat for the other five pillars -->

## Recommended next steps
| # | Step | Addresses | Effort |
|---|---|---|---|

## Not visible from the repository
- <e.g. branch protection: check repository settings>

## Appendix: commands run
```bash
<every command, in order>
```
````
