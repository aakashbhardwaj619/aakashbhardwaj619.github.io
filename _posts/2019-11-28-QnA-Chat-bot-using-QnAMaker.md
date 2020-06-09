---
layout: post
title: "Create QnA Chat Bot using Microsoft Azure QnA Maker, SharePoint and Power Automate"
date: 2019-11-28
---

## Introduction

This article shows how a SharePoint Frequently Asked Questions (FAQ) List data can be set as the data source for a QnA Maker Knowledge Base and used in an SPFx Chat Bot web part so that users can get the answers to their queries from within SharePoint.

## Purpose

The purpose of this article is to demonstrate how to build a QnA Chat Bot using Microsoft Azure QnA Maker and Web App Bot. It also shows how the QnA Knowledge Base can be synced with a SharePoint list data using Power Automate and utilized in SharePoint using the PnP React Bot Framework SPFx web part sample.

## Technical Design

Below are the various components that are part of this solution along with the required steps to set them up.

### QnA Maker

QnA Maker is a cloud-based Natural Language Processing (NLP) service that can be used to find the most appropriate answer for any given natural language input, from your custom knowledge base (KB) of information. The KB will store all the QnA pairs from which the chat bot can find answers to user queries. Follow the below steps to create a new QnA KB:

-   Sign in to the [QnAMaker.ai](https://qnamaker.ai/) portal with your Azure credentials.
-   On the QnA Maker portal, select Create a knowledge base.
-   On the Create page, select Create a QnA service. You are directed to the [Azure portal](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) to set up a QnA Maker service in your subscription.
-   In the QnA Maker portal, select your QnA Maker service from the drop-down lists (**ABQNAService**  in this case). If you created a new QnA Maker service, be sure to refresh the page.

![Creating QnA Maker Knowledge Base](/public/images/QnAMaker-chatbot/CreateKB.PNG)

-   Give a name to your knowledge base eg.  **SPQnA**  in this case.
-   For now skip adding any custom data source in the  **Populate your KB**  step and just add _professional_ Chit-chat to your KB.
-   Select Create your KB.
-   Publish the KB once it is created so that we can use it in our Bot

![Publish the KB](/public/images/QnAMaker-chatbot/PublishKB.PNG)

### Azure Web App Bot

The Azure Web App Bot will be our client application that will use the above created KB to answer user queries. Follow the below steps to create the bot:

-   Select the  **Create Bot**  option from the previous step where the KB was published.

![Create Bot from the published KB](/public/images/QnAMaker-chatbot/PublishKB2.PNG)

-   Enter the required information like Bot handle, Resource Group, Location, Pricing Tier, App Name and Application Insights location.  **Do not change other properties**.

![Create Web App Bot page](/public/images/QnAMaker-chatbot/CreateBot.PNG)

-   Once the deployment is successful you can use the  **Test in Web Chat**  feature to test the chat bot.

![Test in Web Chat](/public/images/QnAMaker-chatbot/TestChat.png)

### SharePoint FAQ List

The SharePoint list will be the medium through which content editors would add or update existing question answer pairs that will be synced with the QnA Maker KB.

-   Create a new SharePoint List (Titled FAQ for this sample), rename its Title column to  **Question**  and add a new column  **Answer**  (Multiple lines of text).

![SharePoint FAQ List](/public/images/QnAMaker-chatbot/FAQList1.PNG)

### Power Automate Flow

This will be the main controller through which our QnA Maker KB will be kept in sync with the SharePoint FAQ list. To access the KB we will be using the  [QnA Maker REST API V4.0](https://docs.microsoft.com/en-us/rest/api/cognitiveservices/qnamaker/knowledgebase)  throught an  **HTTP Premium**  Action. To call the API we would need to have our  **KB Id**  as well as the  **Ocp-Apim-Subscription-Key**. These can be obtained as follows:

-   To get the  **KB Id**, go to the  **My knowledge bases** section in QnA Maker and click on  **View Code**  for the your KB (**SPQnA**  in this case). The below highlighted section is the KB Id.

![Get the Knowledgebase Id](/public/images/QnAMaker-chatbot/KBID.PNG)

-   To get the  **Ocp-Apim-Subscription-Key**  go to your QnAMaker service (**ABQNAService**  in this case) in Azure portal and get the value of  **Key1**.

![Get the Ocp-Apim-Subscription-Key](/public/images/QnAMaker-chatbot/KBKey.PNG)

Follow the below steps to create the Flow:

-   Login to  [https://flow.microsoft.com/](https://flow.microsoft.com/)  and create a new Flow that is triggerred  **When an item is created or modified**  in the SharePoint FAQ List.
-   Initialize the below shown variables as we would be using them later on.

![Trigger flow when an item is created or midified in the SharePoint list](/public/images/QnAMaker-chatbot/Flow1.png)

-   Add a  **Condition**  action to check if a new FAQ item has been created or an existing item has been modified. If the Created time is equal to the Modified time then it means that the trigger item is a new item added to the list and we will call the appropriate API to add a new QnA Pair in our KB.
-   In the Yes branch of above condition add a new  **HTTP** action. Here we will call the  **Update Knowledgebase**  endpoint to add a new QnA pair to our KB(Ref:  [https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da7600](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da7600)). Use the KB Id and Subscription Key as shown below in the action's URI and Headers fields. Add the Trigger item's  **Title**  and  **Answer**  values to the new QnA pair's  **question**  and  **answer**  properties in the HTTP body section along with an  **ItemID**  metadata value. Also set the  **StatusCode**  and  **EndpointURL**  variables to the HTTP action response Status Code and Location as we will be using these in next steps.

> Since the QnA pair's  **id**  property is immutable, we will also add a metadata name value pair for  **ItemID**. This will be used to map our SharePoint list items with the QnA pairs so that we can easily update or delete the QnA pair by updating or deleting the SharePoint list item itself.

![Add new QnA Pair to Update Knowledgebase for new item trigger](/public/images/QnAMaker-chatbot/Flow2.png)

-   In the No branch of the above condition we would handle the item modified scenario. Add a new  **HTTP**  action to call the  **Download Knowledgebase**  endpoint to download the KB (Ref:  [https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/knowledgebases_download](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/knowledgebases_download)).

> Here we would need to first download the KB to find the QnA pair  **id**  value of the QnA pair with metadata property  **ItemId**  equal to the trigger item ID. We will then delete the QnA pair with this id and add a new QnA pair with the updated question and answer.

-   Once we have the KB downloaded we will add a  **Parse JSON**  action followed by a  **Filter** action to get the QnA pair with metadata ItemID value equal to our trigger item ID. Set the QNAId variable to this QnA pair's  **id**.

![Download Knowledgebase and filter to find the QnA pair Id](/public/images/QnAMaker-chatbot/Flow3.png)

-   Now that we have the QnA pair id of the updated FAQ we will call the  **Update Knowledgebase**  endpoint to delete this QnA pair and add a new one with the updated question and answer. Also set the StatusCode and EndpointURL variables to the HTTP action response Status Code and Location like we did earlier.

![Delete and Add New QnA Pair using Update Knowledgebase endpoint](/public/images/QnAMaker-chatbot/Flow4.png)

-   After the previous Condition action, add another Condition to check if the Status Code variable is set to  **202**. If yes then it means that our previous HTTP request has been accepted but is yet to be processed since the Update Knowledgebase task is an  **asynchronous** operation.
-   In the Yes branch of the above condition add a  **Do Until**  action so that we can wait until the Operation Status (Ref:  [https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/operations_getoperationdetails](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/operations_getoperationdetails)) of the Update Knowledgebase HTTP operation (EndPointURL variable we set in the last step) becomes  **Succeeded**.

> We will keep calling the endpoint to get the async operation's status until it gets succeeded so that we can proceed to next step once the async operation has been successful.

![Get Update Knowledgebase operation status and publish Knowledgebase](/public/images/QnAMaker-chatbot/Flow5.png)

-   Once the Update Knowledgebase operation is successful we call the  **Publish Knowledgebase**  endpoint (Ref:  [https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75fe](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75fe)) so that we can use the updated Knowledgebase in our chat bot.

### React Bot Framework SPFx Web Part

The  [React Bot Framework SPFx sample](https://github.com/SharePoint/sp-dev-fx-webparts/tree/master/samples/react-bot-framework)  web part will be used to provide user the interface to chat with the bot inside SharePoint.

-   Git clone the  [https://github.com/SharePoint/sp-dev-fx-webparts/tree/master/samples/react-bot-framework](https://github.com/SharePoint/sp-dev-fx-webparts/tree/master/samples/react-bot-framework)  repo to local system.
-   Run  _gulp bundle --ship_ command to build and bundle the artifacts.
-   Run  _gulp package-solution --ship_  command to create the sppkg package file for the web part.
-   Deploy the app package to the App Catalog and install the app in a SharePoint site.
-   Go to the Web App Bot created in  [https://portal.azure.com/](https://portal.azure.com/)  and configure  **Direct Line Channel**  that will be used in the SPFx web part.

![Configure Direct Line Channel](/public/images/QnAMaker-chatbot/DirectLineChannel1.png)

-   Show and copy the Secret Key

![Direct Line Channel Secret Key](/public/images/QnAMaker-chatbot/DirectLineChannel2.png)

-   Add the SPFx web part on any page in the SharePoint site and edit it. Paste the Secret Key copied in the previous step to the Direct Line Secret property and add other color properties if required.
-   The web part is ready to use now and should return queries to the questions similar to the ones added in FAQ list

![React Bot Framework Web Part](/public/images/QnAMaker-chatbot/ChatWebPart.png)

## Conclusion

Thus we saw how a SharePoint List data can be synced to a QnA Maker Knowledge Base using Power Automate and utilized in a chat bot that can be hosted in SharePoint as an SPFx web part.
