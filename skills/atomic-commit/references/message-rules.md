# Advanced Commit Message Rules

Read this file only when `intent` or the candidate diff requires a body, issue
footer, or breaking-change notation. Keep ordinary Conventional Commits to one
line.

## Breaking Changes

When the change is explicitly breaking:

- Add `!` as `type!:` or `type(scope)!:`.
- Add `BREAKING CHANGE: <description>` as a footer.
- Describe the user-visible impact or migration requirement in English.

Block when compatibility may break but the impact is unclear. Request the
missing impact through the Result Contract; do not invent it.

## Issue References

Use only issue references present in `intent` or the candidate diff:

- Use `Closes #123` when the commit fixes and closes the issue.
- Use `Refs #456` when the commit relates to the issue without closing it.

Never infer an issue number from unrelated text.

## Body and Footers

Write the body in English only when it adds user impact, risk, migration steps,
or necessary rationale. Use footers only for breaking changes or explicit issue
references. Omit both when no such information exists.
