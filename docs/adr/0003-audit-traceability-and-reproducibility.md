---
id: 0003
title: Audit traceability and calculation reproducibility
status: accepted
date: 2025-09-30
deciders: [head-of-engineering, legal-counsel, dpo, ministry-liaison]
consulted: [operations-lead, security-officer]
tags: [audit, compliance, calculation, regulatory]
touches:
  - services/**
  - infra/audit-log/**
supersedes: []
superseded-by: null
related: [0001, 0002]
---

# Audit traceability and calculation reproducibility

## Context and problem

National pension law and the applicable EU regulations require that
every decision the system communicates to a citizen (their
entitlement, the calculated amount, a denial, an adjustment, a
correction) be reproducible on demand from the inputs and the rules
that produced it, months or years after the fact. They also require
that every change to a citizen's record be attributable to a person
or a system event, with a timestamp, and that the audit trail itself
be tamper-evident. These are hard constraints and they shape almost
every other architectural choice this system will ever make.

## Decision drivers

- Regulatory: pension law explicitly requires reproducibility of
  historical decisions and non-repudiation of record changes
- Operational: without reproducibility, citizen complaints become
  unwinnable arguments about what the system did months ago
- Trust: citizens have the legal right to challenge any decision and
  the state has the legal obligation to defend it with evidence

## Considered options

1. Standard application logging supplemented by database change
   tracking, replayed against best-effort rule versions
2. Immutable append-only audit log for all citizen-affecting events,
   plus a versioned rule engine that records the rule-set version
   used for each calculation, plus event-sourced projections for
   citizen-visible state

## Decision outcome

Chose option 2. All citizen-affecting changes are recorded as events
in an append-only audit log with cryptographic chaining so tampering
is detectable. All calculations that produce a citizen-visible number
take a `ruleSetVersion` as input and record the version alongside the
inputs and the output. Historical decisions can therefore be replayed
by loading the recorded inputs and the recorded rule-set version and
executing the calculation deterministically.

## Consequences

Positive: legal defensibility of every decision; citizen inquiries
have concrete answers; changes to calculation rules can be rolled
out without breaking the reproducibility of past decisions; the
audit log itself is a first-class artifact rather than a
best-effort byproduct.

Negative: no in-place updates to citizen records; storage grows
faster than in a mutable model; rule engine changes require
disciplined versioning; a bug in the rule engine that changes past
behaviour is a serious incident requiring rollback and rerun.

## How the agent applies this

- Any code that changes citizen-visible state must write to the
  audit log with a specific event type, the actor (person or system
  identifier), the timestamp, and the change payload
- Any code that calculates a benefit amount, an entitlement, or any
  citizen-visible number must accept a `ruleSetVersion` parameter
  and record it alongside the calculation output
- Any code that reads a historical decision must do so through the
  audit log, never by reconstructing what the current rules would
  produce today
- The calculation engine must be deterministic: same inputs plus
  same rule-set version equals same output, always

## Revisit when

Never. This is a regulatory constraint and cannot be relaxed by
engineering decision alone. If law changes, this ADR is superseded
by a new one recording the changed constraint; the mechanism does
not go away.
