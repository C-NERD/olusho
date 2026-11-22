# Revocation

Withdrawal of consent is the hardest part of this design and the part most often promised without
a mechanism. This document states what is possible, what is not, and why.

## Two different kinds of "no"

These are separate operations with different actors, remedies and costs. Conflating them produces
designs that cannot be implemented.

**1. An individual withdraws their own record.**
The data subject exercises a personal right. The record stops being served; if it has reached
model weights, something has to be retrained. No committee is involved — under data protection law
withdrawal and erasure are rights of the individual, not powers of a collective acting on their
behalf.

**2. The authorising body refuses or revokes an access grant.**
The committee declines a request, or cuts an existing grant. The key is revoked. This is immediate,
costs nothing, and requires no retraining.

The expensive remedy belongs to the first case only. Any claim that models must stay small enough
to retrain "when the community says no" attaches the cost to the actor that does not trigger it.

A caveat worth recording: there is a serious literature arguing that community-level consent is
the workable model in African research contexts where individual consent is impractical at scale.
That position is defensible, but it does not remove the individual statutory right, and it changes
which remedy applies. Both mechanisms are specified here deliberately.

## Why machine unlearning is not the mechanism

It is tempting to propose that a recipient "unlearns" withdrawn data. The literature does not
support it.

- Only **retraining from scratch** without the data is exact. Everything else is approximate.
- Approximate unlearning in large language models is **shallow and reversible**: published results
  show that supposedly forgotten knowledge is recovered by relearning attacks using a small, and
  even loosely related, set of data points.
- **The evaluation is itself unreliable.** Work in this area has shown that standard metrics can
  indicate complete removal while the model demonstrably retains the targeted knowledge. You
  cannot currently measure whether unlearning worked.
- **Regulators have no settled answer either.** The UK ICO treats erasure requests as applicable
  to training data but not absolute; France's CNIL states explicitly that its guidance holds only
  "in the light of the current state of the art" and calls for research. Full retraining is widely
  treated as the most compliant route, and legal scholars disagree on whether a data subject can
  compel it.

Proposing unlearning as the enforcement mechanism would therefore be proposing something that
cannot be verified. It is not used here.

## Three levers that do work, strongest first

### 1. Keep the data out of the weights

If the data is queried from a retrieval layer at inference time rather than baked into parameters,
withdrawal is a deletion from an index: immediate, total, and verifiable. This is the only lever
that delivers genuine instant revocation.

It is therefore a **condition of access**, not a hope: derived artifacts must keep the data in a
revocable position in the stack.

### 2. Where weights are unavoidable, bound the cost

Sharded training — partitioning the data so each shard trains an isolated model whose outputs are
aggregated — means a withdrawal forces retraining only of the shard containing that record, not
the entire model. SISA is the best-known exact method of this kind.

Pair it with **honest latency**. "Withdrawal honoured at the next scheduled retrain, within N
days" is implementable and checkable. "Instant erasure from a trained model" is not.

### 3. Govern weight export

Once trained weights leave the environment, revocation is over permanently: there is nothing to
detect, no gate to close, and no remedy. This is the highest-leverage condition in the entire
design and the one most frequently forgotten, because attention goes to the data and not to what
is derived from it.

Where retraining must happen, running it inside the custodian's environment also converts
verification from auditing a claim into observing an event.

## Why this argues for small models

Revocation feasibility is inversely proportional to model size. Retraining a frontier model to
honour one person's withdrawal is unthinkable; retraining a small, purpose-built model is a
routine scheduled job.

Small models are therefore not merely cheaper or greener here. They are the only scale at which
withdrawal of consent can be mechanically honoured — which makes "small" a precondition for the
right to withdraw being real, rather than an aesthetic preference.

## The honest limit

For models already trained and released on data already collected, the data is gone. Not
difficult, not expensive — gone.

This belongs in the consent conversation itself rather than buried in a specification. Telling a
data subject "you may withdraw consent for future use; use that has already happened cannot be
undone" is part of consent being informed. A system that implies otherwise misleads the people it
claims to serve.
