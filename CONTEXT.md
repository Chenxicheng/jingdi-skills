# Skill Repository

This repository develops predictable, self-contained Skills and their supporting resources.

## Language

**Coordinator**:
The main agent that starts a workflow, delegates bounded work, validates the returned contract, and reports the result.
_Avoid_: Master agent, orchestrator

**Commit Guide**:
The subagent-only operational reference for atomic commits. The Coordinator passes its path without loading its contents.
_Avoid_: Agent instructions, embedded prompt

**Skill Metadata**:
The Codex-facing `agents/openai.yaml` interface and invocation configuration for a Skill.
_Avoid_: Agent definition, subagent configuration

**Commit Batch**:
An ordered set of atomic candidate groups committed sequentially. Successful commits remain when a later group fails.
_Avoid_: Transaction, all-or-nothing commit set

**Result Contract**:
The strict JSON contract returned by the Subagent after self-verification and schema-checked by the Coordinator.
_Avoid_: Commit summary, agent response

**Delegation Envelope**:
The minimal task-local input passed to a fresh Subagent without the Coordinator's conversation history.
_Avoid_: Forked context, full handoff

**Issue Reference**:
A non-closing relationship stating that a commit is related to an explicitly identified Issue.
_Avoid_: Related Issue, Issue Link

**Issue Closure**:
A completion relationship stating that a commit resolves an explicitly identified Issue and authorizes automatic closure.
_Avoid_: Closing Reference, Fix Link
