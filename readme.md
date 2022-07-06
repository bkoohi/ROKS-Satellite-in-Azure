### 1- Validation

1- Prior to creating a statellite location in Azure, check Usage-Quotas in Azure portal or cli to make sure there are at least 12 vCPU allowed:

<img width="1237" alt="image" src="https://user-images.githubusercontent.com/6279125/176729940-8b3d170f-6792-4c3c-9062-39f865efd826.png">

or from cli:
```
az login
```
```
az vm list-usage --location "East US" --output table | grep DASv4
```
```
% az vm list-usage --location "East US" --output table | grep DASv4
Standard DASv4 Family vCPUs               0               40
Standard NDASv4_A100 Family vCPUs         0               0
% 
```

```
 % az vm list-usage --location "East US" --output table | grep Regional 
Total Regional vCPUs                      0               81
Total Regional Low-priority vCPUs         0               3
% 

```
By default the limit is 10 vCPU per region and it needs to be modified from portal to at least 12 or higher.


### 2- Deployement 
1- Install Azure AZ Command line in your local host or laptop following in Mac OS:
```
brew update && brew install azure-cli
```

For other OSes, follow these instructions: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-macos?view=azure-cli-latest

2- Login in Azure using:
```
az login
```
It will bring up Azure Login in a browser and prompts for login

3- To test your ID and account:
```
az account list
```

4- Find your subscription ID:
```
az account show
```
Look for 
"id: xxxxxx"

5- Setup the subscription 
```
az account set --subscription=xxxxx
```
6- Create a principal service 
```
az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/xxxxx" -n"satelline-service-principal"
```

7- Go to IBM Cloud Satellite offering using Azure Quick Start
https://cloud.ibm.com/satellite/locations/create/azure

Enter AppID, Tenant ID and passwd generated in previous step and select Verify

Once verification has been comepleted, you should be able to get a summary of account such as this:

<img width="1554" alt="image" src="https://user-images.githubusercontent.com/6279125/176719384-4d14034d-4eb3-4468-9133-75d40a13d4ab.png">


8- Once account has been validated and such cost estimate generated as shown in above then click on Create Location

9- Check the cluster creation status and progress:
<img width="1566" alt="image" src="https://user-images.githubusercontent.com/6279125/176720140-8f1b3711-09d1-42a3-8cff-9aed4bc45ad5.png">


10 - To review the detail, select Overview and then under "Schematics workspace status", select View Logs: 

<img width="1191" alt="image" src="https://user-images.githubusercontent.com/6279125/177604136-20917498-9f63-46ff-9dd0-2cf51d5562d0.png">

11- Under schematics workspace , expand the executing job to find the detail: [https://cloud.ibm.com/schematics/workspaces]

It may take a 5-10 min to deploy 3 VMs for Satellite control plane and 3 VMs for ROKS cluster

12 - Resources created in Azure can be listed via Azure portal: https://portal.azure.com/#view/HubsExtension/BrowseAll

13- 





