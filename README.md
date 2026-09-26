# Software Architecture Notes

Short, practical notes about software architecture, Domain-Driven Design, CQRS, and distributed systems.

The focus is on the decisions behind architectural patterns: what problem they solve, when they introduce unnecessary complexity, and which trade-offs they bring.

Written for experienced developers, technical leads, architects, and engineering managers who need to make and explain architectural decisions.

## Approach

Each article starts with a concrete problem, considers the simplest viable solution, and ends with a decision supported by explicit constraints and trade-offs.

```text
Context → Problem → Constraints → Options → Trade-offs → Decision
```

Examples stay small and use TypeScript, with NestJS and PostgreSQL only where they help explain the decision. The recurring domain is a Todo application with one business rule: **a user cannot have more than three active todos**.

## Chapters

No articles are published yet. The first chapter has an initial outline; the remaining chapters are planned.

| # | Chapter | Central question | Status |
| --- | --- | --- | --- |
| 1 | [When a Modular Monolith Is Enough](articles/modular-monolith.md) | Do we need independent services, or strong module boundaries? | Outline |
| 2 | When CQRS Is Overkill | Does the read/write model justify the added complexity? | Planned |
| 3 | Domain Events Are Not Integration Events | Should an internal domain concept become a public contract? | Planned |
| 4 | Why Saving Data and Publishing an Event Is Hard | How do we connect a database transaction with reliable messaging? | Planned |
| 5 | Bounded Contexts Are More Than Folders | Is the model boundary visible and enforceable in code? | Planned |
| 6 | Your Domain Should Not Know About HTTP | Would the domain still make sense without HTTP? | Planned |
| 7 | Testing Architecture, Not Just Business Logic | How do we prevent architectural boundaries from eroding? | Planned |
| 8 | Where Should a Transaction End? | Which operations need to be immediately consistent? | Planned |
| 9 | Microservices Are an Operational Decision Too | Which benefits justify the operational cost of distribution? | Planned |

## Repository layout

```text
articles/
└── modular-monolith.md    First chapter outline
examples/                 Reserved for small, standalone examples
diagrams/                 Reserved for diagrams that need separate files
```

Chapters are developed incrementally. Code and diagrams will be added when they clarify a specific architectural decision.
