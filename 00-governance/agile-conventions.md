# Agile Conventions

> This document proposes a lightweight way to plan and review documentation work for the Simple Stock Flow challenge. The challenge README defines the deliverables and their dependency order. Team-specific methods marked **[assumption]** should be confirmed before being treated as agreed policy.

## 1. Purpose

Organize documentation tasks so that each change has a clear outcome, source, review path, and submission status.

The challenge evaluates how the supplied data model is interpreted and applied in the documents. The amount of text is not the goal; clarity and traceability are.

## 2. Work items

Track work at the level of a document or a small group of related documents. Each item should include:

- **Objective:** what the document change will clarify or deliver.
- **Target:** exact file path and name.
- **Source:** relevant sections, decisions, access patterns, or findings from `spec/data-model.md`.
- **Dependencies:** documents or decisions that must be available first.
- **Acceptance conditions:** what a reviewer should be able to confirm.
- **Open questions:** anything that cannot be resolved from the supplied source.

Do not create a work item that requires treating an unsupported assumption as a fact. Record missing information as an open question or mark an inference **[assumption]**.

## 3. Documentation workflow

Follow the challenge's stated reconstruction order, adjusting only when a dependency or deadline requires it:

1. Complete or review `05-architecture/`.
2. Write `04-requirements/`.
3. Write `03-product/`.
4. Write `02-domain/`.
5. Write `01-context/`.
6. Return to `05-architecture/consistency-check.md` and complete the cross-document check.

The challenge README defines the required files in each folder. Do not write a replacement for the supplied data-model source.

## 4. Prioritization

Prioritize work that unblocks dependent documents or resolves inconsistencies. Keep known model contradictions visible while working; do not wait for unavailable source documents if the supplied model and consistency findings allow the work to proceed with a clearly marked assumption.

**[Assumption]** If several independent documents are ready, work on the highest-priority or most deadline-sensitive item first. The challenge does not prescribe a planning tool or a prioritization framework.

## 5. Review and collaboration

Before marking a document ready for submission:

- Check its claims against the cited model sections.
- Confirm that inferences are marked **[assumption]**.
- Check terminology and cross-document consistency.
- Record unresolved owner decisions rather than silently resolving them.
- Use `definition-of-ready.md` and `definition-of-done.md` as checklists.

Reviewer, approval roles, review cadence, and meeting schedule: **[TODO: decide with the team if needed]**.

## 6. Estimation and iterations

The challenge does not define story points, sprint length, estimation method, or a backlog tool.

- Estimation method: **[TODO]**.
- Iteration or sprint duration: **[TODO]**.
- Planning tool: **[TODO]**.

Do not add estimates or dates to challenge documents as if they came from the data model.

## 7. Delivery and progress

- Save and review the document before staging it.
- Stage only the intended files and check `git status`.
- Commit with a message that describes the documentation change.
- Push to the team's fork and verify the commit appears there before the challenge deadline.

The last commit pushed before the deadline stated in the challenge README is the submission.