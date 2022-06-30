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



