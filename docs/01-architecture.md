# Architecture

Design stage. Component boundaries are intended to be stable; mechanisms inside them are not yet
settled. Items marked **[open]** are unresolved and listed in
[90-open-questions.md](90-open-questions.md).

This document is the overview. Two mechanisms have their own specifications:
[06-request-protocol.md](06-request-protocol.md) for how a request is expressed, rendered and
authorised, and [07-data-plane.md](07-data-plane.md) for how data moves between the store and the
sandbox.

## Components

| Component | Responsibility |
|---|---|
| Data store | Holds the data. Never serves raw records outward. PostgreSQL, on its own host. |
| Submission service | Accepts a request: a declarative specification, the code to run, stated purpose, requester identity. |
| Renderer | Turns a specification into the plain-language view the committee reads. Security-critical. |
| Authorisation service | Holds the access policy; verifies committee signatures on a grant. |
| Job runner | Trusted. Verifies the grant, extracts the approved slice, feeds and collects the sandbox. |
| Execution sandbox | Runs the requester's code against a materialised extract, with no network namespace at all. |
| Output review queue | Human disclosure control on results before release. |
| Audit log | Append-only record of every request, decision, execution and release. |
| Public log view | Plain-language, publicly readable rendering of the audit log. |
| Synthetic mirror | Structurally identical fake dataset, public, for developing against. |

## Request flow

1. **Develop against the synthetic mirror.** The requester writes and tests their code against
   public synthetic data with the same schema. They never need the real dataset to build.
2. **Submit.** A declarative specification of the data wanted, plus the code to run, the stated
   purpose, the requester's identity, and the intended position of any derived artifact (see Terms
   of access below). Detail in [06-request-protocol.md](06-request-protocol.md).
3. **Render.** The specification is turned into plain language for the committee. If it cannot be
   described, it cannot be signed.
4. **Authorise.** A grant is valid only when signed by at least *m* of *n* committee members, over
   a payload binding the specification to the exact render they were shown. The operator cannot
   produce a valid grant with operator credentials alone.
5. **Execute.** The job runner verifies the grant, extracts the approved slice, and starts the
   sandbox with the extract mounted read-only, no network and no interactive shell. Raw records are
   never rendered to the requester. Detail in [07-data-plane.md](07-data-plane.md).
6. **Review outputs.** Results enter a disclosure-control queue. A trained reviewer checks them
   against release rules before anything leaves.
7. **Release and log.** Approved outputs are returned. Request, decision, signatures, execution
   and release are written to the append-only log and surfaced in the public view.

## Separation of key-holder from decision-maker

This is the load-bearing design decision.

If the party that operates the infrastructure also decides who may use it, there is no governance —
only a benevolent custodian, which lasts exactly as long as the current leadership. Community
control that depends on the custodian's goodwill is not control.

Therefore: the operator runs the system and holds operational credentials, and still cannot open
it. Authorisation requires *m-of-n* signatures from the governing body, verified by the
authorisation service. Access becomes a precondition enforced in software rather than a commitment
recorded in a document.

**[open]** Choice of *m* and *n*; key custody for committee members (hardware tokens, custodial
keys, social recovery); what happens when a member loses a key or becomes unreachable; whether
some request classes need a higher threshold than others.

## Output disclosure review

Aggregated results can still leak information about individuals, particularly in small subgroups.
Every real Trusted Research Environment retains a human step here, and so does this design.

For tabular results the reviewer's job is bounded and well-defined: check against release rules
(minimum cell sizes, suppression of small counts, no record-level data) and approve or refuse. This
is a tractable queue of outgoing artifacts — not an attempt to supervise a corporation.

Under the weights-only policy, however, the artifact under review is frequently a **model**, and
that is a materially harder review: establishing that a model will not leak the people inside it is
only partly solved. The queue therefore also carries consent re-checking at release, and
model-specific checks — canary probing, membership-inference testing. See
[05-threat-model.md](05-threat-model.md) §9.

**[open]** Who performs disclosure control, what training they need, how the role is funded, and
what the appeal path is when a researcher's output is refused. Reviewing a *model* rather than a
table is a materially harder problem and is tracked separately at
[90-open-questions.md](90-open-questions.md) §27.

## Audit log

Append-only, tamper-evident, and readable by the governing body without the operator's cooperation.
The point is not that the operator keeps good records; it is that the governed can verify them.

The public view renders the same information in plain language: who asked for what, who authorised
it, what was refused, and what was released. This is the most legible proof that the arrangement
is real.

**[open]** Tamper-evidence mechanism (hash chain, external anchoring, third-party witness) and how
much request detail can be public without exposing the requester's legitimate confidentiality or
re-identifying data subjects.

## Terms of access

Conditions attached to a grant, enforced where possible and contractual where not:

- **Weights only.** Requesters may train and export model weights. Taking a copy of the records
  into any external index or store is prohibited — that is a bulk data release, not a safer
  alternative. See [03-revocation.md](03-revocation.md).
- **Sharded training is mandatory**, not preferred. The cost of honouring a withdrawal must be
  bounded to the affected shard rather than the whole model. A requester who will not shard cannot
  have access.
- **Weight export is governed.** Trained weights leaving the environment ends revocability
  permanently. This is the single highest-leverage condition and the one most often forgotten.
- **Consent is re-checked at release**, not only at extraction. An artifact built from a record
  withdrawn since the extract was taken is not released.

## Development and testing posture

The synthetic mirror is public and is the only dataset used in development, demonstration or
prototyping. This follows OpenSAFELY's dummy-data pattern and has a practical consequence worth
stating: the system can be built, demonstrated and criticised in full before any real data or any
ethics approval is involved.
