---
name: gatekeeper-review-standard
description: >-
  Use this when Gatekeeper Prime (or any reviewer bot) reviews a pull request,
  re-reviews a fix, or decides whether a PR is merge-ready.
---
# Gatekeeper review standard

Follow [builder-reviewer-loop](builder-reviewer-loop.md). Look up the GitHub connector's tool schemas fresh each time.

## Gather
1. PR metadata: author and their association (owner, collaborator, outside contributor, bot), draft state, base and head branch, **full head SHA**, and mergeable state.
2. The full diff. For a huge diff, read the riskiest files first (auth, sandbox, network, config, CI workflows, dependency manifests, migrations) and say the review was partial.
3. Check runs for the head SHA, not just the combined status. Any failed run is red. Read the failing job's log tail.
4. Earlier reviews and comments, so you don't repeat yourself.

## Check
- **Correctness:** does it do what the title claims? Look for edge cases, error handling, and races.
- **Security:** secrets, weakened auth or sandboxing, new listeners or outbound calls, unsafe shell or eval, symlink and path tricks, CI permission changes (`pull_request_target`, new third-party actions).
- **Tests:** behavior changes need tests. A fix needs a regression test that fails on the protected branch, so check out the base, run that test, and confirm it fails. Deleted or skipped tests are a red flag.
- **Plan:** large or security-related PRs should trace back to a plan you approved. If there's none, ask for one before the code review.
- **Scope:** no unrelated refactors or mass reformatting.
- **Attribution:** reused code, text, data, or models must be listed in the PR's sources and the credits file. No removed license, notice, or credit lines. No agent rules pulled in from outside web pages.
- **Honesty:** docs, status badges, and changelogs must match what the code really does.
- **Owner's standing rules:** apply any repo-specific rules saved in memory.

When you can, run it: execute the risky path or a quick probe, and keep the evidence in a scratch folder.

## Post
- Post the review on the PR as a public review pinned to the head SHA. Use inline comments where they help. Rank findings High, Medium, or Low, and give one line of fix advice for each. End with a verdict.
- If the PR author is the same account the bot posts as, GitHub won't allow an approval, so post it as a comment review.
- Message the builder directly for anything they must act on.
- File every non-blocking Low as an issue now and link it.
- Keep review text factual and short. No secrets, tokens, or private paths.

## Re-review
Read only what changed since your last review, but confirm every earlier finding is really fixed. Re-run the regression test. Wait for all required checks, including slow security scanners. For checks still pending, set a one-off watch on that PR that fires when checks finish and deletes itself on merge or close.

## Merge
Only on the owner's explicit yes for that PR. Re-read the head SHA, confirm every check run is green, then squash-merge locked to that SHA. Report the merge commit to the owner and the builder.
