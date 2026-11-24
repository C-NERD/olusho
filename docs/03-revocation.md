# Revocation

Withdrawal of consent is the hardest part of this design and the part most often promised without
a mechanism. This document states what is possible, what is not, and why.

## A note on terms

Two things in this design have confusingly similar names.

The **authorisation service** is ours. It verifies committee signatures and decides whether a job
may run. It is part of the gate.

A **retrieval layer** belongs to whoever uses the data. It means a system that looks records up at
the moment it answers a question, rather than having absorbed them into model parameters during
training. The distinction matters: a system that memorised cannot forget on command, while a system
that looks things up only needs the page removed.

Nothing in this repository implements a retrieval layer. Where one appears below, it is a
constraint on how a requester builds their system, or a service we might operate in a later phase.

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

## Three populations of artifact

What withdrawal can achieve depends entirely on where the derived work currently sits.

### Inside the custody boundary

Training in progress, or trained and waiting on output review. We control these completely.

**Revocation here is free: abort, do not retrain.** Kill the job, discard the artifact. Nothing has
been released, so nothing has to be undone. If the requester still wants the work, they re-extract
against current consent and re-run.

This is the only case where withdrawal is both costless and absolutely enforceable, and it is worth
stating first because it is easy to overlook while worrying about the hard cases.

### Released as weights

A derived, lossy artifact. The records are not literally inside it, but what was learned from them
is. Revocation requires retraining, which is why the cost must be bounded in advance — see
*Bounding the cost* below.

### Released into an external index

The requester holds a searchable copy of the records in their own infrastructure. This is a bulk
data release, not a safe alternative to weights, and it is **prohibited** under this project's
terms of access.

## Where a derived artifact may live

Ordered by how much control the community retains:

| Placement | Revocation | Data leaves custody? |
|---|---|---|
| Retrieval on the custodian's side | Row deletion, instant, needs no cooperation | No |
| Sharded weights | Bounded retraining, next scheduled cycle | No (derived only) |
| Retrieval on the requester's side | Contractual only | **Yes, in bulk** |

Custodian-side retrieval is the strongest arrangement available. The requester's product calls our
service at inference time; the records never leave; withdrawal takes effect on the next query with
nobody's cooperation required; and every call is metered, which turns benefit-sharing into
continuous payment rather than a one-off fee.

It is also the heaviest. It makes a community organisation into production infrastructure for
someone else's product, with the uptime, latency and on-call obligations that implies, and the
serving cost falls on the custodian unless it is priced into the arrangement. Disclosure control
becomes much harder, because nobody can review every response by hand at query volume.

## Policy for this phase: weights only

Requesters may train and export **sharded model weights**. They may not take a copy of the records
into any external index or store. Custodian-side retrieval is recognised as the stronger
arrangement and deliberately deferred: it is a possible second phase, not a commitment, because the
operating burden is beyond what this project can honestly promise. See
[90-open-questions.md](90-open-questions.md) §30.

Two consequences follow, and both are costs rather than conveniences.

First, there is no longer a cheap revocation path. Row deletion is unavailable, so bounding the
retraining cost stops being a preference and becomes a hard condition of every grant.

Second, the artifact under disclosure review is now a model rather than a table. That is a much
harder review than checking cell sizes, and it is treated as its own adversary in
[05-threat-model.md](05-threat-model.md) §9.

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

## Bounding the cost: sharding and slicing

### Sharding is mandatory, not preferred

A grant requires shardable training. A requester who will not shard cannot have access, because
otherwise a single withdrawal means retraining an entire model and we would be promising a remedy
we cannot deliver.

**Sharding** splits the records into *N* groups and trains an independent model on each, combining
their answers at inference time. Only the model for the group containing a withdrawn record ever
saw it, so only that model is retrained. The remaining models are provably unaffected.

### How records are assigned to shards

```
shard = hash(grant_salt + internal_record_id) mod N
```

Three design points, each for a reason:

- **Not `LIMIT`/`OFFSET`.** Those depend on row ordering, so shard boundaries move whenever rows
  are added or removed, destroying the only property that makes sharding useful: being able to prove
  which shard a record was in.
- **Hashed rather than assigned.** If an operator could choose assignments freely and simply record
  them, they could place a known individual in a shard they quietly intend never to retrain. A hash
  lets anyone holding the salt and the identifier recompute the assignment and confirm it was not
  steered. This is an auditability property, not a performance one.
