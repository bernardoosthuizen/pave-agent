# PAVE — An Autonomous Preflight Briefing Agent

*An experiment in whether AI agents can do the tedious half of preflight
briefing and risk assessment — and hand a pilot something worth reading.*

![status: experimental](https://img.shields.io/badge/status-experimental-orange)
![license](https://img.shields.io/badge/license-MIT-green)

---

## What this is

The PAVE checklist — **P**ilot, **A**ircraft, **e**n**V**ironment,
**E**xternal pressures — is a risk assessment framework to help pilots gather relevant information
and address the details that matter. Using this framework it is the aim 
that PAVE Agent provides crew with accurate information to assist in decision making 
and risk mitigation.

The synthesis isn't intellectually hard. It's just *tedious and
heterogeneous* — which is exactly the shape of problem AI agents are
supposed to be good at.

**So: this project runs on a schedule, notices a flight on my calendar,
gathers the scattered public data, reasons about it across the four PAVE
domains, and emails me a briefing the night before I fly.** No prompting,
no dashboard to remember to open. It shows up.

```
Subject: [PAVE] KORD→KBOS 18:00Z — MODERATE (low ceilings, GDP risk)
```

## What this is *not*

**This is an educational project. Do not use it for operational flight
planning.**

It is not a replacement for official weather briefings, ForeFlight,
Leidos, or your own judgment. It is not certified, not verified, and not
affiliated with any aviation authority. It will sometimes be confidently wrong.

Three constraints follow from that, and they're baked into the design:

- **It advises; it never decides.** The agent produces a risk *assessment*
  with the factors it found. It does not issue go/no-go calls. That
  authority belongs to the pilot in command and this project won't
  pretend to borrow it.
- **Every claim is traceable.** Each risk driver carries its source, so
  you can check the raw METAR rather than trusting a paraphrase.
- **It reports its own uncertainty.** When the agent can't parse a flight
  or can't reach a data source, the briefing says so plainly instead of
  filling the gap with plausible-sounding fiction.

If aviation software has one cultural rule worth importing into AI, it's
that a system which hides its own failure modes is more dangerous than
one that has none.

## Why I built it

I'm learning agent development, and I wanted a problem with real
constraints rather than another chatbot wrapper. Aviation is unusually
good for this: the data is public, the domain punishes vagueness, and
there's a genuine correctness standard.

This repo is the artifact of that learning. Alongside the code, the
[**Design decisions**](#design-decisions) section documents the tradeoffs
I made and, more usefully, the ones I got wrong first. The central one:
**the LLM handles judgment, and plain deterministic Python handles
everything else** — scheduling, fetching, formatting, delivery. That
boundary makes the system debuggable, and it's the lesson I'd keep if I
threw out all the rest.

## How it works

Four specialist agents — one per PAVE domain — investigate in parallel. A
supervisor weighs their findings, resolves conflicts, and produces a
structured assessment that gets rendered to email.

```mermaid
flowchart TD
    CRON[cron: daily] --> CAL[Read calendar]
    CAL --> EX[Extract flight details]

    EX --> P["<b>P</b>: Pilot<br/> - currency<br/> - validity<br/> - flight and duty "]
    EX --> A["<b>A</b>: Aircraft<br/> - equipment<br/> - performance<br/> - airworthiness"]
    EX --> V["<b>V</b>: enVironment<br/> - weather<br/> - terrain<br/> - airspace/route<br/> - aerodromes"]
    EX --> E["<b>E</b>: External<br/> - on time performance"]

    P --> SUP[Supervisor<br/>synthesis]
    A --> SUP
    V --> SUP
    E --> SUP

    SUP --> ASSESS[FlightRiskAssessment]
    ASSESS --> MAIL[Render & email]
```

### Risk Assessment Matrix
In progress . . .

> ⚠️ **Educational project. Not for operational flight planning.** Always
> obtain an official weather briefing and exercise your own judgment as
> pilot in command.
