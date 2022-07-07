IBM Cloud documentation for creating a Satellite location in Azure provides overall design and deployment steps: https://cloud.ibm.com/docs/satellite?topic=satellite-azure. It's missing some of prerequisites steps and some of post deployment steps for accessing ROKS portal that are being documented in this readme.

### 1- Validation

1- Prior to creating a satellite location in Azure, check Usage-Quotas for your userid. This quota can be displayed and modified in Azure portal or via cli. By default, there is a 10 vCPU limit per region per user limit. We need to increase this limit to at least 24 vCPU or higher if you are deploying multiple ROKS clusters in Azure. Satellite deployment will fail when VM provisioning hits the quotas limit in Azure:

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
By default the limit is 10 vCPU per region and it needs to be modified from portal to at least 24 or higher.


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
16 - Create a ROKS cluster from IBM Cloud portal in Satellite location in Azure:
https://cloud.ibm.com/kubernetes/catalog/create?platformType=openshift
<img width="1505" alt="image" src="https://user-images.githubusercontent.com/6279125/177611449-d2772954-8001-487f-b24f-f8c3e70e0811.png">

<img width="1507" alt="image" src="https://user-images.githubusercontent.com/6279125/177611546-197c2639-807d-4875-ad69-cded1f5d9c6d.png">

<img width="1489" alt="image" src="https://user-images.githubusercontent.com/6279125/177611676-6faff171-1d82-4574-91dd-f144db43889f.png">


17 - It may take up to 30 min to create the cluster:

<img width="1215" alt="image" src="https://user-images.githubusercontent.com/6279125/177613835-a2be7c0f-fc0a-4dfd-8e6e-5d2e9790b093.png">

During this cluster provisioning, Hosts in Satellite location will undergo a Provisioning process:
<img width="1516" alt="image" src="https://user-images.githubusercontent.com/6279125/177613694-71661278-5d1e-4120-8f44-6d88d22525a9.png">

It may take up to 30 min for cluster proisioning to complete:
<img width="1483" alt="image" src="https://user-images.githubusercontent.com/6279125/177617199-c5e757dc-4dd9-452e-a3a1-a998254c87c0.png">

### 3- Post Deployment
By now, we have Satellite control plane deployed and at least one ROKS cluster in Aure ready for application or CloudPak deployment. However, by default private IPs ( 10.x.x.x ) are used for Control plane and worker nodes deployment. Private network are not accessable from outside of Azure network and therefore cluster web console is not accessable from IBM Cloud. There are two options to make it accessable:
1- Establish a VPN link from IBM Cloud to Azure using wineguard VPN tool: https://jakew.me/wireguard-docker/
2- Update VMs in Azure for control plane and worker nodes and swap private IPs with public IPs.

#### 3.1- Wireguard

#### 3.2- Swapping private IPs with public IPs
1 - VMs in Azure VPC by default will be provisioned using private IPs. In order to access the portal ROKS Web Console, there is a need to assign public IP and NIC to each Controller VMs and Worker node VMs.