- **Salted per grant.** An unsalted hash would place the same person in the same shard number in
  every extract, letting a requester holding two extracts infer that the same individual appears in
  both. The salt gives stability for the lifetime of one model, which is all the stability required.

Assignments are **recorded** in the extract manifest, not recomputed on demand — see
[07-data-plane.md](07-data-plane.md). Hashing happens once, over a few thousand identifiers, and is
negligible beside the extraction query, let alone training.

Changing *N* reshuffles every record's assignment, so the shard count is a long-term commitment
rather than a tuning knob.

### Slicing composes with sharding; it does not compete

**Slicing** splits the records within each shard into sequential stages, saving a checkpoint after
each. A withdrawal resumes from the checkpoint before the affected stage instead of restarting the
shard.

For 10,000 records in 10 shards of 1,000, each sliced into 5 stages of 200:

| Arrangement | Retraining cost |
|---|---|
| Neither | 1.00 |
| Sharding only | 0.10 |
| Sharding + slicing, record in stage 4 | 0.04 |
| Sharding + slicing, record in stage 1 (worst case) | 0.10 |

The worst case for slicing equals sharding alone. Slicing never makes retraining more expensive; it
either helps or breaks even, and averaged over where a record happens to sit it roughly halves the
within-shard cost.

What slicing actually costs is storage and bookkeeping — *N × S* checkpoints, so 50 saved model
states in the example above. Trivial for small models; not trivial for large ones, which is one
more way the small-models argument pays for itself.

### Honest latency

"Withdrawal honoured at the next scheduled retrain, within N days" is implementable and checkable.
"Instant erasure from a trained model" is not.

One known weakness, recorded rather than hidden: the saving from sharding depends on withdrawals
clustering in few shards. If withdrawals scatter across all of them, every shard is retrained and
nothing has been gained. The literature on random partitioning identifies exactly this.

## Govern weight export

Once trained weights leave the environment, revocation is over permanently: there is nothing to
detect, no gate to close, and no remedy. This is the highest-leverage condition in the entire
design and the one most frequently forgotten, because attention goes to the data and not to what is
derived from it.

Where retraining must happen, running it inside the custodian's environment also converts
verification from auditing a claim into observing an event.

## Consent is a state, not an event

Consent is read at each training cycle rather than pushed at the moment it changes. Every scheduled
retrain includes whoever currently consents.

Withdraw today and you are out of the next cycle. Restore consent later and you are back in the
cycle after that. No special machinery is required in either direction.

Two reasons it is built this way:

- **Re-inclusion must not be compulsory.** Restored consent means a requester *may* use the record
  again, not that they *must*. Forcing a retrain to put someone back converts a permission into an
  obligation that nobody asked for.
- **Event-triggered retraining is an attack surface.** If every consent change forced a retrain,
  then repeatedly toggling consent — by one person, or by an adversary controlling several records
  — becomes a way to impose unbounded cost on the holder.

## Consent is re-checked at release

Checking consent when the extract is made is not sufficient. Consider: extract on Monday, training
runs Tuesday, a data subject withdraws on Wednesday, release review happens Thursday. The artifact
is stale with respect to consent and nothing has caught it.

So the output review queue re-checks consent currency at release. If any record in the extract
manifest has been withdrawn since the extract was taken, release is refused and the job returns to
the requester. This is the mechanism that makes *abort, do not retrain* effective for everything
still inside the boundary.

## Why this argues for small models

Revocation feasibility is inversely proportional to model size. Retraining a frontier model to
honour one person's withdrawal is unthinkable; retraining a small, purpose-built model is a routine
scheduled job.

Under a weights-only policy this stops being a rhetorical point and becomes the operative
constraint: model size directly determines whether withdrawal is affordable at all. Small models
are therefore a precondition for the right to withdraw being real, rather than an aesthetic
preference.

## The honest limit

For models already trained and released on data already collected, the data is gone. Not difficult,
not expensive — gone.

This belongs in the consent conversation itself rather than buried in a specification. Telling a
data subject "you may withdraw consent for future use; use that has already happened cannot be
undone" is part of consent being informed. A system that implies otherwise misleads the people it
claims to serve.
