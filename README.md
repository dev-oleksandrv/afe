# AFE — Agent-Friendly Engineering

**A guidebook for building codebases that coding agents can work in safely and well, from a weekend project to a team product.**

Agentic engineering means building software where coding agents write most of the code while people direct and review it. The field moves fast. New tools, file formats, and "best practices" appear every month, and last month's advice is often already outdated. It's easy to get lost.

AFE tries to separate what lasts from what doesn't. It doesn't tell you which agent, model, or tool to use. It names the **problems** every agent-driven codebase has to solve and collects practices that help with each one. It's a guide, not a rulebook: take what fits your project.

## The idea in a minute

Every project, whatever its size, runs into the same six problems:

| Pillar | The question |
|---|---|
| **Verification** | How does the agent know its change is correct? |
| **Legibility** | Is there one obvious way to do each thing here? |
| **Traversable Knowledge** | Can the agent find *why* things are the way they are, and what's in progress? |
| **Freshness** | Is what the agent finds still true? |
| **Operability & Context Economy** | Can the agent run and observe the project alone, without drowning in noise? |
| **Bounded Autonomy** | What may the agent do alone, what needs a human, and what can be undone? |

Each practice belongs to one of two **layers**:

- **Repo**: code, config, scripts, docs, CI, and the repository's platform settings.
- **Harness**: the agent's own setup, meaning instructions, rules, skills, hooks, subagents, tools, permissions, and the environment it runs in.

Practices are grouped by **stage** (starting, growing, at scale), so a weekend project and a product with users can take what they each need from the same guide.

## Why not just write a good `CLAUDE.md`?

Because instructions on their own are suggestions. The most common gap is a project that *describes* its rules well but applies almost none of them while the agent is working. Looking at the repo and the harness separately makes that gap visible.

## How to use it

- **Have a problem?** Start with the [symptom index](GUIDE.md#appendix-b-symptom-index), e.g. "the agent says done and CI is red", and follow it to the right pillar.
- **Setting up a project?** Read the "Starting" recommendations of each pillar in [GUIDE.md](GUIDE.md). That's about an hour of work.
- **Want an outside look?** Give your coding agent [GUIDE.md](GUIDE.md) and [REVIEW.md](REVIEW.md) and ask:
  > Review this repository with AFE (GUIDE.md + REVIEW.md). Run the checks: yes.

  You get a write-up of where each pillar stands, the symptoms already visible, and next steps ranked by value for the effort. See [a real example](examples/streamready-review.md).

### Review any project with Claude Code

You don't need to copy anything into your project. Open Claude Code in the root of the repository you want reviewed and paste this prompt:

```text
Review the current project using the AFE (Agent-Friendly Engineering) guidelines from https://github.com/dev-oleksandrv/afe.

1. Fetch and read both files in full:
   - https://raw.githubusercontent.com/dev-oleksandrv/afe/main/GUIDE.md
   - https://raw.githubusercontent.com/dev-oleksandrv/afe/main/REVIEW.md
2. Follow REVIEW.md exactly: its ground rules, its procedure, and its review template.
3. Run the checks: yes.
4. Don't modify the project. Give me the finished review in the chat, then a short summary of the top next steps.
```

Notes:

- Change `Run the checks: yes` to `no` if the agent shouldn't run setup, tests, or builds. The review then marks those results as "not run".
- To save the review as a file, name a folder outside the project in step 4 (for example ~/afe-reviews/). Agents in a sandbox may only be able to show the review in the chat.
- Claude Code asks for permission to fetch the two URLs. If web access is blocked, download [GUIDE.md](GUIDE.md) and [REVIEW.md](REVIEW.md) somewhere outside your project and point the prompt at those local paths instead.
- The same prompt works with other coding agents that can read URLs or local files.
- For a high-stakes project, run the review in a fresh session so the agent has no prior context about the code.

## Files

| File | For | What's in it |
|---|---|---|
| [GUIDE.md](GUIDE.md) | people and agents | The six pillars: why each matters, symptoms, recommendations by stage, questions to check yourself. Also the harness, putting it into practice, and a symptom index. |
| [REVIEW.md](REVIEW.md) | agents | How to review a repository with the guide, and a template for the write-up. |
| [examples/](examples/) | everyone | Real reviews. |

## Principles

- **Outcome over artifact.** The goal is "the agent can confirm its change works", not "have unit tests".
- **Problems, not mechanisms.** Hooks and skills are tools that serve the pillars; they aren't pillars.
- **Knowledge in neutral places.** What the project knows lives in `AGENTS.md` and `docs/`, where every agent and person can read it. Tool folders like `.claude/` only hold the wiring.
- **Pace layers.** The pillars should last years, the recommendations get reviewed with each version, and the tool details are expected to change every few months.
- **Mistakes drive changes.** Change your setup when an agent actually gets something wrong, not when a new tool comes out.
- **Good for humans first.** Nearly everything here also helps a new engineer on the team.
- **Right-sized.** Same pillars, different depth.

## Status

Version 1.0, October 2026. AFE draws on public writing about agentic engineering, context engineering, and agent-friendly codebases, and on reviews of real projects. It isn't an official standard. Sources are in [GUIDE.md, Appendix C](GUIDE.md#appendix-c-sources).

Feedback, counterexamples, and reviews of your own projects are welcome. Open an issue or a pull request.

## License

© 2026 Oleksandr Voronkov. Licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE). You may share and adapt this guide, including commercially, as long as you give credit and note your changes.
