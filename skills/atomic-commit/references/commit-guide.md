# Committer Execution Guide

Act as the only Committer Subagent for the request. Own every Git command,
commit, and post-commit check. Do not ask the Coordinator to inspect or repair
Git state. The Coordinator performs no Git operation after delegation.

Accept `skill_path`, `project_path`, the original `intent`, optional
`commit_files`, and the Result Contract in the delegation prompt. Return only
JSON that exactly follows that contract.

## Boundaries

- Create one reviewable commit per purpose. Include every implementation, test,
  document, configuration, migration, and generated artifact required for that
  purpose.
- Treat `commit_files` as the maximum allowed scope, not as permission to omit a
  dependency. Block when a complete commit requires a path outside that scope.
- Block hunk-level splitting, mixed purposes in one file, missing dependencies,
  ambiguous ordering, secrets, credentials, private keys, tokens, `.env` files,
  and machine-local configuration.
- Block any binary, generated, or unusually large file whose safety and atomic
  role cannot be verified. Never sample content to bypass a safety or atomicity
  check.
- Do not push, amend, roll back commits, bypass hooks, change Git configuration,
  discard worktree changes, or run interactive Git commands.
- Do not run tests, lint, or builds by default. Run only the exact validation
  explicitly required by `intent`. Allow normal Git hooks to run.

## Phase 1: Preflight Every Group

Complete this phase for all candidate groups before changing the index or
creating any commit.

1. Resolve the repository from `project_path`:

   ```bash
   git rev-parse --show-toplevel
   git rev-parse --verify HEAD
   git branch --show-current
   git rev-parse --short HEAD
   ```

   Block an unborn `HEAD`. Derive `repoName` from the top-level directory name.
   Report a detached branch as `HEAD-detached:<short-hash>`.

2. Read repository status exactly once:

   ```bash
   git status --porcelain=v1 -z --untracked-files=all --no-renames
   ```

   Parse the NUL-delimited output as path arguments. Never split paths on spaces
   or newlines. Treat renames as delete/add. Resolve `commit_files` against
   repository-relative path components and reject paths outside the repository.

3. Block an unmerged index or an in-progress Git operation before grouping.
   Return only compact counts and operation names from these checks:

   ```bash
   set -o pipefail
   git ls-files -u -z | tr -cd '\0' | wc -c
   for marker in MERGE_HEAD CHERRY_PICK_HEAD REVERT_HEAD BISECT_START \
     BISECT_LOG rebase-merge rebase-apply sequencer; do
     marker_path=$(git rev-parse --git-path "$marker")
     test ! -e "$marker_path" || printf '%s\0' "$marker"
   done
   ```

   Treat any nonzero unmerged-entry count or emitted marker as blocking. Do not
   finish, abort, or otherwise modify that state. Put all unresolved or
   unclassified changed paths in `error.files` and direct the user to finish or
   abort the named operation before retrying.

4. Form candidate groups from the status, `intent`, and optional
   `commit_files`. Include every clear group only when the intent is broad, such
   as "commit" or "commit current work". Classify every changed path as a
   candidate-group member, an unrelated ignored file, or a blocking error file.

5. Read numstat only when filenames and intent do not provide enough information
   to form groups, or when performing the mandatory size and binary precheck:

   ```bash
   GIT_LITERAL_PATHSPECS=1 git diff HEAD --numstat -z --no-renames -- \
     "${tracked_files[@]}"
   ```

6. Before any complete tracked diff or untracked content enters model context,
   run a metadata-only size, type, and generation precheck. Use Git and standard
   shell tools so the model receives only byte counts, line counts, binary flags,
   file types, attributes, and producer/source relationships:

   ```bash
   set -o pipefail
   GIT_LITERAL_PATHSPECS=1 git diff HEAD --binary --full-index \
     --no-ext-diff --no-textconv -- "${tracked_files[@]}" | wc -c
   git check-attr -z diff generated linguist-generated -- "${candidate_files[@]}"
   ```

   Gate every untracked path with an `lstat`-based type check before opening it.
   Use runtime standard-library calls equivalent to `lstat` and `readlink`; do
   not use `stat`, `realpath`, or another operation that follows the path.

   - For a symbolic link, inspect and count only the link target returned by
     `readlink`. Never open, resolve, or inspect the target. Commit the link
     target itself; its patch fingerprint must include mode `120000`.
   - For a regular file, record its `lstat` byte size, then run `wc -c` and the
     `--no-index --numstat` binary check below. Recheck that it is still a regular
     file before reading its content.
   - Reject directories, FIFOs, sockets, block devices, character devices, and
     every other file type. Do not open them or attempt to commit them.

   ```bash
   wc -c < "$untracked_regular_file"
   git diff --no-index --numstat -- /dev/null "$untracked_regular_file"
   ```

   Treat exit status `1` from the untracked `--no-index` comparison as the
   expected "different" result, but reject other command failures. Inspect file
   types without dereferencing links. Block if the type changes during the gate.
   Use repository rules, attributes, path conventions, and visible
   producer/source relationships to identify generated artifacts.

   Admit at most 64 KiB of complete text diff or content per candidate group and
   128 KiB across the entire request. Count tracked patch bytes and untracked
   regular-file bytes or symbolic-link target bytes toward these limits. Do not
   put binary or generated content in model context. Continue with an over-limit,
   binary, or generated artifact only when its source is in the same group and a
   deterministic generator, checksum, object ID, or equivalent metadata proves
   its exact role and contents. Record that proof for post-commit verification.
   Otherwise block the whole request. Never sample, truncate, or partially decode
   a file to pass this gate.

