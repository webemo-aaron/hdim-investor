# Who Stands It Up

I am a senior healthcare-integration architect. I have worked inside the interop plumbing that
moves clinical data — IRIS/HealthShare, Mirth, Rhapsody — and I led the replacement of an
identity-matching component in a large regional exchange, moving it from a legacy MPI to
referential matching. I ran integration and analytics on a statewide exchange built on those
engines. That is where the idea for this platform came from: I watched the same quality defects
cost real money, repeatedly, with tools that were never funded to measure them.

Prior professional history is employment. It is not an HDIM customer, reference, or prospect
relationship, and no organization named or unnamed above has any commercial relationship with
HDIM.

## Why this exists

The incumbents structurally will not build it. Instrumenting the quality of data they already
move is not a direction their roadmaps or their P&Ls reward — it measures something they would
rather leave unmeasured. That leaves the problem to someone who has lived it.

The standards floor is already settled — FHIR R4, USCDI, CMS rules — and the cost of building
enterprise healthcare software has collapsed. Those two facts together are why a single
architect can now build what previously needed a funded team.

## How it is built

**Human-designed, AI-implemented.** Senior architectural judgment directs AI execution. That is
the operating model, and it is the reason one person produced the inventory below. The build is
self-funded.

*Founder background and operating model are founder-stated. The inventory that results is
measured from the repository, not asserted.*

## What exists today

Measured by script from `git ls-tree` at a single pinned commit — never from a working tree,
which stale branches pollute. Each figure carries its counting rule, because a number without one
cannot be checked:

| Figure | Count | Counting rule |
|---|---:|---|
| Gradle-registered backend service modules | 77 | Directories under the backend services tree containing a depth-1 Gradle build file, counted as distinct directories |
| Backend service directories | 80 | Depth-1 directories under the same tree, Gradle or not; three are Python services with their own build |
| Backend controller classes | 250 | Scoped to the platform backend; repository-wide the figure is 258 |
| Liquibase changelog files | 481 | Changelog **files**; 61 of those are per-module aggregators, leaving 420 migration changelogs |

Corroborated independently: the Gradle settings file registers 77 service includes, which agrees
with the first figure. These figures **supersede** an earlier published set of 59 / 171 / 362,
which was a stale snapshot and understated the platform by roughly thirty percent.

## What I do on a deployment

I lead it, and I build the customer's own capability while I do. The delivery method is
documented and repeatable: install, configuration, validation, migration, cutover, post-go-live,
each lane doing the work and teaching the team to do it next. The goal is that the customer's IT
team can operate, monitor and maintain the deployment without me.

That is a **designed goal, not a track record.** No customer team has yet run it. It is the
commitment I am accountable for, and the register of dated targets behind it is part of the NDA
materials.

## What I do not claim

- No customer deployments, signed pilots, or reference customers.
- The platform runs in a **DEMO** posture: synthetic data, no production patient data.
- Readiness is self-assessed. No independent audit or certification has been performed.
- Named integrations beyond the engines above are roadmap, labelled as such wherever they appear.

Stating these is the point. A package that lists only strengths gives a reader no way to
calibrate the ones that are real.

## What is under NDA

Code-validated architecture summaries, readiness and validation summaries, the gap register with
its closure paths, and the dated target register are NDA-protected rather than public-safe. They
exist, they are written, and they are available under a confidentiality agreement — ask me and I
will share the access details directly.
