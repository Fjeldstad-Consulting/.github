# Hierarchy

fc-rig is the top of the hierarchy for all Fjeldstad Consulting work. This file says how its rules reach every other repository and every agent, and what happens when levels disagree.

## Levels

| Level | What | Where |
|---|---|---|
| 1. Rig | Rules and principles for all FC work | This repository |
| 2. Organisation | Enforced repository settings: pull requests, signed commits, required checks | Organisation rulesets on GitHub, targeting all repositories, including new ones |
| 3. Repository | Rules specific to one repository | That repository's `AGENTS.md` |
| 4. Local | Values for one machine or account, and legacy rules not yet replaced | Local config and the internal monorepo |

## Precedence

- A higher level wins over a lower one.
- A lower level may add rules or tighten them. It never loosens a rule from a higher level.
- On conflict, stop and flag it to the owner. Never resolve it by assumption.

## Transition

The rig does not yet cover everything. Where it is silent, the existing local rules keep applying until a rig rule replaces them. Where a local rule contradicts the rig, the rig wins.

## Scope

- **Covered:** every repository in the Fjeldstad Consulting organisation and all FC work on any machine.
- **Not covered:** repositories on personal or hobby accounts. They are kept separate on purpose.
- **Client repositories:** the rig governs how FC works, not the client's codebase. A repository the client owns or receives must stand on its own. It never points to this private repository. It gets self-contained copies of whatever rules it needs.

## Any agent, any vendor

- `AGENTS.md` is the canonical instruction file in every repository. Several vendors read it directly.
- Tool-specific files (`CLAUDE.md`, `GEMINI.md`, `.github/copilot-instructions.md` and similar) are pointers to `AGENTS.md`. They hold only adaptations that tool needs. A rule that exists only in a tool-specific file does not exist.
- Every repository's `AGENTS.md` opens with the precedence statement: the rig applies first, and this file may tighten but never loosen.

## Distribution

An agent working in one repository usually sees only that repository. Each repository therefore carries a synced copy of the rig's core rules in `.rig/`: this file, `AGENTS.md`, `docs/classification.md`, `docs/issue-guidelines.md` and `VERSION` with the rig release it was copied from. `.rig/` is never edited by hand; the repository's own `AGENTS.md` sits beside it and may only tighten. If a newer rig is reachable than `.rig/VERSION`, the newer rig wins.

The copy is kept current by the `rig-sync` workflow in this repository. On every release tag it opens one pull request per repository that has the organisation custom property `rig-sync` set to true, updating `.rig/` and the two scan files (`.github/workflows/secret-scan.yml`, `.githooks/pre-commit`) and nothing else. A repository's own pre-commit checks go in `.githooks/pre-commit.local`, which the synced hook runs after gitleaks and the sync never touches. A sync pull request that changes a scan file carries the `Secret scan change:` line and is merged by the owner only; every sync pull request is merged by a person. The key of the organisation's GitHub App lives only in this repository's `rig-sync` environment, never as an organisation secret, so no other repository's workflows can reach it. The environment is open to `v*` tags and `main` only, a repository ruleset lets only the owner create or move `v*` tags, and the job refuses a tag whose commit is not on `main`, so the key is reachable only from code that went through a pull request. There is no version check in the repositories: the open sync pull request is the signal that a repository is behind, and a weekly job keeps an issue here listing sync pull requests open for more than seven days. Client repositories and hobby repositories never get the property.

## Enforcement

Settings that GitHub can enforce are set once at organisation level, not per repository:

- changes to the default branch only through pull requests, from the first pull request onwards
- signed commits
- a green secret scan
- no deletion or force-push

The secret scan is hardened so that a pull request cannot weaken the check that judges it. The workflow reads `.gitleaks.toml` and `.gitleaksignore` only from the default branch, ignores inline `gitleaks:allow` comments, runs on every push to every branch, scans only the history of the commit under review (the merge result for a pull request) so an unrelated branch cannot block merges, and fails if the pre-commit hook has lost its executable mode. The same rule applies to every copy of the workflow in other repositories.

