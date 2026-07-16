---
title: Troubleshoot MCP apps in Microsoft 365 Copilot - Common issues and fixes
description: Fix common issues with MCP apps in Microsoft 365 Copilot, including widget rendering problems, tool discovery failures, authentication errors, and response issues.
author: jasonjoh
ms.author: jasonjoh
ms.localizationpriority: medium
ms.topic: troubleshooting-general
ms.date: 07/14/2026
---

# Troubleshoot MCP apps in Microsoft 365 Copilot

This guide provides troubleshooting advice for common issues you might encounter when developing a [Model Context Protocol (MCP) app to integrate with a declarative agent](plugin-mcp-apps.md) inside Microsoft 365 Copilot.

## Enable developer mode

[Enabling developer mode](prerequisites.md#enabling-developer-mode) surfaces logs and errors in agent responses. This information is essential for debugging. To enable developer mode, type the following command in Microsoft Copilot.

```text
-developer on
```

MCP tools available to your agent show up in the **Actions** section of the debug information card. For details about the debug information card, see [Use developer mode in Microsoft 365 Copilot to test and debug agents](debugging-agents-copilot-studio.md).

## Discovery and entry problems

### No tools listed

If the **Actions** section of the debug information card doesn't list any MCP tools, check the following items:

- Confirm your MCP server is running and you're connecting to the correct MCP endpoint in your plugin manifest.
- Confirm that authentication succeeds. Many MCP servers return no tools until the user signs in, so a failed or skipped sign-in can result in an empty tool list.

#### Dynamic tool discovery

If your agent uses [dynamic tool discovery](plugin-dynamic-tool-discovery.md) (the default), the tools are resolved from the server at runtime. Check that:

- The `RemoteMCPServer` runtime sets `run_for_functions` to `["*"]` and the top-level `functions` array is empty.
- The server's `tools/list` method returns tools. You can inspect the response by using a tool such as [MCP Inspector](https://www.npmjs.com/package/@modelcontextprotocol/inspector).
- A newly discovered or modified tool isn't withheld by runtime validation. Check the debug information for validation errors.

```json
"runtimes": [
  {
    "type": "RemoteMCPServer",
    "spec": {
      "url": "https://api.contoso.com/mcp"
    },
    "run_for_functions": [
      "*"
    ]
  }
]
```

#### Pinned tools

If you pinned a fixed set of tools, verify that the MCP server runtime specified in the `runtimes` property in your plugin manifest:

- Includes the expected tools in the top-level `functions` property.
- References the tools in the `mcp_tool_description` property by either:
  - Referencing a JSON file that contains the tool descriptions in the `file` property **OR**
  - Listing the tool descriptions inline in the `tools` property
- Includes the tool names in the `run_for_functions` property.

```json
"runtimes": [
  {
    "type": "RemoteMCPServer",
    "spec": {
      "url": "https://api.contoso.com/mcp",
      "mcp_tool_description": {
        "file": "mcp-tools.json"
      }
    },
    "run_for_functions": [
      "get_widget",
      "create_widget"
    ]
  }
]
```

### Tools not triggering from Copilot chat

- Revisit your tool and parameter descriptions to ensure they provide sufficient context. Consider rewriting them using "Use this function/parameter when..." phrasing.
- Keep descriptions under 1,024 characters. Text beyond 1,024 characters is ignored.
- Ensure tool visibility is set correctly.
  - For MCP apps, `_meta.ui.visibility` includes `model`.
  - For OpenAI SDK apps, `meta["openai/visibility"]` is set to `public`.

### The wrong tool is selected

- Avoid tools with similar names or overlapping descriptions.
- Add clear differentiators in descriptions explaining when each tool should be used.

## Widget issues

### Widget doesn't render

If the correct MCP tool is called but your UI widget doesn't render in the response, your MCP server is likely only returning structured content with no UI component. Ensure that UI binding is configured correctly.

- For MCP apps, tool definition includes `_meta.ui.resourceUri` set to a registered HTML resource with MIME type `text/html;profile=mcp-app`.
- For OpenAI SDK apps, tool definition includes `_meta["openai/outputTemplate"]` set to a registered HTML resource with MIME type `text/html+skybridge`.

### Widget fails to load

- Open your browser's developer tools and check for Content Security Policy (CSP) violations in the console. Ensure that requests from the widget's host URL are allow-listed. For more information, see [MCP server requirements for MCP apps](plugin-mcp-apps.md#mcp-server-requirements-for-mcp-apps).
- Verify that your widget compiles all HTML and JavaScript dependencies into a single file with no external unresolved assets.

### Widget loads with no data

- Verify the tool's response structure.
  - `content` should contain only the data (model).
  - `structuredContent` should contain both the data and the widget.
  - `_meta` should contain only the widget.
- Ensure `structuredContent` or `_meta` includes the required data.

### Widget has a double scrollbar

The Copilot host container already has a scroll with max height. Disable inner scroll in your widget by setting `overflow: hidden` in your container styles.

### Hyperlinks in widget don't open

Anchor tags `<a>` don't work for external links in Copilot. Use the appropriate platform APIs instead.

- For MCP apps, use `app.openLink`.
- For OpenAI SDK apps, use `window.openai.openExternal`.

### Fullscreen doesn't work in some Copilot hosts

Fullscreen view isn't supported across all Copilot hosts. As a best practice, always check for host capabilities and conditionally display UI elements (such as a fullscreen button). For more information, see [Verify API availability](plugin-mcp-apps.md#verify-api-availability).

## Response issues

### Tool result expiry issues

Ensure tool responses sent through `content` or `structuredContent` aren't excessively large. If your widget requires rich metadata that isn't useful for the model, such as avatar URLs or UI-specific details, include the full data in `_meta` and provide a concise summary in `content`. This approach ensures the model retains key information while supporting an effective multiturn experience.

### Duplicate data in widget and text summary

Resolve this problem by using one of the following options:

- **Optimize data separation:** use `_meta` for widget-specific data and `content` for model-visible summaries.
- **Steer formatting:** use instructions in the declarative agent manifest to guide how responses are structured and presented.

## Authentication problems

For authentication problems - such as sign-in failures, token errors, consent prompts, and auth configuration mismatches - see [Troubleshoot MCP and API plugin authentication](plugin-authentication-troubleshooting.md).

## Related content

- [Add MCP apps to declarative agents in Microsoft 365 Copilot](plugin-mcp-apps.md)
- [Configure authentication for MCP and API plugins in agents](plugin-authentication.md)
- [Use developer mode in Microsoft 365 Copilot to test and debug agents](debugging-agents-copilot-studio.md)
