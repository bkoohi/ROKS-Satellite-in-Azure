### 1- Validation
1- Review IBM Cloud documentation for creating a Satellite location in Azure: https://cloud.ibm.com/docs/satellite?topic=satellite-azure

2- Prior to creating a statellite location in Azure, check Usage-Quotas in Azure portal or cli to make sure there are at least 12 vCPU allowed:

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

It may take up to 30 min to deploy 3 VMs for Satellite control plane and 3 VMs for ROKS cluster

12 - Resources created in Azure can be listed via Azure portal: https://portal.azure.com/#view/HubsExtension/BrowseAll

13- Once terraform execution completes, it generates the following completition message:
```
 2022/07/06 17:18:09 Terraform apply | module.satellite-host.ibm_satellite_host.assign_host[0]: Still creating... [19m0s elapsed]
 2022/07/06 17:18:15 Terraform apply | module.satellite-host.ibm_satellite_host.assign_host[0]: Creation complete after 19m6s [id=cb2rsd8w0emj53scdsb0/azure-eastus-1154-vm-0]
 2022/07/06 17:18:15 Terraform apply | 
 2022/07/06 17:18:15 Terraform apply | Apply complete! Resources: 39 added, 0 changed, 0 destroyed.
 2022/07/06 17:18:15 Terraform apply | 
 2022/07/06 17:18:15 Terraform apply | Outputs:
 2022/07/06 17:18:15 Terraform apply | 
 2022/07/06 17:18:15 Terraform apply | host_ids = [
 2022/07/06 17:18:15 Terraform apply |   "/subscriptions/xxxhiddenxxx/resourceGroups/azure-eastus-4228/providers/Microsoft.Compute/virtualMachines/azure-eastus-1154-vm-0",
 2022/07/06 17:18:15 Terraform apply |   "/subscriptions/xxxhiddenxxx/resourceGroups/azure-eastus-4228/providers/Microsoft.Compute/virtualMachines/azure-eastus-1154-vm-1",
 2022/07/06 17:18:15 Terraform apply |   "/subscriptions/xxxhiddenxxx/resourceGroups/azure-eastus-4228/providers/Microsoft.Compute/virtualMachines/azure-eastus-1154-vm-2",
 2022/07/06 17:18:15 Terraform apply |   "/subscriptions/xxxhiddenxxx/resourceGroups/azure-eastus-4228/providers/Microsoft.Compute/virtualMachines/azure-eastus-1154-vm-3",
 2022/07/06 17:18:15 Terraform apply |   "/subscriptions/xxxhiddenxxx/resourceGroups/azure-eastus-4228/providers/Microsoft.Compute/virtualMachines/azure-eastus-1154-vm-4",
 2022/07/06 17:18:15 Terraform apply |   "/subscriptions/xxxhiddenxxx/resourceGroups/azure-eastus-4228/providers/Microsoft.Compute/virtualMachines/azure-eastus-1154-vm-5",
 2022/07/06 17:18:15 Terraform apply | ]
 2022/07/06 17:18:15 Terraform apply | location_id = "cb2rsd8w0emj53scdsb0"
 2022/07/06 17:18:15 Command finished successfully.
 
 2022/07/06 17:18:15 [34mStarting command: terraform1.1 output -no-color -json[39m[0m
 2022/07/06 17:18:15 Starting command: terraform1.1 output -no-color -json
 2022/07/06 17:18:20 Command finished successfully.
 2022/07/06 17:18:27 [1m[32mDone with the workspace action[39m[0m
 ```

14- To view hosts status and mapping: https://cloud.ibm.com/satellite/locations/cb2rsd8w0emj53scdsb0/hosts

<img width="1518" alt="image" src="https://user-images.githubusercontent.com/6279125/177609311-16966aa2-fb2d-44a8-bb6a-6c643fc92f4f.png">


15 - Wait under Satellite location is Ready and in Normal status from the Satellite portal or from IBM Cloud cli:

```
behzadkoohi@Behzads-MBP login % ibmcloud sat location get --location cb2rsd8w0emj53scdsb0
Retrieving location...
OK
                                   
Name:                           azure-eastus   
ID:                             cb2rsd8w0emj53scdsb0   
Created:                        2022-07-06 12:54:13 -0400 (40 minutes ago)   
Managed From:                   wdc   
State:                          normal   
Ready for deployments:          yes   
Message:                        R0001: The Satellite location is ready for operations.   
Hosts Available:                3   
Hosts Total:                    6   
Host Zones:                     eastus-1, eastus-2, eastus-3   
Provider:                       azure   
Provider Region:                eastus   
Provider Credentials:           yes   
Public Service Endpoint URL:    https://c107.us-east.satellite.cloud.ibm.com:30344   
Private Service Endpoint URL:   -   
OpenVPN Server Port:            -   
Ignition Server Port:           -   
Konnectivity Server Port:       30189   
Logging Key Set:                no   
Activity Tracker Key Set:       no   
behzadkoohi@Behzads-MBP login % 

```

