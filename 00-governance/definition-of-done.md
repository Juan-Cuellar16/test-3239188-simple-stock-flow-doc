# Definition of Done

> This checklist is adapted for the Simple Stock Flow documentation challenge. It defines when a documentation task is complete; it does not add implementation requirements to the challenge.

## 1. Purpose

A document is Done when it is complete enough to review, traceable to the supplied source, and consistent with the other challenge documents.

The challenge README defines the required deliverables and submission deadline. The supplied `spec/data-model.md` is the source for reconstructing system documentation.

## 2. Document completion

- [ ] The file is in the required folder and uses the required filename.
- [ ] The document is written in English.
- [ ] Its purpose and scope are clear.
- [ ] Material claims about the system cite the relevant section, decision, access pattern, or finding from the data model.
- [ ] Claims not established by the model are marked **[assumption]** or recorded as open questions.
- [ ] Database-enforced, domain-only, and pending rules are distinguished accurately.
- [ ] Relevant model contradictions remain visible; the document does not silently present a disputed interpretation as fact.
- [ ] The document does not add unsupported users, technologies, interfaces, security controls, or service-level targets.
- [ ] Links, tables, headings, and Markdown formatting are readable.
- [ ] Cross-references point to the correct files.
- [ ] The supplied data model and unrelated files were not changed.

## 3. Consistency review

- [ ] Terms agree with the context and domain glossaries.
- [ ] Requirements and product statements agree with the traceability matrix and user stories.
- [ ] Architecture statements agree with the architecture overview and consistency check.
- [ ] Exclusions and unresolved decisions are represented consistently across affected documents.
- [ ] Any remaining discrepancy is documented with its source and owner, if known.

## 4. Review

- [ ] The author has reread the final document.
- [ ] Important claims have been checked against their cited source.
- [ ] Any review feedback has been resolved or recorded as an open issue.
- [ ] Reviewer or approver: **[TODO: identify if the team requires one]**.

The challenge does not prescribe a formal review board or reviewer role.

## 5. Submission

- [ ] `git status` confirms that only intended files are staged for the commit.
- [ ] The commit message describes the documentation change.
- [ ] The commit is pushed to the team's fork, with `origin` verified as the fork remote.
- [ ] The pushed commit is visible on GitHub before the deadline stated in the challenge README.
- [ ] The final repository state and recent commit history have been checked.

A local commit that has not been pushed to the fork is not a submitted change under the challenge instructions.

## 6. Not required by this definition

This documentation task does not require implementation code, automated application tests, deployment, or changes to the supplied data model. The challenge evaluates the interpretation and application of the specification in the submitted documents.