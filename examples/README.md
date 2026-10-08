# Examples

Drop-in files for the repo you want Builder Prime and Gatekeeper Prime to work on.

| File | Put it at | Notes |
|---|---|---|
| [ruleset.json](ruleset.json) | Apply with `gh api -X POST repos/OWNER/REPO/rulesets --input ruleset.json` | Replace the check names with your own first. See [setup step 6](../docs/setup.md#6-protect-main-with-a-ruleset). |
| [PULL_REQUEST_TEMPLATE.md](PULL_REQUEST_TEMPLATE.md) | `.github/PULL_REQUEST_TEMPLATE.md` | If your CI checks for a sources section, make the heading match. See [setup step 7](../docs/setup.md#7-add-the-pr-template). |
