# Overview

## The problem

Mental health data from African communities is increasingly used to build AI systems, often
without meaningful consent, community control, or any share of the benefits. The pattern is
familiar: the community supplies the data, and others decide how it is used and who profits.

Most responses to this treat it as a paperwork problem. A community or its representatives sign
an agreement with a data recipient setting out permitted uses, and compliance is expected to
follow. This does not work, and it is worth being precise about why.

## Why detect-and-enforce cannot work

Once the data has been handed over, the community has effectively lost all power to enforce
control over its own data, and every available detection mechanism fails:

- **The recipient logging its own usage?** Self-reporting is worthless without audit rights, and no
  small community organisation will obtain enforceable audit rights inside a commercial training
  pipeline.
- **Monitoring of public signals?** — news, publications, product announcements. You learn about a
  breach only after it has become stale news, if at all.
- **A record keeping committee?** A committee can only record what it is shown. This relocates the
  self-reporting problem rather than solving it.

Enforcement is in no better shape. A community holding a signed agreement and evidence of breach
holds a lawsuit it cannot afford, against an adversary that can.

The failure is structural, not a matter of writing a better agreement. An agreement that cannot
be checked or reversed is a promise; and promises can be broken

## The inversion

Olúṣọ́ moves the control point from detection to access.

The data never leaves the community's custody. A requester does not receive a copy of the data.
Instead the requester's code is executed against the data inside the custodian's environment,
and only aggregated results leave — after review.

Three things follow:

1. **Detection becomes unnecessary.** Every query and every training job has to be submitted in
   order to run at all, so the usage log is a byproduct of the system operating rather than a
   report that must be requested and trusted.
2. **Enforcement becomes revocation.** The remedy for breach is withdrawal of access, exercised
   unilaterally by the custodian. No court, no regulator, no goodwill required.
3. **Benefit-sharing becomes measurable.** Every access is a transaction through a gate, so usage
   can be metered and any agreed share computed from records both parties can see.

## This is deployed practice, not a new idea

The pattern has names and running implementations. Trusted Research Environments and the Five
Safes framework are established in health research governance; commercial equivalents are called
clean rooms.

The clearest precedent is OpenSAFELY, which operates over NHS England patient records at national
scale. Researchers develop their analysis against dummy data, then submit code for automated
remote execution against the real records, never interacting with raw patient data directly; only
aggregated outputs are released, and submitted code is published. The Alan Turing Institute has
open-sourced its own Data Safe Haven implementation, and inventories of European TREs exist.

What is new here is not the mechanism. It is the custodian. In every deployment above, the party
holding the gate is a national health service, a university or a government ministry. The
governance literature on community and African health data sovereignty is mature on paper —
dynamic consent, benefit-sharing, custodianship, controlled access — but it has not been joined
to running access-control software in a setting where **the community itself is the custodian**.

That join, and the unglamorous engineering it exposes, is the contribution. It is an integration
and deployment claim, not an invention claim.

## Non-goals

- **It does not create consent.** Consent is a relationship between people. Software can make an
  agreement auditable and reversible; it cannot generate agreement where none exists.
- **It does not recover data already absorbed into released models.** Whatever has already been
  collected and trained on is gone. Protection here is prospective only.
- **It does not replace the law.** The Nigeria Data Protection Act, the confidentiality provisions
  of the National Health Act, and research ethics approval through NHREC are floors beneath this
  system, not alternatives to it. See [04-legal.md](04-legal.md).
- **It is not a diagnostic or clinical AI system.** It governs access to data; it does not provide
  care, triage or assessment.
- **It is not a general-purpose data platform.**

## Scope and status

Design stage. No implementation, no real data, no host agreement. Where this document refers to a
host organisation operating the infrastructure, that is a proposal and not a commitment by any
party.
