---
layout: post
title: "Deploy SPFx solutions on multiple sites using GitHub Actions"
date: 2020-06-09
---

Introduction
------------

Recently I came across a requirement to create a GitHub Action Workflow for the build and deployment of an SPFx solution to a large number of sites that were using their own site collection scoped app catalogs. The [Office 365 CLI Deploy App](https://github.com/marketplace/actions/office-365-cli-deploy-app) action provides the option to deploy to either the tenant app catalog or a site collection app catalog. Deploying our app package to all those sites using this action seems cumbersome as we would need to repeat this step with different `SITE_COLLECTION_URL` values.

Another approach for this scenario would be to write a separate script that would handle the deploymeny steps and execute this script from the pipeline using the [Office 365 CLI Run Script](https://github.com/marketplace/actions/office-365-cli-run-script) action.

YAML Configuration
-------------------

The GitHub Actions Workflow YAML file for building and deploying the SPFx solution looks like the one below. Here we have added the steps for configuring our environment, running automated tests, bundling our `sppkg` package file and executing the bash deployment script.

> For getting started using GitHub Actions for SPFx please refer to the article [Create GitHub actions for SPFx solution](https://medium.com/@anoopt/create-github-actions-for-spfx-solution-cc4a810b87db) from [Anoop](https://twitter.com/anooptells).

{% raw %}
```
name: SPFx CI CD

on:
  push:
    branches:
      - dev

env:
  packagePath: sharepoint/solution/test-list-items.sppkg

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:

    - name: Checkout code
      uses: actions/checkout@v2

    - name: Setup Node.js environment
      uses: actions/setup-node@v1.4.2
      with:
        node-version: 10.x

    - name: Install dependencies
      run: npm ci
    
    - name: Build solution
      run: gulp build
      
    - name: Test solution
      run: npm test
              
    - name: Bundle and package
      run: |
        gulp bundle --ship
        gulp package-solution --ship
    - name: Upload Build Package
      uses: actions/upload-artifact@v2
      with:
        path: ${{ env.packagePath }}
              
    - name: Office 365 CLI Login
      uses: pnp/action-cli-login@v1.0.0
      with:
        ADMIN_USERNAME: ${{ secrets.adminUsername }}
        ADMIN_PASSWORD: ${{ secrets.adminPassword }}
        
      # Run bash script at the specified path in the repository
    - name: Office 365 CLI Runscript bash
      uses: pnp/action-cli-runscript@v1.0.0
      with:
        O365_CLI_SCRIPT_PATH: './scripts/DeployPackage.sh'

```
{% endraw %}

Bash Deployment Script
----------------------

The below bash script traverses each of the site collections added in the `sites` array and deploys the app package to its site collection app catalog using the [o365 spo app add](https://pnp.github.io/office365-cli/cmd/spo/app/app-add/) command. It also checks the app info to see if the app needs to be installed or upgraded using the [o365 spo app upgrade](https://pnp.github.io/office365-cli/cmd/spo/app/app-upgrade/) and [o365 apo app install](https://pnp.github.io/office365-cli/cmd/spo/app/app-install/) commands. Lastly it sends a completion email, using the [o365 spo mail send](https://pnp.github.io/office365-cli/cmd/spo/mail/mail-send/) command, once the deployment is completed.

> Similar script can also be written using PowerShell when using windows runner

```
#!/bin/bash

# requires jq: https://stedolan.github.io/jq/

packagePath=$packagePath

sites=("https://contoso.sharepoint.com/sites/Site1" "https://contoso.sharepoint.com/sites/Site2" "https://contoso.sharepoint.com/sites/Site3")

echo "Starting Deployment..."

for siteUrl in "${sites[@]}"; do
  echo "Site URL: $siteUrl"
  app=$(o365 spo app add --filePath $packagePath --scope sitecollection --appCatalogUrl $siteUrl --overwrite)
  o365 spo app deploy --id $app --scope sitecollection --appCatalogUrl $siteUrl
  echo "Deployed App..."
  appInfo=$(o365 spo app get --id $app --scope sitecollection --appCatalogUrl $siteUrl --output json)
  appVersion=$(echo $appInfo | jq -r '.InstalledVersion')
  appCanUpgrade=$(echo $appInfo | jq -r '.CanUpgrade')

  if [[ "$appCanUpgrade" = "true" ]]; then
    o365 spo app upgrade --id $app --siteUrl $siteUrl --scope sitecollection
    echo "Upgraded App..."
  fi
  if [ -z "$appVersion" ]; then
    o365 spo app install --id $app --siteUrl $siteUrl --scope sitecollection
    echo "Installed app..."
  fi
done

o365 spo mail send --webUrl ${sites[0]} --to 'aakash.bhardwaj@in8aakbh.onmicrosoft.com' --subject "Multi Site Deployment Completed" --body "<h2>Deployment Completed</h2> <p>Multi Site Deployment of $packagePath has been completed.</p>"
```

> Notice that the variable `packagePath` gets its value from the environment variable set in the YAML file

Conclusion
----------

Thus we saw how we can deploy an SPFx app package to multiple site collections from a custom bash script using Office 365 CLI commands.
