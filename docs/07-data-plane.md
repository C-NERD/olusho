# Data plane

How data moves between the store and the execution sandbox. Design stage; items marked **[open]**
are listed in [90-open-questions.md](90-open-questions.md).

## The constraint everything else follows from

The requester's code must never hold database credentials and must never hold a network socket.

Both follow from one observation. If the sandbox can reach the database, then the requester's code
holds credentials — and read-only credentials still permit `SELECT *` across everything the role
can see. **Read-only protects integrity, not scope.** Those are different properties and conflating
them is the easy mistake.

And once the sandbox can open a connection at all, egress filtering is the only thing between
untrusted code and the internet. That is too weak a control to rest the design on.

## Shape

Three roles, on separate hosts:

| Host | Runs | Trusted? |
|---|---|---|
| Data store | PostgreSQL (primary + read replica) | Yes |
| Runner | Extraction, authorisation checks, output collection | Yes |
| Sandbox | The requester's analysis code | **No** |

Separating the store from the sandbox is about blast radius, not performance: the machine running
untrusted code should not be the machine holding the data.

## Flow

1. The runner verifies the grant (see [06-request-protocol.md](06-request-protocol.md)).
2. The runner — not the sandbox — connects to the read replica over mTLS and extracts exactly the
   approved slice.
3. The extract is materialised as a file (Arrow or CSV).
4. The sandbox starts with **no network namespace at all**, the extract mounted read-only, and one
   writable output directory.
5. The requester's code reads the local file. It has no credentials, no socket, and cannot see a row
   outside the approved slice.
6. The output directory is collected and enters disclosure review.

Because extraction is a bounded one-time read against a replica, the database is under no load at
all while analysis runs. That addresses the performance concern more thoroughly than host
separation alone.

## PostgreSQL hardening

The specification is enforced twice — once when generating the extract query, and again at the
database, so a bug in the compiler is not sufficient to exceed the grant:

- A **role per grant**, created at extraction time and dropped afterwards.
- **Column-level `GRANT`s**, so unapproved columns are unreadable rather than merely unused.
- **Row-level security policies** carrying the approved filter.
- `statement_timeout`, `work_mem` ceilings and connection limits, so a bad extract cannot take the
  store down.
- **pgaudit** logging every statement the runner issues. The audit log then records what was
  actually read, not merely what was requested.
- Extractions run against a **read replica**; analytical reads never touch the primary.

mTLS sits between the runner and PostgreSQL. It is deliberately *not* between the sandbox and
PostgreSQL — mutual authentication is worth little when the adversary is inside the client.

## Sandbox

- No network namespace. Not firewalled — no route.
- Input: the extract, mounted read-only.
- Output: a single writable directory, which is the only thing collected.
- CPU, memory and wall-clock ceilings.
- No interactive shell; raw records are never rendered to the requester.

For a training job the artifact leaving the sandbox is model weights, which hits the governed
weight-export condition in [03-revocation.md](03-revocation.md). Same queue, same review, same
terms.

## Synthetic mirror

The same specification generates a synthetic extract of identical shape. Requesters develop and
test against it, and it is the only data used in demonstrations or prototyping.

This has a practical consequence worth stating: the whole system can be built, demonstrated and
criticised before any real data or any ethics approval exists.

## The limit

None of this addresses leakage through the output channel. Bounding what code can *see* is a real
gain, but code with legitimate access to a slice can still encode that slice into its results.
Disclosure review remains load-bearing and unchanged — see
[05-threat-model.md](05-threat-model.md) §1.

**[open]** Extract format and schema versioning. Sandbox runtime (container versus VM) and who
patches it. Whether row-level security or compiler-side filtering is authoritative when they
disagree. Key and certificate rotation for the runner-to-database channel.
