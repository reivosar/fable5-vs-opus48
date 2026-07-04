# Claude Fable 5 vs Claude Opus 4.8 — Where the Difference Actually Lives

Date: 2026-07-05
Companion diagram: `images/fable5-vs-opus48-diff.svg`

## 1. Executive summary

Claude Fable 5 is not a tuned or prompted variant of Opus 4.8. It is a separate,
higher model tier (the new "Mythos class") that sits above the Opus family.
The gap between the two has two components:

1. **Behavioral defaults** — things Opus 4.8 *can* do but does not do unless
   instructed (search first, delegate to subagents, self-verify, proceed
   autonomously). This part **can be closed with prompting and API settings**.
2. **Raw capability** — the reasoning ceiling and long-horizon coherence
   (not degrading over hundreds of steps). This part **cannot be closed with
   prompting**, because no amount of instruction text adds compute.

Practical consequence: route routine-to-medium work to a well-configured
Opus 4.8 (half the price), and reserve Fable 5 for the hardest reasoning and
the longest autonomous runs.

## 2. What is publicly known

Anthropic does not publish parameter counts, training data volume, or
architecture details for either model. The following is what is stated in
official documentation and inferable from the API surface.

| Dimension | Claude Opus 4.8 | Claude Fable 5 |
|---|---|---|
| Tier | Opus family (most capable Opus) | Mythos class — a new tier above Opus |
| Model ID | `claude-opus-4-8` | `claude-fable-5` (same model as `claude-mythos-5`) |
| Pricing (per MTok) | $5 input / $25 output | $10 input / $50 output (exactly 2x) |
| Context / max output | 1M / 128K | 1M / 128K |
| Tokenizer | Opus 4.7 tokenizer | Same tokenizer — token counts roughly identical |
| Thinking | Adaptive, optional; omitting `thinking` runs without thinking; `{type: "disabled"}` accepted | Always on at the architecture level; `{type: "disabled"}` returns HTTP 400; only omission or `{type: "adaptive"}` accepted |
| Raw chain of thought | Summarized thinking available | Never returned; `display: "summarized"` gives a summary, default is omitted |
| Safety | Standard | Additional safety classifiers; `stop_reason: "refusal"` possible; server-side fallbacks to Opus 4.8 recommended |
| Data retention | Standard options incl. ZDR | Requires 30-day retention; ZDR orgs get 400 on every request |
| Typical turn length | Seconds to minutes | Many minutes on hard tasks is normal (15-minute single requests occur) |

The 2x price at identical context/output limits is the strongest public signal
that inference genuinely costs more — i.e. a larger model and/or more inference
compute, not a prompt-level difference. A same-weights variant would not be
priced differently.

## 3. Where the capability gap lives

### 3.1 Reasoning ceiling

Fable 5 solves problems at the top of the difficulty range that Opus 4.8 does
not reach even at `effort: "max"`. Documented strength areas:

- First-shot implementations of well-specified systems
- End-to-end enterprise deliverables (financial analysis, spreadsheets,
  slides, documents)
- Code review and debugging (higher recall and precision on real bugs)
- Vision on dense or degraded images — explicitly trained to use bash and
  crop tools to preprocess flipped/blurry/noisy inputs itself
- Navigating ambiguity without thrashing

### 3.2 Long-horizon coherence (the structural difference)

Long autonomous tasks are governed by error compounding: if the per-step
success rate is p, the probability of completing an n-step task cleanly is
roughly p^n. Small differences in p become enormous differences at scale:

| Per-step success | 100 steps | 500 steps | 1000 steps |
|---|---|---|---|
| 99.7% | 74% | 22% | 5% |
| 99.9% | 90% | 61% | 37% |

(Illustrative numbers, not published benchmarks.) This is why the felt
difference is "completes an overnight run without human correction" versus
"drifts after a couple of hours". Fable 5's training explicitly emphasizes:

- Long-horizon autonomous execution (overnight-run-class tasks)
- Sustained communication with long-running parallel subagents and peer agents
- Writing and using file-based memory across a run
- Self-verification loops during long builds

### 3.3 Architectural stance on thinking

On Fable 5, reasoning is not an optional feature: the API rejects disabling
it. This reflects a design in which thinking is integral to how the model
operates, whereas on Opus 4.8 it is a mode.

## 4. What prompting and settings CAN close on Opus 4.8

Opus 4.8 follows instructions very well, so Fable-like *behavior* (not
capability) is largely recoverable.

### 4.1 API settings

```python
client.messages.create(
    model="claude-opus-4-8",
    max_tokens=64000,                   # headroom for xhigh effort
    thinking={"type": "adaptive"},      # must be explicit — omission = no thinking
    output_config={"effort": "xhigh"},  # coding/agentic; sweep medium..xhigh per route
    messages=[...],
)
```

- Give the full task specification in one well-specified first turn.
- For agentic loops, consider Task Budgets (beta `task-budgets-2026-03-13`,
  minimum 20,000 tokens) so the model paces itself.

### 4.2 System prompt levers

Opus 4.8 under-reaches by default for search, subagents, memory, and custom
tools, and asks the user more often than Fable 5 would. All of these respond
to explicit instruction:

- **Autonomy** — decide minor choices without asking; only stop for scope
  changes or destructive actions; do not end a turn on a plan or a promise.
- **Act, don't overplan** — when enough information exists, act; recommend
  instead of surveying options.
- **Grounded progress** — audit every progress claim against a tool result;
  report failures verbatim.
- **Capability triggering** — state *when* to search, *when* to delegate to
  subagents, *when* to read/write the memory file; repeat the trigger
  condition in each tool's own `description` (measurable lift).
- **Self-verification** — build and run a checking method; inferred success
  is not success.
- **Scope discipline** — simplest thing that works; no unrequested
  refactoring or speculative abstractions.

Avoid aggressive phrasing ("CRITICAL: YOU MUST") — on Opus 4.8 it causes
overtriggering; plain conditional statements work better.

## 5. What prompting CANNOT close

1. **The reasoning ceiling.** Problems above Opus 4.8's capability stay
   unsolved regardless of effort level or prompt engineering.
2. **Multi-hour autonomous coherence.** The per-step reliability that makes
   overnight runs survive is a property of the weights. Mitigation on
   Opus 4.8: split long tasks into checkpointed segments with human or
   scripted verification between them.
3. **Degraded-vision self-preprocessing.** Fable 5 was trained to fix bad
   images with tools on its own; Opus 4.8 needs the pipeline to do it.

## 6. Cost-aware routing recommendation

| Workload | Model | Rationale |
|---|---|---|
| Routine and medium-difficulty tasks | Opus 4.8 + settings in §4 | Half the price; behavioral gap closable |
| Hardest reasoning, one-shot builds of complex systems | Fable 5 | Above Opus ceiling |
| Multi-hour autonomous runs | Fable 5 | Long-horizon coherence is not promptable |
| Fable 5 in production | Add server-side fallbacks to Opus 4.8 (`server-side-fallback-2026-06-01` beta) | Safety classifiers can decline benign adjacent work |

## 7. Caveats — what is NOT public

Parameter counts, training compute, dataset composition, architecture changes,
and the lineage relationship to Opus 4.8 (from-scratch vs derived) are all
undisclosed. The honest limit of this report is: "a higher tier with more
training and inference compute, specifically trained for long-horizon
autonomous work." Official announcement:
https://www.anthropic.com/news/claude-fable-5-mythos-5
