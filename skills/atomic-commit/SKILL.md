---
name: atomic-commit
description: >-
  Create atomic Git commits by delegating repository inspection and mutation to
  a Codex subagent. Use when the user asks to commit current or scoped changes,
  split mixed work into commits, or follow an approved atomic-commit workflow.
  Do not use for amend, initial commit, push, staging-only work, or planning
  commits without creating them.
---

# Atomic Commit

Coordinate one bounded Commit Batch. The Committer Subagent exclusively runs
Git and verifies its own commits; the Coordinator validates only its JSON.

## Coordinator workflow

1. Resolve `project_path` from the user request or current workspace metadata.
   Ask only when neither identifies one absolute path.
2. Resolve this Skill directory as `skill_path` without opening
   `references/commit-guide.md`.
3. Preserve the user's complete request as `intent`. When the user explicitly
   names files or directories, pass them as the maximum `commit_files` scope;
   otherwise omit `commit_files`.
4. Create one fresh Codex subagent without conversation history. Send only the
   delegation envelope below, then wait for it to finish. Do not pin a model.
5. Validate the returned JSON against the Result Contract. If it is malformed,
   ask the same subagent once to correct only the JSON response without rerunning
   Git. Report a blocked result if the correction still fails.
6. Report the validated result to the user.

The Coordinator does not read the Commit Guide, run Git, inspect repository
state, choose candidate files, write messages, or perform a fallback commit. If
Codex subagents are unavailable, stop and explain that delegation is required.

## Delegation envelope

```text
Act as the sole Committer Subagent for this request.

Before running any Git command, read and follow:
<skill_path>/references/commit-guide.md

Inputs:
- project_path: <absolute project path>
- intent: <verbatim user request>
- commit_files: <optional explicit maximum scope; omit when absent>

Own all repository inspection, candidate grouping, commit creation, and
post-commit verification. Return exactly one JSON object with no Markdown or
surrounding text.

Result Contract

Success:
{"ok":true,"data":{"commits":[{"commitId":"<id>","commitMessage":"<message>","files":["<repo-relative path>"]}],"ignoredFiles":["<repo-relative path>"]}}

Failure after resolving the repository:
{"ok":false,"data":{"repoName":"<name>","branch":"<branch>","commits":[],"ignoredFiles":[]},"error":{"message":"<reason>","files":["<repo-relative path>"],"nextStep":"<smallest required action>"}}

Failure before resolving the repository:
{"ok":false,"data":null,"error":{"message":"<reason>","files":[],"nextStep":"<smallest required action>"}}

For partial success, populate the failure result's data.commits with every
created and verified commit.
```

## Result validation

- Accept only the three Result Contract shapes with no additional top-level
  fields.
- Require non-empty strings and repo-relative `files`; allow an empty
  `ignoredFiles` or failure `error.files` array.
- Require at least one commit on success. A no-change result is a failure.
- Require `commits[].files`, `ignoredFiles`, and `error.files` to be duplicate-free
  and mutually exclusive wherever those fields are present.
- Preserve the Subagent's `error.message` and `nextStep`; do not reinterpret
  repository facts.
