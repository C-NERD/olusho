# Olúṣọ́ (Guard)

**Let the model in, never the data out.**

Olúṣọ́ is a design for community-custodied access to mental health data. The data never leaves the
community's custody: a requester's code runs against it inside a secure environment, and only
reviewed, aggregated results come out. No single party can open the gate alone — not even whoever
runs the servers.

It guards a gate. It does not manufacture consent.

## Status

Design stage. A specification and a list of open questions, nothing more. No implementation, no
real data, no agreement with any host organisation. The architecture is published before it is
built so that it can be argued with first.

## Documents

| Document | What it covers |
|---|---|
| [00-overview](docs/00-overview.md) | The problem, why detect-and-enforce cannot work, the inversion, non-goals |
| [01-architecture](docs/01-architecture.md) | Components, request flow, authorisation, output review, audit log |
| [02-governance](docs/02-governance.md) | Who "the community" is, and who may authorise access |
| [03-revocation](docs/03-revocation.md) | Withdrawal of consent, and why unlearning is not the mechanism |
| [04-legal](docs/04-legal.md) | Nigerian statutory floors: NDPA, National Health Act, NHREC |
| [05-threat-model](docs/05-threat-model.md) | Adversaries, mitigations, and what this cannot defend against |
| [06-request-protocol](docs/06-request-protocol.md) | How a request is expressed, rendered and signed |
| [07-data-plane](docs/07-data-plane.md) | How data moves between the store and the sandbox |
| [08-reuse-and-dependencies](docs/08-reuse-and-dependencies.md) | What we reuse, under what licence, and what is ours to build |
| [90-open-questions](docs/90-open-questions.md) | What is not mine to answer |

If you only read one, read [03-revocation](docs/03-revocation.md). It is the argument the rest
hangs off.

## Licence

Documentation is CC-BY-4.0. See [LICENSE](LICENSE).
