---
layout: post
title: "Implement On-Behalf-Of Flow using C# Azure Function"
date: 2021-07-27
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

Anyone who has tried to call Graph API from a Teams app using Single Sign-On must have come across an authentication flow called On-Behalf-Of. The [OAuth 2.0 On-Behalf-Of (OBO)](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) flow is used when an app calls an API that needs to call another API on behalf of the logged-in user.
 
In this flow, the middle-tier service expects a user access token from the calling app and uses it, along with an Azure AD app's credentials, to secure another access token for calling the downstream service. In this article, we will discuss how to use a C# based Azure Function as the middle-tier API to fetch an access token for calling the downstream API, Microsoft Graph.

# Scenario

![OBOFlowDiagram](/public/images/Function-CSharp-Obo//OBOFlowDiagram.PNG)

1. An HTTP-triggered Azure Function is used as the middle-tier API. This function expects a user access token passed in the Authorization header when it is called from any app.
2. The Function uses the Microsoft Authentication Library (MSAL) to exchange the user access token for another access token for accessing the downstream API, Microsoft Graph. This also requires an AAD App's Client Id and Client Secret.
3. The new access token is returned to the app and can be used to call Microsoft Graph.

# Azure AD App Configuration

1. Go to Azure AD and create a new app registration with an appropriate name and type of supported account. Add the redirect URL `http://localhost:3000` as it would be required later in the Azure Function code.
![Create App Registration](/public/images/Function-CSharp-Obo//CreateAppReg.jpeg)
2. Go to Expose an API and set the APP URI to the default value: `api://<client-id>`.
![Expose API](/public/images/Function-CSharp-Obo//ExposeApi.jpeg)
3. Add a new scope with the appropriate details as this would be required to get the first access token with which the Azure function would be called.
![Add Scope](/public/images/Function-CSharp-Obo//AddScope.jpeg)
4. Create a new `client secret` and copy its value along with the application `client id` as these would be required later in the Azure function.

# Azure Function to exchange token

1. Create a new HTTP-triggered C# Azure Function using VS Code. Follow [this link](https://docs.microsoft.com/en-us/azure/azure-functions/create-first-function-vs-code-csharp) for reference.
2. Add the below NuGet packages as these would be required for validating the first user access token, and exchanging it for another access token to call the downstream API.
```
dotnet add package Microsoft.Identity.Client
dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect
dotnet add package System.IdentityModel.Tokens.Jwt
```
3. Before exchanging the user access token, it needs to be validated to check if the `issuer`, `audience`, and `signing keys` values are correct. Use the below method to validate the token.
```csharp
new JwtSecurityTokenHandler().ValidateToken
```
4. If the token is validated, use MSAL's `AcquireTokenOnBehalfOf` method to exchange the token for another access token that can access the downstream API. This method requires the scope for the downstream API that we are accessing along with the validated token.
```csharp
await app.AcquireTokenOnBehalfOf(scopes, userAssertion).ExecuteAsync();
```

> The entire source code for the Azure Function is available at [function-csharp-obo](https://github.com/aakashbhardwaj619/function-csharp-obo).

# Calling the Azure Function

To call the Azure function, we would need to pass the user access token that can be exchanged for another access token from Azure AD. The below steps can be followed to retrieve the first token using OAuth 2.0 Authorization Code Flow:

1. Access the below URL in a browser replacing the values of `client_id` and `scope`. It gets the authorization code from the authorization endpoint of AAD.
`https://login.microsoftonline.com/common/oauth2/v2.0/authorize?scope=api://<client_id>/user_impersonation&response_type=code&client_id=<client_id>&redirect_uri=http%3A%2F%2Flocalhost%3A3000%2Fredirect`
2. Copy the value of the `code` query parameter from the redirected URL. It is the authorization code and will be used to obtain the access token.
`http://localhost:3000/redirect?code=0.AXEA6a9vzJPdfEm1IR2XGxRxxximwaI6AH9Ji17H3Z9X-OZxAAE...&session_state=da89e5c0-a92a-4fb8-84a8-415eacce1117#`
3. Using Curl or Postman, make a POST request to the below mentioned token endpoint with the `code`, `client_id`, `client_secret`, `grant_type`, `scope`, and `redirect_uri` values.
`https://login.microsoftonline.com/common/oauth2/v2.0/token`
![Postman Token Request](/public/images/Function-CSharp-Obo//PostmanTokenRequest.PNG)
4. This returns the access token that will be used when calling the Azure Function.
5. Make a POST request to the Azure Function with the access token retrieved above as the bearer authorization token. It returns the access token that can be used to call the downstream API (Microsoft Graph, in this case).
![Image](/public/images/Function-CSharp-Obo//PostmanFunctionCall.PNG)
6. Make a call to the downstream API `https://graph.microsoft.com/v1.0/me` with the access token retrieved above as the bearer authorization token. It should return the user information for the logged-in user.
![Image](/public/images/Function-CSharp-Obo//PostmanGraphCall.PNG)

# Conclusion

In this way, Azure Functions can be used as the middle-tier API in an On-Behalf-Of flow and exchange the user access token for another higher privileged access token. Even though this article uses C# Azure Function, Node.Js functions can also be used to achieve similar functionality.
