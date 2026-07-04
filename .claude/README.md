# What settings.json actually does

Claude Code sends an API request (`POST /v1/messages`) on every turn.
Each key in `settings.json` sets one field of that request. This table
spells out every value, including the ones that would otherwise be
silent defaults, so nothing happens implicitly.

| settings.json key | Field of the API request it controls | Value here | What happens if the key is absent |
|---|---|---|---|
| `model` | `model` | `claude-opus-4-8` | Your account's default model (varies by plan) |
| `effortLevel` | `output_config.effort` | `high` | Claude Code picks its own default (xhigh on Opus-class models — more expensive) |
| `alwaysThinkingEnabled` | `thinking` | `true` -> `{"type": "adaptive"}` | Same (defaults to on), written here so it is visible |
| `autoCompactEnabled` | server-side history compaction | `true` | Same (defaults to on), written here so it is visible |
| `fastMode` | `speed` (premium-priced fast output) | `false` | Same (defaults to off) — explicit so nobody enables it by accident |
| `env.CLAUDE_CODE_MAX_OUTPUT_TOKENS` | `max_tokens` | `64000` | Harness decides per request. 64000 gives thinking room at high effort; only tokens actually generated are billed |

## Things that have NO settings key (done automatically every request)

| API request field | Who handles it |
|---|---|
| `system=[... cache_control: ephemeral ...]` | Claude Code places cache breakpoints on the system prompt and history automatically — cached input bills at ~0.1x. Verify with `/cost`: from turn 2 on, most input should show as cache reads |
| Thinking depth per task | Adaptive thinking: the model itself spends little on easy asks, more on hard ones. No per-task tuning needed |
| `messages=[...]` | The conversation itself |

## The only manual habits worth having

1. `/clear` when you switch to an unrelated task (history resend is the
   main everyday cost).
2. `/cost` if you want to see what a session spent.

Everything else in this file is set-and-forget.
