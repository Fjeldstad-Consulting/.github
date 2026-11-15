# .github

Default community health files for the Fjeldstad-Consulting organisation. GitHub uses them in every repository that has no file of its own:

- `SECURITY.md`: how to report a vulnerability
- `SUPPORT.md`: where to ask for help
- `.github/ISSUE_TEMPLATE/`: the Task, Bug and Decision issue forms, and no blank issues; this is their only source
- `.github/pull_request_template.md`: the pull request template; this is its only source
- `profile/README.md`: shown on the organisation page

A repository that needs a different file adds its own copy; the default then no longer applies there.

## The secret scan and the pre-commit hook

`.github/workflows/secret-scan.yml` and `.githooks/pre-commit` are copies of the files in the private repository fc-rig. They are replaced from there and never edited here. Both run [gitleaks](https://github.com/gitleaks/gitleaks), an external tool: the hook checks staged changes before a commit, and the workflow scans the history on pushes and pull requests. Neither finds every secret.

Prerequisites for the hook: git, a POSIX shell (Git Bash on Windows) and `gitleaks` on `PATH`. Check with `gitleaks version`.

Install the hook once per clone, from the root of the clone. The commands are for an ordinary clone that uses the default hooks folder; if `.git/hooks/pre-commit` already exists, keep what it checks.

```sh
cp .githooks/pre-commit .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

The hook then runs on every commit, and it also runs `.githooks/pre-commit.local` if the branch you have checked out has one, so read both files before you install it and commit only on branches you trust. Run the two commands again when `.githooks/pre-commit` changes on `main`. People with access to fc-rig use its installer instead, which does the same and checks the copy.

When a commit is stopped, read the whole output. The two usual last lines:

- `pre-commit: gitleaks not found on PATH, commit stopped.`: a prerequisite is missing. Install gitleaks, or put it on `PATH`, and commit again.
- `pre-commit: gitleaks found a possible secret, commit stopped.`: the scanner rejected the staged change. The lines above name the file and the rule. Remove the secret from the change; if it was ever pushed, treat it as leaked and replace it.

The copied files refer to `docs/hierarchy.md`, section Break glass. That file is in fc-rig and is not available here. What it says for this repository: a pull request that changes the scan files must say so in its description on a line starting with `Secret scan change:` (the scan fails otherwise), so that the owner reads those files before merging; and a false positive is cleared through `.gitleaks.toml` on the default branch, never with an inline comment.

This repository is public, because GitHub serves organisation defaults only from a public `.github`. It therefore carries no copy of the rig's rules (`.rig/`) and is not a sync target: the rig is private, and nothing from it beyond the scan files belongs here. Decided 2026-10-17.
