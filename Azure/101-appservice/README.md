# Deployment of a Web Api  hosted on Azure App Service with Azure AD authentication

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fflecoqui%2FTestNodeJSWebApiAzureAD%2Fmaster%2FAzure%2F101-appservice%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>
<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Fflecoqui%2FTestNodeJSWebApiAzureAD%2Fmaster%2FAzure%2F101-appservice%2Fazuredeploy.json" target="_blank">
    <img src="http://armviz.io/visualizebutton.png"/>
</a>

This template allows you to deploy from Github a REST API  hosted on Azure App Service. Moreover, the REST API service will be directly deployed from github towards Azure App Service.

The REST API (api/values) is actually an JSON echo service, if you send a Json string in the http content, you will receive the same Json string in the http response.
Below a curl command line to send the request:


          curl -d '{"name":"0123456789"}' -H "Content-Type: application/json"  -X POST   https://<hostname>/api/values
          curl -H "Content-Type: application/json"  -X GET   https://<hostname>/api/values?name=0123456789



# DEPLOYING THE REST API ON AZURE SERVICES

## PRE-REQUISITES
First you need an Azure subscription.
You can subscribe here:  https://azure.microsoft.com/en-us/free/ . </p>
Moreover, we will use Azure CLI v2.0 to deploy the resources in Azure.
You can install Azure CLI on your machine running Linux, MacOS or Windows from here: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest 



## CREATE RESOURCE GROUP:
First you need to create the resource group which will be associated with this deployment. For this step, you can use Azure CLI v1 or v2.

* **Azure CLI 1.0:** azure group create "ResourceGroupName" "RegionName"

* **Azure CLI 2.0:** az group create an "ResourceGroupName" -l "RegionName"

For instance:

    azure group create TestNodeJSWebApiAzureADrg eastus2

    az group create -n TestNodeJSWebApiAzureADrg -l eastus2

## DEPLOY THE SERVICES:

### DEPLOY REST API ON AZURE APP SERVICE:
You can deploy Azure Function, Azure App Service and Virtual Machine using ARM (Azure Resource Manager) Template and Azure CLI v1 or v2

* **Azure CLI 1.0:** azure group deployment create "ResourceGroupName" "DeploymentName"  -f azuredeploy.json -e azuredeploy.parameters.json*

* **Azure CLI 2.0:** az group deployment create -g "ResourceGroupName" -n "DeploymentName" --template-file "templatefile.json" --parameters @"templatefile.parameter..json"  --verbose -o json

For instance:

    azure deployment group create TestNodeJSWebApiAzureADrg TestNodeJSWebApiAzureADdep -f azuredeploy.json -e azuredeploy.parameters.json -vv

    az deployment group create -g TestNodeJSWebApiAzureADrg -n TestNodeJSWebApiAzureADdep --template-file azuredeploy.json --parameter @azuredeploy.parameters.json --verbose -o json


When you deploy the service you can define the following parameters:</p>
* **namePrefix:** The name prefix which will be used for all the services deployed with this ARM Template</p>
* **WebAppSku:** The WebApp Sku Capacity, by defualt F1</p>
* **repoURL:** The github repository url</p>
* **branch:** The branch name in the repository</p>
* **configClientID:** The Client ID used for the Azure AD Authentication</p>
* **configClientSecret:** The Client Secret used for the Azure AD Authentication</p>
* **configTenantName:** The Tenant Name used for the Azure AD Authentication</p>
* **configRedirectUrl:** TheRedirect Url used for the Azure AD Authentication</p>
* **configSignOutUrl:** The SignOut Url used for the Azure AD Authentication</p>

All the parameters required for the Azure AD authentication can be automatically created using the following script or bash files:
* **Windows Powershell:** install-webapp-azuread-windows.ps1 </p>
* **Linux Bash:** install-webapp-azuread.sh </p>

Those files are called using the following parameters:
* **Azure resource group :** The Azure Resource group where the App Service will be deployed</p>
* **namePrefix:** The name prefix which will be used for all the services deployed with this ARM Template</p>
* **Tenant Name:** The Tenant Name for the authentication TenantNagitme.onmicrosoft.com </p>
* **Azure Subscription ID for Azure AD:** The Azure Subscription ID associated with the Azure AD used for the authentication </p>
* **Azure Subscription ID for Azure App Service:** The Azure Subscription ID associated with the App Serivce  </p>
* **WebAppSku:** The WebApp Sku Capacity, by default F1</p>

For instance:

.\install-webapp-azuread-windows.ps1 TestNodeJSWebAppAzureADrg testnodewebapp M365x175592 faa1b9e5-22ff-4238-8fb5-5a4d73c49d47 e5c9fc83-fbd0-4368-9cb6-1b5823479b6d S1

./install-webapp-azuread-windows.sh TestNodeJSWebAppAzureADrg testnodewebapp M365x175592 faa1b9e5-22ff-4238-8fb5-5a4d73c49d47 e5c9fc83-fbd0-4368-9cb6-1b5823479b6d S1


Those scripts required two authentication phases one with the Azure  Subscription ID associated with the Azure AD and one with the Azure Subscription ID associated with the Web API Service. 



# TEST THE SERVICES:

## TEST THE SERVICES WITH CURL
Once the services are deployed, you can test the REST API using Curl. You can download curl from here https://curl.haxx.se/download.html 
For instance :

     curl -d '{"name":"0123456789"}' -H "Content-Type: application/json"  -X POST   https://<namePrefix>web.azurewebsites.net/api/values
     curl -H "Content-Type: application/json"  -X POST   https://<namePrefix>web.azurewebsites.net/api/values?name=012345678

</p>

## TEST THE SERVICES WITH VEGETA
You can also test the scalability of the REST API using Vegeta. 
You can deploy a Virtual Machine running Vageta using the ARM Template here: https://github.com/flecoqui/101-vm-simple-vegeta-universal 
While deploying Vegeta, you can select the type of Virtual Machine: Windows, Debian, Ubuntu, RedHat, Centos.

Vegeta will be pre-installed on those virtual machines.

Once connected with the Vegate Virtual Machine, open the Command Shell and launch the following command for instance :</p>


         vegeta attack -duration=10s -rate 1000 -targets=targets.txt | vegeta report 



where the file targets.txt contains the following lines: </p>


          POST http://<namePrefix>web.azurewebsites.net/api/values
          Content-Type: application/json
          @data.json



where the file data.json contains the following lines: </p>


         '{"name":"0123456789"}'


# DELETE THE REST API SERVICES 

## DELETE THE RESOURCE GROUP:

* **Azure CLI 1.0:**      azure group delete "ResourceGroupName" "RegionName"

* **Azure CLI 2.0:**  az group delete -n "ResourceGroupName" "RegionName"

For instance:

    azure group delete TestNodeJSWebApiAzureADrg eastus2

    az group delete -n TestNodeJSWebApiAzureADrg 

# Next Steps

1. Automate the Vegeta Tests  
