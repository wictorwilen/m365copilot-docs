---
title: Build People Connector for Microsoft 365 Copilot (preview)
description: Learn how to build your first Microsoft 365 Copilot connector for people data using the Microsoft Graph SDK (preview).
author: wictorwilen
ms.author: wictorwilen
ms.localizationpriority: medium
ms.date: 06/09/2025
ms.topic: how-to
---

# Build your first custom Copilot connector for people data using the Microsoft Graph SDK (preview)

[Microsoft 365 Copilot connectors for people data](https://learn.microsoft.com/en-us/graph/peopleconnectors) enable you to ingest people data and knowledge from your source systems (for example HR, talent management or other people systems) into Microsoft Graph to make it available to Microsoft 365 Copilot and people experiences such as the profile card and people search. When you ingest your data, Copilot can reason over the data and use it to respond to user prompts.

> [!IMPORTANT]
> Microsoft 365 Copilot connectors for people data built using the Microsoft Graph API are currently in public preview with limited functionality. See additional notes and limitations.

This article walks you through the steps to build your first Copilot connector by using the Microsoft Graph SDK in Visual Studio Code.

## Prerequisites

You need the following prerequisites to complete the steps in this article:

- A Microsoft 365 developer tenant (If you don't have a developer tenant, you might qualify for one through the [Microsoft 365 Developer Program](https://developer.microsoft.com/microsoft-365/dev-program))
- [.NET SDK](https://dotnet.microsoft.com/en-us/download)
- [Visual Studio Code](https://code.visualstudio.com/)
- The ability to admin consent in Microsoft Entra admin center. You must be or complete this step as a Global administrator. See [Grant tenant-wide admin consent to an application](/entra/identity/enterprise-apps/grant-admin-consent#prerequisites) for the required roles.
- Your user must have the role Search Administrator, Cloud Application Developer to see the connector in the Microsoft 365 admin center.

## Overview of Microsoft 365 Copilot connectors for people data

You build Microsoft 365 Copilot connectors for people data in the same way as other Copilot connectors using the Microsoft Graph external connections APIs. To ensure Microsoft 365 recognizes your connection as containing people data, you must follow certain schema requirements and register the connection as a source of profile data.

### Connection schema requirements

To ensure Microsoft 365 recognizes your connection as containing people data, your schema must have the following properties:

- `connectionId` of type `string` representing the Id of the connection (this requirement is expected to be removed before GA).
- `accounts` of type `string` representing the account of the user being enriched. The value of this must be a string encoded JSON object of the profile [userAccountInformation](https://learn.microsoft.com/en-us/graph/api/resources/useraccountinformation?view=graph-rest-beta) entity with the `userPrincipalName` and `externalDirectoryObjectId` properties set to values representing the person to be enriched.

> [!IMPORTANT]
> We expect changes during the public preview and ahead of general availability of this core schema configuration. Please regularly check this page for updates.

### Registering the connection as a profile source

After you create the connection and follow the schema requirements above, you must register the connection as a source of profile data and add it to the list of prioritized sources.

You register the connection as a profile source by using the [Profile source API](https://learn.microsoft.com/en-us/graph/api/peopleadminsettings-post-profilesources?view=graph-rest-beta&tabs=http) with `sourceId` set to the connection ID and the `webUrl` property set to a HTTPS link to either the external system or a page with additional information about the source.

After registration, you must add the connection to the list of prioritized profile sources using the [Profile property settings API](https://learn.microsoft.com/en-us/graph/api/profilepropertysetting-update?view=graph-rest-beta&tabs=http). Add the URL to the profile source, in the format of `https://graph.microsoft.com/beta/admin/people/profileSources(sourceId='connectionId')` where `connectionId` is the unique id of the connection, to the `prioritizedSourceUrls` array. This array represents the order in which Microsoft 365 composes the view of a person. If you want your connection to be the highest prioritized source, add it as the first item in the array.

## Build your first custom connector

The following walkthrough guides you through the minimal steps to create a custom Microsoft 365 Copilot connector for people data. You can find a more elaborate example here: [Github: ContosoHrConnector](TODO)

### Entra ID app registration

Follow these steps to create a new Entra ID app registration for your people connector.

1. Log in to the [Azure Entra ID portal](https://aad.portal.azure.com) using a global administrator role.
1. Select **Applications > App registrations** and click on **+ New registration**.
1. Type the name of your application in the **Name** text box. Ex *ContosoHrConnector*.
1. Click **Register** to complete the registration.
1. Select **API Permissions** and choose **+ Add a permission** to add permissions to the app.
1. Choose **Microsoft Graph** and then **Application permissions** and select the following permission scopes:
    1. **ExternalConnection.ReadWrite.All** (required to create the connection and schema)
    1. **ExternalItem.ReadWrite.All** (required to ingest people data)
    1. **PeopleSettings.ReadWrite.All** (required to add the connection as a profile source)
    1. **Users.Read.All** (preview requirement for this walkthrough)
1. Click **Add Permissions** and then click **Grant admin consent for Contoso** (replace Contoso with your organization’s name) to grant these permissions to the application. Select **Yes** to complete the grant.
1. Select **Certificates & secrets** and create a new secret with **+ New client secret**. Give it an appropriate description and expiry length and click **Add**.
1. Note the **Secret** value and store it in a safe location.
1. Click **Overview** and record the *Application (client) ID* and *Directory (tenant) ID*.

> [!TIP]
> For production scenarios, create two different applications—one to create the connection, schema, and perform the profile source registration, and another for the actual ingestion. Use Managed Identities for credentials instead of storing client secrets.

### Connection application

Follow these instructions to create the console application for the people connector:

1. On a machine where you have installed the [.NET SDK](https://dotnet.microsoft.com/en-us/download), open a terminal window and type the following to create a new console application: `dotnet new console --name ContosoHrConnector`.
1. Navigate to the newly created folder with `cd ContosoHrConnector`.
1. In the console, type the following to add the required packages: `dotnet add package Azure.Identity`, `dotnet add package Microsoft.Extensions.Configuration.Binder`, `dotnet add package Microsoft.Extensions.Configuration.UserSecrets`, `dotnet add package Microsoft.Graph.Beta --prerelease` and `System.CommandLine --prerelease`.
1. Open Visual Studio Code with `code .`.

> [!NOTE]
> This sample application uses pre-release (beta) packages for Microsoft.Graph and System.CommandLine during the preview.

In *Program.cs*, replace all code with the following. This code creates a simple CLI experience with a *setup* command to set up the connection, a *register* command to register the connection as a profile source, and a sync command to ingest people data.

```csharp
using System.CommandLine;

var rootCommand = new RootCommand("Contoso HR Connector");
var setupCommand = new Command("setup", "Setup the Contoso HR Connector");
var registerCommand = new Command("register", "Register the Contoso HR Connector");
var syncCommand = new Command("sync", "Sync the Contoso HR Connector");
rootCommand.AddCommand(setupCommand);
rootCommand.AddCommand(registerCommand);
rootCommand.AddCommand(syncCommand);

setupCommand.SetHandler(() =>
{
    Console.WriteLine("Setting up the Contoso HR Connector...");
    // Add setup logic here
});

registerCommand.SetHandler(() =>
{
    Console.WriteLine("Registering the Contoso HR Connector...");
    // Add registration logic here
});

syncCommand.SetHandler(() =>
{
    Console.WriteLine("Syncing the Contoso HR Connector...");
    // Add sync logic here
});

// Start the command line
await rootCommand.InvokeAsync(args);
```

### Set up authorization to Microsoft Graph

To connect the application to Microsoft Graph using the Entra ID app registration, run the following commands in the terminal window, in the directory of the console application. Replace the client ID, tenant ID, and client secret with the values you stored in a safe location.

``` bash
dotnet user-secrets init
dotnet user-secrets set settings:clientId <client-id>
dotnet user-secrets set settings:tenantId <tenant-id>
dotnet user-secrets set settings:clientSecret <client-secret>
```

Update the imports so they match the following:

``` csharp
using System.CommandLine;
using System.Text.Json;
using Azure.Identity;
using Microsoft.Extensions.Configuration;
using Microsoft.Graph;
using Microsoft.Graph.Beta;
using Microsoft.Graph.Beta.Models.ExternalConnectors;
using Microsoft.Kiota.Authentication.Azure;
```

Then update *Program.cs* by adding the following code just before the `setupCommand.SetHandler` row:

``` csharp
var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();
var credential = new ClientSecretCredential(
    config["settings:tenantId"], config["settings:clientId"], config["settings:clientSecret"]);
var httpClient = GraphClientFactory.Create();
var authProvider = new AzureIdentityAuthenticationProvider(
    credential, scopes: ["https://graph.microsoft.com/.default"]);
var graphClient = new GraphServiceClient(authProvider, "https://graph.microsoft.com/beta");
```

### Creating the connection and schema

To add the connection and schema creation, replace the `setupCommand.SetHandler` logic with the following code. This code first registers the connection with an Id and a name and then creates the schema. The schema contains the required properties as well as the *positions* out-of-the-box entity and a custom property called *favoriteColor*. Since creating the schema takes a few minutes, the code polls every five seconds to verify that the operation is complete.

``` csharp
var CONNECTOR_ID = "contosohrconnector1";
setupCommand.SetHandler(async () =>
{
    Console.WriteLine("Setting up the Contoso HR Connector...");

    var newConnectionParameters = new ExternalConnection
    {
        Id = CONNECTOR_ID,
        Name = "Contoso HR Connector",
    };

    var newConnection = await graphClient.External.Connections.PostAsync(newConnectionParameters);

    if (newConnection == null)
    {
        Console.WriteLine("Failed to create the Contoso HR Connector.");
        return;
    }

    Console.WriteLine("Configuring the schema, this may take a few minutes...");
    var requestInfo = graphClient.External.Connections[newConnection.Id].Schema.ToPatchRequestInformation(new Microsoft.Graph.Beta.Models.ExternalConnectors.Schema
    {
        BaseType = "microsoft.graph.externalItem",
        Properties =
           [
                new() {
                     Name = "connectionId",
                     Type = Microsoft.Graph.Beta.Models.ExternalConnectors.PropertyType.String
                },
                new() {
                     Name = "accounts",
                     Type = Microsoft.Graph.Beta.Models.ExternalConnectors.PropertyType.String
                },
                new() {
                     Name = "positions",
                     Type = Microsoft.Graph.Beta.Models.ExternalConnectors.PropertyType.String
                },
                new() {
                     Name = "favoriteColor",
                     Type = Microsoft.Graph.Beta.Models.ExternalConnectors.PropertyType.String
                }
           ]
    });

    // We will poll for the operation to complete, so the request
    // is customized to include a header that says we want an async response
    var requestMessage = await graphClient.RequestAdapter.ConvertToNativeRequestAsync<HttpRequestMessage>(requestInfo);
    _ = requestMessage ?? throw new Exception("Could not create native HTTP request");
    requestMessage.Method = HttpMethod.Patch;
    requestMessage.Headers.Add("Prefer", "respond-async");

    var responseMessage = await httpClient.SendAsync(requestMessage) ??
           throw new Exception("No response returned from API");

    // Wait for the operation to complete
    if (responseMessage.IsSuccessStatusCode)
    {
        var operationId = responseMessage.Headers.Location?.Segments.Last() ??
            throw new Exception("Could not get operation ID from Location header");
        do
        {
            var operation = await graphClient.External
                .Connections[newConnection.Id]
                .Operations[operationId]
                .GetAsync() ?? throw new ServiceException("Operation not found");

            if (operation?.Status == ConnectionOperationStatus.Completed)
            {
                return;
            }
            else if (operation?.Status == ConnectionOperationStatus.Failed)
            {
                throw new ServiceException($"Schema operation failed: {operation?.Error?.Code} {operation?.Error?.Message}");
            }
            // Wait 1 minute and then try again
            await Task.Delay(60000);
        } while (true);
    }
    else
    {
        throw new ServiceException("Registering schema failed",
            responseMessage.Headers, (int)responseMessage.StatusCode);
    }
});
```

You can now run the console application in your terminal window with the following: `dotnet run setup`. After the app completes, you have correctly created your connection and schema in your tenant.

### Registering the connection as a profile source

To ensure Microsoft 365 correctly propagates the people data in the connection into each user's profile, you must register the connection as a profile source and add it to the profile source prioritization list. You do this via two operations in Microsoft Graph.

Replace the code in the `registerCommand.SetHandler` method with the code below to implement this in the console app.

The first operation adds the connection using the connection id, a display name, and a web url.

The second operation reads the current list of prioritized profile sources and adds this new one on the top of the list, making it the most prioritized source of profile data.

``` csharp
registerCommand.SetHandler(async () =>
{
    Console.WriteLine("Registering the Contoso HR Connector...");
    _ = await graphClient.Admin.People.ProfileSources
        .PostAsync(new Microsoft.Graph.Beta.Models.ProfileSource
        {
            SourceId = CONNECTOR_ID,
            DisplayName = "Contoso HR Connector",
            WebUrl = "https://hr.contoso.com"
        }) ?? throw new ServiceException("Failed to register the Contoso HR Connector");

    var propertySettings = await graphClient.Admin.People.ProfilePropertySettings.GetAsync() ?? throw new ServiceException("No response returned from API");

    var globalSettings = propertySettings.Value?.SingleOrDefault(x => x.Name is null) ?? throw new ServiceException("No response returned from API");`  

    var sources = globalSettings.PrioritizedSourceUrls;
    var sourceId = globalSettings.Id;

    if (sources == null || sources.Count == 0)
    {
        sources = new List<string>([$"https://graph.microsoft.com/beta/admin/people/profileSources(sourceId='{CONNECTOR_ID}')"]);
    }
    else
    {
        sources.Insert(0, $"https://graph.microsoft.com/beta/admin/people/profileSources(sourceId='{CONNECTOR_ID}')");
    }

    var newPropertySetting = new Microsoft.Graph.Beta.Models.ProfilePropertySetting
    {
        PrioritizedSourceUrls = sources
    };

    var responseMessage = await graphClient.Admin.People.ProfilePropertySettings[sourceId]
        .PatchAsync(newPropertySetting) ?? throw new ServiceException("No response returned from API");
});
```

Save your file and run the following command in the terminal window to register the connection as a profile source: `dotnet run register`.

> [!NOTE]
> If you remove the connection, you must also remove the connection from the list of prioritized urls as well as from the list of profile sources.

### Synchronizing people profiles

The final step of creating this connection is to ingest data about people into Microsoft 365.

For this sample, you use a fixed set of people data, which you should replace with your own logic to connect to your source systems. Add the following code just before the `syncCommand.SetHandler` method. Replace the UPN value with correct values for your tenant and the users you want to enrich.

``` csharp
var people = new[]
{
    new { UPN = "alexw@contoso.com", 
        Department = "Engineering", 
        Position = "Software Engineer", 
        FavoriteColor = "Blue" },
    new { UPN = "luisg@contoso.com", 
        Department = "Marketing", 
        Position = "Marketing Manager", 
        FavoriteColor = "Green" }
};
```

Replace the code in the `syncCommand.SetHandler` method with the following code. This code iterates through the list of users and adds or updates them in the connection one by one. Note how the `accounts` and `positions` properties are serialized as JSON strings.

> [!TIP]
> Use a serializer to ensure proper formatting of the profile entity properties, instead of manually creating the JSON strings, as Microsoft Graph discards any malformed property.

``` csharp
syncCommand.SetHandler(async () =>
{
    Console.WriteLine("Syncing the Contoso HR Connector...");

    foreach (var person in people)
    {
        Console.WriteLine($"Syncing {person.UPN}...");
        var personIdentifier = person.UPN.Replace("@", "_at_").Replace(".", "_dot_");
        Console.WriteLine($"Person identifier: {personIdentifier}");
        var graphUser = await graphClient.Users.GetAsync(requestConfiguration =>
            requestConfiguration.QueryParameters.Filter = $"userPrincipalName eq '{person.UPN}'") ?? throw new ServiceException($"User with UPN {person.UPN} not found in Microsoft Graph.");

        var user = graphUser.Value?.FirstOrDefault();
        var oid = user?.Id ?? throw new ServiceException($"User with UPN {person.UPN} does not have an OID.");

        var newItem = new ExternalItem
        {
            Id = personIdentifier,
            Acl =
            [
                new Acl
                {
                    AccessType = AccessType.Grant,
                    Type = AclType.Everyone,
                    Value = "EVERYONE"
                }
            ],
            Properties = new Properties
            {
                AdditionalData = new Dictionary<string, object>
                {
                    { "connectionId", CONNECTOR_ID },
                    { "accounts", JsonSerializer.Serialize(new[]
                        {
                            new {
                                userPrincipalName = person.UPN,
                                externalDirectoryObjectId = oid,
                            }
                        }
                    )},
                    { "positions", JsonSerializer.Serialize(new[]
                        {
                            new {
                                detail = new
                                {
                                    jobTitle = person.Position,
                                    company = new
                                    {
                                        department = person.Department
                                    },
                                },
                                isCurrent = true
                            }
                        }
                    )},
                    { "favoriteColor", person.FavoriteColor }
                }
            }
        };

        await graphClient.External
              .Connections[CONNECTOR_ID]
              .Items[personIdentifier]
              .PutAsync(newItem);
    }
});
```

> [!NOTE]
> During the public preview, the externalObjectId is required in the accounts property. This is done via a lookup of the user in Microsoft Graph. Microsoft intends to remove this extra property and Microsoft Graph call prior to general availability.

To run this sync, type the following command into your terminal window: `dotnet run sync`.

## Notes and limitations

- Microsoft 365 treats all ingested data as organizational public data.
- You must set the ACL exactly as shown in the code example above.
- The schema requires that `connectionId` and `accounts`, as described above, must be present.
- Microsoft Graph discards people data without matching `userPrincipalName` and `externalDirectoryObjectId` in the `accounts` entity collection.
- Microsoft 365 only supports the following reserved profile entities for enrichment, and you must follow the JSON schema for the entities.
  - [`accounts`](https://learn.microsoft.com/en-us/graph/api/resources/useraccountinformation?view=graph-rest-beta). Max 1, see above for minimum schema requirements.
  - [`positions`](https://learn.microsoft.com/en-us/graph/api/resources/workposition?view=graph-rest-beta). Max 1 position.
  - [`names`](https://learn.microsoft.com/en-us/graph/api/resources/personname?view=graph-rest-beta). Max 1 name.
  - [`notes`](https://learn.microsoft.com/en-us/graph/api/resources/personannotation?view=graph-rest-beta). Max 1 note.
  - [`emails`](https://learn.microsoft.com/en-us/graph/api/resources/itememail?view=graph-rest-beta). Max 3 e-mails.
  - [`addresses`](https://learn.microsoft.com/en-us/graph/api/resources/itemaddress?view=graph-rest-beta). Max 3, one of each of Home, Work and Other.
  - [`anniversaries`](https://learn.microsoft.com/en-us/graph/api/resources/personanniversary?view=graph-rest-beta). One of each wedding, birthday and work.
  - [`phones`](https://learn.microsoft.com/en-us/graph/api/resources/itemphone?view=graph-rest-beta)
  - [`webAccounts`](https://learn.microsoft.com/en-us/graph/api/resources/webaccount?view=graph-rest-beta)
  - [`webSites`](https://learn.microsoft.com/en-us/graph/api/resources/personwebsite?view=graph-rest-beta). Max 1 web site.
  - [`skills`](https://learn.microsoft.com/en-us/graph/api/resources/skillproficiency?view=graph-rest-beta)
  - [`projects`](https://learn.microsoft.com/en-us/graph/api/resources/projectparticipation?view=graph-rest-beta)
  - [`awards`](https://learn.microsoft.com/en-us/graph/api/resources/personaward?view=graph-rest-beta)
  - [`certifications`](https://learn.microsoft.com/en-us/graph/api/resources/personcertification?view=graph-rest-beta)
- You must provide valid string encoded JSON objects for profile entities. Microsoft Graph ignores invalid values.
- You must always provide profile entities as an array of entities.
- Microsoft Graph treats any other properties, besides the reserved profile entities above in the connection schema, as a custom property.
- Custom properties show up in profile cards as notes during the preview, but Microsoft will remove them before or at general availability.
- Microsoft 365 might take up to 48 hours after you ingest data about a person before it becomes available in people experiences or Copilot.
- Connections with people data do not support staged connections.
- Indexed items in connections with people data only appear in people search.

## Related content

- [Copilot connectors API](/graph/connecting-external-content-connectors-api-overview?context=%2Fmicrosoft-365-copilot%2Fextensibility%2Fcontext)
- [Copilot connector samples](samples.md#copilot-connector-samples)
- [Community samples](https://github.com/pnp/graph-connectors-samples)
