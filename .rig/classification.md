# Classification rule

Status: approved 2026-10-02. Applies to every file before it enters this repository.

## Purpose

This repository teaches any AI model how Fjeldstad Consulting works: method, writing, security, quality gates and conventions. It must never teach anyone what Fjeldstad Consulting knows about its clients, money or people. The rule below draws that line.

## Three levels

| Level | What it covers | Where it lives |
|---|---|---|
| Shareable | Working method, writing style, security principles, quality gates, folder and naming conventions, generic templates, approved skills and agents, generic lessons about tools and environments | This repository |
| Local | Identity, account addresses, machine paths, key IDs, personal background, voice sources and the voice profile | Local config on the machine (see `local-config.example.md`), never committed |
| Never out | Client data, finance, legal documents, operational status, task lists, risk register, decision history with internal context | The internal monorepo, which has no remote |

## The test

Ask three questions about every file and every paragraph:

1. Does it name or describe a person, a client or a prospect?
2. Does it contain a number, status or fact about the business (revenue, pipeline, prices agreed with someone, open risks)?
3. Does it contain a value that belongs to one machine or one account (address, path, key ID, token)?

Any yes means the content does not go in. A value can be replaced by a description of the value and a pointer to local config. The principle can usually stay while the fact goes.

## Decisions and history

The repository holds principles, not history. When an old decision matters, write the principle itself into the rule, skill or agent. Do not reference internal decision numbers, task numbers or risk IDs, since they mean nothing outside the monorepo and leak its structure.

## Work coordination

Agents working from this repository coordinate through GitHub issues. Issues follow the same rule: no client names, no business figures, no local values.

## Enforcement

- Pre-commit hook runs gitleaks on staged changes and ignores inline `gitleaks:allow` comments. It is off until enabled once per clone with `git config core.hooksPath .githooks`.
- The `secret-scan` workflow runs gitleaks on the full history for every pull request and every push to any branch, so commits from clones without the hook are still scanned. It reads `.gitleaks.toml` and `.gitleaksignore` only from the default branch, never from the change under review (`docs/hierarchy.md`, Break glass).
- `main` accepts changes only through pull requests with signed commits.
- Every new or changed skill, agent, rule or template is checked against this rule before commit.
- A full scan of tree and history runs before each release.
