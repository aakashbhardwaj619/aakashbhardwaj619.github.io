---
layout: post
title: "Add SharePoint site to Teams Personal App"
date: 2020-01-16
---

## Introduction

With the increasing adoption of Microsoft Teams it is a common requirement from users to get most of their daily work done within Teams itself. This also leads to a use case to render a frequently visited SharePoint site/news page or an intranet home page as a personal Teams app so that it can be accessed in Teams without having to switch to the browser.

There is currently an option to add a SharePoint Page or List as a tab in a Teams channel but there is no Out of the Box approach for adding a SharePoint site as a personal Teams app. This functionality can be achieved within minutes by creating a custom app using Microsoft Teams AppStudio. In this article we will see how to create this no code custom app and also about its distribution and deployment.

## Creating the App

App Studio is a Teams app which can be used to quickly create custom Microsoft Teams apps without writing any code. It helps streamlining the process of creating the app manifest JSON file and the app package consisting of the manifest and icons. Follow  [https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/build-and-test/app-studio-overview](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/build-and-test/app-studio-overview)  for installing the app from Teams store as we would need this to create our custom app.

Once App Studio is installed follow the below steps for creating the new app:

-   Open App Studio and in the Manifest Editor tab of App Studio click on  **Create a New App.**

![Create New App](/public/images/Teams-Personal-App/TeamsApp-1.png)

-   Fill out basic details about the app in the  **App Details**  section like App Name, App ID, App Package Details, Developer Information and App URLs for privacy statement and terms of use.

![App Details](/public/images/Teams-Personal-App/TeamsApp-2.png)

-   In the  **Tabs**  section click on  **Add a personal tab**  button and enter the Tab Name, ID and Content URL (Page URL) of the SharePoint page(s) that you want to render.

> The conten page will be rendered in an iFrame in Teams and will only support read operations

![Tab Details](/public/images/Teams-Personal-App/TeamsApp-3.png)

-   In the  **Domains and Permissions**  section you will find your site URL in the Valid Domains section.
-   In the  **Test and Distribute**  section click on  **Install**  to install your app for testing. While in testing the app would not be available to all other users.
-   If you find that your page is displayed in your Teams app then it is ready to be installed in the tenant for all users.

![SharePoint in Teams personal app](/public/images/Teams-Personal-App/TeamsApp-4.png)

-   Click on the  **Download** button in  **Test and Distribute**  section to download the app package for this app that will consist of the app manifest along with the app icons.

## Distribution and Deployment

For an app to be available to all users its app package needs to be uploaded to the tenant Teams App Catalog. Follow the below steps for doing so:

-   Go to the  **Apps**  section and upload your custom app for all users using the  **Upload for <<tenantName>>**  option.

![Upload app package to tenant app catalog](/public/images/Teams-Personal-App/TeamsApp-5.png)

-   The app would now be available for all users to add. However, if you want to pin this app to the Teams sidebar so that it automatically becomes available to everyone that can also be done using the next steps.

### Pinning the app to sidebar

-   Go to  **Teams Admin Center**  at  [https://admin.teams.microsoft.com/dashboard](https://admin.teams.microsoft.com/dashboard).
-   Go to  **Setup Policies**  within Teams Apps.
-   Edit the  **Global Org Wide**  policy settings and click on  **Add Apps**  to add your custom app to the sidebar pinned apps.

![Setup Policies](/public/images/Teams-Personal-App/TeamsApp-6.png)

-   Search for and add the custom app. Update the sort order for all the apps if necessary.

![App order](/public/images/Teams-Personal-App/TeamsApp-7.png)

-   The custom app should now be visible in the Teams app sidebar for all users.

![No alt text provided for this image](/public/images/Teams-Personal-App/TeamsApp-8.png)

## Conclusion

Thus, we saw how we can add frequently accessed SharePoint site pages as tabs to a Teams personal app without writing any custom code using App Studio.
