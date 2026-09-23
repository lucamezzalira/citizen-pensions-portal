---
id: 0002
title: Kafka to data lake for overnight calculations, with PII controls
status: accepted
date: 2026-02-08
deciders: [head-of-engineering, data-lead, security-officer]
consulted: [dpo, platform-team]
tags: [architecture, data, privacy, batch]
touches:
  - services/entitlements/**
  - services/documents/**
  - services/notifications/**
  - infra/kafka/**
  - infra/data-lake/**
supersedes: []
superseded-by: null
related: [0001, 0003]
---

# Kafka to data lake for overnight calculations, with PII controls

## Context and problem

The portal is transactional and citizen-facing during the day, and
its data feeds several overnight batch processes: actuarial
projections, aggregate reporting for the ministry, and reconciliation
against the tax authority's records. Running these calculations
against the transactional database would degrade daytime performance
and would violate the principle that reporting workloads should not
share resources with citizen-facing workloads. The data protection
officer also requires that personal data leaving the transactional
system be minimised and pseudonymised at the boundary, because a data
lake is a broader-access system than the transactional store.

## Decision drivers

- Overnight calculations must not compete with daytime transactional
  load
- PII exposure widens as data moves from the transactional store into
  systems used by more people, so the boundary is the right place to
  minimise
- The batch results must be joinable back to specific citizens when
  required (individual statements, specific inquiries), but the
  aggregate calculations themselves should not need identity
- The stream must be replayable so a data lake outage does not lose
  events

## Considered options

1. Nightly database replication to a read replica used for batch
2. Direct queries against the transactional database, off-hours
3. Domain events published to Kafka, consumed by a data lake loader,
   with PII pseudonymised at the publish boundary

## Decision outcome

Chose option 3. Each of the four domains publishes domain events to
Kafka when state changes: entitlement calculated, payment issued,
document generated, notification sent. The events carry the
information the batch calculations need, with citizen identity
replaced by a pseudonymous stable identifier at the publish boundary.
The mapping from pseudonym to real identity is held only in the
transactional store, and reads that need to be re-identified must
join back through it.

## PII handling at the boundary

The published event schema uses `citizenRef` (a pseudonymous string
derived from the citizen ID plus a rotating salt) rather than the
citizen ID itself. Names, addresses, tax IDs, dates of birth, and
contact details are not published; if a batch calculation needs any
of these it must lookup by `citizenRef` against the transactional
store through an audited API. The salt rotates monthly; retention on
Kafka is capped at ninety days, so the salt is always current for any
data still in-broker.

## Consequences

Positive: the transactional store is not queried by batch
workloads; PII exposure to the data lake is minimised by design; the
event log is replayable and audit-friendly; the pattern generalises
to future batch needs without a new integration each time.

Negative: two-system operational cost; downstream calculations
needing PII must make an extra call and be logged for doing so; the
rotating salt requires operational discipline; a bug in the
pseudonymisation code would leak PII into a wider system.

## How the agent applies this

- Any code publishing to Kafka must go through the pseudonymisation
  helper, never publish citizen IDs directly
- Any code adding a field to an event schema must confirm the field
  is not PII before it lands, and must add it to the DPO review list
  otherwise
- Any code consuming from Kafka should treat `citizenRef` as opaque
  and never attempt to reverse it locally

## Revisit when

The batch calculation latency requirement drops below four hours (at
which point the event-driven model is too slow and a different
architecture may be needed), or the data lake becomes the primary
system for a workload currently served transactionally.
