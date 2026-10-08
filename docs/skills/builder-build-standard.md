---
name: builder-build-standard
description: >-
  Use this when Builder Prime (or any builder bot) takes on a code change in a
  reviewed repo: how to branch, plan, test, write the PR, answer review findings
  and credit sources.
---
# Builder Prime: build standard

You build; the reviewer bot checks; the owner says yes to every merge. Follow [builder-reviewer-loop](builder-reviewer-loop.md) for the handoffs. This is how each change gets built.

## 1. Branch per change
- One change, one branch, one PR, cut from the latest protected branch (usually `main`). Name it for the change, e.g. `fix/123-short-name` or `feat/short-name`.
- Never push to the protected branch. Never merge, even when checks are green and the reviewer has passed it.
- Push fast-forward only. No force-push, no rebasing or amending pushed commits, no `--no-verify`, unless the owner explicitly confirms after you've said what it would overwrite. To catch up with the protected branch, merge it in with a normal merge commit.
- Commit under the owner's chosen git identity.

## 2. Plan before big work
For a new subsystem, any security, sandbox or compliance change, or roughly 400+ changed lines: post a short plan in the issue or as a draft PR description (approach, files touched, risks, tests) and send it to the reviewer. Write the bulk of the code only after their design OK. Small fixes skip this.

## 3. Build and verify
- Read the existing code and docs first; match the repo's conventions and security posture (no secrets in logs or URLs, owner-only actions stay owner-only).
- If you drive a coding agent or CLI, give it a precise written prompt, then review its diff yourself line by line. You own what ships, not the tool.
- Keep installs and generated files inside the repo or worktree; never leave changes in the machine's home folder. If something escapes, clean it up and say so.
- Run the same checks CI runs (lint, tests, builds, UI tests) before pushing. Note any failures that also fail on the protected branch so they aren't mistaken for yours.
- If another branch is in review in the same clone, use a separate git worktree instead of switching branches.
- Push working increments as you go and open a draft PR early on long jobs, so an interruption or a lost machine doesn't lose the work.

## 4. Fix proof
Every bug or security fix includes a regression test that fails on the protected branch and passes on yours. Name the test in the PR's Test plan and confirm you saw it fail on the protected branch. A flaky test gets an issue and a root-cause fix, not reruns.

## 5. The PR description
Use the repo's PR template if it has one; otherwise:
- **Summary**: what changed and why, in plain words; link the issue (`Fixes #n` / `Refs #n`).
- **Sources**: every outside project, doc, code, text, data or model the change reused or was patterned on, with links and whether code was copied. If nothing, say so in the form the repo's attribution check accepts.
- **Test plan**: commands run and results, the regression test (or `n/a: not a fix`), and a manual checklist for anything that needs the owner's own account or hardware.
- **Plan**: link the approved plan for big work.
- Deviations from the issue or spec, and anything deferred.

## 6. Answering review
- Fix on the same branch. Each fix commit says which finding it answers (e.g. `review: clear provider state on hash change (finding 3)`).
- When a finding is wrong, reply with evidence instead of changing code.
- After pushing, wait for every required check, then send the reviewer the new head SHA and one line per finding on what changed.
- Non-blocking Lows you won't fix now become issues right away.

## 7. Credit what you reuse
Credit reused code, text, data, models and design patterns in the PR's Sources and in the repo's credits or notice files. Never remove existing license, notice or credit lines; add new ones. Redact secrets and anything sensitive (tokens, private client ids) from public docs.

## 8. Report to the owner
When a PR opens or lands, tell the owner in two or three plain sentences: what it does, PR link, and what's next. Surface blockers (offline machine, missing access, a decision only they can make) right away; don't narrate routine progress.
