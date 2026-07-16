---
title: Configure anonymous authentication
description: Learn how to configure anonymous access for MCP and API plugins in agents running in Microsoft 365 Copilot.
author: amitharjani93
ms.author: amith
ms.localizationpriority: medium
ms.date: 07/14/2026
ms.topic: how-to
---

# Configure anonymous authentication

For Model Context Protocol (MCP) servers or APIs that don't require any authentication, or for developer environments where authentication isn't yet implemented, plugins can access the MCP server or API anonymously.

This article uses MCP plugins as the default walkthrough. The same steps apply to API plugins built from an OpenAPI document, except where noted.

Unlike the other authentication schemes, anonymous access requires no registration, credentials, or auth config - you set the authentication type to `None` directly in the plugin manifest.

## Add anonymous authentication to the plugin manifest

When you add an MCP server that requires no authentication in Microsoft 365 Agents Toolkit or through the declarative agent developer skill, the plugin manifest is set to `None` for you. Set it manually only when you edit an existing plugin manifest yourself.

To configure anonymous access, set the `type` property of the [runtime authentication object](plugin-manifest-2.4.md#runtime-authentication-object) to `None`.

```json
"auth": {
  "type": "None"
},
```

> [!NOTE]
> Anonymous access is intended for development and testing. Add authentication before you deploy your plugin to production.

## Related content

- [Configure authentication for MCP and API plugins in agents](plugin-authentication.md)
- [Configure OAuth 2.0 authentication](plugin-authentication-oauth.md)
- [Configure Microsoft Entra SSO authentication](plugin-authentication-entra-sso.md)
- [Troubleshoot MCP and API plugin authentication](plugin-authentication-troubleshooting.md)
