---
title: Prompt and capabilities reference for the Microsoft 365 Insights Agent
description: Find reference information for the Microsoft 365 Insights Agent, including supported metrics and prompts, sustained-usage queries, and more.
ms.topic: reference
ms.date: 06/30/2026
author: lauragra
ms.author: fwilliams
ms.reviewer: lauragra
ms.localizationpriority: medium
---

# Prompt and capabilities reference for the Microsoft 365 Insights Agent

This article provides a catalog of the queries the Microsoft 365 Insights Agent answers, how it answers each query class, and what it doesn't answer. Use this information as the source of truth when you integrate the agent into a runbook, develop internal training material, or validate whether a customer-reported behavior aligns with the documented contract.

## Query classes

Queries fall into five classes. The agent behaves deterministically within each class.

| Class | What the agent does |
|---|---|
| Supported (core) | Computes the answer from aggregate activity data and responds with the numeric value plus a short narrative. |
| Sustained-usage and growth | Computes a defined proxy or derived signal and always appends the required interpretation caveat. |
| Not yet supported | Returns a standard "on the roadmap" response that names the missing capability. No fabricated number. |
| Not supported by design | Returns a standard "we don't have user-level identity" response and points to the nearest aggregate signal. No fabricated number. |
| Handled by another capability | Returns a standard response that identifies the capability that owns the answer. |

In every example, replace `{Agent Name}` with the name of the agent you want to ask about. Replace `{Agent A}` and `{Agent B}` with two agents you want to compare.

## Supported core prompts

The following prompts are the validated core of the release. The agent answers each one from aggregate activity data.

| # | Example prompt | Metric returned | Computation |
|---|---|---|---|
| 1 | Compare DAU and WAU for {Agent Name} | Daily active users compared to weekly active users for the same agent | Two parallel distinct-user counts over the day and over the 7-day window |
| 2 | Show me {Agent Name}'s key usage stats | DAU, WAU, and MAU summary block | Three distinct-user counts (1-day, 7-day, and 30-day windows) |
| 3 | How many unique users did {Agent Name} have over time? | Unique-user time series | Series of distinct-user counts across consecutive windows |
| 4 | How does daily usage of {Agent Name} compare to monthly usage? | DAU compared to MAU | Two distinct-user counts (1-day and 30-day windows) shown side by side |
| 5 | Compare MAU and WAU for {Agent Name} | Monthly active users compared to weekly active users | Two parallel distinct-user counts (30-day and 7-day windows) |
| 6 | What's the weekly active user trend for {Agent Name}? | WAU trend line | Series of 7-day distinct-user counts across recent weeks |
| 7 | How many unique users did {Agent Name} have last week? | Single WAU number anchored to the prior 7-day window | Distinct-user count for the prior 7-day window |
| 8 | Provide all usage insights for {Agent Name} | Full summary block | DAU, WAU, MAU, trend, plus narrative shape commentary |
| 9 | Give me a full usage summary for {Agent Name} | Full summary block | Same as #8; variant phrasing supported |
| 10 | Show me usage metrics of {Agent Name} | Full summary block | Same as #8; variant phrasing supported |
| 11 | Which agents have higher usage this week? | Ranked list of agents by recent unique-user count | Pairwise (or N-wise) sort of 7-day distinct-user counts across agents in scope |
| 12 | What's the net new user growth for {Agent Name}? | Net new user delta across the recent windows | Difference between unique users in the most recent window and the prior window |
| 13 | Show me daily active users for {Agent Name} | DAU value or DAU time series, depending on the date qualifier | Distinct-user counts by day |
| 14 | Compare {Agent A} to {Agent B} | Side-by-side DAU, WAU, and MAU for two agents | Two parallel metric pulls for the same windows |

## Sustained-usage and growth prompts (five derived patterns)

These higher-order queries use the same aggregate activity data, and they always include the required interpretation caveat in the response.

### Rolling retention proxy

| Field | Value |
|---|---|
| Example prompts | What's the 7-day retention for {Agent Name}? / Show me the 30-day rolling retention for {Agent Name} |
| Supported windows (N) | 1, 7, 30, 60, or 90 days |
| Definition | `unique_users[t − N → t] / unique_users[t − 2N → t − N]`, expressed as a ratio or percentage |
| Required caveat in response | "This is computed as an aggregate window-over-window ratio, not user-level cohort retention." |

