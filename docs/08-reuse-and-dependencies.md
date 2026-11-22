# Reuse and dependencies

The working principle: implement only what does not already exist. This document records what we
take from elsewhere, under what licence, and what is genuinely ours to build.

## What OpenSAFELY has published

OpenSAFELY is an open-source Trusted Research Environment operating over NHS England records at
national scale. Three of its components are relevant, all licensed **GPL-3.0-or-later**, copyright
University of Oxford:

| Component | What it gives us |
|---|---|
| [ehrql](https://github.com/opensafely-core/ehrql) | A query model, SQL generation across database dialects, and synthetic data generation from the same definition |
| [airlock](https://github.com/opensafely-core/airlock) | Output review and release: requesters raise release requests, trained checkers review independently, outputs are approved or refused |
| [job-runner](https://github.com/opensafely-core/job-runner) | Running jobs in containers inside a secure environment |

Airlock already implements independent review by two separate output checkers before anything is
released. That is the disclosure-control step in [01-architecture.md](01-architecture.md), built and
in production.

## What we inherit specifically

- `ehrql/query_model/` — the intermediate representation. `nodes.py` declares an explicit public
  API surface with its own `__all__`, and nothing in the codebase or docs marks it internal or
  unstable.
- `ehrql/query_engines/base_sql.py` — generic SQLAlchemy-backed SQL generation, with per-dialect
  engines layered on top.
- `ehrql/dummy_data/generator.py` — synthetic data generated from the same definition that drives
  the real extract. This is the synthetic mirror in [07-data-plane.md](07-data-plane.md), free.
- `ehrql/measures/disclosure_control.py` — disclosure control already implemented.
- `ehrql/file_formats/` — Arrow and CSV extract writers.
- `ehrql/query_engines/in_memory.py` and `local_file.py` — engines that let tests run without a
  database.

## What does not exist, and is therefore ours

- **m-of-n community authorisation.** OpenSAFELY has institutional approval: projects get approved,
  researchers get accredited. There is no cryptographic multi-party authorisation by a community
  committee. See [06-request-protocol.md](06-request-protocol.md).
- **Plain-language rendering for non-specialist reviewers.** Their output checkers are trained
  statisticians reading tables. Ours are community health workers deciding whether a request should
  happen at all.
- **Revocation propagation.** A research TRE has no consent-withdrawal machinery: no retrieval-layer
  invalidation, no sharded retraining, no governed weight export. See
  [03-revocation.md](03-revocation.md).
- **Benefit-sharing metering**, and the community-facing public access log.

This is worth stating plainly, because it defines the contribution: the data plane is largely
solved and published. The governance layer is not. We are not rebuilding a TRE — we are building
the layer no TRE has, on top of one that already works.

## Decision: target the IR, do not sign it

We compile our canonical specification into ehrQL's query model and inherit its SQL generation.

We do **not** sign ehrQL's serialised query model. Its serialiser carries execution context — file
paths, module details, error types — because it is transport between their components rather than a
description of a data scope. The reasoning is in
[06-request-protocol.md](06-request-protocol.md).

## The PostgreSQL engine

ehrQL's production backends are TPP (MSSQL) and EMIS. There is no PostgreSQL support anywhere in
the repository, so this is ours to contribute.

The scope is small. `base_sql.py` is ~1400 lines with 107 methods, of which only 9 are abstract —
nearly everything is inherited. The complete SQLite engine is 191 lines plus a 20-line dialect, and
SQLAlchemy already ships a first-class PostgreSQL dialect, so we may not need a dialect file at all.

The real work is not the skeleton but the function-coverage matrix: every date and string operation
needs correct PostgreSQL semantics. Most comes free from SQLAlchemy; the time goes into edge cases
such as day-of-month overflow in month arithmetic.

## Licensing consequence

Importing `ehrql.query_model` makes our code a derivative work. **Our code is therefore
GPL-3.0-or-later.** This is compatible with working in the open, so nothing is lost.

Note the asymmetry: OpenSAFELY's *code* is GPL, but their *documentation* is not openly licensed —
the docs reserve rights beyond non-commercial research and study. Reuse the code; do not copy their
prose or schema descriptions into these documents.

## Pinning and divergence

`ehrql/VERSION` reads `dev`, there are no release tags in the tree, and the project states it is not
currently accepting external contributions. So:

- Pin to a specific commit SHA, not a branch.
- Expect the PostgreSQL engine to live as a permanent fork delta that we rebase ourselves.
- Upstreaming is not available to us, so budget for carrying the difference indefinitely.

## Implementation language

**Python**, for two reasons.

The first is mechanical: the IR is Python, so targeting it requires Python. An FFI bridge from
another language into CPython to drive dataclasses would be worse than either pure option.

The second matters more. This system's value depends on people who are not its author being able to
read, verify, maintain and inherit it — a future maintainer at the host organisation, an auditor, a
security reviewer, whoever picks it up years from now. That is the substance of
[90-open-questions.md](90-open-questions.md) §12 and §14. A language with a small talent pool makes
the project's two hardest open questions harder, in a project whose premise is that it must outlive
whoever wrote it.

## The independent verifier (Nim)

One component is deliberately *not* Python: a standalone verifier, written in Nim and shipped as a
single static binary.

It takes a signed grant, the canonical specification and the approved render, and checks that the
signatures are valid and that the render reproduces from the specification.

The point is independence. A second, separately written implementation of verification surfaces
ambiguities in our own specification that a single implementation can never reveal — which is why
serious protocols want more than one. It also means verification does not depend on trusting the
code that produced the grant: anyone can run the binary against a published grant and check that
the main system is telling the truth.

It is a good fit for Nim on the merits, not as a gesture: the interface is narrow, it needs nothing
from the ecosystem beyond Ed25519, and it ships with no interpreter or runtime to install on a
reviewer's machine.

Sequencing: it is written **after** the Python path works, against a specification that has stopped
moving. Ed25519 in Nim is best obtained by binding libsodium — audited C behind a thin wrapper —
rather than relying on a pure-Nim implementation.

**[open]** How much of verification the binary must cover to be a meaningful second implementation,
rather than a partial check that creates false confidence.
