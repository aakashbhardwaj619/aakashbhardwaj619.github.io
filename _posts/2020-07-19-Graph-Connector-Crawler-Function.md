---
layout: post
title: "Microsoft Graph Connector crawling using Azure Functions"
date: 2020-07-19
---

Introduction
------------

It has been a while since [Microsoft Graph Connectors](https://docs.microsoft.com/en-us/microsoftsearch/connectors-overview) went into public preview. It is a great way to index third party data to appear in Microsoft Search results. While Microsoft provides a couple of in built connectors to index data from sources such as Azure DevOps, ServiceNow, File Share, etc., to index data from any other custom service a connector can be built in Microsoft Graph that creates a connection and pushes items to the Microsoft Search index.

The basic steps involved in setting up a connector include making some Graph API calls for creating a connection, registering a schema that describes the type of external data and finally indexing the data. Detailed information regarding these API calls can be found [here](https://docs.microsoft.com/en-us/graph/api/resources/indexing-api-overview?view=graph-rest-beta)

The above mentioned steps help in the one time setup and indexing of external data in Microsoft Search index. However, for regular update of data an appropriate crawling mechanism needs to be implemented because currently there is no such provision available by default for connectors other than the Microsoft built connectors. This article describes one such approach for scheduled crawling of external data using an Azure timer-triggered function.

Azure Function Crawler
---------------

To keep the connector's external data up to date, the search indexing API calls need to be triggered at a regular schedule. This can be achieved using timer-triggered Azure functions. Depending on the amount of data and business requirement either a full crawl or an incremental crawl can be setup.

The below approach shows how to implement crawling of external data using a `dotnet` function. For this sample, the external data being indexed in Microsoft Search is coming from a SharePoint list but the approach can be modified to fetch data from any other custom service as well.

## Prerequisites

### Setup the custom connector

- Register an Azure AD App with `ExternalItem.ReadWrite.All` application permission on Microsoft Graph. Also create a Client Secret for the app and make a note of it along with App Id and Tenant Id.
- Create a connection
- Register a schema for the type of external data (For example, `Appliances` data as shown in below mentioned GitHub sample)
- Create a vertical
- Create a result type

> Refer to the sample [https://github.com/microsoftgraph/msgraph-search-connector-sample](https://github.com/microsoftgraph/msgraph-search-connector-sample) for setting up the connector and registering schema

### Create a SharePoint List 

Create a SharePoint list with the below columns (in accordance with the schema registered) that will serve as the external data source.

- `Title` of type `Single line of Text`
- `Description` of type `Single line of Text`
- `Appliances` of type `Single line of Text`
- `Inventory` of type `Number`
- `Price` of type `Number`

![SharePoint List](/public/images/SearchConnectorCrawler/SearchConnectorList.png)

Azure Function
--------------

- Create a timer triggered Azure function using VS Code as specified at [this link](https://docs.microsoft.com/en-us/azure/azure-functions/functions-develop-vs-code?tabs=csharp)
- Add the Nuget packages for `Microsoft.Graph.Beta` and `Microsoft.Identity.Client` using the below commands
```csharp
dotnet add package Microsoft.Graph.Beta --version 0.19.0-preview
dotnet add package Microsoft.Identity.Client --version 4.16.1
```
- For local development, update the `local.settings.json` file with the below configuration updating the required Ids, secrets and CRON schedule. Once deployed to Azure Function App add the same values as application settings.
```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "<STORAGE_ACCOUNT_CONNECTION_STRING>",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet",
    "schedule": "<CRON_SCHEDULE>",
    "appId": "<APP_ID>",
    "tenantId": "<TENANT_ID>",
    "secret": "<APP_SECRET>",
    "connectionId": "<CONNECTION_ID>",
    "siteId": "<SITE_ID>",
    "listId": "<LIST_ID>"
  }
}
```
- Create the below `MSALAuthenticationProvider.cs` class that will be used for authentication by the `GraphServiceClient` object

<script src="https://gist.github.com/aakashbhardwaj619/9ae23126f83ce59eb384217e5c2efc36.js"></script>

- Create the below `GraphHelper` class that contains methods for fetching SharePoint list items and updating the search index using `GraphServiceClient`
  - The `AddOrUpdateItem` method adds/updates an item to the search index
  - The `GetIncrementalListData` method fetches the items updated since a particular time (last trigger time of the function) and is used for incremental crawling
  - The `GetFullListData` method fetches all the items in the SharePoint list and is used for setting up full crawl

<script src="https://gist.github.com/aakashbhardwaj619/b76d4a4154bd93aaf572ec690b896088.js"></script>

- Update the function class with below code that calls the methods to get SharePoint list data and update the search index

<script src="https://gist.github.com/aakashbhardwaj619/9263208d3131573cd25c7ee00862e20a.js"></script>

- Add/Update an item in the SharePoint List and check if the search results return the updated item results after the function is triggered

![Search Results](/public/images/SearchConnectorCrawler/SearchResults.png)

> The entire source code for this sample is available at [this link](https://github.com/aakashbhardwaj619/function-search-connector-crawler)

Conclusion
----------

![Azure Function Logs](/public/images/SearchConnectorCrawler/AzureFunctionLogs.png)

In this way, Azure functions can be used for scheduled crawling of external data in Microsoft Search to ensure that latest data is available in the search results. The above sample can be extended to deal with data from other sources and handle source item deletion scenarios as well.
