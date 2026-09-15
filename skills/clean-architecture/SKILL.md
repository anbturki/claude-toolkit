---
name: clean-architecture
description: Audit and enforce architecture-level boundaries - the Dependency Rule, layer separation, ports and adapters, and dependency inversion at module boundaries. Catches reverse imports (a lower layer importing a higher one), framework leakage into the domain, and business logic coupled to IO. Use when reviewing module structure, layering, or whether the domain is isolated from frameworks and IO. Operates above clean-code (lines/functions) and SOLID (classes) - this is the system level.
user-invocable: true
allowed-tools: Read, Edit, Grep, Glob, Bash(grep *), Bash(rg *), Bash(find *)
argument-hint: "[file path, layer name, or 'full' for the whole repo]"
---

# Clean Architecture

Structure above the class level. Where SOLID governs how a class depends on another, Clean Architecture governs how a *layer* depends on another. The single load-bearing rule:

> **The Dependency Rule:** source-code dependencies point only inward. An inner layer must never know anything about an outer layer. Nothing in an inner circle can name anything in an outer circle.

The circles, inner to outer: **Entities** (enterprise business rules) -> **Use Cases** (application business rules) -> **Interface Adapters** (controllers, presenters, gateways) -> **Frameworks & Drivers** (web, DB, UI, external services).

## Scope

`$ARGUMENTS` - a file, a layer directory, or `full`. If empty, default to the working-tree changes.

## Step 1: Discover the layer map

Projects name layers differently. Find the convention before judging violations. Common shapes:

| Style | Inner -> outer |
|---|---|
| Onion / Hexagonal | `domain` -> `application` -> `infrastructure` / `adapters` |
| Uncle Bob layers | `entities` -> `usecases` -> `adapters` / `controllers` -> `frameworks` |
| Feature-first | `core`/`domain` -> `features` -> `app`/`pages` |
| DDD | `domain` -> `application` -> `infrastructure` / `interfaces` |

```bash
# What top-level layers exist?
find <scope> -maxdepth 3 -type d | grep -iE 'domain|core|entit|usecase|application|service|adapter|infra|controller|presentation|ui|web|api' | head -40

# Read the project's own statement of layering, if any
rg -n -i 'layer|architecture|dependency rule|boundary' CLAUDE.md README.md docs/ 2>/dev/null | head
```

If the project documents a layer order, that is the source of truth. If it doesn't, infer the inner->outer order from the directory names above and **state the assumption in your report** before flagging anything.

## Step 2: Detect Dependency Rule violations (the core check)

A reverse import - an inner layer importing from an outer layer - is the primary finding. Grep for each layer importing anything more outer than itself.

```bash
# Example (Onion): domain must not import application or infrastructure
rg -n "from ['\"].*(application|infrastructure|adapters)" <scope>/domain
rg -n "import .*(application|infrastructure|adapters)"      <scope>/domain

# Example: application must not import infrastructure / framework concretions
rg -n "from ['\"].*(infrastructure|adapters|express|fastify|drizzle|prisma|axios)" <scope>/application

# Python equivalent
rg -n "^(from|import) .*(infrastructure|adapters|frameworks)" <scope>/domain
```

Each match is a finding. The inner layer should depend on an **abstraction it owns** (an interface/port), and the outer layer should implement it.

## Step 3: Detect framework / IO leakage into the core

The domain and use-case layers must be plain - no framework, no IO, no DB, no HTTP. Grep the inner layers for outer-world imports:

```bash
# Inner layers must not reference frameworks, ORMs, HTTP clients, or runtime IO
rg -n -i 'express|fastify|elysia|next|react|drizzle|prisma|mongoose|sequelize|axios|fetch\(|fs\.|process\.env' <scope>/domain <scope>/usecases 2>/dev/null
```

Findings here mean business rules are coupled to a delivery mechanism - they can't be tested or reused without booting the framework.

