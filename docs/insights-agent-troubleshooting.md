---
title: Troubleshoot the Microsoft 365 Insights Agent
description: Resolve common issues with the Microsoft 365 Insights Agent, including missing agent picker entries, no-data responses, unexpected not-supported responses, and computation questions.
ms.topic: troubleshooting
ms.date: 06/30/2026
author: lauragra
ms.author: fwilliams
ms.reviewer: lauragra
ms.localizationpriority: medium
---

# Troubleshoot the Microsoft 365 Insights Agent

This article lists the most common issues that can occur with the Microsoft 365 Insights Agent, and provides resolution steps.

## No data for the agent you asked about

You ask one of the supported prompts - for example, "Provide all usage insights for {Agent Name}" - and the response says there's no data for the agent you named.

**Most likely causes, in order:**

1. **Typo or wrong agent name.** The agent does an exact match on the agent name visible to you. Confirm the name as it appears in your tenant catalog, and then try again.
1. **The agent has no activity yet.** New agents need at least one prior window of activity before windowed metrics return nonzero values. If the agent shipped to your tenant very recently, this condition is expected, and it resolves itself as activity accumulates.
1. **You don't have visibility into the agent.** The Insights Agent reports on agents you have access to. If you don't see the agent in your normal Copilot agent surface, you won't get data from the Insights Agent either.

<!-- markdownlint-disable MD036 -->
**Resolution**

Verify the agent name, confirm with the agent owner that activity exists, and then retry. If the issue persists for an agent you know is active, contact your Microsoft contact.

## You asked for retention and the response includes a caveat you don't understand

You ask "What's the 7-day retention for {Agent Name}?", and the response gives a percentage along with a sentence like "This is computed as an aggregate window-over-window ratio, not user-level cohort retention."

**Cause**

This caveat is by design. The Insights Agent reports aggregate metrics only; it doesn't track individual users. Classical retention - the same set of users returning over time - requires user-level identity. Instead, the agent provides a proxy: the ratio of unique users in the most recent N-day window to unique users in the prior N-day window. The caveat is required so that you don't interpret the number as cohort retention.

**Resolution**

Read the proxy as a sustained-usage signal: whether the active-user pool is growing, holding, or shrinking across consecutive equal-length windows. If you need cohort retention, that requires a different data source than the Insights Agent provides.

## You asked about install or uninstall counts and got a "not yet supported" response

You ask "How many people installed {Agent Name} last week?", and the agent says the metric is on the roadmap but not in this release.

**Cause**

The current version doesn't yet connect the install and uninstall signal to the Insights Agent data layer. It's tracked on the roadmap for a future release.

**Resolution**

For now, use activity metrics - DAU, WAU, MAU, and unique users over time - as a proxy for adoption. If install and uninstall metrics are critical to your evaluation of the agent, let your Microsoft contact know. The product team prioritizes the roadmap based on customer feedback.

## Trend query returns an unexpectedly small number or an error

A trend-shaped prompt - for example, "What's the weekly active user trend for {Agent Name}?" or "How does daily usage of {Agent Name} compare to monthly usage?" - returns a smaller-than-expected result or a vague intent-classification error.

**Most likely cause**

Time-windowed queries depend on a recent prompt-template revision that ships with the agent. Earlier builds had a known issue parsing time-based intent on these specific prompts.

**Resolution**

Make sure you're on the most recent Insights Agent build. (Your Microsoft contact can confirm.) If you're on the current build and still see the issue, file the exact prompt text and the response with your Microsoft contact. This information helps the engineering team narrow down language-understanding gaps.

## Pairwise compare shows numbers that don't match your Power BI dashboard

You ask "Compare {Agent A} to {Agent B}", and the values differ from what your tenant's Power BI usage dashboard shows.

**Likely causes**

One of the following causes likely explains the gap:

1. **Different time anchor.** The Insights Agent computes from "today" or a date you specify in the prompt. Dashboards typically run on a fixed refresh schedule. A small difference in the anchor day can yield different counts.
1. **Different uniqueness definition.** The Insights Agent counts distinct users with at least one session in the window. A dashboard might count distinct sessions, prompts, or signed-in accounts. Those aren't the same denominator.
1. **Different agent-version scope.** If an agent was renamed or republished, the Insights Agent and a dashboard might disagree on which underlying agent version each metric attributes activity to.

**Resolution**
Anchor both surfaces to the same date, and confirm that both count distinct users (not distinct sessions). If the discrepancy persists, contact your Microsoft contact with the exact prompt and dashboard screenshots so that engineering can verify the attribution path.

## Fastest-growing response returns the same agent every week

You regularly ask "Which agent is growing fastest this week?" and the same agent keeps appearing at the top.

**Cause**

The fastest-growing computation ranks agents by week-over-week percentage change in unique users. Agents with small absolute bases can show large percentage swings, and agents with stable usage at scale can hold the top slot for weeks.

**Resolution**

This behavior is expected, not a bug. If you want to filter for a minimum scale - for example, only agents above a certain MAU floor - ask in your Microsoft contact channel. We're tracking demand for ranked queries that exclude low-base outliers.

## Related content

- [Sample prompts](insights-agent-sample-prompts.md)
- [Prompt and capabilities reference](insights-agent-prompt-reference.md)
- [Microsoft 365 Insights Agent overview](insights-agent-overview.md)
