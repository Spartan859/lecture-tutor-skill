# Installation Guide

This guide is the source of truth for installing and validating the `lecture-tutor` skill from scratch.

Repository:
- `https://github.com/Epiphanythu/lecture-tutor-skill`

Raw files for this skill:
- `https://raw.githubusercontent.com/Epiphanythu/lecture-tutor-skill/main/SKILL.md`
- `https://raw.githubusercontent.com/Epiphanythu/lecture-tutor-skill/main/INSTALL.md`
- `https://raw.githubusercontent.com/Epiphanythu/lecture-tutor-skill/main/README.md`
- `https://raw.githubusercontent.com/Epiphanythu/lecture-tutor-skill/main/LICENSE`

Companion `pdf` skill references:
- Repository path: `https://github.com/anthropics/skills/tree/main/skills/pdf`
- Reference raw files for policy-permitted local installs:
- `https://raw.githubusercontent.com/anthropics/skills/main/skills/pdf/SKILL.md`
- `https://raw.githubusercontent.com/anthropics/skills/main/skills/pdf/LICENSE.txt`
- Recommended extra reference files:
- `https://raw.githubusercontent.com/anthropics/skills/main/skills/pdf/forms.md`
- `https://raw.githubusercontent.com/anthropics/skills/main/skills/pdf/reference.md`

State model used in this guide:
- `pdf status` is one of: `discoverability-confirmed`, `path-validated`, `unavailable`
- `latex status` is one of: `xelatex-ready`, `fallback-only`, `unavailable`
- `install result` is one of: `strictly validated`, `path-validated`, `incomplete`

Important naming rule:
- the repository name is `lecture-tutor-skill`
- the installed local skill folder must be `lecture-tutor`
- do not install this skill into a folder named `lecture-tutor-skill`

## For Humans

Copy and paste this prompt to your LLM agent:

```text
Install and validate the `lecture-tutor` skill by following this guide exactly:
https://raw.githubusercontent.com/Epiphanythu/lecture-tutor-skill/main/INSTALL.md
```

## For LLM Agents

Follow this order exactly.

1. Detect the host agent family.
Use one of these families: `OpenCode`, `Claude Code`, or `.agents/skills style setup`.
Use observable signals rather than guesswork:
- choose `OpenCode` if the environment, docs, or local paths clearly refer to `OpenCode` or `opencode`
- choose `Claude Code` if the environment, docs, or local paths clearly refer to `Claude Code` or `.claude`
- choose `.agents/skills style setup` if the environment uses the `.agents/skills` convention and does not present itself as `OpenCode` or `Claude Code`
- if the signals conflict or no family can be identified, stop and report the blocker instead of guessing

2. Resolve the default local skills directory for that family.
Use these defaults:
- `OpenCode` -> `~/.config/opencode/skills/`
- `Claude Code` -> `~/.claude/skills/`
- `.agents/skills style setup` -> `~/.agents/skills/`

3. Create `<skills-dir>/lecture-tutor/`.

4. Install `lecture-tutor` from the raw URLs listed in this guide.
Treat the raw URLs in this file as the authoritative install source for the skill contents.
Download these three files into `<skills-dir>/lecture-tutor/` with these exact filenames:
- `SKILL.md`
- `README.md`
- `LICENSE`

Do not rename the folder to `lecture-tutor-skill`.
Do not rely on repo metadata like `.git/` to make the skill work.

5. Verify that `<skills-dir>/lecture-tutor/SKILL.md` exists directly inside the folder.
If the host caches skill discovery per session, restart the host, reload the skill list, or run its discoverability refresh step before continuing.

6. Check the companion `pdf` status.
Treat it as:
- `discoverability-confirmed` if the platform exposes an installed or bundled skill list that includes `pdf`
- `path-validated` if a local file exists at `<skills-dir>/pdf/SKILL.md`
- `unavailable` if neither of the above is true

7. Prefer a bundled or already discoverable `pdf` skill.
Only if `pdf` is still `unavailable`, and only if the host platform and the companion skill license both clearly permit a local file install, install it from the companion references listed above.
Create `<skills-dir>/pdf/` and download at least:
- `SKILL.md`
- `LICENSE.txt`
Recommended additional files for fuller compatibility:
- `forms.md`
- `reference.md`

Do not present this local `pdf` copy flow as unconditional or universally supported.

8. Re-check the `pdf status`.
If it is still `unavailable`, report the `install result` as `incomplete` for full `lecture-tutor` use and stop.

9. Check the local LaTeX environment with these commands, in this order:
- `xelatex --version`
- `pdflatex --version`
- `latexmk --version`

10. Interpret the LaTeX check as a capability probe.
Treat it as:
- `xelatex-ready` if `xelatex --version` returns a usable version string
- `fallback-only` if `xelatex` is unavailable but `pdflatex --version` or `latexmk --version` returns a usable version string
- `unavailable` if none of the three commands returns a usable version string

Missing `xelatex` means the default Chinese-first compiled PDF path is not fully validated.

11. Verify installation state.
If the platform exposes a skill list or installed-skill view, confirm that `lecture-tutor` appears there.
If the platform does not expose such a mechanism, confirm that:
- the selected `<skills-dir>` matches the host-family rule, and
- `lecture-tutor/SKILL.md` is directly inside that directory.

12. Report the final result with:
- detected agent family
- resolved skills directory
- installed skill path
- file list in `<skills-dir>/lecture-tutor/`
- whether `pdf` is `discoverability-confirmed`, `path-validated`, or `unavailable`
- whether LaTeX is `xelatex-ready`, `fallback-only`, or `unavailable`
- whether the `install result` is `strictly validated`, `path-validated`, or `incomplete`

## Validation Standard

An installation counts as `strictly validated` only if all of the following are true:
- `<skills-dir>/lecture-tutor/SKILL.md` exists
- `<skills-dir>/lecture-tutor/README.md` exists
- `<skills-dir>/lecture-tutor/LICENSE` exists
- companion `pdf` skill is `discoverability-confirmed` or `path-validated`

LaTeX is a runtime capability check, not the installation criterion.