| Smell | Why it's wrong | Fix |
|---|---|---|
| Entity imports the ORM / `@Entity` decorators from the DB library | Domain now can't exist without the DB | Keep entities as plain types; map to DB rows in the infrastructure layer |
| Use case calls `fetch()` / `axios` / `db.query` directly | Business rule coupled to IO; untestable without network/DB | Define a port (interface) in the use-case layer; inject an adapter |
| Domain reads `process.env` | Config (an outer concern) leaks inward | Pass config in at the boundary |
| Controller contains business rules | Logic trapped in the delivery layer, not reusable | Move the rule into a use case; controller only translates IO |

## Step 4: Ports & Adapters (dependency inversion at the boundary)

The mechanism that lets an inner layer "use" an outer resource without depending on it:

- The **inner layer defines the interface** (the *port*) - e.g. `interface UserRepository` lives in `domain`/`application`.
- The **outer layer implements it** (the *adapter*) - e.g. `DrizzleUserRepository` lives in `infrastructure`.
- Wiring happens at the **composition root** (main / bootstrap), the only place allowed to know all concretions.

Check for the anti-pattern: a use case `new`-ing its own infrastructure.

```bash
# Use cases / services instantiating concretions instead of receiving ports
rg -n 'new [A-Z][A-Za-z]*(Repository|Client|Service|Gateway|Db|Store)\(' <scope>/usecases <scope>/application <scope>/domain 2>/dev/null
```

Each hit means the dependency points outward and should be inverted: depend on the port, inject the adapter.

## Step 5: Component coupling - no dependency cycles (ADP)

The **Acyclic Dependencies Principle**: the dependency graph between modules/packages must be a DAG. A cycle (`A -> B -> C -> A`) fuses the three into one un-releasable, un-testable unit - you can't reason about, build, or deploy any of them in isolation.

```bash
# TS/JS: detect import cycles with the project's tooling if present
npx madge --circular <scope> 2>/dev/null || npx dpdm --no-warning --no-tree -T <scope>/**/*.ts 2>/dev/null

# Language-agnostic first pass: find mutually-importing file pairs by hand
rg -n "import .*B" <scope>/A* ; rg -n "import .*A" <scope>/B*   # adapt names to suspected pair

# Python
pip show pydeps >/dev/null 2>&1 && pydeps <scope> --show-cycles 2>/dev/null
```

Each cycle is a must-fix. Break it by: (a) extracting the shared piece into a new component both depend on, or (b) inverting one edge with a port (Dependency Inversion) so the arrow flips.

## Step 6: Component coupling - stability direction (SDP + SAP)

- **Stable Dependencies Principle (SDP):** depend in the direction of stability. A component should only depend on components more stable (harder to change) than itself. Instability `I = fan-out / (fan-in + fan-out)`; `I=0` is maximally stable (many depend on it, it depends on nothing), `I=1` is maximally unstable. Dependencies must point toward lower `I`. A volatile module that many stable modules depend on is a structural hazard.
- **Stable Abstractions Principle (SAP):** a component should be as abstract as it is stable. Stable components (`I≈0`) must be abstract (interfaces/policies) so they can be extended without modification - this is OCP at the component scale. A stable *and* concrete component is rigid (the "zone of pain"); an unstable *and* abstract one is useless (the "zone of uselessness").

What to flag: a frequently-changing module (config, a specific vendor adapter, UI glue) that many other modules import directly. Fix by inverting onto an abstraction the stable side owns.

## Step 7: Component cohesion - what belongs in a module (REP/CCP/CRP)

When deciding which classes share a component, three principles tension against each other:

| Principle | Rule | Smell when violated |
|---|---|---|
| **REP** - Reuse/Release Equivalence | The unit of reuse is the unit of release; group what's reused together and versioned together | A "grab-bag" package where consumers want one class but must take twenty |
| **CCP** - Common Closure | Group classes that change for the same reason at the same time (SRP for components) | One edit forces recompiling/redeploying ten unrelated packages |
| **CRP** - Common Reuse | Don't force a consumer to depend on things it doesn't use; classes used together belong together | A module drags in transitive deps for code the consumer never calls |

These pull in different directions (REP+CCP push packages larger; CRP pushes smaller). Early projects lean CCP (developability); mature ones lean REP/CRP (reusability). Flag the extreme cases, not the judgment calls.

