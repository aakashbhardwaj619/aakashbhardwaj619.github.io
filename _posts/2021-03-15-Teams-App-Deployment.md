# Teams App Deployment to Azure Web App

Recently I was working on a Teams app development using [yo teams](https://github.com/pnp/generator-teams) generator for one of my clients. The documentation for getting started and creating a Teams tab from scratch was really helpful and allowed for a smooth development experience. 

But when it was time to deploy the application to an Azure Web App to check if everything was working as expected, outside of the local developer environment, I found the documentation a little bit lacking as most of the articles I found only mentioned `ngrok` for serving the app locally. With no prior experience on working with an Express based Node.Js application I found the deployment process a bit tricky. So once I was able to set it up successfully, I decided to put together the different pieces I learned related to the deployment process. This article describes how to deploy a Teams app to Windows Azure Web App using the **Local Git Deployment** option.

## Prerequisites

- A Node.Js based Teams app needs to be created using the [yo teams](https://github.com/pnp/generator-teams) generator.
- An Azure subscription is required to create the Azure Web App.

## Azure Web App

Create a new Azure Web App with the below options using the Azure portal:

![Azure Web App Options](/public/images/Teams-App-Deployment/CreateAzureWebApp.png)

- **Code** as the Publish option.
- **Windows** as the Operating System.
- **Node** as the Runtime Stack.

## Kudu Deployment Script

Using Git deploy to publish an app to Azure Web App enables automatic deployment once the changes are pushed. But sometimes we want to customize the deployment process to add some more steps to the deployment process. There is a custom deployment script generator for Kudu called [Kudu Script](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script) that generates a deployment script which can be updated as per the requirement.

For our app deployment we will use a custom deployment script using the below steps:

- Install kuduscript tool globally using the below command
```
npm install kuduscript -g
```

- Run the below command in the root of the Teams app repository to create the deployment script for a Node.js application
```
kuduscript -y --node
```
- This will generate two files 
  - `.deployment` - Contains the command to run for deploying your site.
  - `deploy.cmd` - Contains the deployment script (or deploy.sh if running on Mac/Linux)

- Update the `deploy.cmd` file with the below content that contains steps specific for a `yo teams` generated project
```cmd
@if "%SCM_TRACE_LEVEL%" NEQ "4" @echo off

:: ----------------------
:: KUDU Deployment Script
:: Version: 1.0.9
:: ----------------------

:: Prerequisites
:: -------------

:: Verify node.js installed
where node 2>nul >nul
IF %ERRORLEVEL% NEQ 0 (
  echo Missing node.js executable, please install node.js, if already installed make sure it can be reached from current environment.
  goto error
)

:: Setup
:: -----

setlocal enabledelayedexpansion

SET ARTIFACTS=%~dp0%..\artifacts

IF NOT DEFINED DEPLOYMENT_SOURCE (
  SET DEPLOYMENT_SOURCE=%~dp0%.
)

IF NOT DEFINED DEPLOYMENT_TARGET (
  SET DEPLOYMENT_TARGET=%ARTIFACTS%\wwwroot
)

IF NOT DEFINED NEXT_MANIFEST_PATH (
  SET NEXT_MANIFEST_PATH=%ARTIFACTS%\manifest

  IF NOT DEFINED PREVIOUS_MANIFEST_PATH (
    SET PREVIOUS_MANIFEST_PATH=%ARTIFACTS%\manifest
  )
)

IF NOT DEFINED KUDU_SYNC_CMD (
  :: Install kudu sync
  echo Installing Kudu Sync
  call npm install kudusync -g --silent
  IF !ERRORLEVEL! NEQ 0 goto error

  :: Locally just running "kuduSync" would also work
  SET KUDU_SYNC_CMD=%appdata%\npm\kuduSync.cmd
)
goto Deployment

:: Utility Functions
:: -----------------

:SelectNodeVersion

IF DEFINED KUDU_SELECT_NODE_VERSION_CMD (
  :: The following are done only on Windows Azure Websites environment
  call %KUDU_SELECT_NODE_VERSION_CMD% "%DEPLOYMENT_SOURCE%" "%DEPLOYMENT_TARGET%" "%DEPLOYMENT_TEMP%"
  IF !ERRORLEVEL! NEQ 0 goto error

  IF EXIST "%DEPLOYMENT_TEMP%\__nodeVersion.tmp" (
    SET /p NODE_EXE=<"%DEPLOYMENT_TEMP%\__nodeVersion.tmp"
    IF !ERRORLEVEL! NEQ 0 goto error
  )
  
  IF EXIST "%DEPLOYMENT_TEMP%\__npmVersion.tmp" (
    SET /p NPM_JS_PATH=<"%DEPLOYMENT_TEMP%\__npmVersion.tmp"
    IF !ERRORLEVEL! NEQ 0 goto error
  )

  IF NOT DEFINED NODE_EXE (
    SET NODE_EXE=node
  )

  SET NPM_CMD="!NODE_EXE!" "!NPM_JS_PATH!"
) ELSE (
  SET NPM_CMD=npm
  SET NODE_EXE=node
)

goto :EOF

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:: Deployment
:: ----------

:Deployment
echo Handling node.js deployment.

:: 1. Select node version for build
call :SelectNodeVersion
:: 2. Install npm packages
IF EXIST "%DEPLOYMENT_SOURCE%\package.json" (
  pushd "%DEPLOYMENT_SOURCE%"
  call :ExecuteCmd !NPM_CMD! install
  IF !ERRORLEVEL! NEQ 0 goto error
  popd
)
:: 3. Build it
IF EXIST "%DEPLOYMENT_SOURCE%\package.json" (
  pushd "%DEPLOYMENT_SOURCE%"
  call :ExecuteCmd !NPM_CMD! run-script build
  IF !ERRORLEVEL! NEQ 0 goto error
  popd
)
:: 4. KuduSync
IF /I "%IN_PLACE_DEPLOYMENT%" NEQ "1" (
  call :ExecuteCmd "%KUDU_SYNC_CMD%" -v 50 -f "%DEPLOYMENT_SOURCE%" -t "%DEPLOYMENT_TARGET%" -n "%NEXT_MANIFEST_PATH%" -p "%PREVIOUS_MANIFEST_PATH%" -i ".git;.hg;.deployment;deploy.cmd"
  IF !ERRORLEVEL! NEQ 0 goto error
)
:: 5. Select node version for run
call :SelectNodeVersion

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
goto end

:: Execute command routine that will echo out when error
:ExecuteCmd
setlocal
set _CMD_=%*
call %_CMD_%
if "%ERRORLEVEL%" NEQ "0" echo Failed exitCode=%ERRORLEVEL%, command=%_CMD_%
exit /b %ERRORLEVEL%

:error
endlocal
echo An error has occurred during web site deployment.
call :exitSetErrorLevel
call :exitFromFunction 2>nul

:exitSetErrorLevel
exit /b 1

:exitFromFunction
()

:end
endlocal
echo Finished successfully.

```

## Local Git Deployment

There are various deployment options present in Azure such as Azure DevOps Repos, GitHub, bitbucket, and Local git repository. If we just want to deploy the app for testing and don't want to create any pipelines yet, we can use the [Local Git Deployment](https://docs.microsoft.com/en-us/azure/app-service/deploy-local-git?tabs=cli) option.

The below steps can be followed for enabling local git deployment on our Azure Web App:

- Go to the **Deployment Center** settings tab for the Azure Web App and save **Local Git** as the deployment option. This would generate a Local Git Uri. Copy this as this would be needed later.
![Local Git Deployment Settings](/public/images/Teams-App-Deployment/LocalGitDeployment.png)
- Go to the project's root directory and initialize an empty Git repository using the below command.
```
git init
```
- Add an `azure` Git remote pointing to the Local Git repository URL copied in one of the previous steps using the below command.
```
git remote add azure LOCAL_GIT_REPO_URL
```
- Add and commit the changes.
```
git add .
git commit -m "Initial commit"
```
- Push the changes to the web app using the below command.
```
git push azure master
```
- The first time changes are pushed some credentials would be required. To get the credentials go to the **Local Git** tab of the web app's **Deployment Center** settings and use the Application Scope Username and Password.
![Git Deployment Credentials](/public/images/Teams-App-Deployment/GitDeploymentCredentials.png)
- Once the deployment is completed, go to the app URL and check if our Teams app is working or not.
![Deployed App](/public/images/Teams-App-Deployment/DeployedApp.png)

> There is also an [Azure App Service extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice) for VS Code, currently in preview, that can be used to carry local git deployment of our app. Using it the credentials don't have to be entered manually while pushing the code changes if we are signed in to our Azure subscription in VS Code. Deployment steps using the extension are mentioned at [this link](https://docs.microsoft.com/en-in/azure/app-service/quickstart-nodejs?pivots=platform-windows#deploy-the-app-to-azure).

## Troubleshooting

If we see that the app is not working as expected, we can go to the **Advanced Tools** section of the Web App and make the below checks in the Kudu explorer.

- Check that all the build generated files are present at the path `site/wwwroot/dist`.

- Check that a `web.config` file is present at `site/wwwroot`. This file that tells the IIS server to use `issnode` module for hosting the application and also specifies `dist/server.js` as the entry point to the application. It is automatically generated during git deployment process and has a content similar to the below file.

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
     This configuration file is required if iisnode is used to run node processes behind
     IIS or IIS Express.  For more information, visit:

     https://github.com/tjanczuk/iisnode/blob/master/src/samples/configuration/web.config
-->

<configuration>
  <system.webServer>
    <!-- Visit http://blogs.msdn.com/b/windowsazure/archive/2013/11/14/introduction-to-websockets-on-windows-azure-web-sites.aspx for more information on WebSocket support -->
    <webSocket enabled="false" />
    <handlers>
      <!-- Indicates that the server.js file is a node.js site to be handled by the iisnode module -->
      <add name="iisnode" path="dist/server.js" verb="*" modules="iisnode"/>
    </handlers>
    <rewrite>
      <rules>
        <!-- Do not interfere with requests for node-inspector debugging -->
        <rule name="NodeInspector" patternSyntax="ECMAScript" stopProcessing="true">
          <match url="^dist/server.js\/debug[\/]?" />
        </rule>

        <!-- First we consider whether the incoming URL matches a physical file in the /public folder -->
        <rule name="StaticContent">
          <action type="Rewrite" url="public{REQUEST_URI}"/>
        </rule>

        <!-- All other URLs are mapped to the node.js site entry point -->
        <rule name="DynamicContent">
          <conditions>
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="True"/>
          </conditions>
          <action type="Rewrite" url="dist/server.js"/>
        </rule>
      </rules>
    </rewrite>
    
    <!-- 'bin' directory has no special meaning in node.js and apps can be placed in it -->
    <security>
      <requestFiltering>
        <hiddenSegments>
          <remove segment="bin"/>
        </hiddenSegments>
      </requestFiltering>
    </security>

    <!-- Make sure error responses are left untouched -->
    <httpErrors existingResponse="PassThrough" />

    <!--
      You can control how Node is hosted within IIS using the following options:
        * watchedFiles: semi-colon separated list of files that will be watched for changes to restart the server
        * node_env: will be propagated to node as NODE_ENV environment variable
        * debuggingEnabled - controls whether the built-in debugger is enabled

      See https://github.com/tjanczuk/iisnode/blob/master/src/samples/configuration/web.config for a full list of options
    -->
    <!--<iisnode watchedFiles="web.config;*.js"/>-->
  </system.webServer>
</configuration>

```

## Conclusion

In this way a Node.Js based Teams app can be deployed to an Azure Web App using Local Git Deployment. For production scenarios Azure pipelines or GitHub Actions can be set up for automatic deployment and [this article](https://www.wictorwilen.se/blog/deploying-yo-teams-and-node-apps/) by Wictor Wilen can be used for such cases.