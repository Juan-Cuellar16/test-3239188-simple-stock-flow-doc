# Documentation Rules

> These rules are adapted to the Simple Stock Flow documentation challenge. The challenge README defines the deliverables; the supplied data model is the source for reconstructing system documentation.

## 1. Purpose

Keep the project documents consistent, readable, and traceable to the source. Do not use this governance file to introduce requirements that are absent from the challenge or data model.

## 2. Language and format

- Write deliverable documents in English, as required by the challenge README.
- Use Markdown headings, lists, and tables to make documents easy to review.
- Use the folder names and filenames specified by the challenge.
- Keep terminology and identifiers consistent with `spec/data-model.md`.
- Use `**[assumption]**` to mark statements that are inferred and not established by the data model.

## 3. Traceability

- Support every material statement about the system with a reference to a section of `spec/data-model.md`, such as `§2.3`, `Q9 / §6.1`, or `FK-2`.
- Cite the relevant decision identifiers, task identifiers, or consistency findings when they appear in the model.
- Distinguish between:
  - Rules enforced by the database engine.
  - Rules enforced only by the domain.
  - Work marked as pending.
- Do not present a pending change as an implemented guarantee.
- If a statement cannot be traced to the supplied model, mark it as **[assumption]** or leave it as an open question.

## 4. Source authority and contradictions

The challenge README defines the task and required deliverables. The supplied `spec/data-model.md` is the input for reconstructing the system documentation.

Some linked source documents are not included in the challenge. Do not treat them as documents that were independently reviewed; use only the information about them included in `spec/data-model.md`.

If the model contradicts itself:

1. Identify the conflicting statements.
2. Consult the findings in `05-architecture/consistency-check.md` when applicable.
3. State the adopted interpretation and label any unresolved status.
4. Do not silently remove or rewrite a model contradiction.

Do not modify or replace the supplied `spec/data-model.md`.

## 5. Scope and accuracy

- Do not introduce unsupported users, technologies, interfaces, services, data, or business rules as facts.
- Do not add customer data, multi-currency support, seller-level report breakdowns, or Product attributes beyond the model's decided set (D-05, DP-02/03, model §§1 and 12).
- Do not infer a message broker, asynchronous event delivery, or a microservices architecture from ports, candidate event names, or folder structure alone (§6, §12).
- Keep open decisions visible and identify who needs to resolve them when known.
- Do not copy DistriLink-specific assumptions or requirements into Simple Stock Flow documents.

## 6. Cross-document consistency

When adding or changing a document:

- Check that its terminology agrees with `01-context/glossary.md` and `02-domain/glossary.md`.
- Check related requirements against `04-requirements/traceability-matrix.md`.
- Check architecture claims against `05-architecture/overview.md` and `05-architecture/consistency-check.md`.
- Preserve exclusions and unresolved conflicts in dependent documents.
- Update cross-references if a file is renamed or moved.

## 7. Challenge deliverables

The challenge README lists documents in `01-context` through `05-architecture` and states that `06-data` is the supplied input and is not to be written as a deliverable. Keep the provided data model as source material; do not create a competing data-model document.

The `00-governance` folder is supplemental project guidance. It does not replace or expand the required challenge deliverables.

## 8. Review checklist

Before submitting a document, check that:

- [ ] It uses the required path and filename.
- [ ] It is written in English.
- [ ] Material system claims have source references.
- [ ] Inferences are marked **[assumption]**.
- [ ] Pending and disputed model states are not presented as settled facts.
- [ ] Links and cross-references point to existing documents.
- [ ] The document agrees with related files, or the inconsistency is recorded.
- [ ] No unrelated files or the supplied data model were changed.

## 9. Changes to these rules

A change to this file should include the reason for the change and any affected documents. The challenge does not define a formal approval workflow; team approval requirements are **[TODO]**.