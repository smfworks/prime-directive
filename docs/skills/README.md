# The skills

These are the instructions Builder Prime and Gatekeeper Prime actually follow, copied from the templates as plain markdown. In Grok Bot they come with the templates, so you don't need to do anything with them. They're here so you can read exactly what the bots are told, and so you can adapt them for another agent setup.

| File | Used by | What it covers |
|---|---|---|
| [builder-reviewer-loop.md](builder-reviewer-loop.md) | Both | The shared protocol: roles, the nine-step loop, the merge gate, guardrails |
| [builder-build-standard.md](builder-build-standard.md) | Builder | Branching, planning, fix proof, the PR description, answering review, credits |
| [builder-getting-started.md](builder-getting-started.md) | Builder | The first-run setup questions |
| [gatekeeper-review-standard.md](gatekeeper-review-standard.md) | Reviewer | What to gather, what to check, how to post, re-review, and merge |
| [gatekeeper-getting-started.md](gatekeeper-getting-started.md) | Reviewer | The first-run setup questions, routines, and branch protection |

## Using them outside Grok Bot

Each file starts with a short YAML header (`name`, `description`), the same shape many agent tools use for skills or rules files.

- **Builder agent:** give it `builder-reviewer-loop.md` and `builder-build-standard.md`, for example in its `AGENTS.md`, `CLAUDE.md`, or skills folder.
- **Reviewer agent:** give it `builder-reviewer-loop.md` and `gatekeeper-review-standard.md`. Run it as a **separate agent with its own session**, not the builder reviewing itself.
- **Getting-started files:** treat them as a setup checklist. Answer the questions once and put the answers in each agent's config or memory.

Some lines assume Grok Bot features, such as "save its id from the teammates list," messaging the other bot directly, and routines that fire on GitHub events. Swap in whatever your setup has: a shared issue thread for handoffs, a GitHub Action or cron job for the PR watch, or you passing the baton by hand. The rules themselves — plan first, failing-on-main tests, public SHA-pinned reviews, Lows become issues, a protected main, and a human yes on every merge — carry over unchanged.
