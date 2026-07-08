---
title: Sample prompts for the Microsoft 365 Insights Agent
description: Get a catalog of ready-to-use prompts you can ask the Microsoft 365 Insights Agent to understand activity and adoption of agents in your tenant.
ms.topic: concept-article
ms.date: 06/30/2026
author: lauragra
ms.author: fwilliams
ms.reviewer: lauragra
ms.localizationpriority: medium
---

# Sample prompts for the Microsoft 365 Insights Agent

The Microsoft 365 Insights Agent is a Copilot agent that answers natural language questions about how the agents in your tenant are used. This article lists the prompts that the Insights Agent currently supports, organized by the question you're trying to answer.

## Prompt overview

Every prompt is a pattern that the agent is validated against. You don't have to phrase your question exactly as shown; the agent handles natural variations. Use the prompts in this article as a starting point for your own agent prompts.

The agent reports aggregate metrics only. It doesn't identify individual users, and it tells you explicitly when a question requires user-level identity that it doesn't have. For more information, see [Questions the agent won't answer](#questions-the-agent-wont-answer).

## Core supported prompts

The Insights Agent currently supports the following core prompts. The agent answers each one from aggregate activity data and returns a grounded, numerical response.

### Get a snapshot of an agent

Use these prompts when you want the basics in one shot.

| # | Prompt |
|---|---|
| 1 | Show me {Agent Name}'s key usage stats |
| 2 | Provide all usage insights for {Agent Name} |
| 3 | Give me a full usage summary for {Agent Name} |
| 4 | Show me usage metrics of {Agent Name} |

### See trends over time

Use these prompts to understand whether usage is moving up, down, or flat.

| # | Prompt |
|---|---|
| 5 | How many unique users did {Agent Name} have over time? |
| 6 | What's the weekly active user trend for {Agent Name}? |
| 7 | How does daily usage of {Agent Name} compare to monthly usage? |
| 8 | Show me daily active users for {Agent Name} |

### Look at a specific window

Use this prompt when you have a particular period in mind.

| # | Prompt |
|---|---|
| 9 | How many unique users did {Agent Name} have last week? |

### Compare metrics within one agent

Use these prompts to put two activity measures side by side for the same agent.

| # | Prompt |
|---|---|
| 10 | Compare DAU and WAU for {Agent Name} |
| 11 | Compare MAU and WAU for {Agent Name} |

### Compare across agents

Use these prompts to put two or more agents next to each other.

| # | Prompt |
|---|---|
| 12 | Compare {Agent A} to {Agent B} |
| 13 | Which agents have higher usage this week? |

### Measure growth

Use this prompt to see net new user growth.

| # | Prompt |
|---|---|
| 14 | What's the net new user growth for {Agent Name}? |

## Questions the agent won't answer

The Insights Agent reports aggregate metrics only. By design, it doesn't track individual users, capture verbatim feedback, or hold install and uninstall signal. If you ask one of the following kinds of questions, the agent tells you it can't compute the answer and points you to the nearest aggregate metric that's available:

- **Individual-user questions.** Anything that asks who used the agent, how a specific person behaved, or how many of the same users came back. The agent doesn't have user-level identity.
- **Cohort retention percentages.** Classical retention—the same set of users returning over time—requires user-level identity. The agent doesn't produce a cohort retention number. For an aggregate proxy you can ask about, see [Sustained-usage and growth patterns](#sustained-usage-and-growth-patterns).
- **Install and uninstall counts.** Install signal isn't currently connected to the Insights Agent data layer.
- **Feedback summarization.** A separate capability handles verbatim feedback for individual agents.

When the agent declines a question, it returns a deterministic message that names the constraint and points you to the closest supported signal. It doesn't fabricate a number.

## Sustained-usage and growth patterns

In addition to the core prompts, the agent supports a small set of higher-order questions about whether the active-user pool is sustained, how concentrated usage is, and which agents are growing fastest. The agent computes these metrics from aggregate activity data, and the response always includes the appropriate caveat.

### Rolling retention proxy

Ask either of the following questions to get a window-over-window sustained-usage signal:

- What's the 7-day retention for {Agent Name}?
- Show me the 30-day rolling retention for {Agent Name}.

The agent calculates the ratio of unique users in the most recent N-day window to unique users in the prior N-day window, where N can be 1, 7, 30, 60, or 90. The response always includes the caveat that this ratio is an aggregate window-over-window ratio, not user-level cohort retention.

### Stickiness

Ask either of the following questions:

- How sticky is {Agent Name}?
- What's the DAU/MAU ratio for {Agent Name}?

The agent returns DAU divided by MAU as a percentage, interpreted as how many days of the month an average monthly user is active.

### Adoption trend

Ask either of the following questions:

- Is adoption of {Agent Name} increasing?
- What's the adoption trend for {Agent Name}?

The agent returns the direction and magnitude of the unique-user count week-over-week or month-over-month.

### Fastest-growing agent

Ask either of the following questions:

- Which agent is growing fastest this week?
- Show me which agent is growing fastest.

The agent returns a ranked list of the top agents by week-over-week percentage change in unique users.

### Main usage pattern

Ask the following question:

- What are the main usage patterns for {Agent Name}?

The agent returns a narrative description of the temporal usage shape, such as weekday-heavy and business-hours-concentrated, steady, or spiky, based on the trailing 30-day DAU distribution. The agent doesn't try to cluster users by persona or intent.

## Tips for writing your own prompts

Apply the following tips when you create prompts for the agent:

- **Use natural language.** You don't have to match the canonical phrasing in this article. *"How is X doing this month?"* and *"Give me the monthly stats for X"* both work.
- **Substitute any agent name visible to you.** You can replace `{Agent Name}` with any agent name you have access to in your tenant.
- **Anchor to a date when you need precision.** Phrases like *"last week,"* *"this month,"* and *"since May 1"* work and shape the response window.
- **If the agent says it can't compute something, read the pointer.** The response names the nearest supported metric. Try that one.

## Related content

- [Microsoft 365 Insights Agent overview](insights-agent-overview.md)
- [Prompt and capabilities reference](insights-agent-prompt-reference.md)
- [Troubleshoot the Insights Agent](insights-agent-troubleshooting.md)
