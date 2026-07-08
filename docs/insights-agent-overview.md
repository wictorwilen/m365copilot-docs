---
title: Microsoft 365 Insights Agent overview
description: Learn how the Microsoft 365 Insights Agent helps you understand activity and adoption of agents in your tenant by asking natural-language questions.
ms.topic: overview
ms.date: 06/30/2026
author: lauragra
ms.author: fwilliams
ms.reviewer: lauragra
ms.localizationpriority: medium
---

# Microsoft 365 Insights Agent overview

The Microsoft 365 Insights Agent is a Copilot agent that answers natural language questions about how the agents in your tenant are used. Ask about daily, weekly, and monthly active users, rolling retention, stickiness, adoption trends, or how one agent compares to another. The agent returns a grounded, aggregate answer right in chat.

## Why use the Insights Agent

Agent owners, IT admins, and business stakeholders all need to know how their agents are performing. Today, that information is located in dashboards, Power BI reports, and ad hoc Kusto queries that most of these audiences can't run themselves. As a result:

- It takes a long time to learn whether anyone is using an agent after you ship it.
- Understanding usage depends on a small group of analysts.
- Teams define basic terms like *retention* and *stickiness* inconsistently.

The Insights Agent removes barriers to the information. It gives you a consistent, documented set of activity metrics. It answers in plain language in the same Copilot Chat surface where your agents live, and it reports aggregate signal only, so the results align with privacy review requirements.

## Who it's for

The Insights Agent is built for three audiences:

- **Agent owners and PMs.** You shipped an agent and need to know whether usage is growing, where it's plateauing, and how it compares to peer agents in your tenant.
- **IT administrators and Copilot champions.** You're accountable for the Copilot footprint across your organization. You need adoption trends and a defensible way to discuss them with your business sponsors.
- **Business stakeholders and executives.** You funded an agent investment and want a periodic, plain-language read on whether it's paying off, without learning a BI tool.

The agent isn't designed for individual user analysis or cohort attribution. For more information, see [Scope boundaries](#agent-scope).

## What you can do with the Insights Agent

By using the Insights Agent, you can:

- Get daily, weekly, and monthly active users for any agent you can see in your tenant.
- Compare metrics across two agents side by side.
- See unique user counts over rolling windows (last 7 days, last 30 days, or a custom anchor date).
- Get an aggregate rolling retention proxy (1, 7, 30, 60, or 90 day windows) without exposing individual identity.
- Get usage concentration (stickiness) (DAU divided by MAU) for any agent.
- Identify the fastest-growing agent in your tenant by week-over-week change in unique users.
- Get a narrative read of an agent's main usage pattern (weekday-heavy, business-hours-concentrated, steady, or spiky) from the trailing 30-day distribution.

## Available metrics

The agent currently supports the following metrics. It computes each metric deterministically from aggregate activity data. It doesn't generate numbers or estimate them from a model.

| Metric | What it answers | How it's computed |
|---|---|---|
| Daily active users (DAU) | How many distinct users used the agent today, or on a specified day | Count of distinct users with at least one session in the day |
| Weekly active users (WAU) | How many distinct users used the agent in a 7-day window | Count of distinct users with at least one session in the 7-day window |
| Monthly active users (MAU) | How many distinct users used the agent in a 30-day window | Count of distinct users with at least one session in the 30-day window |
| Unique users over time | How the unique-user count changes across windows | Series of distinct-user counts across consecutive windows |
| Pairwise compare | Side-by-side DAU, WAU, and MAU for two agents | Two parallel metric pulls plotted together |
| Rolling N-day retention proxy | Whether the active-user pool is sustained, growing, or shrinking | `unique_users[t − N → t] / unique_users[t − 2N → t − N]`, where N is 1, 7, 30, 60, or 90 |
| Stickiness | How concentrated usage is among monthly users | DAU divided by MAU, expressed as a percentage |
| Adoption trend | Direction of the unique user count week-over-week or month-over-month | Sign and magnitude of the most recent week-over-week or month-over-month delta |
| Fastest growing | Which agent grew the most this week | Ranked week-over-week percentage delta of unique users across agents in scope |
| Main usage pattern | Temporal shape of usage | Categorical narrative from the DAU distribution over the trailing 30 days |

> [!IMPORTANT]
> The rolling retention proxy is an **aggregate window-over-window ratio**, not user-level cohort retention. The agent always includes this caveat in its response. Cohort retention requires user-level identity, which the Insights Agent doesn't collect by design.

## How it works

A typical interaction follows this pattern:

1. You ask the Insights Agent a question in Microsoft 365 Copilot Chat. For example, "What's the weekly active user trend for {Agent Name}?"
1. The agent matches your intent to the supported metrics.
1. If the query is supported, the agent pulls the relevant aggregate metrics for the agent and window you asked about, and then returns a grounded answer with the underlying numbers.
1. If the query isn't supported - for example, individual-user retention - the agent returns a deterministic, scoped-down response that points you to the closest supported signal.

For a list of supported prompts and how to phrase them, see [Sample prompts](insights-agent-sample-prompts.md).

## Two ways to get usage data

You can get usage data for the agents in your tenant by asking the Insights Agent directly in Microsoft 365 Copilot Chat or by pulling the same signal programmatically through Work IQ.

| | In-product (Insights Agent) | Programmatic (Work IQ) |
|---|---|---|
| **What it is** | Ask the Insights Agent natural-language questions in Copilot Chat | Pull the same usage metrics from the Work IQ command line and endpoints |
| **Best for** | Agent owners, IT admins, and sponsors who want a quick, plain-language read | Developers automating the data into dashboards, scorecards, or pipeline checks |
| **Cost** | Included with your Copilot license; no additional cost | Metered through Copilot Credits |

Both paths return the same documented, aggregate metrics. The difference is the surface and how you consume the data.

### In-product experience (no additional cost)

Using the Insights Agent inside Microsoft 365 Copilot Chat is included with your Copilot license. There's no additional charge for asking usage questions in product. For most agent owners, IT admins, and business sponsors, the in-product experience is the fastest way to get a grounded answer.

### Programmatic access with Work IQ

When you want the same metrics in your own systems—for example, feeding a weekly adoption scorecard, populating a dashboard, or adding a usage check to an automated pipeline—use Work IQ. Through its command line and endpoints, you can request the supported usage metrics programmatically and integrate the results into your own workflows. Work IQ programmatic access is a metered, consumption-based capability billed through Copilot Credits.

## Agent scope

The Insights Agent has a narrow scope. The following capabilities are currently out of scope and are handled by separate capabilities:

- **Individual-user analysis.** The agent reports aggregate metrics only. It doesn't track which specific users used an agent.
- **Install and uninstall trends.** Install signal isn't currently connected to the Insights Agent data layer.
- **Engagement depth and session quality.** These metrics require either user-level identity or richer telemetry than the current data layer provides.

## Related content

- [Sample prompts](insights-agent-sample-prompts.md)
- [Prompt and capabilities reference](insights-agent-prompt-reference.md)
- [Troubleshoot the Insights Agent](insights-agent-troubleshooting.md)
