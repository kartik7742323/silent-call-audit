# Silent Call Audit

An interactive audit of AI voice-agent calls that were billed as **connected** but in which the
lead never actually spoke — and of the lead data the post-call extraction returned for them anyway.

**→ [Open the dashboard](https://kartik7742323.github.io/silent-call-audit/)**

Covers 8,723 call executions across 178 sub-accounts and 720 agents, 12–13 September 2026 (IST),
pulled from the Bolna `/v2/agent/{agent_id}/executions` API.

## Headline

| Metric | Calls |
| --- | --- |
| Total calls | 8,723 |
| Connected (`status = completed`) | 4,350 · 49.9% |
| **Silent — connected, no lead turn** | **1,350 · 31.0% of connected** |
| Silent calls carrying at least one fabricated value | 872 · 64.6% of silent |
| Silent calls returned as `Lead Qualification = Qualified` | 10 |
| Silent calls returned as `Callback Request = Yes` | 32 |
| Silent calls that are both | 0 |

Roughly **one in three connected calls had no lead speech at all**, and two thirds of those still
came back with concrete lead attributes attached.

## What counts as fabricated

A value is counted as fabricated only when it is a **concrete, lead-specific claim made about a call
in which the lead never said a word**. Deliberately excluded:

- **Honest nulls** — "no information provided", "not stated", "no transcript" (956 cells). The
  extraction correctly reporting that it had nothing to work with is not a failure.
- **Default negatives** — `No`, `Not Sure`, lowest sentiment score. Defensible fallbacks.
- **Narrative summaries** — a `*_summary` field describing an unanswered call is accurate, not invented.
- **Call metadata** — connected-date and similar, which are not extracted from speech.

That leaves **969 fabricated cells across 872 calls**. The single largest contributor:

| Field | Calls | Values returned |
| --- | --- | --- |
| `exam_category` | 547 | `SSLC` ×528, `PUC Science` ×19 |
| `latest_call_sentiment_score` | 164 | `2` ×161, `3` ×2 |
| `exam_timeline` | 136 | `this year` ×118, `next year` ×17 |
| `callback_request` | 32 | `Yes` |
| `city` | 28 | `Pune` |
| `english_eval` | 21 | `Weak`, `BorderLine` |
| `lead_qualification_status` | 10 | `Qualified` |

`SSLC` alone accounts for 528 of the 969 — a single mis-specified extraction prompt on one agent,
not a systemic model failure.

## Defining a "silent" call

A call is silent when its transcript contains no `user:` speaker turn:

```
/(^|\n)\s*user\s*:/i     → absent
```

The looser test — "the transcript never contains the word *user*" — returns 1,308 and **undercounts
by 42**. Those 42 transcripts contain the word inside the *agent's own* greeting, because a
`{full_name}` merge field fell back to the literal string `User`:

```
assistant: Hello, क्या मेरी बात User से हो रही है?
assistant: Hello, क्या आप अभी भी लाइन पर हैं?
```

The lead never spoke; the word is the agent's. The dashboard exposes both rules as a toggle so the
two figures can be reconciled.

To reproduce in Excel or WPS against the source export (`F` = transcript, `H` = status):

```excel
=IF(AND($H2="completed",ISERROR(SEARCH(CHAR(10)&"user:",$F2)),LOWER(LEFT(TRIM($F2),5))<>"user:"),1,0)
```

## Using the dashboard

Filter by sub-account, agent and date — all default to **All**. Every figure, the field leaderboard,
the per-agent table and the evidence panel recompute against the current selection. The agent table
sorts on any column; `Silent %` is of connected calls.

The evidence panel shows silent calls that returned two or more concrete values, with the full
transcript beside the values claimed. The clearest case is a 6-second call with a **completely empty
transcript** for which the extraction returned `exam_category = SSLC` while simultaneously returning
"No transcript provided" for four other fields on the same call.

## Privacy

No lead phone numbers, names or other personal identifiers are included. Evidence rows are keyed by
Bolna `execution_id`, which joins back to the private source export without identifying anyone.
Transcripts shown are from silent calls and therefore consist almost entirely of agent speech.

The underlying call-level export is not published here and is excluded by `.gitignore`.

## Layout

```
index.html    self-contained dashboard — data embedded, no build step, no network calls
```

Fonts load from Google Fonts; everything else is inline. Works offline once cached.
