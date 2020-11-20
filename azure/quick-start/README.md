Quick Start Guide
---


Prerequisites
---
 1. Azure CLI.    
 [Link](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
 2. Terraform 0.12.X or higher  
    [Download](https://releases.hashicorp.com/terraform/)  
    [Install Guide](https://learn.hashicorp.com/tutorials/terraform/install-cli?in=terraform/azure-get-started)
3. SSH Public key to install on nodes  

Authenticating to Azure CLI using username/password  
---
[Terraform Azure Provider](https://www.terraform.io/docs/providers/azurerm/guides/service_principal_client_secret.html)

```az login```  

``` 
You have logged in. Now let us find all the subscriptions to which you have access...  
  {
    "cloudName": "AzureCloud",  
    "id": "xxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",  
    "isDefault": true,  
    "name": "PlayGround",  
    "state": "Enabled",  
    "tenantId": "xxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",  
    "user": {  
      "name": "xxxxxxxx@nearform.com",  
      "type": "user"  
    }  
  }  
  ```  
**NOTE:**  "id:" = ```your subscription ID```  

Setting to a subscription if you have more than one  
```az account set --subscription="SUBSCRIPTION_ID"```  

Creating a service principle for connecting to Azure (optional)
---

```az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/SUBSCRIPTION_ID"```  

  ```
  Creating a role assignment under the scope of "/subscriptions/xxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  Retrying role assignment creation: 1/36
{
  "appId": "xxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "displayName": "azure-cli-2020-10-12-20-55-35",
  "name": "http://azure-cli-2020-10-12-20-55-35",
  "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```
Logging in using Service Principle 
---
```az login --service-principal -u CLIENT_ID -p CLIENT_SECRET --tenant TENANT_ID```

Optionally you can set env vars to configure your az cli client  
```
export ARM_CLIENT_ID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx"  
export ARM_CLIENT_SECRET="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx"   
export ARM_SUBSCRIPTION_ID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx"  
export ARM_TENANT_ID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx"   
```
Executing the Terraform Plan
---
You will need to provide some variables unique to your environment in the ``` main.tf ``` file:  
```
  source = "git::https://github.com/nearform/taurus-azure.git?ref=dev"  
  sub_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx"  
  location = "West US" # What region to deploy all your resources  
  environment = "dev" # used for labeling/tagging
  namespace = "testing" # used for labeling/tagging 
  aks_dns_prefix = "dev"  # must contain between 3 and 45 characters, and can contain only letters, numbers, and hyphens.   
  cluster_name = "AKSCluster"  # prefix to identify resource. no dashes allowed   
  acr_name_prefix = "acr"  # 5 or more alphanumeric characters and globally unique  
  storagename_prefix = "storage"  # prefix used to name a misc resource group.  Will hold DB and KeyVault
  db_name_prefix = "postgres"  # prefix used to identify the database
```
Connecting to AKS Cluster
---

Using the commands below is an easy way to download/install/configure kubectl.  The command uses the default location for the Kubernetes configuration file, which is ~/.kube/config. You can specify a different location for your Kubernetes configuration file using --file.
```
az aks install-cli  
az aks get-credentials --resource-group "myResourceGroup" --name "myAKSCluster"   
```

GitHub Actions (GHA) CI/CD
---

There are two provided workflows to deploy to AKS using the helm charts, Frontend and Backend.  GHA will build the docker images, push the images to ACR with a tag of the commit SHA.  GHA will also deploy to AKS using helm charts overrriding the image tag with the correct commit SHA.  

Terraform is building a service principle user to use in GHA.  It will create an Enterprise App with a appID.  This appID is the username and it is assigned a password.  Both are available using the terraform output command.  

GHA actions will only run on the specified branch and if there are chanages made to ```  'app/backend/*' ```  or  ``` 'app/frontend/*'.  ```    

GHA secrets will need to be provided to the Github Repository.  These variables are available after the terraforming had completed.  
Use ``` terraform output ``` to retrieve the values for the following secrets.

ACR_PASSWORD  
ACR_USERNAME  
ACR_RESOURCE_GROUP  
ACR_SERVER  
AZURE_CREDS  
K8_CLUSTER_NAME  

** NOTE: For AZURE_CREDS use the following template and fill out the appropriate information
``` 
{
  "clientId": "<client id>",
  "clientSecret": "<client secret>",
  "subscriptionId": "<subscription id>",
  "tenantId": "<tenant id>",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```
Change helm charts
---

You will have to change ```app/frontend/values.yaml``` and ```app/backend/values.yaml```  files with the new ACR FQDN so GHA will override with the correct path.
