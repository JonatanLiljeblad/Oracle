# ORACLE

### Experimental Intelligence for Understanding Complex Software Systems

![Status: concept and planning](https://img.shields.io/badge/status-concept%20and%20planning-yellow)
![License: TBD](https://img.shields.io/badge/license-TBD-lightgrey)

> **ORACLE is a software-system laboratory designed to understand, observe, experiment with, and verify complex software systems.**

ORACLE goes beyond static code analysis and conventional AI assistants.

Instead of only reading source code or answering questions about a system, ORACLE is intended to **build a model of a running system, form hypotheses about its behavior, perform controlled experiments, analyze the results, and identify weaknesses or unexpected behavior.**

The long-term goal is to create an environment where software can be investigated almost like a scientific subject:

```text
Observe
   |
Understand
   |
Form Hypothesis
   |
Run Experiment
   |
Analyze
   |
Verify
   |
Explain / Improve
   |
Repeat
```

**Current status:** ORACLE is in the concept and planning stage. There is no application implementation, runnable environment, or test suite yet. The capabilities, architecture, technology choices, and directory layout below describe the intended direction, not existing functionality.

## Table of Contents

- [Why ORACLE?](#why-oracle)
- [Vision](#vision)
- [Goals](#goals)
- [Core Capabilities](#core-capabilities)
- [Example Investigation](#example-investigation)
- [Architecture](#architecture)
- [Technology](#technology)
- [Development Roadmap](#development-roadmap)
- [Proposed Project Structure](#proposed-project-structure)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Development and Testing](#development-and-testing)
- [Observability](#observability)
- [Design Principles](#design-principles)
- [Security](#security)
- [Research Direction](#research-direction)
- [Long-Term Goal](#long-term-goal)
- [Contributing](#contributing)
- [License](#license)

## Why ORACLE?

Modern software systems are increasingly difficult to understand. A production application may contain multiple services, databases, caches, asynchronous workers, queues, external APIs, distributed state, retries, timeouts, concurrent processes, and infrastructure dependencies.

A system can look correct in source code while behaving unexpectedly at runtime.

Traditional tools answer individual questions:

> What does this function do?
>
> Which service calls this API?
>
> What happened in this log?

ORACLE is designed around a broader question:

> **How does the system actually behave, and what happens when its assumptions are challenged?**

This makes ORACLE an experimental platform rather than simply another developer dashboard.

## Vision

The long-term vision is a **general-purpose laboratory for complex software systems**.

A user should eventually be able to provide an authorized software system and let ORACLE:

1. Discover its architecture.
2. Build a machine-readable model of the system.
3. Observe its runtime behavior.
4. Identify assumptions and potential failure modes.
5. Form testable hypotheses.
6. Create isolated experiments.
7. Inject controlled failures or environmental changes.
8. Compare predicted and observed behavior.
9. Verify important system properties within explicitly stated model assumptions.
10. Explain findings in a reproducible report.
11. Generate regression tests or improvement suggestions.

The central idea is simple:

> **Don't just analyze software. Experiment on it.**

## Goals

### Understand systems

Construct a useful representation of services, APIs, dependencies, databases, queues, processes, external integrations, data flow, and infrastructure relationships.

### Observe systems

Collect and correlate logs, metrics, distributed traces, events, system state, and performance characteristics.

### Experiment with systems

Run controlled experiments involving service failures, process termination, network latency, packet loss, resource constraints, configuration changes, dependency failures, and concurrency conditions.

### Investigate intelligently

Use AI-assisted reasoning to identify suspicious behavior, generate hypotheses, propose experiments, analyze results, connect evidence across sources, and prioritize investigations.

### Verify important properties

Allow critical system assumptions to be expressed as explicit invariants and checked using formal methods where appropriate.

### Make findings reproducible

Every meaningful investigation should produce a record containing:

```text
Experiment
|-- System version
|-- Configuration
|-- Hypothesis
|-- Initial state
|-- Workload and inputs
|-- Actions
|-- Observations
|-- Evidence
|-- Result
`-- Reproduction information
```

Reproducibility is a core principle of the project.

## Core Capabilities

All capabilities in this section are planned.

### System Discovery

ORACLE should analyze a target system and construct a machine-readable model of its architecture.

Potential discoveries include service-to-service communication, API and database dependencies, message queues, cache usage, execution paths, configuration dependencies, and infrastructure relationships.

The result should be a continuously evolving **system graph**, informed by source code, runtime data, and infrastructure definitions.

### Runtime Observability

ORACLE should correlate multiple forms of runtime evidence:

| Evidence | Examples |
| --- | --- |
| Logs | Timestamp, service, severity, message, metadata |
| Metrics | CPU, memory, latency, throughput, error rate, queue depth, database activity |
| Traces | Request paths across APIs, databases, caches, and workers |
| Events and state | Deployments, restarts, configuration changes, observed system state |

The objective is not simply to collect telemetry, but to connect it back to the system model.

### Experiment Engine

Experiments should be isolated, controlled, measurable, and reproducible. Each experiment should be a first-class object rather than an ad-hoc test.

Example investigations:

| Controlled change | Observation | Question |
| --- | --- | --- |
| Terminate a worker during processing | Retry behavior and duplicate work | Can a job execute twice? |
| Add 250 ms of network latency | Timeout frequency and cascading effects | How does latency propagate through the system? |
| Limit database connections to 10 | Queueing and failure thresholds under concurrent load | Where does capacity become a bottleneck? |

### Intelligent Investigation

The AI component is intended to function as an **investigator**, not simply a conversational interface.

Given enough evidence, it may propose a hypothesis such as:

```text
Prediction:
A worker restart during message processing
may cause the same job to execute twice.

Supporting evidence:
The worker acknowledges messages after processing,
and no deduplication mechanism has been observed.

Recommended experiment:
Terminate the worker during the processing
window and observe message semantics.
```

An authorized experiment could then test the prediction and update the investigation:

```text
Evidence -> Hypothesis -> Experiment -> Result
   ^                                      |
   `----------- New Evidence -------------'
```

AI-generated claims must remain distinguishable from observations and verified results.

### Verification

Not every important property can or should be learned statistically. Some properties are better expressed explicitly:

```text
Invariant:
A transaction must not be committed twice.
```

ORACLE should eventually help translate architectural assumptions into verification-oriented representations and use formal methods where practical.

A verification result should identify the model, assumptions, and scope of the check. A failure should include a counterexample when available:

```text
Invariant violated in the model.

Counterexample:
Begin -> Commit -> Failure -> Retry -> Duplicate commit
```

The goal is to connect **formal reasoning with real system behavior**. Passing a runtime experiment does not prove an invariant, and verifying a model does not by itself prove that the implementation matches it.

## Example Investigation

The following is an illustrative future workflow, not an existing feature.

Imagine ORACLE is connected to a distributed application:

```text
                 Frontend
                    |
                   API
                    |
         +----------+----------+
         |          |          |
      Postgres    Redis      Queue
                    |          |
                    +-----+----+
                          |
                        Worker
```

ORACLE notices that job processing appears to depend on retry behavior and proposes:

> A worker crash during processing may result in duplicate execution.

After authorization, it creates an isolated environment and:

1. Starts a workload.
2. Observes a job entering its processing window.
3. Terminates the worker.
4. Restarts the worker and allows message redelivery.
5. Collects traces and compares resulting state.

An example report might contain:

```text
Status:
REPRODUCED

Finding:
Duplicate job execution detected.

Affected components:
Worker / Queue

Evidence:
Matching execution traces and state changes
for the same job identifier.

Suggested next step:
Introduce an idempotency key.
```

The investigation should retain the system version, experiment definition, inputs, actions, and evidence so it can be reproduced later.

## Architecture

The proposed architecture keeps the core system modular:

```text
                         ORACLE PLATFORM
                               |
          +--------------------+--------------------+
          |                    |                    |
       Console          Investigation          Experiment
      / Frontend           Engine                Engine
                               |                    |
                     +---------+---------+          |
                     |                   |          |
                System Model       AI / Reasoning   |
                     |                              |
          +----------+-----------+                  |
          |          |           |                  |
      Telemetry     Code    Infrastructure          |
          |          |           |                  |
          +----------+-----------+------------------+
                                 |
                   Persistent Investigation State
                     PostgreSQL / Event Store
```

Individual components should be able to evolve independently. The system model connects source, infrastructure, and runtime evidence; investigation and experiment engines use that model and persist their findings.

## Technology

The stack is **proposed, not implemented or finalized**. The initial direction is:

| Layer | Candidate technology |
| --- | --- |
| Core systems | Rust |
| Backend APIs | FastAPI / Rust services |
| Frontend | Next.js + TypeScript |
| Database | PostgreSQL |
| Cache / coordination | Redis |
| Event streaming | Kafka / Redpanda |
| Observability | OpenTelemetry |
| AI / ML | Python, PyTorch where needed |
| Formal methods | TLA+ |
| Containers | Docker |
| Infrastructure | Kubernetes / cloud environments |
| Testing | Unit, integration, property-based, and experiment-based testing |

Technology choices should follow engineering requirements rather than novelty. Not every candidate needs to be introduced in the first implementation.

## Development Roadmap

The roadmap describes the intended progression, not a release schedule.

### Phase 1 - System Model

- [ ] Repository ingestion
- [ ] Service discovery
- [ ] Dependency graph
- [ ] API mapping
- [ ] Database relationship mapping
- [ ] Initial architecture visualization

### Phase 2 - Runtime Intelligence

- [ ] Log ingestion
- [ ] Metrics ingestion
- [ ] Distributed trace ingestion
- [ ] Correlation engine
- [ ] Runtime system graph

### Phase 3 - Experimentation

- [ ] Isolated experiment environments
- [ ] Failure injection
- [ ] Network fault simulation
- [ ] Resource constraints
- [ ] Experiment definitions
- [ ] Reproducible experiment runs

### Phase 4 - AI Investigation

- [ ] Hypothesis generation
- [ ] Evidence retrieval
- [ ] Experiment recommendation
- [ ] Result analysis
- [ ] Investigation memory
- [ ] Automated reports

### Phase 5 - Verification

- [ ] System invariants
- [ ] Formal models
- [ ] Counterexample reporting
- [ ] Verification workflow
- [ ] Runtime-to-formal-model connections

### Phase 6 - Autonomous Laboratory

- [ ] Closed-loop investigations
- [ ] Continuous experimentation
- [ ] Automated regression generation
- [ ] Long-running investigations
- [ ] Distributed experiment execution
- [ ] Investigation prioritization

Autonomous execution remains subject to explicit target authorization, isolation, and execution limits.

## Proposed Project Structure

The repository may evolve toward a structure similar to this. These application directories do not exist yet.

```text
oracle/
|-- apps/
|   |-- console/               # Web interface
|   `-- api/                   # Public API
|-- crates/
|   |-- core/                  # Core domain logic
|   |-- model/                 # System representation
|   |-- graph/                 # Dependency graph
|   |-- telemetry/             # Logs, metrics, traces
|   |-- experiments/           # Experiment engine
|   |-- verification/          # Verification integration
|   `-- agents/                # AI investigation
|-- services/
|   |-- collector/             # Telemetry collection
|   |-- orchestrator/          # Experiment orchestration
|   `-- worker/                # Background execution
|-- models/                    # ML / reasoning components
|-- verification/              # Formal specifications
|-- infrastructure/
|   |-- docker/
|   |-- kubernetes/
|   `-- observability/
|-- experiments/               # Reproducible experiment definitions
|-- docs/
|   |-- architecture/
|   |-- research/
|   `-- decisions/
|-- tests/
|   |-- unit/
|   |-- integration/
|   `-- system/
`-- README.md
```

The layout should evolve with the architecture rather than require empty scaffolding in advance.

## Getting Started

You can clone the repository to explore the project direction:

```bash
git clone https://github.com/JonatanLiljeblad/Oracle.git
cd Oracle
```

**There is currently nothing to build or run.** Dependency manifests, infrastructure definitions, and application startup commands will be documented when the first implementation is introduced.

Rust, Python, Node.js, Docker, PostgreSQL, and Redis are potential development dependencies, not current installation requirements.

## Configuration

The intended configuration approach is environment variables rather than hard-coded values.

Potential settings include database and Redis connection URLs, an OpenTelemetry endpoint, AI provider and model selection, and the execution environment. Setting names and defaults are not yet defined.

When configuration is implemented, a committed `.env.example` should document supported settings using non-secret placeholders. Local secret-bearing `.env` files must be excluded from version control.

Secrets must never be committed or included in experiment reports, logs, or AI requests.

## Development and Testing

ORACLE is intended to be developed as an engineering and research project, favoring small, reviewable changes and reproducible environments.

Build, formatting, lint, and test commands will be added alongside the tooling they operate on. No such commands are currently available.

The planned testing strategy includes:

| Test level | Purpose |
| --- | --- |
| Unit | Validate individual functions and components |
| Integration | Validate communication between services and infrastructure |
| System | Validate complete workflows against realistic environments |
| Property-based | Exercise behavior across generated input spaces |
| Experiment | Run controlled fault scenarios and check expected outcomes |
| Reproducibility | Rerun an experiment with the recorded version, configuration, initial state, inputs, and definition |

Testing is central to ORACLE because the platform itself is designed around experimental evidence.

## Observability

ORACLE itself should be observable.

The platform should expose requests, errors, latency, CPU and memory use, queue depth, experiment execution, agent decisions, investigation duration, and verification results.

Distributed tracing should make it possible to follow an investigation across services. Agent decision records should capture supporting evidence and actions without exposing secrets.

> **The system investigating software should itself be engineered like production software.**

## Design Principles

### Evidence over speculation

Distinguish between **observed**, **inferred**, **predicted**, and **verified** claims. AI-generated claims should not automatically be treated as facts.

### Reproducibility over magic

Record what happened, the conditions under which it happened, the supporting evidence, and how the result can be reproduced.

### Deterministic where possible

Prefer deterministic experiments and simulations where practical. Record seeds and relevant environmental conditions when nondeterminism cannot be eliminated.

### Safety by isolation

Experiments that can affect system state or infrastructure should run in controlled environments. Never assume an arbitrary target is safe to modify.

### Modular architecture

Keep the AI layer, experiment engine, telemetry system, system model, and user interface independently replaceable.

### Measure everything that matters

Back performance claims with measurements: experiment duration, throughput, latency, resource utilization, failure recovery time, prediction accuracy, and false-positive rate.

## Security

ORACLE is intended for **authorized systems only**.

Because the platform would interact with software systems and execute experiments, its design must prioritize:

- Explicit target authorization and approved experiment scope.
- Isolated environments and least-privilege execution.
- Restricted network access and secret isolation.
- Audit logging of experiment actions and authorization.
- Sandboxed execution and protection against arbitrary command execution.
- Bounded execution, cancellation, and cleanup of experiment resources.

Source code, telemetry, and experiment artifacts may contain sensitive information. Any external AI integration must respect data-access boundaries and explicit approval for sending data outside the environment.

These are design requirements, not implemented security guarantees.

## Research Direction

ORACLE sits at the intersection of distributed systems, software engineering, artificial intelligence, observability, testing, formal verification, and data engineering.

Potential research questions include:

| Area | Question |
| --- | --- |
| System understanding | How accurately can architecture be reconstructed from source code and runtime evidence? |
| Experiment selection | Can an AI agent maximize useful information while minimizing execution cost? |
| Failure discovery | Can autonomous experimentation discover failure modes conventional testing misses? |
| Verification | Can runtime observations help generate or improve formal models of concurrent and distributed systems? |
| Human-AI collaboration | How should developers interact with an AI system that actively investigates software rather than merely answering questions? |

These questions may evolve as the project develops.

## Long-Term Goal

The ultimate goal is to let a developer connect ORACLE to a complex application and ask:

> **"What don't we know about this system?"**

ORACLE should then investigate: identify uncertainty, propose experiments, collect evidence, test assumptions, and turn observations into reproducible engineering knowledge.

The desired end state is not another coding assistant. It is:

> **A laboratory for discovering how software systems actually behave.**

## Contributing

Contributions should prioritize correctness, reproducibility, measurable performance, clear architecture, strong testing, and useful documentation.

At this stage, contributions can help refine the architecture, define the first system-model implementation, and establish concrete development tooling. Once implementation exists, pull requests should update relevant tests and documentation.

## License

A license has not yet been selected. No license grant is provided by this README; licensing terms will be added separately.
