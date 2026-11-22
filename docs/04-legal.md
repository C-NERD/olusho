# Statutory floors (Nigeria)

Governance design does not displace law. Whatever a community committee decides, the obligations
below apply to whoever holds and processes the data. They are floors beneath Olúṣọ́, not
alternatives to it.

**This document is an engineer's orientation, not legal advice.** Section-level citations should be
confirmed against the current gazetted texts, and the arrangement should be reviewed by a
Nigerian lawyer before any real data is involved.

## Nigeria Data Protection Act 2023 (NDPA)

Regulator: the Nigeria Data Protection Commission (NDPC).

The Act sets out principles that bear directly on this design: personal data must be processed
fairly, lawfully and transparently; collected for specified, explicit and legitimate purposes and
not further processed in a way incompatible with those purposes; and adequate, relevant and limited
to the minimum necessary for those purposes.

Consequences for Olúṣọ́:

- **Purpose limitation is enforceable by a regulator**, independently of the committee. A grant the
  committee approves must still have a lawful basis and a compatible purpose.
- **The operator is a data controller or processor** with statutory duties — security, breach
  handling, records of processing — that cannot be delegated to a governance committee.
- **Withdrawal and erasure are individual rights.** This is why
  [03-revocation.md](03-revocation.md) treats individual withdrawal separately from committee
  refusal: a committee cannot withdraw on a person's behalf, and a person does not need committee
  approval to withdraw.

Mental health data is sensitive personal data, which attracts stricter treatment.

**[open]** Whether the host operator is controller, processor, or joint controller with the
committee; whether any cross-border element (a requester outside Nigeria submitting code, or
results leaving the country) triggers transfer conditions.

## National Health Act 2014, ss. 25–29

- **s. 25** imposes an obligation to keep health records.
- **s. 26** makes all information concerning a user of health services — including health status,
  treatment, and stay in a health establishment — confidential.

Confidentiality under s. 26 is the baseline the architecture has to respect by default. A system
in which raw records never leave custody is substantially easier to reconcile with s. 26 than one
that hands copies to third parties under contract.

## Research ethics: NHREC

Research use of the data requires ethics approval through the National Health Research Ethics
Committee regime, under the National Code for Health Research Ethics. Institutional health research
ethics committees operate under that national framework.

Consequences:

- A requester wanting to conduct research must hold ethics approval **in addition to** a committee
  grant. The two are not substitutes, and the gate should require evidence of both.
- Prototyping and demonstration on the **synthetic mirror** involves no human subjects data, and so
  needs no approval. This is a deliberate property of the development posture in
  [01-architecture.md](01-architecture.md), and it is what makes the system buildable before any
  approval exists.

## How the layers compose

```
Individual statutory rights (NDPA)        <- cannot be overridden by committee
        v
Confidentiality + records duties (NHA)    <- bind the operator
        v
Research ethics approval (NHREC)          <- gates research purposes
        v
Community authorisation (m-of-n)          <- this project's contribution
        v
Technical enforcement (the gate)          <- makes the layer above real
```

The project's claim is narrow and sits at the bottom two layers: the law already says who has
rights, and ethics review already says which research may proceed. Neither is self-enforcing when
the data has been handed over. The gate is what makes the layers above it checkable.
