# Microservices documentation applicability

> Current status: not applicable to the modeled system. This file is retained because it is part of the requested governance skeleton.

## 1. Current system boundary
The model and architecture overview describe one application/service with a PostgreSQL database and external image storage. They do not define independently deployed business microservices, a service catalog, or asynchronous messaging (model §6.1, §12; 05-architecture/overview.md).

## 2. Current documentation
- Use the challenge's repository-level documents in 01-context through 05-architecture.
- Keep spec/data-model.md as the supplied source.
- Do not create per-microservice folders or contracts unless an approved architecture introduces separate services.
- Do not infer a broker or asynchronous communication from candidate domain-event names.

## 3. Revisit trigger
Revisit this document if the architecture is formally split into independently deployed services. Define service boundaries, data ownership, communication contracts, operational ownership, and required service documents at that time.
