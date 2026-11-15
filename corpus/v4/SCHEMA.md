# PRAMANA v4 — sequence schema

**v4 supersedes v3, which is withdrawn.** v3 could be solved by one structural fact (see
"Why v4 exists"). Do not score against v3.

One trace is an ordered sequence of 12 events from one to four agents. Each attack trace
opens with a healthy prefix drawn from the benign generator and then compromises at some
point, so the signal is deviation from the agent's own recent behaviour.

- `holdout.jsonl` — 260 traces, 3120 events, **no labels**. Score against this.
- `holdout_labels.jsonl` — the answer key. Withheld while v4 is the live season.
- 130 attack and 130 benign traces; agent counts matched across labels.

## Published fields

`agent_id`, `agent_profile_type`, `agent_role`, `app_name`, `destination`,
`egress_destinations`, `framework`, `is_demo`, `latency_ms`, `model`, `pii_detected`,
`pii_types`, `prompt_hash`, `prompt_length`, `session_id`, `step`, `token_count_in`,
`token_count_out`, `tool_call_count`, `tool_calls`, `tool_calls_last_n`.

Events are content-free: hashes, counts, lengths, latencies, tool names and opaque
destination identifiers. No prompt or response text exists anywhere in the corpus.

## Why v4 exists

The v3 builder tested shortcuts only on scalar fields and skipped list-valued fields.
Three defects slipped through, all the same shape: the two generators emit different
things and the union-fill writes a constant that then marks the class.

| v3 defect | measured |
|---|---|
| `tool_calls_last_n == []` somewhere in the trace (attack generators never emit it) | balanced **1.0000** |
| `egress_destinations` non-empty (benign generator never emits a non-empty list) | 0.85 alone |
| any of five attack-only counters, OR-ed | 0.888 |

A one-line rule therefore beat every entry on the v3 board, including ours.

## What changed

1. **Only fields both generators can emit are published.** Sixteen attack-only fields were
   dropped, among them `cvr_score`, `blast_radius_score`, `verdict_origin` and the
   remediation fields, which are outputs of our own platform and not telemetry.
2. **`tool_calls_last_n` is recomputed for both classes** as one rolling window of the
   agent's last five tool calls, so it cannot differ in kind between classes.
3. **Destinations are opaque and habitual.** Every destination is an opaque identifier. The
   pool is partitioned across the benign archetypes, so each destination is some
   archetype's habit and none is attack-only. A destination is informative only relative to
   the agent's own history. 4% of healthy events visit one novel destination, and the rate
   of any egress in healthy traces is matched to the attack traces.
4. **Tool and PII vocabularies overlap.** Healthy events occasionally use a tool from the
   attack vocabulary (8%) — shells, scanners, database, mail and web tools, but never
   overtly offensive tooling — compromise events keep using ordinary tools (35%), and
   healthy PII events draw from the pooled type list.
5. **Healthy agents have bad moments.** 12% of healthy stretches contain one excursion of
   4–7σ from the agent's own baseline. In v3, no benign trace exceeded 3.52σ.
6. **The guard now covers structure.** The build refuses a corpus if any single structural
   fact (a field present, empty, or a list containing an element) scores above 0.85, or if
   an OR of near-silent facts scores above 0.75. v4: best fact 0.6115, disjunction 0.7212.

## Limits, stated

- Tool names are real labels. The vocabulary overlap is deliberate but synthetic, so a
  healthy agent running a shell or a scanner is a stand-in for an unusual-but-benign choice.
- A detector that learns each profile's habitual destinations from the labelled public
  split can still use novelty well. That is baseline-aware detection and is meant to be
  rewarded; it is not a shortcut because the habits are not exclusive to one class.
- The structural guard is a fixed family of tests. It rules out the failure modes we
  know about; it does not prove no other shortcut exists.
