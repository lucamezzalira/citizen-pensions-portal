---
id: 0001
title: Monolith over microservices for the portal backend
status: accepted
date: 2025-11-14
deciders: [head-of-engineering, cto, lead-architect]
consulted: [operations-lead, security-officer]
tags: [architecture, deployment, team-shape]
touches:
  - services/**
  - infra/**
supersedes: []
superseded-by: null
related: [0002, 0003]
---

# Monolith over microservices for the portal backend

## Context and problem

The citizen pensions portal serves roughly four million citizens
across a country with a stable pension system, and the engineering
team maintaining it is fifteen developers organised into four
domain-aligned pairs plus a platform pair. External consultants have
repeatedly proposed decomposing the portal into microservices to
"modernise" it. This ADR records the decision to stay monolithic and
the reasoning behind that choice, so the same conversation does not
have to be relitigated every time a new team member joins or a new
consultant arrives.

## Decision drivers

- Pension calculations must be transactionally consistent across
  entitlement, payment, and audit data; distributed transactions
  across services would either add complexity or reduce guarantees
- The team size makes owning fifteen services operationally
  impossible without either doubling the team or accepting an oncall
  burden that would drive attrition
- Public administration procurement makes it easy to add servers to
  an existing monolith and hard to procure a fleet of small services
  with their own infrastructure needs
- Regulatory audit requirements (ADR 0003) are easier to satisfy in
  a single deployable than across a distributed system where audit
  events themselves become subject to distributed-systems failure
  modes

## Considered options

1. Full microservices decomposition, one service per domain
2. Modular monolith with clear domain boundaries and independent
   deployability of the frontend, but a single backend deployment
3. Status quo, an implicit monolith with weak internal boundaries

## Decision outcome

Chose option 2: a modular monolith with the four domains (profile,
entitlements, documents, notifications) as internal modules with
explicit boundaries and a single deployment target for the backend.
The frontend is separately deployable. Modules communicate through
explicit interfaces and never reach into each other's persistence.

## Consequences

Positive: transactional consistency is preserved; deployment
complexity stays within what the team can operate; audit logging
lives in one place; changes touching multiple domains are trivially
atomic.

Negative: scaling is coarse-grained (the whole backend scales
together); a team wanting to use a different language for one domain
cannot; a bug in one module can affect the availability of all four.

## How the agent applies this

- New features go in the existing services under `services/<domain>/`,
  never in newly-created top-level directories
- Cross-domain reads go through the target domain's public interface,
  not through direct database access
- If a change appears to require distributed transactions, the
  design is wrong for this codebase; escalate before implementing

## Revisit when

The team grows past forty developers, or a specific domain requires
independent scaling patterns (bursty load, dedicated regional
deployment) that the monolith cannot accommodate. Neither is on the
current three-year roadmap.
