# When a Modular Monolith Is Enough

> Status: outline. The scenario and section notes below are a starting point for the article.

A growing application needs clearer ownership and fewer changes that ripple across unrelated features. Before introducing independent services, we need to decide whether explicit boundaries inside one deployment solve the problem.

**Central question:** Do we actually need independent services, or do we only need strong boundaries?

## The problem

Scenario to develop:

- One team maintains a Todo application backed by PostgreSQL.
- Account management owns user details; Todo management owns todos and their lifecycle.
- A user cannot have more than three active todos.
- Features increasingly reach into each other's tables and implementation details.
- There is no current requirement for independent deployment or scaling.

Explain which pain comes from coupling and which, if any, comes from sharing a deployment.

## The simple solution

Start with one application and one database. Introduce explicit module interfaces, keep implementation details private, and assign ownership of tables and migrations.

Points to develop:

- Contrast a tightly coupled monolith with a modular monolith.
- Keep business rules close to the data and behavior they govern.
- Access another module through its public interface instead of its repositories or tables.
- Explain why folders alone do not enforce boundaries.

## When the pattern helps

- The team needs clearer ownership while retaining a shared release process.
- Business capabilities can be separated without network communication.
- Operational simplicity matters more than independent deployment.

Discuss how bounded contexts can coexist in one deployment, without assuming that every module represents a bounded context.

## When it becomes unnecessary complexity

- A small CRUD application may need only a few clear components.
- Interfaces and abstractions should protect meaningful boundaries.
- Adding brokers, generic event buses, or separate databases needs its own justification.

## Example

Planned example: an account module and a Todo module inside one application, with explicit public interfaces and table ownership in a shared PostgreSQL database.

Use a small boundary diagram and, if useful, a short TypeScript example. Show where the three-active-todos rule belongs. If the example includes creation logic, address concurrent requests rather than relying on an unprotected count-then-insert check.

## Trade-offs

### Benefits

- One deployment and simpler local development.
- In-process communication between modules.
- Explicit ownership without operating multiple services.

### Costs

- Releases and application scaling remain shared.
- Failures can affect the whole application.
- Module boundaries require discipline and automated enforcement.
- A shared database makes accidental cross-module coupling easy.

## Decision

Working decision: start with a modular monolith because the current problem is coupling within one team's application. Make module interfaces and database ownership explicit.

Revisit the decision when independent deployments, different scaling needs, stronger failure isolation, or separate team ownership create a concrete need. Explain why extracting a service also adds network failures, data coordination, and operational work.

## Key takeaway

Strong boundaries do not require separate services. Choose the deployment model from the application's constraints and make the cost of that choice explicit.

---

[Back to the chapter index](../README.md#chapters)