A repository that lacks the secret-scan workflow cannot merge until the workflow is added, so it fails safe. The one exception is the first commit of a new repository: the required check is not enforced on creation of the default branch, so a repository can be created from the template or receive its first signed push in one step; that commit is scanned by the push run and reported, and everything after it goes through a pull request. New repositories are created from `fc-repo-template`, which carries the workflow, the hook, `.gitignore`, `AGENTS.md` with the `CLAUDE.md` pointer, and a script for the settings a template does not copy. The security policy, the support note and the issue template come from the organisation's public `.github` repository, which GitHub shows as defaults in every repository without its own copy.

## Break glass

Nobody can bypass the rulesets, including the owner. The first commit of a new repository is the only change the required check does not gate; see Enforcement.

### False positive in the secret scan

The scan never trusts an allowlist that arrives in the same pull request as the finding, so a false positive is cleared in two steps:

1. Open a separate pull request against the default branch that adds the narrowest possible allowlist entry to `.gitleaks.toml` (create the file with `[extend]` and `useDefault = true` if it does not exist). Scope the entry to one rule and one file, and state in `description` why it is not a secret:

   ```toml
   [[rules]]
   id = "slack-bot-token"
   [[rules.allowlists]]
   description = "Documented example token in the docs, not a credential"
   paths = ['''^docs/example\.md$''']
   ```

   Never put the value itself in the allowlist: `.gitleaks.toml` is not scanned, so a literal value in it would be a secret merged with a green check. Match a path, or a non-secret part of the line with `regexTarget = "line"`. The workflow rejects a trusted config that drops `useDefault = true` or extends from a path or URL, because such a path resolves inside the change under review.
2. After it is merged, re-run the check on the blocked pull request. It now scans against the updated allowlist. Locally, merge or rebase the default branch into the blocked branch first, because the pre-commit hook reads `.gitleaks.toml` from the working tree.

Use `.gitleaksignore` only for a finding that is already in the history of the default branch and cannot be rewritten. Its entries are bound to a commit or a line position, so they are not the route for open pull requests. Inline `gitleaks:allow` comments are ignored everywhere.

### The scan cannot protect itself

GitHub runs the workflow file from the pull request, so a pull request can edit `.github/workflows/secret-scan.yml`, or add another workflow whose job is also named `gitleaks`, and the required check turns green without a scan. The organisation plan offers no rule that takes the workflow from a repository the pull request cannot change, and the ruleset requires no second reviewer. The control is therefore a rule, enforced as far as a workflow can enforce it:

1. **Scan files.** `.github/workflows/` (every file in it), `.githooks/pre-commit`, `.gitleaks.toml` and `.gitleaksignore`. Changing any of them is a change to the scan, whatever the reason.
2. **Explicit note.** A pull request that changes a scan file must say so in its description, on a line that starts with `Secret scan change:` (plain, as a list item, a heading or in bold) followed by what changes and why, and a link to the issue when there is one. The `gitleaks` job fails a pull request that changes a scan file without that line, and runs again when the description or the base branch is edited. The line is for the reader, not the check: it makes the change impossible to miss.
3. **Owner reads, owner merges.** The owner reads the changed scan files line by line before merging. An agent that opens such a pull request writes the line itself. It never merges the pull request, and never adds the line to a pull request it did not open in order to get it past the check.
4. **Issue first for anything that weakens.** A change that removes a step, widens an allowlist or relaxes a trigger is proposed in an issue before the pull request is opened, so the reasoning is on record where it can be disputed.

The check in point 2 lives in the workflow the pull request can edit, so it stops mistakes, not attackers. Points 3 and 4 are the owner's discipline, not controls: nothing in the repository stops an account with write access from merging. Closing that would need a required reviewer or a merge restriction in the organisation ruleset, which needs a second person.

### Blocked legitimate change

If GitHub Actions is down or a check blocks a change that cannot wait for the route above:

1. Prefer a fix through a pull request.
2. If a pull request is impossible, an organisation owner disables the ruleset, makes the change, and enables the ruleset again straight away.
3. Record what was done and why in an issue in this repository, following `docs/issue-guidelines.md`.

## Versions

Changes to the rig go through pull requests like everything else. Releases are tagged (`v1.0`, `v1.1` and so on). Repositories record which version they follow, so a rule change reaches them as a deliberate update and not as a surprise.