### Stickiness

| Field | Value |
|---|---|
| Example prompts | How sticky is {Agent Name}? / What's the DAU/MAU ratio for {Agent Name}? |
| Definition | DAU divided by MAU, expressed as a percentage |
| Interpretation | A higher ratio means monthly users tend to use the agent on more days within the month |

### Adoption trend

| Field | Value |
|---|---|
| Example prompts | Is adoption of {Agent Name} increasing? / What's the adoption trend for {Agent Name}? |
| Definition | Sign and magnitude of the unique-user delta week-over-week or month-over-month |
| Response shape | Direction (increasing, decreasing, or flat) plus the percentage change |

### Fastest-growing agent

| Field | Value |
|---|---|
| Example prompts | Which agent is growing fastest this week? / Show me which agent is growing fastest |
| Definition | Highest week-over-week percentage growth in unique users across agents in scope |
| Response shape | Ranked list of the top agents with their week-over-week percentage deltas |

### Main usage pattern

| Field | Value |
|---|---|
| Example prompts | What are the main usage patterns for {Agent Name}? |
| Definition | Narrative description of the temporal usage shape, derived from the trailing 30-day DAU distribution |
| Response shape | Categorical narrative (for example, weekday-heavy and business-hours-concentrated, steady, or spiky); the agent doesn't attempt persona or intent clustering |

## Not yet supported

The following query classes are tracked for a future release. The agent returns a standard response that names the capability.

| Example prompt | Standard response |
|---|---|
| Show install trends for {Agent Name} | "Support for install and uninstall metrics is on the Insights Agent roadmap but not available in this release. We're focused on activity metrics in the current release." |
| How many people installed {Agent Name} last week? | "Support for install and uninstall metrics is on the Insights Agent roadmap but not available in this release. We're focused on activity metrics in the current release." |
| Which agents have the best retention? | "Best-retention superlatives require a cohort retention number, which the current release doesn't produce. For an aggregate sustained-usage signal, ask for rolling retention on a specific agent." |

## Not supported by design

The following queries require user-level identity. The Insights Agent doesn't track individual users. This is a deliberate privacy choice. The agent returns a standard response and points to the nearest aggregate signal.

| Example prompt | Standard response |
|---|---|
| How many users uninstalled {Agent Name}? | "Insights Agent reports aggregate usage metrics and doesn't track individual users. Uninstall counts require user-level identity to compute." |
| What's the engagement depth for {Agent Name}? | "Engagement depth requires per-user session detail that the agent doesn't collect. The closest aggregate signal is the full usage summary." |
| What's the retention rate for {Agent Name}? | "Cohort retention as classically defined requires user-level identity. The aggregate sustained-usage signal is the rolling retention proxy; ask for 7-day, 30-day, 60-day, or 90-day rolling retention." |
| Show me 7-day retention for {Agent Name} | Redirected to the rolling retention proxy with the aggregate caveat |
| How many users returned at least twice last week? | "We don't track individual return visits. The closest aggregate signal is the rolling retention proxy plus a DAU-versus-unique-user comparison." |
| Compare engagement depth across top 5 agents | Redirected to pairwise agent compare on supported metrics |

## Decision flow

To find out in advance whether the agent answers a new prompt, use the following flow.

```text
Does the question ask for one of the 14 supported metrics or a known phrasing variant?
  YES -> Supported. Expect a numeric answer with the agent name and time window stated.
  NO  -> continue
Does the question ask for rolling retention, stickiness, adoption trend, fastest-growing, or main usage pattern?
  YES -> Supported. Expect the value plus the required interpretation caveat.
  NO  -> continue
Does the question mention install, uninstall, or "best retention" superlatives?
  YES -> Not yet supported. Expect the roadmap response.
  NO  -> continue
Does the question require knowing about individual users (cohort retention, engagement depth, return visits)?
  YES -> Not supported by design. Expect the privacy-posture response with a pointer to the nearest aggregate signal.
  NO  -> continue
  NO  -> Generic out-of-scope fallback.
```

## Related content

- [Microsoft 365 Insights Agent overview](insights-agent-overview.md)
- [Sample prompts](insights-agent-sample-prompts.md)
- [Troubleshoot the Insights Agent](insights-agent-troubleshooting.md)
