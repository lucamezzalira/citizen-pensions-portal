# Citizen Pensions Portal — Feedforward Exercise

This repo is the starting point for the feedforward exercise on day
two of the Harness and Loop Engineering workshop. It contains the
skeleton of a citizen-facing pensions portal for a public
administration, plus three ADRs recording the architectural
constraints the team works under, plus templates for the artifacts
you will produce.

## The scenario

You are joining an engineering team that maintains a citizen
pensions portal for a national public administration. The portal is
a modular monolith with four domain modules: profile, entitlements,
documents, and notifications. The wireframes for the portal are in
`mock/citizen-portal-wireframes.pdf`; read them first, they set the
context.

The team has decided to introduce a harness for AI-assisted
development. Your group's job is to write the feedforward artifacts
(the AGENTS.md files and one or more skills) that would guide an
agent working on this codebase.

## What you have to work with

- `docs/adr/` contains three ADRs recording the decisions the team
  has already made. Read them first; they are load-bearing and your
  artifacts must respect them
- `services/{domain}/README.md` describes what each domain owns
- `harness/templates/AGENTS.md.template` is the shape of an
  AGENTS.md file, with commented sections telling you what belongs
  where and which sections are mandatory
- `harness/templates/SKILL.md.template` is the same for a skill
- `mock/citizen-portal-wireframes.pdf` shows what the citizen sees

## What you will produce

Working as a group of three to five:

1. **One root `AGENTS.md`** at the repo root, orienting an agent
   who arrives at the codebase for the first time
2. **Four domain-level `AGENTS.md`** files, one at each of
   `services/{profile,entitlements,documents,notifications}/AGENTS.md`,
   carrying what the agent needs to know when working specifically
   in that domain
3. **At least one skill** under `harness/skills/{your-skill-name}/SKILL.md`,
   describing a workflow you decide is worth capturing based on the
   scenario. Pick something that spans more than one domain if you
   can; that will make the sharing conversation richer

## How to spend your time

- **10 minutes**: read the materials together (this README, the
  three ADRs, the four domain READMEs, the two templates)
- **15 minutes**: discuss and decide. What stays in the root
  AGENTS.md and what belongs in each domain file? Which skill are
  you writing? What are the ADRs already covering that you should
  not repeat?
- **50 minutes**: write
- **15 minutes**: self-review. Does anything in your AGENTS.md files
  repeat the ADRs? Does the skill have steps, or is it a rule in
  disguise? Would a colleague joining today find these useful?

## After you finish

Each group presents in three minutes, structured as:

- What did you put in the root AGENTS.md and why?
- What did you put in each domain AGENTS.md and why?
- What skill did you pick and why?
- What did you deliberately leave out?

The last question is the most important. Every group will feel the
pull to over-specify, and what you chose not to include says as
much about your reasoning as what you did.

## A note before you start

There is no single right answer here. Every group will produce
something similar in shape and different in emphasis, and the
sharing at the end is where the lesson lands because you will see
that divergence directly. What you are practising is the reasoning
that goes into these artifacts, which is the part most teams
delegate to an agent without thought.
