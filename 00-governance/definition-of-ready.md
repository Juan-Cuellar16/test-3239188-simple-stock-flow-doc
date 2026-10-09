# Definition of Ready

> This checklist is adapted for documentation work in the Simple Stock Flow challenge. It helps confirm that a document task is clear enough to begin; it does not add requirements to the challenge.

## 1. Purpose

A documentation task is Ready when its intended outcome, source material, and completion conditions are clear enough for someone to work on it without inventing missing system facts.

The challenge README defines the deliverables. The supplied `spec/data-model.md` is the source for reconstructing the system documentation.

## 2. General task readiness

Before starting a task, confirm that:

- [ ] The objective is stated in one or two clear sentences.
- [ ] The target document and exact repository path are identified.
- [ ] The expected change is described: create, update, review, or reconcile.
- [ ] Related documents that may be affected are identified.
- [ ] The required result can be reviewed in the repository.

## 3. Source readiness

- [ ] Relevant sections of `spec/data-model.md` are identified.
- [ ] Any relevant decision, access pattern, foreign-key policy, task, or consistency finding is noted.
- [ ] The source information is available in the supplied model; linked but undelivered documents are not required as additional sources.
- [ ] Contradictions or stale statements relevant to the task are listed.
- [ ] Missing information can be left as an open question instead of being invented.

## 4. Content readiness

- [ ] The document's audience and purpose are clear.
- [ ] Material claims can be cited to the supplied model.
- [ ] Inferences can be marked **[assumption]**.
- [ ] The task does not require adding unsupported users, technologies, interfaces, rules, or service-level targets.
- [ ] The expected language is understood: challenge deliverables are written in English.
- [ ] Any known exclusions or scope boundaries relevant to the document are understood.

## 5. Dependencies and decisions

- [ ] Dependencies on other documents are identified.
- [ ] Required owner decisions are available, or the document can proceed while recording them as unresolved.
- [ ] The task does not silently decide a conflict already documented in `05-architecture/consistency-check.md`.
- [ ] Any affected cross-references can be updated as part of the task.

## 6. Acceptance conditions

Before work begins, define what will count as complete. For a documentation task, acceptance conditions may include:

- The correct file exists at the required path.
- The content is in English and uses the agreed Markdown structure.
- Claims include model references, and unsupported inferences are marked **[assumption]**.
- Relevant uncertainties and conflicts remain visible.
- Related documents and links are consistent.

**[TODO: add any team-specific review or approval requirement.]**

## 7. Items not defined by the challenge

The challenge does not prescribe a planning tool, sprint assignment, story-point estimate, team reviewer, or formal approval board. These are not readiness blockers unless the team separately agrees that they are required.