2- Identify az_resource_group and az_resource_prefix within the Schematics workspace:
![image](https://user-images.githubusercontent.com/6279125/177636646-875509fa-7bbc-458c-8490-bee0156c7bac.png)

set up two variables for Resource Group name and Prefix:
```
% VM_PREFIX=azure-eastus-1154                                             
% SAT_RG=azure-eastus-4228
```
3- Reconfigure VMs with public nic:
```
for i in {0..5}; do az network public-ip create --resource-group $SAT_RG --name $VM_PREFIX-vm-0-
public --version IPv4 --sku Standard --zone 1 2 3; done
```

Display VMs public nic IPs:
```
for i in {0..5}; do az network public-ip show -g $SAT_RG --name $VM_PREFIX-vm-$i-public | grep ipAddress; done
```
```
Output:
  "ipAddress": "20.231.234.79",  <===== Satellite Control plane
  "ipAddress": "20.231.232.154", <===== Satellite Control plane
  "ipAddress": "20.231.234.229", <===== Satellite Control plane
  "ipAddress": "20.231.234.241", <===== Cluster / Worker nodes
  "ipAddress": "20.231.235.85",  <===== Cluster / Worker nodes
  "ipAddress": "20.231.234.247", <===== Cluster / Worker nodes
```

Update VM IPs
```
for i in {0..5}; do az network nic ip-config update --name $VM_PREFIX-nic-internal --nic-name $VM_PREFIX-nic-$i --resource-group $SAT_RG --public-ip-address $VM_PREFIX-vm-$i-public | grep ipAddress; done
```
```
Output:
    "ipAddress": null,
    "ipAddress": null,
    "ipAddress": null,
    "ipAddress": null,
    "ipAddress": null,
    "ipAddress": null,
```
4- Update ROKS cluster configuration in IBM Cloud with new public IPs.

4.1 Set Resource group to Default unless a custom Resource group was used for deployment:
```
ibmcloud target -g Default
```
4.2 Identify Satellite location name used for ROKS deployment:
```
ibmcloud sat location ls
```
```
Output:
% ibmcloud sat location ls
Retrieving locations...
OK
Name           ID                     Status   Ready   Created       Hosts (used/total)   Managed From   
azure-eastus   cb2rsd8w0emj53scdsb0   normal   yes     4 hours ago   6 / 6                wdc  
```

4.3 Update DNS record for Satellite location with public IPs of Controller plane VMS:

```
ibmcloud sat location dns register --location azure-eastus --ip 20.231.234.79 --ip 20.231.232.154 --ip 20.231.234.229
```
```
Output:
% ibmcloud sat location dns register --location azure-eastus --ip 20.231.234.79 --ip 20.231.232.154 --ip 20.231.234.229
Registering a subdomain for control plane hosts...
OK
Subdomain                                                                                       Records   
a429a27d8f60474dd517d-6b64a6ccc9c596bf59a86625d8fa2202-ce00.us-east.satellite.appdomain.cloud   a429a27d8f60474dd517d-6b64a6ccc9c596bf59a86625d8fa2202-c000.us-east.satellite.appdomain.cloud   
a429a27d8f60474dd517d-6b64a6ccc9c596bf59a86625d8fa2202-c000.us-east.satellite.appdomain.cloud   20.231.234.229, 20.231.232.154, 20.231.234.79   
a429a27d8f60474dd517d-6b64a6ccc9c596bf59a86625d8fa2202-c001.us-east.satellite.appdomain.cloud   20.231.232.154   
a429a27d8f60474dd517d-6b64a6ccc9c596bf59a86625d8fa2202-c002.us-east.satellite.appdomain.cloud   20.231.234.79   
a429a27d8f60474dd517d-6b64a6ccc9c596bf59a86625d8fa2202-c003.us-east.satellite.appdomain.cloud   20.231.234.229 
```

4.4 Updaate Ingress subdomain
Find ROKS cluster deployed in the env:
```
ibmcloud ks cluster ls
```

```
Output:
 % ibmcloud ks cluster ls
OK
Name                  ID                     State    Created       Workers   Location       Version                 Resource Group Name   Provider   
mycluster-satellite   cb2skpaw0djunrr0l08g   normal   3 hours ago   3         azure-eastus   4.9.37_1544_openshift   default               satellite   
```

Identify Ingress subdomain for your ROKS cluster:
```
ibmcloud ks cluster get --cluster mycluster-satellite | grep "Ingress Subdomain"
```

```
Output
% ibmcloud ks cluster get --cluster mycluster-satellite | grep "Ingress Subdomain"
Ingress Subdomain:              mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud   
behzadkoohi@Behzads-MBP kubernetesnodeapp % 
```

Add 3 public IPs for worker nodes to ingress subdomain:
```
 %ibmcloud oc nlb-dns add --ip 20.231.234.241 --cluster mycluster-satellite --nlb-host mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud   
Adding IP(s) 20.231.234.241 to NLB host name mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud in cluster mycluster-satellite ...
Note: It might take a few minutes for your changes to be applied.
Ok
%

 %ibmcloud oc nlb-dns add --ip 20.231.235.85 --cluster mycluster-satellite --nlb-host mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud  
Adding IP(s) 20.231.235.85 to NLB host name mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud in cluster mycluster-satellite ...
Note: It might take a few minutes for your changes to be applied.
OK
%

 % ibmcloud oc nlb-dns add --ip 20.231.234.247 --cluster mycluster-satellite --nlb-host mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud  
Adding IP(s) 20.231.234.247 to NLB host name mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud in cluster mycluster-satellite ...
Note: It might take a few minutes for your changes to be applied.
OK
%

Validate ROKS cluster NLB IPs:
```
ibmcloud oc nlb-dns ls --cluster mycluster-satellite
OK
Hostname                                                                                       IP(s)                                                                    Health Monitor   SSL Cert Status   SSL Cert Secret Name                                        Secret Namespace    Status   
mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud   10.0.1.5,10.0.2.5,10.0.3.4,20.231.234.241,20.231.234.247,20.231.235.85   disabled         created           mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000   openshift-ingress   OK   
```

Remove Private IPs from NLB:
```
 ibmcloud oc nlb-dns rm classic --ip  10.0.1.5 --cluster mycluster-satellite --nlb-host mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud
Deleting IP 10.0.1.5 from NLB host name mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud in cluster mycluster-satellite...
Note: It might take a few minutes for your changes to be applied.
OK
 % ibmcloud oc nlb-dns rm classic --ip  10.0.2.5 --cluster mycluster-satellite --nlb-host mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud
Deleting IP 10.0.2.5 from NLB host name mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud in cluster mycluster-satellite...
Note: It might take a few minutes for your changes to be applied.
OK
% ibmcloud oc nlb-dns rm classic --ip  10.0.3.4 --cluster mycluster-satellite --nlb-host mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud
Deleting IP 10.0.3.4 from NLB host name mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud in cluster mycluster-satellite...
Note: It might take a few minutes for your changes to be applied.
OK
 % 
```

Validate NLB public IPs

```
ibmcloud oc nlb-dns ls --cluster mycluster-satellite 
OK
Hostname                                                                                       IP(s)                                         Health Monitor   SSL Cert Status   SSL Cert Secret Name                                        Secret Namespace    Status   
mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud   20.231.234.241,20.231.234.247,20.231.235.85   disabled         created           mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000   openshift-ingress   OK   
 % 
```


5 - Open up OpenShift Web console from Openshift portal in IBM Cloud:
https://cloud.ibm.com/kubernetes/clusters?platformType=openshift

<img width="1517" alt="image" src="https://user-images.githubusercontent.com/6279125/177617416-139ce9c8-60e0-4f86-8b4c-ca26af085daa.png">


6 - access command line
```
% oc version
Client Version: 4.10.6
Kubernetes Version: v1.22.8+f34b40c
%
```

### 4- Demo application deployment
1 - Deploy sample app

```
oc new-app rails-postgresql-example
--> Deploying template "openshift/rails-postgresql-example" to project my-example

     Rails + PostgreSQL (Ephemeral)
     ---------
     An example Rails application with a PostgreSQL database. For more information about using this template, including OpenShift considerations, see https://github.com/sclorg/rails-ex/blob/master/README.md.
     
     WARNING: Any data stored will be lost upon pod destruction. Only use this template for testing.

     The following service(s) have been created in your project: rails-postgresql-example, postgresql.
     
     For more information about using this template, including OpenShift considerations, see https://github.com/sclorg/rails-ex/blob/master/README.md.

     * With parameters:
        * Name=rails-postgresql-example
        * Namespace=openshift
        * Memory Limit=512Mi
        * Memory Limit (PostgreSQL)=512Mi
        * Git Repository URL=https://github.com/sclorg/rails-ex.git
        * Git Reference=
        * Context Directory=
        * Application Hostname=
        * GitHub Webhook Secret=Mjby5RdEkV2nSLRvdxCfY2ibmSI2orMYLth6BE4r # generated
        * Secret Key=2yhn0q5o0bf7txkpljlfal4kvy0hhee81mrhm1t6x8gk72fdea6wernvbqcd74fcmn0vslui2di28517tl7vsul16r8ojptshy5ei1rvpiwefehgnayexrl4beq8nul # generated
        * Application Username=openshift
        * Application Password=secret
        * Rails Environment=production
        * Database Service Name=postgresql
        * Database Username=userVNU # generated
        * Database Password=5hHtPkxO # generated
        * Database Name=root
        * Maximum Database Connections=100
        * Shared Buffer Amount=12MB
        * Custom RubyGems Mirror URL=

--> Creating resources ...
    secret "rails-postgresql-example" created
    service "rails-postgresql-example" created
    route.route.openshift.io "rails-postgresql-example" created
    imagestream.image.openshift.io "rails-postgresql-example" created
    buildconfig.build.openshift.io "rails-postgresql-example" created
    deploymentconfig.apps.openshift.io "rails-postgresql-example" created
    service "postgresql" created
    deploymentconfig.apps.openshift.io "postgresql" created
--> Success
    Access your application via route 'rails-postgresql-example-my-example.mycluster-satellite-0db9129938ea8a3367aac00ffb8e4b76-0000.us-east.containers.appdomain.cloud' 
    WARNING: No container image registry has been configured with the server. Automatic builds and deployments may not function.
    Build scheduled, use 'oc logs -f buildconfig/rails-postgresql-example' to track its progress.
    Run 'oc status' to view your app.
behzadkoohi@Behzads-MBP temp % 

```
