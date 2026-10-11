# marola-agents

The inventory of every agent marola uses or plans to use: the coding agents that write and review
the code, the Claude Projects and their routines, the agents inside the product, the ones CI
calls, and the ones still on paper. For each one it says where the agent lives, what triggers it,
which model runs it, what it may touch and what it may not.

**Status:** new, not yet a submodule of the [umbrella](https://github.com/marola-dev/marola). The
design MIP comes later. Inventory read on 2026-10-11 from every repo's `main`.

Nothing here is loaded by Claude Code or any other tool. Each agent keeps living in the repo that
runs it, and this repo describes it. Skills are catalogued separately, in
[marola-skills](https://github.com/marola-dev/marola-skills).

## What counts as an agent

A model that acts on its own between a trigger and a result: it reads, decides and writes a
report, a comment, a commit or an answer. A script that only calls an API (the arXiv digest, the
wiring generator) is not one, and neither is a bot with no model in it (Scala Steward, the
pointer-sync PR).

## Coding agents (Claude Code subagents)

| Agent | Repo | Model | Tools | Job | Writes? |
|---|---|---|---|---|---|
| `mip-reviewer` | marola-devkit plugin | fable | Read, Grep, Glob, Bash | Reviews one PR of a MIP task stack against its `.tasks.md` row; correctness gaps only | No, it reports |
| `mip-claims-auditor` | marola-devkit plugin | haiku | Read, Grep, Glob, Bash, WebFetch | Finds unsourced external claims in a MIP's §4 | No, it reports |
| `jar-verifier` | marola-app | sonnet | Bash | Checks a Kyo API against the pinned jar with `javap` | No, it reports |
| `news-fact-check` | marola-site | opus | Read, Grep, Glob, Bash, WebFetch | A news post's first review: every claim against its link, pt-BR and en parity | No, it reports |
| `news-copy-review` | marola-site | opus | Read, Grep, Glob | A news post's second review: language, voice, clarity | No, it reports |
| `revisor-proposta` | h0ffmann/ww3-gpu | opus | Read, Grep, Glob, Bash | Scientific review of the UFRJ/DEL project proposal (ABNT, register, citations) | No, it reports |
| `security-reviewer` | h0ffmann/ww3-gpu | opus | Read, Grep, Glob, Bash | Security review of a branch's diff: secrets, hooks, workflows, prompts | No, it reports |

Every one of them is read-only by design: it reports, a person or the calling session fixes.

## Claude Projects and their routines

| Project | Settings copy | Agents |
|---|---|---|
| Marola Agent | the umbrella's `CLAUDE_PROJECT.config.md` | A coordinator session in the project chat and one thread session per task |
| Wave Forecaster | h0ffmann/ww3-gpu `.claude/project/` (ADR-0005) | The same shape, for the WW3-on-GPU book and lab |

| Routine | Project | Schedule | Job |
|---|---|---|---|
| A2A inbox (`para:marola`) | Marola Agent | every 3 h | Answers `[A2A]` issues addressed to marola in marola-dev/marola and h0ffmann/ww3-gpu |
| Preparar post LinkedIn | Marola Agent | Mon–Wed 9:52 (São Paulo) | Reminds the day before a post and suggests a topic |
| Lembrete LinkedIn | Marola Agent | Tue–Thu 9:52 (São Paulo) | Reminds at posting time |
| A2A (`para:wave-forecaster`) | Wave Forecaster | every 3 h | The other side of the same channel |
| Daily deps update | Wave Forecaster | daily 9:52 (São Paulo) | Opens or drives one dependency-update PR in ww3-gpu |
| Weekly settings snapshot | Wave Forecaster | Fridays 2:30 (São Paulo) | Refreshes `.claude/project/` from the live settings |

## Agents in the product (marola-app)

| Agent | Where | Model | Job | Limit |
|---|---|---|---|---|
| Summarizer | `core/…/llm/`, `recommendation_prompt.json` | a local Ollama model (llama3.2 by default, marola-sea) | Writes the short summary of a beach's best hour | Never changes a score or drops an alert |
| Reviewer | `core/…/llm/Reviewer.scala`, `review_prompt.json` | the same | Checks the summary against the numbers | Can only reject or correct the text |
| Ask / chat | `cli/…/agent/ChatServer.scala`, the site's chat widget | the same | Answers ocean questions from the sourced corpus (RAG), with the safety footer | Answers only from marola-corpus |
| MCP tool server | `cli/…/agent/SwimConditionsMcpServer.scala` | none of its own | Exposes the pipeline to an outside agent | Read-only tools |
| Vision | `core/…/vision/`, `local/…/vision/` | a local Ollama vision model | Describes a photo a user sends ("is this a jellyfish?", visibility) | Answers that user only |

## Agents in CI

| Agent | Repo | Trigger | Job |
|---|---|---|---|
| Gemini review | every marola-dev repo, via marola-devkit's `gemini-review.yml` | a person requests the `gemini` team on a PR | One review per request, then a fix commit only where the fix still applies, never under `.github/` |
| DSPy prompt compile | marola-ml `compile-prompt.yml` | dispatch by a person | Compiles the summarizer and reviewer prompts against a model served in the job, then opens a PR to marola-app |

## Planned

| Agent | For | Where it is written down | State |
|---|---|---|---|
| Post writer | Drafts a news post in pt-BR and en, ready for the two news reviewers | marola-site's `news-post` skill does the writing today | Idea |
| Site UI agents | Discipline in UI work on marola.dev: design review, worst-case data, motion review, screenshots before and after | marola-site's `site-frontend` skill orders the frontend skills today | Idea |
| Hazard detection agent | Detects hazards and compares them with public alerts | umbrella `FUTURE-WORK.md` §9.2 | Candidate, gated on a human-confirmation design |
| Escalation, digest and answer actors | marola as actors on Pekko, each also an MCP tool | umbrella `AGENT-FRAMEWORKS-SURVEY.md` §2–§3 | Candidate |
| "Ask marola about marola" | Q&A over the project's own docs (`ask_project_question`) | umbrella `AGENT-STACK-SURVEY.md` §5 | Candidate |
| Hosted agent over HTTP/SSE MCP | The MCP server reachable from outside | umbrella `ARCHITECTURE.md` §5c | Candidate, Phase 2 |
| Evaluation harness | Per-agent scoring and cross-agent traces over the benchmark and the reviewer | umbrella `FUTURE-WORK.md` §4.1 | Candidate |

A planned agent moves to its section above when the PR that adds it merges.

## Rules every agent here follows

- **Read-only unless its job is to write**, and then only what its row says.
- **No paid model or resource without a person's go-ahead.** The product's agents run on a free
  local Ollama model; CI's run on free tiers or inside the job.
- **No model id, secret or session link in a file pushed to a repo.** A row names the model as
  its definition does, never a dated id.
- The org invariants apply as in every marola repo: [AGENTS.md](AGENTS.md).

## Keeping it current

A PR that adds, removes or changes an agent in any repo updates its row here. A row says where the
agent is defined, so `git grep` or a look at that file checks it.

## Links

- [marola-skills](https://github.com/marola-dev/marola-skills): the catalogue of skills
- [The umbrella](https://docs.marola.dev/): MIPs, phase list, ways of working
- [AGENTS.md](AGENTS.md): what an agent may and may not do in this repo
