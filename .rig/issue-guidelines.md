# Issue guidelines

Applies to every issue in every Fjeldstad Consulting repository, whoever writes it, human or agent.

## The rule

An issue must be understandable and finishable by any agent or person who has never seen the conversation it came from. Anyone who reads it should be able to say what the issue is about, why it matters, and how to prove it is done, without asking.

## Required sections

1. **Background.** What the situation is and why it matters, in plain terms. Name the files, branches, pull requests or pages involved and link them. Do not refer to chats, meetings, local files or anything else the reader cannot open.
2. **Current state.** The facts the issue rests on, with the date and how they were checked (command, page, API call). A reader should be able to repeat the check.
3. **Scope.** What the issue covers, and what it deliberately does not cover.
4. **Constraints and risks.** What must not break, what must not be done, and what could go wrong.
5. **Done when.** A checklist of results, not activities. Each item must be verifiable by someone else, ideally with a command or an observable outcome. Avoid items like "improve", "look into" or "clean up" without a measurable result.
6. **How to verify.** The concrete steps that prove each item in Done when.

Options or a proposed approach may be added when they help. Keep them apart from the facts.

## Writing

- One issue per outcome. Split an issue that would need two different pull requests.
- Title states the outcome, for example "Delete merged and obsolete branches", not "Branches".
- Write in the working language of the repository.
- Follow `docs/classification.md`: no client data, business figures or local values in issues.
- Facts that change after the issue is written are updated in the issue, with a date, not only in comments.

## Closing

Close an issue only when every Done when item is checked, and link the pull request or command output that proves it. If the scope changed, say how in the closing comment.
