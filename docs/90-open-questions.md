# Open questions

Written down because they are unresolved. Several are mine to answer, and the design should not pretend otherwise.

## For the community and host organisation to decide

1. **Should this exist at all**, in this form, over this dataset?
2. **Committee composition** — who sits on it, and are data subjects a majority?
3. **Selection and removal** of committee members, and by whom.
4. **The signing threshold** (*m* of *n*), and whether some request classes need a higher bar.
5. **Acceptable purposes.** Is commercial model training ever acceptable? Research only? What is
   categorically refused?
6. **The benefit-sharing split**, and what counts as a benefit — money, co-authorship, services,
   infrastructure, training.
7. **Disclosure thresholds.** How small a subgroup is too small to report on, decided with the
   people in those subgroups.

## Assumptions that need testing, not asserting

08. **Scarcity.** Does this dataset have enough distinctive value that requesters will accept these
    constraints rather than going elsewhere? The entire leverage of the design rests on this, and it
    is currently an assumption. See [05-threat-model.md](05-threat-model.md) §8.
09. **Demand.** Is there in fact a queue of legitimate requesters, or is the gate being built before
    anyone wants through it?
10. **Appetite for governance work.** Committee membership is unpaid attention. Will people serve,
    and keep serving past the first few months?

## Sustainability

11. **Hosting.** Where does the dataset physically live, at what recurring cost, and who pays after
    any grant period?
12. **Operations.** Who keeps it running, patches it, and responds at 2am?
13. **Key custody.** How do committee members hold keys safely, and what is the recovery path when
    a key is lost or a member becomes unreachable?
14. **Exit.** What happens to the dataset, the keys and the logs if the host organisation ceases to
    operate? Devices and data should outlive the organisation that held them.

## Legal and regulatory

15. **Controller or processor?** Whether the operator is data controller, processor, or joint
    controller with the committee under the NDPA — this determines who carries which duties.
16. **Cross-border elements.** A requester outside Nigeria submitting code, or results leaving the
    country: which transfer conditions apply?
17. **Interaction with ethics approval.** Should the gate require NHREC or institutional approval as
    evidence before a grant can be signed, and who verifies it?

## Technical

18. **Output budgeting.** How to bound cumulative inference across many individually benign
    queries, which is the unsolved problem in every TRE.
19. **Tamper-evidence** for the audit log: hash chain, external anchoring, third-party witness —
    what is defensible at this scale and affordable to run?
20. **Request rendering.** Can the plain-language rendering of a request be partly automated without
    becoming a self-description the requester effectively writes?
21. **Disclosure-control staffing.** Who performs output review, trained how, funded how, with what
    appeal path for refusals?
22. **Sharding in practice.** Whether shardable training is realistic for the model sizes a
    requester would actually want, or whether retrieval-layer-only is the honest condition.

## Design decisions taken, with consequences still open

23. **Our table schema.** ehrQL's schemas describe UK primary care. Ours does not exist yet, and
    its shape is a question for the community as much as for engineering: what the dataset contains
    determines what can ever be asked of it.
24. **The specification schema.** The JSON Schema for a request, and how it versions in lockstep
    with the renderer — neither may advance alone, because an unrenderable construct must be
    unsignable.
25. **ehrQL coupling.** We pin a commit SHA and carry the PostgreSQL engine as a fork delta. What
    is the cost of rebasing, and what is the exit if their query model changes under us?
26. **Verifier scope.** How much of verification the independent Nim binary must cover to be a
    meaningful second implementation rather than a partial check that creates false confidence.

## Known tension

27. The protections for small subgroups weaken as the community gets smaller and more specific —
    which is exactly the setting this project exists to serve. There may be no clean resolution,
    only an explicit decision made with the community about what it is willing to publish about
    itself.
