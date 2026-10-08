# Setup guide

This walks you from zero to your first merged PR with Builder Prime and Gatekeeper Prime. Plan on about 30 minutes, most of it answering setup questions.

**You'll need:**
- A Grok Bot account
- A GitHub account and a repo you want the pair to work on (yours, or one you can change settings on)
- For Builder Prime: either a computer you've registered with Grok Bot, or access to cloud coding agents

**Contents**
1. [Import both templates](#1-import-both-templates)
2. [Run Gatekeeper Prime's setup chat](#2-run-gatekeeper-primes-setup-chat)
3. [Run Builder Prime's setup chat](#3-run-builder-primes-setup-chat)
4. [Choose where Builder Prime codes](#4-choose-where-builder-prime-codes)
5. [Pair the bots](#5-pair-the-bots)
6. [Protect main with a ruleset](#6-protect-main-with-a-ruleset)
7. [Add the PR template](#7-add-the-pr-template)
8. [Check the routines](#8-check-the-routines)
9. [Your first PR, start to finish](#9-your-first-pr-start-to-finish)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. Import both templates

Open each link and add the template to your Grok Bot:

- **Builder Prime:** [https://x.ai/bot/W_emQTXhJxJWJOi-c3hjD](https://x.ai/bot/W_emQTXhJxJWJOi-c3hjD)
- **Gatekeeper Prime:** [https://x.ai/bot/re7nU5kngjZPu3qNtvzUx](https://x.ai/bot/re7nU5kngjZPu3qNtvzUx)

Each one becomes its own bot with its own chat. Neither template contains anyone else's repos, accounts, or secrets. Everything specific to you gets filled in during setup.

Do Gatekeeper Prime first. It's the one that sets up branch protection, so it's good to have it in place before any code is written.

## 2. Run Gatekeeper Prime's setup chat

Open Gatekeeper Prime's chat. It introduces itself and asks one question at a time:

1. **GitHub.** Is GitHub connected? If not, it walks you through connecting it, then asks for your GitHub login.
2. **Repos.** Which repos it should guard, which one is the priority, and whether any repo has special rules. Mention anything a reviewer should enforce, such as "no cloud APIs in this repo," "this is a mirror we don't own," or "every PR must update the changelog."
3. **Builder.** Which of your bots builds the code. If Builder Prime isn't set up yet, say so and come back to this after step 3.
4. **Name.** What to call it. "Gatekeeper Prime" is fine, or give it a name of your own.
5. **Rhythm.** When you want the morning digest, which day the weekly sweep should run, and whether you read on your phone. If you do, it keeps everything short.
6. **Public reviews.** Whether it can post reviews on your PRs under your GitHub account. We recommend yes. Public reviews are rule 1.
7. **Protection.** Whether it can add a branch ruleset to the priority repo. It lists the check names it found, shows you the exact ruleset, and waits for your yes. See [step 6](#6-protect-main-with-a-ruleset).

When it's done, it saves what it learned, creates its routines, and tells you what's live.

## 3. Run Builder Prime's setup chat

Open Builder Prime's chat. Same idea, one question at a time:

1. **GitHub.** Is it connected, what's your login, and do your repos live under a personal account or an organization?
2. **Where it codes.** Your own computer or cloud coding agents. See [step 4](#4-choose-where-builder-prime-codes) before you answer.
3. **Repos.** Which repos it works on, which is the priority, and any special rules.
4. **Reviewer.** Which bot reviews its work. Pick Gatekeeper Prime.
5. **Name.** What to call it.

It also asks for the **git name and email** its commits should use. Use something you're happy to see in a public commit history. GitHub's `you@users.noreply.github.com` address works well if you'd rather not publish your email.

## 4. Choose where Builder Prime codes

Builder Prime needs somewhere to clone your repo, run the tests, and push branches. You have two options.

**Option A: one of your own registered computers.**
Builder Prime runs commands on a machine you've registered with Grok Bot. Every command needs your local-tool approval, and you can grant that from your phone. You get full control and your own toolchain. The trade-off is that work pauses when the machine is off or asleep.

During setup, tell it:
- Which computer to use
- Where to clone repos, for example `~/code`
- Whether there's a coding CLI or agent on that machine you want it to drive
- The git identity for commits

Make sure that machine can already push to the repo over git, with SSH keys or the GitHub CLI signed in, and has your project's toolchain installed so it can run the same checks CI runs.

**Option B: cloud coding agents.**
Builder Prime hands coding jobs to cloud agents. That needs nothing running on your hardware and works while your laptop sleeps. The trade-off is that you can't watch the work happen on a machine you own, and any project-specific tools have to be installable in the cloud environment.

Either way, Builder Prime reviews the coding tool's diff itself before pushing. It owns what ships, not the tool.

## 5. Pair the bots

The two bots find each other by id, and each setup chat asks for its partner.

1. In Gatekeeper Prime's setup, at the **Builder** question, pick Builder Prime from your list of bots. It saves Builder Prime's id.
2. In Builder Prime's setup, at the **Reviewer** question, pick Gatekeeper Prime. It saves Gatekeeper Prime's id.
3. Each bot sends the other a short intro: who it is, which repos, and which rules it follows. Each asks the other to save its id.

**How to check it worked:** ask either bot, "Who's your partner, and what's their id?" Both should name the other.

If you set up Gatekeeper Prime before Builder Prime existed, just tell Gatekeeper Prime, "My builder is now Builder Prime," and it'll update its memory and send the intro.

## 6. Protect main with a ruleset

This is rule 5: main is protected by GitHub, not by good intentions. Once it's on, nobody can push straight to `main` or merge with a red check. That includes both bots and you.

> Rulesets are free on public repos. On private repos they need a paid GitHub plan.

### Find your check names

Required checks are matched by name, so you need the exact names GitHub shows on a PR.

**From the web:** open any recent PR, scroll to the checks box, and expand it. Each line is a check name. Matrix jobs show up with their matrix values, for example `test (3.12)`.

**From the terminal** (with the [GitHub CLI](https://cli.github.com)):

```bash
# All checks on a PR
gh pr checks 42 --repo OWNER/REPO

# Check runs on a specific commit
gh api repos/OWNER/REPO/commits/SHA/check-runs --jq '.check_runs[].name'

# Some services, such as secret scanners, report as commit statuses instead
gh api repos/OWNER/REPO/commits/SHA/status --jq '.statuses[].context'
```

Include every check you'd never merge without: tests, lint, build, your attribution check if you have one, and your secret scanner.

### The ruleset

Here's the shape we use, also in [examples/ruleset.json](../examples/ruleset.json). Replace the check names with yours:

```json
{
  "name": "protect-main",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": { "include": ["~DEFAULT_BRANCH"], "exclude": [] }
  },
  "bypass_actors": [],
  "rules": [
    { "type": "deletion" },
    { "type": "non_fast_forward" },
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 0,
        "dismiss_stale_reviews_on_push": false,
        "require_code_owner_review": false,
        "require_last_push_approval": false,
        "required_review_thread_resolution": false,
        "allowed_merge_methods": ["squash"]
      }
    },
    {
      "type": "required_status_checks",
      "parameters": {
        "strict_required_status_checks_policy": false,
        "do_not_enforce_on_create": false,
        "required_status_checks": [
          { "context": "lint" },
          { "context": "test (3.12)" },
          { "context": "build" },
          { "context": "attribution" },
          { "context": "GitGuardian Security Checks" }
        ]
      }
    }
  ]
}
```

What each piece does:

| Rule | Effect |
|---|---|
| `deletion` | Nobody can delete `main`. |
| `non_fast_forward` | No force-pushes to `main`. |
| `pull_request` with `allowed_merge_methods: ["squash"]` | Changes land only through a PR, and only as a squash merge. |
| `required_approving_review_count: 0` | Your yes in chat is the gate, not a GitHub approval. Raise it to `1` if the reviewer posts from its own GitHub account (see the [FAQ](../README.md#faq)). |
| `required_status_checks` | Every listed check has to pass on the PR's head commit before it can merge. |
| `bypass_actors: []` | No one can override any of it. |

### Apply it

**Let Gatekeeper Prime do it.** It shows you the ruleset during setup and creates it on your yes.

**Or use the GitHub CLI:**

```bash
gh api -X POST repos/OWNER/REPO/rulesets --input examples/ruleset.json
```

**Or use the web:** go to **Settings → Rules → Rulesets → New ruleset → New branch ruleset**, target the default branch, and turn on *Restrict deletions*, *Block force pushes*, *Require a pull request before merging* (allowed merge methods: squash only), and *Require status checks to pass*. Leave the bypass list empty.

**Check it:** open the repo's **Settings → Rules → Rulesets** and confirm it says *Active*. Or run `gh api repos/OWNER/REPO/rules/branches/main`.

## 7. Add the PR template

A PR template makes every PR answer the questions the reviewer will ask anyway. Copy [examples/PULL_REQUEST_TEMPLATE.md](../examples/PULL_REQUEST_TEMPLATE.md) into your repo at `.github/PULL_REQUEST_TEMPLATE.md`:

```markdown
## Summary
<!-- What changed and why, in plain words. Link the issue: Fixes #123 / Refs #123 -->

## Sources
<!-- Every outside project, doc, code, text, data, or model this change reused or was patterned on,
     with links and whether code was copied. If nothing: "None: original work." -->

## Regression test
<!-- For a bug or security fix: the test that fails on main and passes here, and how you saw it fail on main.
     Not a fix? Write "n/a: not a fix". -->

## Plan
<!-- Big work only (new subsystem, security change, or ~400+ lines): link the plan the reviewer approved.
     Otherwise: "n/a: small change". -->

## Test plan
<!-- Commands you ran and their results. A manual checklist for anything that needs the owner's own account or hardware. -->
- [ ] 
```

| Section | Why it's there |
|---|---|
| **Summary** | The reviewer checks the code against what the PR claims. |
| **Sources** | Credit for anything reused. If your repo has an attribution check in CI, make this heading match what it looks for. |
| **Regression test** | Rule 2. The reviewer runs this test against `main` to confirm it really fails there. |
| **Plan** | Rule 3. Big PRs trace back to an approved design. |
| **Test plan** | What was actually run, so "it works" is a claim the reviewer can check. |

Builder Prime adds the template in a small PR if you ask it to. That PR goes through the loop like any other, which also makes it a good first PR.

## 8. Check the routines

Gatekeeper Prime creates three routines during setup. Builder Prime doesn't need any, since it works when the reviewer or you hand it something.

| Routine | When it runs | What it does |
|---|---|---|
| **PR watch** | When a PR on your priority repo opens, gets new commits, or merges, and when CI fails on `main` | Reviews or re-reviews the new head, waits for checks to finish, sends findings to Builder Prime, and sends you a merge-ready note when it's time. |
| **Morning digest** | Daily, at the time you picked | One short message: open PRs and their state, what merged, anything waiting on you, red CI. |
| **Weekly sweep** | Weekly, on the day you picked | Looks at open Low-priority issues, hands Builder Prime up to three to pick up, and sends you one line about it. |

To change a time or turn one off, just tell Gatekeeper Prime, for example: "Move the digest to 7:30" or "Pause the weekly sweep."

When checks are still running, the PR watch doesn't poll. It sets a one-off watch on that PR that fires when the checks finish and deletes itself once the PR merges or closes. That's rule 6.

## 9. Your first PR, start to finish

Start small. A typo fix, a missing test, or the PR template from step 7 is perfect.

1. **Ask Builder Prime.** For example: "Add the PR template from prime-directive to `.github/` in my-repo."
2. **It builds.** It cuts a branch from `main`, makes the change, runs the checks locally, and opens a PR with the template filled in. It tells you in two or three sentences, with the link.
3. **The watch fires.** Gatekeeper Prime reads the diff at the head commit and posts a public review on the PR with a verdict: merge-ready, changes needed, or close.
4. **If changes are needed,** Gatekeeper Prime messages Builder Prime directly. Builder Prime pushes fix commits to the same branch, each naming the finding it answers, then sends back the new head SHA.
5. **Re-review.** Gatekeeper Prime confirms each finding is really fixed, re-runs any regression test against `main`, and waits for every required check, including slow scanners.
6. **You get the note.** Something like:
   > **my-repo #12 is merge-ready** at `a1b2c3d`. All 5 checks are green. Adds the PR template; docs only, no code changes. Merge?
7. **Say yes.** Reply with a clear yes for that PR, for example "Yes, squash-merge #12." Gatekeeper Prime re-reads the head SHA, confirms the checks are green, and squash-merges locked to `a1b2c3d`.
8. **Done.** It reports the merge commit to you and to Builder Prime. Any non-blocking Lows are already filed as issues.

A yes covers one PR at one commit. If anything is pushed after your yes, Gatekeeper Prime asks you again.

## 10. Troubleshooting

**A required check says "Expected — waiting for status to be reported" and never finishes.**
The name in the ruleset doesn't match what CI reports. Matrix jobs, renamed jobs, and workflows that only run on `push` (not `pull_request`) are the usual causes. Look up the real names (see [step 6](#find-your-check-names)) and update the ruleset.

**The merge fails with "Head branch was modified."**
That's the SHA lock working. Something was pushed after your yes, so Gatekeeper Prime will show you the new commit and ask again.

**The secret scanner shows up minutes after everything else.**
Some scanners report several minutes after a push. Gatekeeper Prime waits for every required check, so a merge-ready note can lag a little behind the last test.

**Gatekeeper Prime's review shows up as "commented," not "approved."**
Both bots are using the same GitHub account that opened the PR, and GitHub doesn't let an account approve its own PR. That's expected. Your yes is the gate. For formal approvals, give the reviewer its own GitHub account.

**Builder Prime seems stuck.**
If it codes on your computer, there's probably a command waiting for your local-tool approval. Check your notifications. Also check that the computer is on and connected.

**The bots aren't talking to each other.**
Ask each one, "Who's your partner?" If either has the wrong id or none, tell it the right bot, and it'll update and send a fresh intro.

**Builder Prime got blocked pushing to `main`.**
Good. It should never push there. It should be working on a branch. If it keeps trying, remind it of its build standard.

**A test passes sometimes and fails other times.**
Don't let anyone rerun it until it's green. Rule 6 says a flaky test gets an issue and a root-cause fix. Ask Builder Prime to file and fix it.

**A first-time outside contributor's PR has no checks.**
GitHub holds Actions runs on PRs from first-time contributors until a maintainer approves them. Approve the run in the PR's checks section, after reading the diff yourself.

**I want different rules for a specific repo.**
Tell Gatekeeper Prime, for example: "In my-repo, every PR must update CHANGELOG.md." It saves the rule and applies it in every review there. Tell Builder Prime too, so it gets it right the first time.
