# Threat model

Adversaries, what the design does about them, and what it cannot do. Stated in the open because a
governance system whose failure modes are undocumented should not be trusted with anyone's data.

## 1. Requester exfiltrates data through permitted outputs

The requester runs approved code but crafts outputs that carry record-level information out —
overly granular aggregates, small-cell counts, or encodings hidden in numeric results.

*Mitigation:* disclosure-control review on every outgoing artifact, minimum cell sizes, suppression
of small counts, no record-level returns, and limits on volume and frequency of outputs per grant.

*Residual:* a determined requester with many legitimate-looking queries can still accumulate
information. Output budgeting helps and does not eliminate it. This is the central unsolved problem
of every TRE and is not solved here either.

## 2. Operator grants itself access

The party running the infrastructure uses operator credentials to run jobs the committee never
approved.

*Mitigation:* the separation in [01-architecture.md](01-architecture.md) — execution requires a
grant carrying *m-of-n* committee signatures, verified independently of operator credentials. The
operator cannot mint a valid grant.

*Residual:* the operator controls the environment and could in principle bypass the software
entirely. Tamper-evident logging makes this detectable after the fact rather than impossible. Key
custody held outside the operator raises the cost further.

## 3. Committee capture

A requester cultivates enough committee members to obtain routine approval, or the host influences
appointments to favour its partners.

*Mitigation:* composition weighted to data subjects, term limits and rotation, declared conflicts
with recusal, and the public log — which makes a pattern of approvals visible to people outside the
committee.

*Residual:* governance, not engineering. The log makes capture visible; it cannot prevent it.

## 4. Rubber-stamping

The committee has formal veto power and approves everything because requests are unintelligible to
them. The system reports healthy governance while exercising none.

*Mitigation:* plain-language rendering of every request as a required component, not a convenience
(see [02-governance.md](02-governance.md)).

*Residual:* the most likely real-world failure mode, and the hardest to detect from inside. A
refusal rate of zero over a long period should be treated as a defect signal.

## 5. Weights walk out

A requester trains on the data inside the environment, exports the weights, and from then on holds
a derived artifact beyond any revocation.

*Mitigation:* weight export is explicitly governed, and grants require shardable training so that
a withdrawal after export has a bounded remedy the holder can actually perform.

*Residual:* once weights are out, revocation is permanently over. There is no technical remedy,
which is why this is a condition at grant time rather than a control afterwards.

## 6. Re-identification of small subgroups

Mental health data in a defined locality can identify people through combinations of
non-identifying fields, especially for rare presentations or small communities.

*Mitigation:* disclosure control tuned for small populations, suppression thresholds set with the
community rather than by convention, and refusal of query shapes that isolate individuals.

*Residual:* the smaller and more specific the community, the weaker these protections are — which
cuts against the project's own premise of serving specific communities. An honest tension, not a
solved problem.

## 7. Compromise of the custodian

Key theft, insider action, ransomware, or simple loss of the infrastructure.

*Mitigation:* ordinary security engineering — key management, least privilege, backups, incident
response — plus the structural point that a community-held dataset is a smaller and less attractive
target than an aggregated national one.

*Residual:* a small organisation's security capacity is limited. This is a real argument for shared
or pooled security support rather than something each community solves alone.

## 8. The arrangement is simply bypassed

Requesters decline the constraints and obtain equivalent data elsewhere, ungoverned.

*Mitigation:* none available in software.

*Residual:* this is the project's central strategic assumption — that the dataset is distinctive
enough that access on these terms is worth accepting. If it is not, the gate guards an empty room.
It is listed in [90-open-questions.md](90-open-questions.md) as an assumption to be tested with the
community, not asserted.

## 9. The released model leaks its own training data

Under a weights-only policy (see [03-revocation.md](03-revocation.md)) the artifact leaving the
environment is a model trained on individual mental health records. Models memorise. Membership
inference can establish that a particular person was in the training set, and extraction attacks
can recover fragments of training records from the weights themselves.

This is a disclosure of released data with no remedy, because the artifact is already out.

*Mitigation:* keep models small and extracts narrow, so there is less capacity to memorise relative
to the data; canary insertion and extraction probing before release; membership-inference testing as
part of disclosure review; differential privacy during training where the accuracy cost is
acceptable.

*Residual:* substantial, and worse than it looks. Disclosure-reviewing a model is a far harder
problem than disclosure-reviewing a table — checking minimum cell sizes is tractable, while
establishing that a model will not leak the people inside it is only partly solved. This is the
largest cost of choosing weights-only, and it is tracked as
[90-open-questions.md](90-open-questions.md) §27.

## What this design does not defend against

- Data already collected and already trained on. Protection is prospective only.
- A state compelling disclosure through lawful process.
- A community deciding, legitimately, to grant access on terms an outsider would consider unwise.
  The system enforces the community's decision; it does not second-guess it.
