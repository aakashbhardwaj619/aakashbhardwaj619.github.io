

Introduction
------------

It has been a while since Microsoft Search Connectors went into public preview. It is a great way to index third party data to appear in Microsoft Search results. While Microsoft provides a couple of in built connectors to index data from sources such as Azure DevOps, ServiceNow, File Share, etc., to index data from any other custom service users need to build their own custom connectors.

The basic steps involved in setting up a custom connector include some Graph API calls for Creating a connection to the data source, creating a schema that describes the type of external data and indexing the external data. Details information regarding these API calls can be found at [this link](https://docs.microsoft.com/en-us/graph/api/resources/indexing-api-overview?view=graph-rest-beta)

The above steps help in the one time setup and indexing of external data in Microsoft Search. But for regular refresh of this data, an appropriate crawl strategy needs to be implemented. This article describes the implementation of crawling using an Azure function.

> Refer to the sample [https://github.com/microsoftgraph/msgraph-search-connector-sample](https://github.com/microsoftgraph/msgraph-search-connector-sample) to see how Microsoft Graph indexing API can be used in .Net Core application for setting up a connector and indexing data 

Crawling using Azure Functions
---------------

To keep the connector's external data up to date, the API calls to index the latest data need to be called at a regular schedule. This can be achieved using timer triggered Azure functions. Depending on the amount of data and business requirement either a full crawl or an incremental crawl can be setup.

The below approach shows how to implement inremental crawling of external data through a Dotnet function. The sample data for this example has been picked as a SharePoint list data, but the approach can be modified to fetch the data from any other custom service as well.

## Prerequisites

### Setup the custom connector

- Create a connection
- Register a schema

> Refer to the sample [https://github.com/microsoftgraph/msgraph-search-connector-sample](https://github.com/microsoftgraph/msgraph-search-connector-sample) for setting up the connector and registering schema

### Create a SharePoint List 

In this step a SharePoint List will be created from which external data would be indexed to our search connector. Create a SharePoint list with the below columns (in accordance with the schema registered).

- `Title` of type `Single line of Text`
- `Description` of type `Single line of Text`
- `Appliances` of type `Single line of Text`
- `Inventory` of type `Number`
- `Price` of type `Number`

Azure Function
--------------

- Create a timer triggered Azure function using VS Code as specified at [this link]()
- Add the Nuget packages for `Microsoft.Graph.Beta` and `Microsoft.Identity.Client` using the below commands
```csharp
dotnet add package Microsoft.Graph.Beta --version 0.19.0-preview
dotnet add package Microsoft.Identity.Client --version 4.16.1
```
- Update the `local.settings.json` file with the below configuration
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
- Create the below `MSALAuthenticationProvider.cs` class that will be used for authentication by `GraphServiceClient` object

```csharp
public class ClientCredentialAuthProvider : IAuthenticationProvider
  {
    private IConfidentialClientApplication _msalClient;
    private int _maxRetries = 3;

    public ClientCredentialAuthProvider(string appId, string tenantId, string secret)
    {
      _msalClient = ConfidentialClientApplicationBuilder
          .Create(appId)
          .WithTenantId(tenantId)
          .WithClientSecret(secret)
          .Build();
    }

    public async Task AuthenticateRequestAsync(HttpRequestMessage request)
    {
      int retryCount = 0;

      do
      {
        try
        {
          var result = await _msalClient
              .AcquireTokenForClient(new[] { "https://graph.microsoft.com/.default" })
              .ExecuteAsync();

          if (!string.IsNullOrEmpty(result.AccessToken))
          {
            request.Headers.Authorization =
                new AuthenticationHeaderValue("bearer", result.AccessToken);
            break;
          }
        }
        catch (MsalServiceException serviceException)
        {
          if (serviceException.ErrorCode == "temporarily_unavailable")
          {
            var delay = GetRetryAfter(serviceException);
            await Task.Delay(delay);
          }
          else
          {
            throw new Exception("Error: ", serviceException);
          }
        }
        catch (Exception exception)
        {
          throw new Exception("Error: ", exception);
        }

        retryCount++;
      } while (retryCount < _maxRetries);
    }

    private TimeSpan GetRetryAfter(MsalServiceException serviceException)
    {
      var retryAfter = serviceException.Headers?.RetryAfter;
      TimeSpan? delay = null;

      if (retryAfter != null && retryAfter.Delta.HasValue)
      {
        delay = retryAfter.Delta;
      }
      else if (retryAfter != null && retryAfter.Date.HasValue)
      {
        delay = retryAfter.Date.Value.Offset;
      }

      if (delay == null)
      {
        throw new MsalServiceException(
            serviceException.ErrorCode,
            "Missing Retry-After header."
        );
      }

      return delay.Value;
    }
  }
```

- Create the below `GraphHelper` class that contains methods for fetching updated list items since last trigger time and updating the search index

```csharp
public class GraphHelper
  {
    private GraphServiceClient _graphClient;

    public GraphHelper(IAuthenticationProvider authProvider)
    {
      _graphClient = new GraphServiceClient(authProvider);
    }

    public async Task AddOrUpdateItem(string connectionId, ExternalItem item)
    {
      try
      {
        // The SDK's auto-generated request builder uses POST here,
        // which isn't correct. For now, get the HTTP request and change it
        // to PUT manually.
        var putItemRequest = _graphClient.External.Connections[connectionId]
            .Items[item.Id].Request().GetHttpRequestMessage();

        putItemRequest.Method = HttpMethod.Put;
        putItemRequest.Content = _graphClient.HttpProvider.Serializer.SerializeAsJsonContent(item);

        var response = await _graphClient.HttpProvider.SendAsync(putItemRequest);
        if (!response.IsSuccessStatusCode)
        {
          throw new ServiceException(
              new Error
              {
                Code = response.StatusCode.ToString(),
                Message = "Error indexing item."
              }
          );
        }
      }
      catch (Exception exception)
      {
        throw new Exception("Error: ", exception);
      }
    }

    public async Task<IListItemsCollectionPage> GetFullListData(string siteId, string listId)
    {
      var queryOptions = new List<QueryOption>()
      {
        new QueryOption("expand", "fields")
      };
      var items = await _graphClient.Sites[siteId].Lists[listId].Items
        .Request(queryOptions)
        .GetAsync();

      return items;
    }

    public async Task<IListItemsCollectionPage> GetIncrementalListData(string siteId, string listId, string lastModified)
    {
      string filterQuery = $"fields/Modified ge '{lastModified}'";

      var queryOptions = new List<QueryOption>()
      {
        new QueryOption("expand", "fields"),
        new QueryOption("filter", filterQuery)
      };
      var items = await _graphClient.Sites[siteId].Lists[listId].Items
        .Request(queryOptions)
        .GetAsync();

      return items;
    }
  }
```

- Update the function class with below code that calls the methods to fetch the updated list items and update the search index

```csharp
public static class CustomSearchCrawler
  {
    private static GraphHelper _graphHelper;
    private static ILogger _log;

    [FunctionName("CustomSearchCrawler")]
    public static async void Run([TimerTrigger("%schedule%")] TimerInfo myTimer, ExecutionContext context, ILogger log)
    {
      _log = log;

      _log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");

      var appConfig = LoadAppSettings(context);
      if (appConfig == null)
      {
        _log.LogInformation("The configuration values are missing.");
        return;
      }

      var authProvider = new ClientCredentialAuthProvider(
        appConfig["appId"],
        appConfig["tenantId"],
        appConfig["secret"]
      );

      _graphHelper = new GraphHelper(authProvider);

      string lastModifiedTime = myTimer.ScheduleStatus.Last.ToUniversalTime().ToString("s") + "Z";
      _log.LogInformation("Last Trigger time: {0}", lastModifiedTime);

      //Get data to be indexed
      
      //Incremental Crawl
      var listItems = await _graphHelper.GetIncrementalListData(appConfig["siteId"], appConfig["listId"], lastModifiedTime);
      //Full Crawl
      //var listItems = await _graphHelper.GetFullListData(appConfig["siteId"], appConfig["listId"]);

      if (listItems.Count == 0)
      {
        _log.LogInformation("Search index is up to date...");
      }
      else
      {
        await UpdateSearchIndex(listItems, appConfig["connectionId"], appConfig["tenantId"]);
      }
    }

    private static async Task UpdateSearchIndex(dynamic listItems, string connectionId, string tenantId)
    {
      _log.LogInformation("Updating custom search connector index...");
      foreach (var listItem in listItems)
      {
        int lid = Convert.ToInt32(listItem.Id.ToString());
        _log.LogInformation("Updating SharePoint list item: {0}", lid);

        string[] parts = listItem.Fields.AdditionalData["Appliances"].ToString().Split(',');
        List<string> partsList = new List<string>(parts);

        AppliancePart part = new AppliancePart()
        {
          PartNumber = lid,
          Description = listItem.Fields.AdditionalData["Description"].ToString(),
          Name = listItem.Fields.AdditionalData["Title"].ToString(),
          Price = Convert.ToDouble(listItem.Fields.AdditionalData["Price"].ToString()),
          Inventory = Convert.ToInt32(listItem.Fields.AdditionalData["Inventory"].ToString()),
          Appliances = partsList
        };

        var newItem = new ExternalItem
        {
          Id = part.PartNumber.ToString(),
          Content = new ExternalItemContent
          {
            // Need to set to null, service returns 400
            // if @odata.type property is sent
            ODataType = null,
            Type = ExternalItemContentType.Text,
            Value = part.Description
          },
          Acl = new List<Acl>
          {
            new Acl {
              AccessType = AccessType.Grant,
              Type = AclType.Everyone,
              Value = tenantId,
              IdentitySource = "Azure Active Directory"
            }
          },
          Properties = part.AsExternalItemProperties()
        };

        await _graphHelper.AddOrUpdateItem(connectionId, newItem);

        _log.LogInformation("Item updated!");
      }
      _log.LogInformation("Custom search connector index update completed...");
    }

    private static IConfigurationRoot LoadAppSettings(ExecutionContext context)
    {
      var appConfig = new ConfigurationBuilder()
      .SetBasePath(context.FunctionAppDirectory)
      .AddJsonFile("local.settings.json", optional: true, reloadOnChange: true)
      .AddEnvironmentVariables()
      .Build();

      if (string.IsNullOrEmpty(appConfig["appId"]) ||
          string.IsNullOrEmpty(appConfig["tenantId"]) ||
          string.IsNullOrEmpty(appConfig["secret"]) ||
          string.IsNullOrEmpty(appConfig["connectionId"]) ||
          string.IsNullOrEmpty(appConfig["siteId"]) ||
          string.IsNullOrEmpty(appConfig["listId"]))
      {
        return null;
      }

      return appConfig;
    }
  }
```

> The entire source code for this sample can be found at 

<script src="https://gist.github.com/aakashbhardwaj619/9ae23126f83ce59eb384217e5c2efc36.js"></script>

Conclusion
----------

In this way, Azure functions can be used for scheduled crawling of external data in Microsoft Search.
