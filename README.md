# Claude Fable 5 vs Claude Opus 4.8

An analysis of where the difference between Claude Fable 5 (Mythos-class tier)
and Claude Opus 4.8 actually lives — what is closable with prompting and API
settings, and what is a property of the model itself.

![Fable 5 vs Opus 4.8 — where the difference lives](images/fable5-vs-opus48-diff.svg)

## Contents

| File | Description |
|---|---|
| [REPORT.md](REPORT.md) | Detailed report: public facts, capability gap analysis, prompting levers, routing recommendation |
| [images/fable5-vs-opus48-diff.svg](images/fable5-vs-opus48-diff.svg) | One-page visual summary of the differences |

## TL;DR

- Fable 5 is a separate, higher model tier — not a tuned Opus 4.8. The 2x
  price at identical context/output limits reflects genuinely higher
  inference cost.
- The gap splits into two parts:
  - **Behavioral defaults** (search-first, subagent delegation,
    self-verification, autonomy) — closable on Opus 4.8 with explicit
    prompting plus `thinking: adaptive` and `effort: high/xhigh`.
  - **Raw capability** (reasoning ceiling, multi-hour autonomous coherence)
    — not closable; no instruction text adds compute.
- Long-horizon coherence dominates in practice because per-step errors
  compound exponentially over hundreds of steps.
- Route routine work to a well-configured Opus 4.8; reserve Fable 5 for the
  hardest reasoning and the longest autonomous runs.

## Caveats

Parameter counts, training compute, and architecture details are not public.
This analysis is based on official documentation and the API surface as of
July 2026. Official announcement:
https://www.anthropic.com/news/claude-fable-5-mythos-5
