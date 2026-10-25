# .github

Default community health files for the Fjeldstad-Consulting organisation. GitHub uses them in every repository that has no file of its own:

- `SECURITY.md`: how to report a vulnerability
- `SUPPORT.md`: where to ask for help
- `.github/ISSUE_TEMPLATE/`: the task template from fc-rig, and no blank issues
- `profile/README.md`: shown on the organisation page

A repository that needs a different file adds its own copy; the default then no longer applies there. The secret-scan workflow and the pre-commit hook are copies from fc-rig `main`, as in every repository; install the hook once per clone as fc-rig describes.

This repository is public, because GitHub serves organisation defaults only from a public `.github`. It therefore carries no copy of the rig's rules (`.rig/`) and is not a sync target: the rig is private, and nothing from it beyond the scan files belongs here. Decided 2026-10-17 (fc-rig #47).
