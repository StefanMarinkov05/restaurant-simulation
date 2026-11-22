# Restaurant Simulation ("Saar la Carte")

A tick-based simulation of a group of restaurants over several evenings: reservations, seating and table
merging, waiters, kitchens with cooks and a pantry, delivery drivers, regular and casual customer groups,
events, incidents and ratings. Input is three JSON files (food, restaurants, scenario), output is a log of
everything that happens.

Group project of the **Software Engineering Lab (SoPra) 2026, Saarland University**: 5 students, design
phase plus 18-day implementation phase, Kotlin.

## Results (course grading server, final commit)

| Check | Result | Required |
|---|---|---|
| Mandatory system tests (validation 25/25, component 57/60, full 15/15) | **97 / 100** | 97 |
| Mutants caught by our own tests | **38 / 40** | 35 |
| Our system tests valid against the reference implementation | 433 / 436 | |
| Unit and integration tests (local) | 840, all passing | |

The course hides 40 deliberately broken versions of the reference implementation ("mutants"); a group's
system tests have to detect them.

## Architecture

```mermaid
flowchart LR
    A[CLI args] --> B[Load config<br/>food, restaurants, scenario]
    B -->|schema + semantic validation| C{valid?}
    C -->|no| X[Configuration rejected]
    C -->|yes| D[Build simulator]
    D --> E[Evening setup<br/>incidents, reservations,<br/>planning, procurement]
    E --> F[Tick 1..24]
    F --> F
    F -->|evening over, ticks left| E
    F -->|maxTicks reached| R[Statistics report]
```

Each tick runs six phases per open restaurant, in ascending restaurant id: arrivals, seating and ordering;
kitchen; serving; delivering; eating; escorting.

| Package | Responsibility |
|---|---|
| `config` | three-stage parsing (JSON schema, entities, cross-file rules), `Validator` |
| `simulation` | `Simulator`, clock, evenings and ticks, ratings |
| `foh` | front of the house: tables, waiters, reservations, delivery desk and drivers; `visit` holds the customer-visit state machine |
| `kitchen` | cooks, cook roster, pantry and supplier |
| `sharedPackage` | domain model: customer groups (regular, casual, event), orders, meals, food preferences |
| `incident` | staff, recipe, packaging and availability incidents |
| `logging` | every log message of the specification, with levels |

Design documents (PlantUML source and PDF) are in [`design/`](design): class diagram, simulation state
machine, parser sequence, first-tick preparation sequence and one scenario sequence.

## Team and contributions

| Member | Implementation | Testing |
|---|---|---|
| Ognjen Stojicic | simulation core, CLI and config loading, customer-visit state machine, seating, ordering, serving, escorting, event seating | features of Biborka and Constantin; many system tests (delivery, incidents, kitchen) |
| Teodor Vasilev | logging, food parsing, tables and merging, waiter assignment, incidents, event customers | features of Ognjen and Constantin; most mutant-killing system tests |
| **Stefan Marinkov** | **scenario parsing (customers, incidents), cross-file `Validator`, customer-group model (regular, casual), customer decisions, food preferences, waiting and patience clocks, ratings** | **features of Ognjen and Teodor; system-test registration and a large share of the system tests** |
| Biborka Fancsali | restaurant parsing, supplier, pantry, cooking | features of Teodor and Stefan |
| Constantin Hartmann | statistics, menu, browsing, delivery | features of Biborka |

Each feature had one implementer and two testers who were not the implementer.

### My part (Stefan)

- **Parsing and validation:** `ScenarioParser`, `CustomerGroupSerialiser`, `IncidentSerialiser` and the DTOs;
  `Validator`, the cross-file validation every parser uses.
- **Customer model:** `CustomerGroup` hierarchy (`RegularCustomerGroup`, `CasualCustomerGroup`), `Customer`,
  `History`, `FoodPreference`, the decision which restaurant to visit, the waiting, eating and give-up clocks,
  and `ratingFor`.
- **Testing:** unit tests for logging, simulator, tables, waiters, ordering, serving, incidents; wrote
  `SystemTestRegistration` and about 28% of the system-test code (validation, delivery, kitchen, incidents).
- **Bugs found and fixed while testing:** tick log format, delivery rating and eating lifecycle, ingredient
  packaging calculation, a patience-window off-by-one, delivery-order timing.

## Build and run

Requires JDK 21.

```sh
./gradlew build            # compile, detekt, unit + integration tests
java -jar libs/selab.jar \
  --food src/systemtest/resources/example/food.json \
  --restaurants src/systemtest/resources/example/restaurants.json \
  --scenario src/systemtest/resources/example/scenario.json \
  --maxTicks 30 --logLevel IMPORTANT
```

The system tests (`src/systemtest`) need the course's system-test API, which is only available on the
course server; see the comment in `build.gradle.kts`. Everything else builds without it.

## Stack

Kotlin 2.4 (JVM 21), Gradle, kotlinx.serialization, json-sKema (JSON schema validation), kotlinx-cli,
JUnit 5 and Mockito, JaCoCo, detekt (strict, no suppressions allowed).

## Credits and notes

- The specification, the JSON schemas, the build setup and the system-test API were provided by the
  Software Engineering Chair, Saarland University. The specification is not included here.
- This repository is a single snapshot of the group repository; per-author history stays in the course
  GitLab. Contributions are listed above as reported to the course.
- Generative AI use was declared per member to the course. I used Claude Code to scan the course forum for
  clarifications, write and debug tests, and run the build tooling; every change was reviewed before commit.