7. Inspect each admitted candidate group with the smallest complete diff:

   ```bash
   GIT_LITERAL_PATHSPECS=1 git diff HEAD --unified=0 --no-renames -- "${tracked_files[@]}"
   ```

   Run this only when the group has tracked files. Inspect each admitted
   untracked text file in full. Expand context only around a specific ambiguous
   hunk when the zero-context diff cannot establish purpose, dependency, safety,
   or commit-message meaning. Do not reread the whole diff merely for more
   context.

8. Verify atomicity, completeness, sensitive-data safety, and dependency order
   for every group. Order dependent groups topologically; otherwise use stable
   path order. If any group fails, create no commits.

9. Run any validation explicitly required by `intent`. Finish every requested
   validation before creating a commit. If validation fails, create no commits.

## Phase 2: Commit Groups Sequentially

For each approved group, preserve its repository-relative paths as an array and
write a NUL-delimited literal pathspec:

```bash
work_dir=$(mktemp -d)
printf ':(literal)%s\0' "${candidate_files[@]}" > "$work_dir/candidate.pathspec"
```

Create an English imperative Conventional Commit subject of at most 72
characters:

```text
<type>(optional-scope): <imperative summary>
```

Use `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`,
`chore`, or `revert`. Read `<skill_path>/references/message-rules.md` only when
the intent or diff requires a body, issue footer, or breaking-change notation.

When the group contains untracked files, create a separate literal pathspec and
mark only those paths as intent-to-add:

```bash
printf ':(literal)%s\0' "${untracked_files[@]}" > "$work_dir/untracked.pathspec"
git add -N --pathspec-from-file="$work_dir/untracked.pathspec" --pathspec-file-nul
```

Immediately before committing, capture the current base commit and a canonical
fingerprint of the complete candidate patch. Enable pipeline failure detection
and reject a failed diff or hash command. The patch form includes full object
IDs, binary data, and mode changes without exposing its contents to the model:

```bash
set -o pipefail
base_commit=$(git rev-parse HEAD)
candidate_patch_id=$(
  GIT_LITERAL_PATHSPECS=1 git diff "$base_commit" --unified=0 --binary \
    --full-index --no-ext-diff --no-textconv -- "${candidate_files[@]}" |
    git hash-object --stdin
)
```

Commit only the candidate paths so unrelated staged changes remain untouched:

```bash
git commit --only -m "$subject" \
  --pathspec-from-file="$work_dir/candidate.pathspec" --pathspec-file-nul
```

Add separate `-m` arguments only for a required body or footer. If any step after
`git add -N` fails before the commit is created, reset only the temporary
intent-to-add paths:

```bash
git reset -q HEAD \
  --pathspec-from-file="$work_dir/untracked.pathspec" --pathspec-file-nul
```

Report that the index may have changed if this cleanup fails. Remove the
temporary directory after successful commit or successful cleanup.

After each commit, verify the committed file set and message:

```bash
git diff-tree --no-commit-id --name-only -r -z --no-renames HEAD
git show -s --format='%h%x00%B' HEAD
```

Hash the actual commit patch from the recorded base with the same canonical
options:

```bash
actual_patch_id=$(
  GIT_LITERAL_PATHSPECS=1 git diff "$base_commit" HEAD --unified=0 --binary \
    --full-index --no-ext-diff --no-textconv -- "${candidate_files[@]}" |
    git hash-object --stdin
)
test "$candidate_patch_id" = "$actual_patch_id"
```

After the fingerprints match, require a clean candidate scope relative to the
new `HEAD`. This is a required scoped post-commit read, not a second full-repository
status read:

```bash
GIT_LITERAL_PATHSPECS=1 git status --porcelain=v1 -z \
  --untracked-files=all --ignored=matching --no-renames -- \
  "${candidate_files[@]}"
```

Require empty output. Any tracked, staged, unstaged, untracked, or ignored
residual means a hook changed the worktree after Git constructed the commit's
temporary index. Treat the created commit as unverified: preserve it, put its ID
in `error.message`, put the candidate paths in `error.files`, and return the
partial-failure result without rollback or amend.

Require the actual file set to equal the candidate group exactly. Require all
natural-language subject, body, and footer text to be English. Require the patch
fingerprints and every recorded deterministic artifact proof to match. This
detects hooks that rewrite candidate content or file modes. Record the actual
commit ID, full message, and files only after all checks pass.

If a commit, hook, cleanup, or post-commit check fails, stop. Preserve every
commit already created; do not amend or roll it back. Include every previously
verified commit in the failure result, identify any unverified current paths as
error files, identify any created but unverified commit ID in `error.message`,
and state the smallest safe next step.

## Result Semantics

Follow the delegation prompt's Result Contract exactly; it is the sole source of
truth for JSON keys and shapes.

- Put each changed path in exactly one of `data.commits[*].files`,
  `data.ignoredFiles`, or `error.files`.
- Return ignored unrelated paths even on success.
- After repository resolution, include the resolved repository name, branch,
  completed commits, and ignored files in every failure result.
- Before repository resolution, use the contract's null data form.
- On partial success, put every verified completed commit in `data.commits` and
  put the failed or unverified paths in `error.files`.
- Return no Markdown, commentary, or text outside the JSON value.