## Step 8: Boundaries and the Humble Object pattern

Architectural boundaries exist to isolate volatile, hard-to-test code from stable policy. At each boundary, split behavior into:

- A **Humble Object** - the thin, hard-to-test edge that only touches the framework/IO (the view, the controller adapter, the DB gateway impl). Almost no logic.
- A **testable core** - the presenter/use-case/entity that holds the logic and is pure.

Check: is there logic trapped inside a hard-to-test shell (a React component with business rules in it, a SQL gateway computing domain decisions, a controller validating business invariants)? That logic should move across the boundary into a plain, unit-testable object, leaving the shell humble.

## Step 9: Secondary architecture smells

- **Screaming architecture.** Top-level folders should name the *domain* (`billing`, `scheduling`, `accounts`), not the *framework* (`controllers`, `services`, `models` as the only organizing axis). If the structure screams "Rails" or "Express" instead of what the app does, note it.
- **Stable-dependency direction.** Volatile things (UI, DB, frameworks) depend on stable things (policies, entities), never the reverse.
- **Boundaries match change rate.** Things that change together live together; things that change for different reasons are separated by a boundary.
- **No god-module the whole app imports.** A single `utils`/`shared` that every layer reaches into quietly couples everything; it defeats the boundaries.

## What NOT to do

- **Don't impose layers on a small project that doesn't need them.** A 3-file script doesn't need entities/usecases/adapters. Architecture is a cost; apply it where the domain is large or long-lived. Flag missing layering only when the coupling is actually biting.
- **Don't add a port with a single adapter and no second one in sight** unless it's crossing an IO boundary (DB, network, FS) - those invert for testability even with one impl. A pure in-memory abstraction with one implementation is premature.
- **Don't rename folders to textbook layer names** just to match the diagram; follow the project's existing convention.
- **Don't flag a framework import in the framework layer** - that's where it belongs. Leakage is only a finding when it's in an *inner* layer.

## Severity rubric

| Severity | Findings |
|---|---|
| **must-fix** | Dependency Rule violation (inner imports outer); dependency cycle between modules; business logic that cannot be tested without a framework/DB booted |
| **should-fix** | Framework/IO leak into an inner layer; use case `new`-ing its own infrastructure instead of receiving a port; logic trapped in a humble shell; a volatile module many stable modules depend on |
| **nice-to-have** | Screaming-architecture naming; cohesion (REP/CCP/CRP) imbalance; a `shared`/`utils` god-module; missing layering on code that isn't yet hurting |

Calibrate to project size. On a small or short-lived codebase, most of the above drop a severity level - architecture is a cost you pay against change rate and lifespan, not a checklist to maximize.

## Output (when used as a check)

```
Layer map (assumed): <inner> -> ... -> <outer>   [source: CLAUDE.md | inferred]
Checks run: dependency-rule | cycles | framework-leak | ports | cohesion | stability

Findings:
N. [must-fix | should-fix | nice-to-have] <file:line>
   Evidence: <the import line / grep output / cycle path>
   Finding: <which principle, which boundary, which direction>
   Recommendation: <invert via port, break cycle by extraction, move logic, etc.>

(If none: "Dependency Rule respected, no cycles. Verified by: <commands run>.")
```

## See also

- [[clean-code-srp]] - SRP at the class/function level; Clean Architecture is SRP applied to layers (one reason to change per boundary)
- [[deep-audit]] - covers SOLID at the class level (Dimension 1b) and delegates to this skill for the full boundary audit (Dimension 1c)
- [[system-design]] - design the boundaries and ports up front, before the code exists
- [[full-audit]] - spawns this skill as the architecture lane

**Source:** Robert C. Martin, *Clean Architecture* (2017): Part III (Component Principles - REP/CCP/CRP cohesion; ADP/SDP/SAP coupling), Part IV (Architecture - the Dependency Rule, boundaries, policy vs detail), Part V (the Clean Architecture diagram, Screaming Architecture, the Humble Object pattern, Main as composition root).
