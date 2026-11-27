# HRS - Spec-Driven Development Workshop

This is the companion application for the two-day workshop **Spec-Driven Development with the AI Unified Process**.

You build a hotel reservation system (HRS) from a three-line vision to tested code with the **AI Unified Process**
and an AI agent (Claude Code recommended; Copilot CLI, Codex or others work too). On day 2 you reverse engineer an
existing system and harden the harness around the agent.

| Part | When                 | Topic                                                                 | Labs  | Slides                                   |
|------|----------------------|-----------------------------------------------------------------------|-------|------------------------------------------|
| 1    | Day 1, 10:00–13:00   | From vision to use cases                                              | 1–4   | [`slides/part-1.html`](slides/part-1.html) |
| 2    | Day 1, 14:00–17:00   | From use cases to code and tests                                      | 5–8   | [`slides/part-2.html`](slides/part-2.html) |
| 3    | Day 2, 10:00–13:00   | Reverse engineering, harness engineering, guardrails, hooks, Werkstatt | 9–12  | [`slides/part-3.html`](slides/part-3.html) |

Open [`slides/index.html`](slides/index.html) for the overview.

## Start Your Environment

Choose one. Both work equally well.

### Codespaces

1. Open [`github.com/martinelli-workshops/sdd-lab`](https://github.com/martinelli-workshops/sdd-lab)
2. **Code → Codespaces → Create codespace on main**
3. Java, Maven, the AI Unified Process plugins and Playwright are installed for you
4. You work on your own branch `workshop/<github-user>`

### Local

1. Install **JDK 25** and **Claude Code** (Copilot CLI or Codex also work)
2. `git clone` the repository
3. Open `claude` in the project and accept the AI Unified Process plugins it offers
4. Check: `./mvnw spring-boot:run`

## What Is in the Repository

- [`docs/vision.md`](docs/vision.md): the vision of the hotel owners. This is where part 1 starts.
- [`docs/architecture/stack.md`](docs/architecture/stack.md): which AI Unified Process layer plugin builds and tests
  each layer. Part 2 uses it.

## The Labs

### Part 1: From vision to use cases

1. `/requirements`: requirements catalog and glossary from the vision
2. `/use-case-diagram` and `/entity-model`
3. `/use-case-spec UC-###`: specify the use case for searching available rooms
4. `/spec-review` and `/open-questions`

### Part 2: From use cases to code and tests

5. `/use-case-implement UC-###`: Flyway migration, jOOQ repository, Spring service, Vaadin view
6. `/use-case-test UC-###`, `./mvnw verify`, `/use-case-coverage UC-###`
7. Change a business rule ("a stay is at most 14 nights") in the use case first, then implement, test and check the
   coverage again
8. Choose a track: `/business-process` → `/test-case BP-###` → `/test-case-implement TC-###`, or a second use case

### Part 3: Brownfield and the harness

Prepare: clone the `h2` branch of [`AI-Unified-Process/petclinic`](https://github.com/AI-Unified-Process/petclinic)
(`git clone -b h2 …`, embedded H2, no Docker) and run `./mvnw verify` once (jOOQ code generation, frontend build).

You can work on your own project in labs 9 and 11 instead of the reference projects: on your machine, on a branch of
its own, with `aiup-core` and the layer plugins of your stack. Check first that you may use the code with an AI agent.

9. Clone [`spring-projects/spring-petclinic`](https://github.com/spring-projects/spring-petclinic), or take one module
   of your own project: `/stack`, `/reverse-engineer` and `/spec-review`
10. In the AI Unified Process PetClinic: run the sensors (`./mvnw -q test -Dgroups=sensor`), then try to break the
    traceability, the ArchUnit rules and the status hooks
11. Back in HRS or in your own project: write an ADR with `/architecture-decision`, add the `archunit-junit5` dependency to the POM by hand,
    let the agent enforce the ADR with an ArchUnit rule, add a Stop hook
12. Start a Werkstatt run: approve a specified use case on a change branch and run
    `~/.aiup/marketplace/pipelines/werkstatt/run-local.sh UC-###` (needs `git`, `python3`, `jq`)

## AI Unified Process Plugins

The lab uses the AI Unified Process layer plugins from the
[AI Unified Process marketplace](https://github.com/ai-unified-process/marketplace). They are configured for the
project in `.claude/settings.json` (Claude Code) and `.github/copilot/settings.json` (Copilot CLI), so the agent
offers to install them when you open the repository:

| Plugin             | Layer                        | Skills                                                                                    |
|--------------------|------------------------------|-------------------------------------------------------------------------------------------|
| `aiup-core`        | Specification, orchestration | `/requirements`, `/use-case-diagram`, `/entity-model`, `/use-case-spec`, `/use-case-implement`, `/use-case-test`, `/use-case-coverage`, `/reverse-engineer`, ... |
| `aiup-flyway`      | Migration                    | `flyway-schema`                                                                           |
| `aiup-jooq`        | Persistence                  | `jooq-persistence`, `jooq-persistence-test`                                               |
| `aiup-spring-boot` | Service                      | `spring-service`, `spring-service-test`                                                   |
| `aiup-vaadin`      | UI, E2E                      | `vaadin-flow-ui`, `vaadin-flow-ui-test`, `vaadin-e2e-test`                                |

To install them manually in Claude Code:

```bash
claude plugin marketplace add ai-unified-process/marketplace
claude plugin install aiup-core@ai-unified-process-marketplace
claude plugin install aiup-flyway@ai-unified-process-marketplace
claude plugin install aiup-jooq@ai-unified-process-marketplace
claude plugin install aiup-spring-boot@ai-unified-process-marketplace
claude plugin install aiup-vaadin@ai-unified-process-marketplace
```

The combined stack plugin `aiup-vaadin-jooq` is not used in this lab; disable it if you have it installed.

## Tech Stack

- **Java 25** with **Spring Boot 4.1**
- **Vaadin 25** for the UI (Java-based views)
- **jOOQ** for type-safe SQL and data access
- **Flyway** for database migrations
- **H2** as the database (embedded, no external setup required)
- **Vaadin Browserless Testing** for server-side UI tests
- **Playwright** with **Drama Finder** for browser-based end-to-end tests

## Running the Application

The application uses an embedded H2 database — no Docker or external database setup required.

```bash
./mvnw spring-boot:run
```

Or run `HrsApplication.main()` directly from your IDE.

## Build and Code Generation

The Maven build uses a two-step pipeline during `generate-sources`:

1. **Flyway** runs database migrations against a file-based H2 database
2. **jOOQ** generates type-safe Java code from the H2 schema

```bash
./mvnw compile
```

## Running Tests

Unit tests (Browserless) are run by **Surefire** with `mvnw test`. Integration tests (Playwright) use the `*IT` suffix
and are run by the **Failsafe** plugin, which is bound to the `integration-test` and `verify` phases.

**Unit tests only:**

```bash
./mvnw test
```

**All tests (unit + integration):**

```bash
./mvnw verify
```

The `verify` phase ensures that the Failsafe plugin picks up all `*IT` classes (e.g. Drama Finder `AbstractBasePlaywrightIT` subclasses)
and reports their results correctly. Always use `verify` instead of `integration-test` directly, so that test
failures are properly detected.
