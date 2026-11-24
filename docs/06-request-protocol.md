# Request protocol

How a request for access is expressed, made intelligible, and authorised. Design stage; items
marked **[open]** are listed in [90-open-questions.md](90-open-questions.md).

## Two artifacts, not one

A request needs to be readable by a committee of non-specialists *and* precise enough to compile
to SQL. One format cannot be both without becoming either unreadable or unexecutable. So there are
two artifacts, and only one of them is authoritative:

- **The specification** — canonical, machine-readable, the thing that compiles and executes.
- **The render** — plain language, *generated from* the specification, never hand-authored.

The second point is load-bearing. If a human writes the description separately it drifts from the
query, and a committee approving a description that does not match what executes is worse than a
committee that does not exist.

## Format

The requester authors YAML. At submission it is canonicalised to JSON per
[RFC 8785](https://www.rfc-editor.org/info/rfc8785/) (JSON Canonicalization Scheme), so that the
bytes being signed are stable and reproducible.

A specification states: which tables, which columns, which row filters, what time window, and the
stated purpose. It does not contain code.

### Why not sign the analysis code, or the query IR

Signing a program would be meaningless: a program's data reach is not determinable from its bytes,
so a signature over it authorises nothing in particular.

Signing ehrQL's serialised query model is also wrong, even though we compile to it (see
[08-reuse-and-dependencies.md](08-reuse-and-dependencies.md)). Their serialiser carries execution
context — file paths, module details, error types — because it is transport between their own
components, not a description of a data scope. A signature over it would cover incidental
implementation detail and break whenever their internals change.

The canonical specification is the only artifact whose meaning is fully determined by its bytes.
That is why it is the one that gets signed.

## What a signature covers

Each committee member signs a payload binding the specification to what they were actually shown:

| Field | Why |
|---|---|
| Specification hash | The scope that will execute |
| Render hash | The exact text the signer read |
| Renderer version | Which code produced that text |
| Grant terms | Expiry, output limits, permitted artifact position |

So a signature means: *I was shown this text, it was produced from this specification by this
version of the renderer, and I authorise it.* If the renderer changes later, what the committee saw
remains provable.

Signing the specification alone would attest to something nobody read. Signing the render alone
would authorise prose that is not what executes. Binding them is the only honest option.

## The renderer is security-critical

The renderer is part of the trusted computing base. A bug in it silently defeats the entire
governance layer: a faulty or hostile renderer can display "age band and presenting complaint"
while the specification asks for full names, and every signature over it remains cryptographically
valid.

Consequences:

- **Deterministic and pure.** The same specification must always produce the same text, so anyone
  can re-render and compare.
- **Versioned**, with the version recorded inside the signed payload.
- **Independently re-runnable**, so a third party can verify a render matches its specification. A
  second, independently written verifier is specified in
  [08-reuse-and-dependencies.md](08-reuse-and-dependencies.md).
- **Fail closed.** If the renderer meets a construct it cannot describe, the request becomes
  *unsignable* rather than being rendered with that clause silently omitted. Silent omission is the
  fatal bug: a committee approving a specification containing a clause nobody told them about.

Fail-closed rendering caps the expressiveness of the specification at *what can be explained to a
community health worker*. That is a real constraint on what requesters may ask for, and it is the
right one. The schema and the renderer therefore evolve together; neither may advance alone.

## Sequence

1. The requester authors `request.yaml`.
2. It is canonicalised to JSON (RFC 8785).
3. The renderer produces the plain-language view. If it cannot, the request stops here.
4. The committee reads the render. The specification and the generated SQL are available to anyone
   who wants them.
5. Each authorising member signs the bound payload above.
6. Once *m* of *n* signatures are collected, a grant is issued.
7. The job runner verifies the signatures, re-renders from the signed specification and checks the
   result against the approved render hash, and only then derives SQL and extracts.

Step 7 closes the loop: execution refuses if the approved render does not reproduce from the
specification being run.

SQL is always derived at execution time from the signed specification. It is never submitted
alongside it, or a requester could have one thing authorised and ship another.

## Two gates, two competencies

| Gate | Who | What they decide |
|---|---|---|
| Authorisation | The committee | What data may be touched, going in |
| Disclosure review | Trained output checkers | What results may leave, coming out |

The requester's analysis code sits between the two. It is hashed and recorded in the audit log, so
the trail shows exactly what ran — but the committee does not approve its semantics. They cannot,
and should not be asked to. Nothing about the code is safe because it was signed; it is bounded by
the data it can reach and reviewed by what it emits.

**[open]** The JSON Schema for the specification. Key custody for signers, and recovery when a key
is lost. Values of *m* and *n*, and whether some request classes need a higher threshold. The
renderer's vocabulary — how filters are phrased so a lay reviewer genuinely understands them, and
who validates comprehensibility rather than mere accuracy.
