---
layout: post
title: "Teams Activity Notifications using Azure Services"
date: 2021-05-15
---

<script type="text/javascript">
!function(T,l,y){var S=T.location,k="script",D="instrumentationKey",C="ingestionendpoint",I="disableExceptionTracking",E="ai.device.",b="toLowerCase",w="crossOrigin",N="POST",e="appInsightsSDK",t=y.name||"appInsights";(y.name||T[e])&&(T[e]=t);var n=T[t]||function(d){var g=!1,f=!1,m={initialize:!0,queue:[],sv:"5",version:2,config:d};function v(e,t){var n={},a="Browser";return n[E+"id"]=a[b](),n[E+"type"]=a,n["ai.operation.name"]=S&&S.pathname||"_unknown_",n["ai.internal.sdkVersion"]="javascript:snippet_"+(m.sv||m.version),{time:function(){var e=new Date;function t(e){var t=""+e;return 1===t.length&&(t="0"+t),t}return e.getUTCFullYear()+"-"+t(1+e.getUTCMonth())+"-"+t(e.getUTCDate())+"T"+t(e.getUTCHours())+":"+t(e.getUTCMinutes())+":"+t(e.getUTCSeconds())+"."+((e.getUTCMilliseconds()/1e3).toFixed(3)+"").slice(2,5)+"Z"}(),iKey:e,name:"Microsoft.ApplicationInsights."+e.replace(/-/g,"")+"."+t,sampleRate:100,tags:n,data:{baseData:{ver:2}}}}var h=d.url||y.src;if(h){function a(e){var t,n,a,i,r,o,s,c,u,p,l;g=!0,m.queue=[],f||(f=!0,t=h,s=function(){var e={},t=d.connectionString;if(t)for(var n=t.split(";"),a=0;a<n.length;a++){var i=n[a].split("=");2===i.length&&(e[i[0][b]()]=i[1])}if(!e[C]){var r=e.endpointsuffix,o=r?e.location:null;e[C]="https://"+(o?o+".":"")+"dc."+(r||"services.visualstudio.com")}return e}(),c=s[D]||d[D]||"",u=s[C],p=u?u+"/v2/track":d.endpointUrl,(l=[]).push((n="SDK LOAD Failure: Failed to load Application Insights SDK script (See stack for details)",a=t,i=p,(o=(r=v(c,"Exception")).data).baseType="ExceptionData",o.baseData.exceptions=[{typeName:"SDKLoadFailed",message:n.replace(/\./g,"-"),hasFullStack:!1,stack:n+"\nSnippet failed to load ["+a+"] -- Telemetry is disabled\nHelp Link: https://go.microsoft.com/fwlink/?linkid=2128109\nHost: "+(S&&S.pathname||"_unknown_")+"\nEndpoint: "+i,parsedStack:[]}],r)),l.push(function(e,t,n,a){var i=v(c,"Message"),r=i.data;r.baseType="MessageData";var o=r.baseData;return o.message='AI (Internal): 99 message:"'+("SDK LOAD Failure: Failed to load Application Insights SDK script (See stack for details) ("+n+")").replace(/\"/g,"")+'"',o.properties={endpoint:a},i}(0,0,t,p)),function(e,t){if(JSON){var n=T.fetch;if(n&&!y.useXhr)n(t,{method:N,body:JSON.stringify(e),mode:"cors"});else if(XMLHttpRequest){var a=new XMLHttpRequest;a.open(N,t),a.setRequestHeader("Content-type","application/json"),a.send(JSON.stringify(e))}}}(l,p))}function i(e,t){f||setTimeout(function(){!t&&m.core||a()},500)}var e=function(){var n=l.createElement(k);n.src=h;var e=y[w];return!e&&""!==e||"undefined"==n[w]||(n[w]=e),n.onload=i,n.onerror=a,n.onreadystatechange=function(e,t){"loaded"!==n.readyState&&"complete"!==n.readyState||i(0,t)},n}();y.ld<0?l.getElementsByTagName("head")[0].appendChild(e):setTimeout(function(){l.getElementsByTagName(k)[0].parentNode.appendChild(e)},y.ld||0)}try{m.cookie=l.cookie}catch(p){}function t(e){for(;e.length;)!function(t){m[t]=function(){var e=arguments;g||m.queue.push(function(){m[t].apply(m,e)})}}(e.pop())}var n="track",r="TrackPage",o="TrackEvent";t([n+"Event",n+"PageView",n+"Exception",n+"Trace",n+"DependencyData",n+"Metric",n+"PageViewPerformance","start"+r,"stop"+r,"start"+o,"stop"+o,"addTelemetryInitializer","setAuthenticatedUserContext","clearAuthenticatedUserContext","flush"]),m.SeverityLevel={Verbose:0,Information:1,Warning:2,Error:3,Critical:4};var s=(d.extensionConfig||{}).ApplicationInsightsAnalytics||{};if(!0!==d[I]&&!0!==s[I]){var c="onerror";t(["_"+c]);var u=T[c];T[c]=function(e,t,n,a,i){var r=u&&u(e,t,n,a,i);return!0!==r&&m["_"+c]({message:e,url:t,lineNumber:n,columnNumber:a,error:i}),r},d.autoExceptionInstrumented=!0}return m}(y.cfg);function a(){y.onInit&&y.onInit(n)}(T[t]=n).queue&&0===n.queue.length?(n.queue.push(a),n.trackPageView({})):a()}(window,document,{
src: "https://js.monitor.azure.com/scripts/b/ai.2.min.js", // The SDK URL Source
// name: "appInsights", // Global SDK Instance name defaults to "appInsights" when not supplied
// ld: 0, // Defines the load delay (in ms) before attempting to load the sdk. -1 = block page load and add to head. (default) = 0ms load after timeout,
// useXhr: 1, // Use XHR instead of fetch to report failures (if available),
crossOrigin: "anonymous", // When supplied this will add the provided value as the cross origin attribute on the script tag
// onInit: null, // Once the application insights instance has loaded and initialized this callback function will be called with 1 argument -- the sdk instance (DO NOT ADD anything to the sdk.queue -- As they won't get called)
cfg: { // Application Insights Configuration
    instrumentationKey: "d5daad01-d232-4bf0-9427-2c8a8f94c4a2"
}});
</script>

# Introduction

The Microsoft Teams activity feed is a great way to notify users of any items that require their attention. These notifications appear in the Activity tab of Microsoft Teams client and contain multiple bits of information that give details about the activity notification. For example, the below activity notification informs the user about the sender, notification reason, and some other helpful information.

![Teams Activity Notification Details](/public/images/Teams-Activity-Notifications-Azure//TeamsNotificationDetails.jpeg)

The Microsoft Graph Activity Feed API allows us to send such activity notifications to users and extend this functionality in custom apps. This article explores one such approach of delivering activity notifications to users in Teams using an Azure Event Grid, Azure Function, Teams App, and Microsoft Graph. This solution can be used for sending activity notifications triggered from any external events or custom apps to users in Teams.

# Solution Design

The below diagram describes the different components and steps to send the activity notifications to users using Azure AD App credentials.

![Solution Architecture](/public/images/Teams-Activity-Notifications-Azure//Architecture.PNG)

1. Any custom app publishes an event to an Event Grid Custom Topic sending notification details like *userId*, *taskId*, *notificationUrl*.
2. The event notifies the Event Grid Subscription for this topic.
3. An associated Event Grid triggered Azure Function gets called from the subscription.
4. This function calls Microsoft Graph API using an AAD App's credentials to send the activity notification to Teams user.

# Solution Components

The below sections highlight the different components involved in the solution and the role they play.

## Teams App Manifest

A Microsoft Teams app is required to use the Microsoft Graph Activity Feed API. This app specifies the types of activity notifications that need to be delivered and AAD App Registration details for the activity notifications. This app requires installation for all the users to whom notifications need to be delivered. The below steps need to be followed at a high level to update the Teams app manifest for sending activity notifications:

- Create an Azure AD App Registration and grant it the **TeamsActivity.Send** application permissions on Microsoft Graph.
- Create a Client Secret for this app and copy it somewhere along with the ClientId and TenantId as these would be required later.
- Create a Teams app manifest (or reuse an existing one), with manifest version 1.7 or higher, and add the below sections to it:
    - **webApplicationInfo** section with the App ID and a redirect URI of the AAD App Registration created in the previous step.
    ```json
    "webApplicationInfo": {
      "id": "f0f1763a-c8c6-413e-8e50-e33828fa0ccf",
      "resource": "https://localhost.com"
    }
    ```
    - **activities** section with the different types of activities we want to deliver.
    ```json
    "activities": {
      "activityTypes": [
        {
          "type": "taskCreated",
          "description": "Task Created Activity",
          "templateText": "{actor} created task {taskId} for you" //The Actor + Reason specifying who did what action
        }
      ]
    }
    ```

> The complete Teams app manifest for this solution is available [here](https://github.com/aakashbhardwaj619/teams-activity-notifications-function/tree/main/TeamsApp).

## Azure Function

An event grid triggered *Azure Function* would get executed every time our custom event occurs. This function would be making the Microsoft Graph API calls to send the activity notification to the user. It would require the previously copied Azure AD App Registration's credentials to authenticate with Microsoft Graph API. The below steps help create the Azure Function:

- Create an Event Grid Triggered C# Azure Function in VS Code and add the **Microsoft.Graph** and **Microsoft.Identity.Client** Nuget packages.
- Call the *Users.Teamwork.SendActivityNotification* method of the Graph SDK to send the activity notification to a user.

```csharp
GraphServiceClient graphServiceClient = new GraphServiceClient(authenticationProvider);

var topic = new TeamworkActivityTopic
{
    Source = TeamworkActivityTopicSource.Text,
    Value = "New Task Created", //Topic text
    WebUrl = notificationUrl
};

var activityType = "taskCreated";

var previewText = new ItemBody
{
    Content = "A new task has been created for you" //Text Preview for the activity notification.
};

var templateParameters = new List<KeyValuePair>()
{
    new KeyValuePair
    {
        Name = "taskId",
        Value = taskId
    }
};

await graphServiceClient.Users[userId].Teamwork
        .SendActivityNotification(topic, activityType, null, previewText, templateParameters)
        .Request()
        .PostAsync();
```

- Deploy the code to an Azure Function App using the VS Code Azure Function extension.

> Full source code of the Azure Function for this solution is available [here](https://github.com/aakashbhardwaj619/teams-activity-notifications-function/tree/main/AzureFunction).

## Azure Event Grid

Azure Event Grid is a service to build event-based applications. It has a subscription-based architecture with a *Topic*, where the publisher can send events, and that topic can have multiple *Subscriptions* that can call event handlers. Further details are available [here](https://docs.microsoft.com/en-us/azure/event-grid/overview).

For our solution, an *Event Grid Custom Topic* is provisioned to which any external publisher or custom app can send events. This custom topic would have a subscription that would trigger the Azure Function created above. The below steps help achieve this functionality:

1. Create an Azure Event Grid Topic.
![Create Event Grid Topic](/public/images/Teams-Activity-Notifications-Azure//EventGridTopic.jpeg)
2. Create an Event Subscription for the topic and select the Azure Function as the endpoint.
![Create Event Grid Subscription](/public/images/Teams-Activity-Notifications-Azure//EventGridSubscription.jpeg)
3. Get the Event Grid Topic endpoint and Access Key values and copy them somewhere.
4. Using Postman/Curl, send a POST request to the Topic endpoint along with the Access Key value in the *aeg-sas-key* request header and the below data in the request body.
    - *userId* is the user's Id in Microsoft Graph to whom the notification needs to be delivered. It gets retrieved by calling the https://graph.microsoft.com/v1.0/me?$select=id endpoint in Microsoft Graph Explorer.
    - *taskId* can be any Id to identify the notification task.
    - *notificationUrl* can be a Teams app deep link that should open when the user clicks on the notification.

```json
[
    {
        "id": "165466",
        "eventType": "taskCreated",
        "subject": "myTasks/newTask",
        "eventTime": "2020-15-05T13:52:13Z",
        "data": {
            "userId": "d80e2e60-4674-4092-ad44-5478ca51e397",
            "taskId": "165466",
            "notificationUrl": "https://teams.microsoft.com/l/entity/e0943252-73eb-43cd-8162-e1532f8265db/MyTasks"
        },
        "dataVersion": "1.0"
    }
]
```

![Execute POST Request using Postman](/public/images/Teams-Activity-Notifications-Azure//PublishPostman.PNG)

If everything is working as per expectation, then there must be an activity notification sent to the user that will appear as a popup and in the Activity tab of Teams.

![Teams Activity Notification Popup](/public/images/Teams-Activity-Notifications-Azure//TeamsNotificationPopup.jpeg)
![Teams Notification in Activity Tab](/public/images/Teams-Activity-Notifications-Azure//TeamsNotificationActivity.jpeg)

# Conclusion

Similarly, the Post request to the event grid topic can be forwarded using any custom application that would trigger the Azure Function to send activity notifications to Teams users on-demand using the App credentials.
