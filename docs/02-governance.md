# Governance

## Three roles, kept apart

"The community" is used loosely in this field, and the looseness hides the design question. Three
distinct roles have to be separated before anything else can be decided:

| Role | Who | What they hold |
|---|---|---|
| **Data subjects** | The young people, service users and community health workers the data is about | The moral claim. This is "the community". |
| **Operator / custodian** | The organisation running the infrastructure | The keys, the uptime, and the legal duties of a data controller or processor |
| **Authorising body** | A committee accountable to the data subjects | The decision on each access request |

Conflating the second and third is the usual failure. An operator that also decides who may
access the data is not governed; it is trusted. Trust is not a mechanism.

## Testing a candidate authorising body

The test is: **whose interests is this body structurally accountable to?** Applying it rules out
the obvious institutional candidates.

**A professional licensing council** (in Nigeria, the Medical and Dental Council of Nigeria) is
not a candidate. Its statutory mandate under the Medical and Dental Practitioners Act (Cap M8) is
determining standards of knowledge and skill for entry to the profession, accrediting training,
registration, and professional discipline. It regulates practitioners. It has no mandate over
patient data and no representative relationship to service users. Placing it in this role would
put the doctors' regulator in charge of service users' data — an inversion of the problem.

**Hospital or institutional management** has real custody capacity but divergent interests:
research output, partnership agreements, and reputation protection when something goes wrong.
Institutional custodianship is the model this design exists to replace. There is also a scope
mismatch — a hospital holds its own clinical records, whereas community mental health data is
generated in schools, settlements and community programmes.

**Local government** brings electoral cycles, capture risk, and in the Nigerian case the thinnest
capacity and budget of any tier. It makes the arrangement hostage to the next local election.

**A committee of data subjects** is the right shape. It is not yet a design.

## What a committee has to specify before it is a design

A committee's legitimacy comes entirely from how it is constituted. Naming one decides nothing.
The following must be answered, and they are not the author's to answer:

- **Composition.** Are data subjects a majority? The host organisation's own brief names young
  people, people with lived experience, and community health workers.
- **Selection.** Who chooses members, by what process, and who can remove them.
- **Powers.** A veto on access, or advisory input? Advisory is decoration. If the committee cannot
  refuse, there is no governance.
- **Terms and rotation.** How long members serve; how turnover avoids both capture and amnesia.
- **Conflicts of interest.** Declaration and recusal rules, especially where a requester is a
  partner, employer or funder of the host.
- **Deadlock.** What happens when the committee cannot reach the signing threshold.
- **Funding.** Who pays for members' time, and what happens to the committee after any grant ends.

## Whose decisions these are

To be explicit, because a specification written by one engineer about someone else's data can
easily read as a plan imposed on a community:

**Not the author's to decide:** committee composition and selection, the veto threshold, what
purposes are acceptable, the benefit-sharing split, what the dataset contains, and whether the
arrangement should exist at all.

**The author's to decide and defend:** whether the mechanism can enforce whatever the community
decides — that the operator cannot self-authorise, that withdrawal actually takes effect, that the
log cannot be quietly edited, and that all of this runs on infrastructure the community can afford
to keep running.

The architecture is deliberately indifferent to the policy. It should enforce an *m-of-n* threshold
without caring what *m* is, or who sits on the committee.

## Capacity, or the rubber-stamp problem

A committee of non-technical members handed technical access requests will approve them. Community
control without translation is theatre, and it is the most likely way this design fails in
practice while appearing to work.

So the review workflow is part of the system, not an afterthought: each request has to be rendered
into a plain-language statement of what the code would actually do, what it would touch, and what
would leave. If a community health worker cannot form a view from that rendering, the rendering is
the bug.

**[open]** Who produces that rendering, and whether it can be partly automated without becoming a
summary the requester effectively writes about themselves.
