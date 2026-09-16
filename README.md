# Silent Call Audit

An interactive audit of AI voice-agent calls that were billed as **connected** but in which the
lead never actually spoke — and of the lead data the post-call extraction returned for them anyway.

**→ [Open the dashboard](https://kartik7742323.github.io/silent-call-audit/)**

Covers 35,766 call executions across 178 sub-accounts and 722 agents, 12–15 September 2026 (IST),
pulled from the Bolna `/v2/agent/{agent_id}/executions` API.

## Headline

| Metric | Calls |
| --- | --- |
| Total calls | 35,766 |
| Connected (`status = completed`) | 17,489 · 48.9% |
| **Silent — connected, no lead turn** | **7,204 · 41.2% of connected** |
| Silent calls carrying at least one fabricated value | 1,787 |
| Silent calls returned as `Lead Qualification = Qualified` | 14 |
| Silent calls returned as `Callback Request = Yes` | 72 |
| Silent calls that are both | 0 |

## The rate is getting worse

The silent share of connected calls rose sharply over the four days:

| Date (IST) | Total | Connected | Silent | Silent % of connected |
| --- | --- | --- | --- | --- |
| 12 Sep | 6,529 | 3,339 | 1,002 | 30.0% |
| 13 Sep | 2,194 | 1,011 | 348 | 34.4% |
| 14 Sep | 9,029 | 3,882 | 1,968 | **50.7%** |
| 15 Sep | 18,014 | 9,257 | 3,886 | **42.0%** |

On 14 September, **more than half** of all calls billed as connected had no lead speech in them at
all. Volume rose 4× over the same window, so this is not a small-sample artefact.

## What counts as fabricated

A value is counted as fabricated only when it is a **concrete, lead-specific claim made about a call
in which the lead never said a word**. Deliberately excluded:

- **Honest nulls** — "no information provided", "not stated", "no transcript", "call did not
  proceed". The extraction correctly reporting that it had nothing to work with is not a failure.
- **Default negatives** — `No`, `Not Sure`, lowest sentiment score. Defensible fallbacks.
- **Narrative summaries** — a `*_summary` field describing an unanswered call is accurate, not invented.
- **Call metadata and dial-attempt flags** — connected-date, `mio_ai_voice_attempted`. The attempt
  genuinely happened regardless of whether anyone spoke.

That leaves **2,015 fabricated cells across 1,787 calls**:

| Field | Calls | Values returned |
| --- | --- | --- |
| `latest_call_sentiment_score` | 838 | `2` ×834, `3` ×3, `4` ×1 |
| `exam_category` | 548 | `SSLC` ×528, `PUC Science` ×19 |
| `exam_timeline` | 201 | `this year` ×177, `next year` ×24 |
| `score` | 97 | `1` ×97 |
| `city` | 86 | `Pune` ×86 |
| `callback_request` | 72 | `Yes` ×72 |
| `english_eval` | 54 | `Weak` ×43, `BorderLine` ×11 |
| `interested_course` | 39 | `Undergraduate` ×30, `Masters` ×9 |
| `lead_qualification_status` | 17 | `Qualified` ×14, `Not Qualified` ×3 |

Two patterns dominate. `exam_category = SSLC` (528 calls) is a single mis-specified extraction
prompt on one agent. `latest_call_sentiment_score = 2` (834 calls) is a sentiment judgement of a
conversation that never happened — scoring silence as mildly negative rather than as absent.

## Defining a "silent" call

A call is silent when its transcript contains no `user:` speaker turn:

```
/(^|\n)\s*user\s*:/i     → absent
```

The looser test — "the transcript never contains the word *user*" — returns 7,141 and **undercounts
by 63**. Those transcripts contain the word inside the *agent's own* greeting, because a
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
