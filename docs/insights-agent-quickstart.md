---
title: Quickstart - Use the Microsoft 365 Insights Agent
description: Ask your first question of the Microsoft 365 Insights Agent and confirm the agent returns a grounded usage answer for an agent in your tenant.
ms.topic: quickstart
ms.date: 06/30/2026
author: lauragra
ms.author: fwilliams
ms.reviewer: lauragra
ms.localizationpriority: medium
---

# Quickstart: Use the Microsoft 365 Insights Agent

The Microsoft 365 Insights Agent is a Copilot agent that answers natural language questions about how the agents in your tenant are used. This quickstart describes how to ask your first question of the Microsoft 365 Insights Agent and confirm that it returns a grounded usage answer for an agent in your tenant. The interaction takes under two minutes.

## Prerequisites

Before you begin, make sure you have the following items:

- A Microsoft 365 Copilot license assigned to your account.
- At least one Copilot agent deployed in your tenant that you have permission to use. The Insights Agent reports activity for agents you already have access to.
- A few days of agent activity in your tenant. Some metrics, such as rolling retention, adoption trend, and fastest-growing, need at least one prior window of data to compute a delta. New tenants might see partial results until enough activity accumulates.

## Step 1: Open the Insights Agent

Sign in to [Microsoft 365 Copilot Chat](https://m365.cloud.microsoft/chat) by using a work account that's assigned a Copilot license in your tenant.

In the agent picker, search for **Insights Agent**, and then select it.

## Step 2: Pick an agent to ask about

Have the name of an agent deployed in your tenant ready. This is the agent you want usage information for.

You can ask about any agent that's visible to you in your tenant. You don't need to be the owner of the agent.

## Step 3: Ask your first question

Paste the following prompt into the chat, and replace `{Agent Name}` with the name of the agent you want usage data for.

```text
Provide all usage insights for {Agent Name}
```

Select **Enter**, and then wait for the response.

## Step 4: Read the response

A successful response includes the following:

- A short narrative summary that names the agent you asked about and identifies the time window that the numbers cover.
- One or more aggregate metrics from the supported-metric list. For example, daily active users, weekly active users, monthly active users, or unique users over time.
- The numerical values themselves, expressed as counts or percentages.
- A caveat line on any metric that requires interpretation. For example, the rolling retention proxy is always labeled as an aggregate window-over-window ratio, not user-level cohort retention.

If the agent doesn't have data for the agent you named, it says so explicitly and suggests a related question you can ask instead. It doesn't fabricate a number.

## Step 5: Try a comparison

After you have a baseline response, try one of the comparison prompts to see how the agent handles a two-metric or two-agent question:

```text
Compare DAU and WAU for {Agent Name}
```

To compare two agents, use:

```text
Compare {Agent A} to {Agent B}
```

The response shape is the same: a narrative summary, and then the numerical comparison.

## Confirm successful setup

You completed the quickstart when:

- The Insights Agent appears in your Microsoft 365 Copilot Chat agent picker.
- A prompt against a real agent in your tenant returns a grounded numerical response that states the agent name and time window.
- A comparison prompt returns parallel metrics for both items being compared.

If a step doesn't produce the result described, see [Troubleshoot the Insights Agent](insights-agent-troubleshooting.md).

## Related content

- [Sample prompts](insights-agent-sample-prompts.md)
- [Prompt and capabilities reference](insights-agent-prompt-reference.md)
- [Microsoft 365 Insights Agent overview](insights-agent-overview.md)
