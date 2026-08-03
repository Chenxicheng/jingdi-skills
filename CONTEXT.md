# Skill Repository

This repository develops predictable, self-contained Skills and their supporting resources.

## Language

**Coordinator**:
The main agent that starts a workflow, delegates bounded work, validates the returned contract, and reports the result.
_Avoid_: Master agent, orchestrator

**Committer Subagent**:
The Codex subagent that exclusively inspects repository state and creates and verifies atomic commits.
_Avoid_: Commit agent, Git worker

**Commit Guide**:
The subagent-only operational reference for atomic commits. The Coordinator passes its path without loading its contents.
_Avoid_: Agent instructions, committer prompt

**Skill Metadata**:
The Codex-facing `agents/openai.yaml` interface and invocation configuration for a Skill.
_Avoid_: Agent definition, subagent configuration

**Progressive Diff**:
The Committer Subagent's staged inspection of repository changes, expanding from paths to hunks only when a decision requires more context.
_Avoid_: Full diff, sampled diff

**Commit Batch**:
An ordered set of atomic candidate groups that passes complete preflight before its first commit. Successful commits remain when a later group fails.
_Avoid_: Transaction, all-or-nothing commit set

**Commit Result**:
The strict JSON contract returned by the Committer Subagent after self-verification and schema-checked by the Coordinator.
_Avoid_: Commit summary, agent response

**Delegation Envelope**:
The minimal task-local input passed to a fresh Committer Subagent without the Coordinator's conversation history.
_Avoid_: Forked context, full handoff
