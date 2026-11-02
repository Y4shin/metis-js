# Per-repository agent configuration

The workflow skills read this directory for repository-specific settings.
Each section starts empty; a skill fills its entry when it first needs the
answer, and the answer is reused from then on.

## Commands

The commands a skill may run, and the one it must not run. Cross-reference
`docs/testing.md` for test commands and `docs/dev-env.md` for environment
commands rather than repeating them here.

- Install: not defined
- Build: not defined
- Full verification before a commit: not defined
- Never run: not defined

## Review

- Reviewers to ask, or none: not defined
- Files a reviewer must always see: not defined
- Known-dead code to ignore: not defined

## Shipping

- Branch that lands work: not defined
- Merge style: not defined
- Release command: not defined

## Conventions that override a skill default

A skill's default is used unless an entry here overrides it. State the skill,
the default it would use, and what to do instead.

(none recorded)
