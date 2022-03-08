# az

## intro


The Azure cloud command line interface is the command **`az`**. This page describes its use in relation to 
(or from) the bash shell.


## bash environment variables 

Run these commands from a bash shell on an Azure Virtual Machine.


```
SUB=8888aa7a-dd88-4886-b888-7a888889d845                      <- subscription: copy this using the copy icon in the Azure console
RG=demoRG                                                     <- resource group
LOCATION=eastus                                               <- region
STGE_ACCT=mydemostorageaccount                                <- storage account
BLOB_CONT=mydemoblobcontainer                                 <- container, specifically a blob container under the storage account


echo $SUB                                                     <-protocol for reviewing an environment variable in bash


az group show --resource-group $RG                            <- first example of using the Azure CLI


echo that was a close call > test.txt                         <- create a one-line text file
cat test.txt                                                  <- verify it worked
```


At this point we have a set of environment variables that describe working Azure resources. These include at the top
level a **Subscription** and within that a **Resource Group**.  A Resource Group is a virtual construct intended to
associate some related resources. These include Virtual Machine (VM) Instances and Storage Accounts. Associated resources
tend to be located in a specific region. Here we are working in the **East US** region; and we have used the Azure 
(browser) console to create the other resources. 


As a working metaphor: A Storage Account is like the trunk of a car. A container within that storage account is like a grocery bag 
placed in the trunk of the car. The particular working example here is a **blob** container. **Blob** is the Azure term for 
object storage: Cheap storage, effectively infinite capacity, and file contents are not directly addressable. Hence the file is
treated as an object when at rest in object storage, hence 'blob'.


In the source tutorial the Azure blob container was created with "anonymous contributor" enabled. This is not the complete 
story for enabling agents to create data within this blob container. A second component is needed: The would-be contributor
must have the appropriate **Role** associating themselves to the blob container. The following rather long command accomplishes
this:


```az ad signed-in-user show --query objectId -o tsv | az role assignment create --role "Storage Blob Data Contributor" --assignee @- --scope "/subscriptions/$SUB/resourceGroups/$RG/providers/Microsoft.Storage/storageAccounts/$STGE_ACCT"


- This command requires some disassembly. 
- It would be a perfect world if every prior step was done as a cli call



produces the following: 


```
WARNING: Graph API will make the above call obsolete at some point. 
See [https://docs.microsoft.com/cli/azure/microsoft-graph-migration](https://docs.microsoft.com/cli/azure/microsoft-graph-migration).
{
  "canDelegate": null,
  "condition": null,
  "conditionVersion": null,
  "description": null,
  "id": "/subscriptions/8388888a-dd39-4886-b888-7ad888888885/resourceGroups/demoRG/providers/Microsoft.Storage/storageAccounts/demosarobfatland/providers/Microsoft.Authorization/roleAssignments/ed2cbf2e-4895-478a-9d54-c06b42814aae",
  "name": "ed2cbf2e-4895-478a-9d54-c06b42814aae",
  "principalId": "6aa1a880-0541-4467-bf63-2cd43481a9d7",
  "principalType": "User",
  "resourceGroup": "demoRG",
  "roleDefinitionId": "/subscriptions/8388aa7a-dd39-4996-b9a9-7adb5b69d845/providers/Microsoft.Authorization/roleDefinitions/ba92f5b4-2d11-453d-a403-e96b0029c9fe",
  "scope": "/subscriptions/8388aa7a-dd39-4996-b9a9-7adb5b69d845/resourceGroups/demoRG/providers/Microsoft.Storage/storageAccounts/demosarobfatland",
  "type": "Microsoft.Authorization/roleAssignments"
}
```

From here we can use the cli to upload a file to the blob container.


```
az storage blob upload --account-name $STGE_ACCT --container-name $BLOB_CONT --name demo_test.txt --file test.txt --auth-mode login
```

The above may produce an error: **`You do not have the required permissions needed... etcetera`**


Wait two minutes, run it again. This is the slow propagation of effects of the prior command through the system. Now we have:


```
Finished[#############################################################]  100.0000%
{
  "client_request_id": 
  
  ...etcetera JSON...

"request_server_encrypted": true,
  "version": "2020-10-02",
  "version_id": null
}
```


In the Azure console: Navigate to containers for the Storage Account, 
look at the blob container, look at its IAM tab, there examine Role Assignments: 
Myself the User should be listed there as a Storage Blob Data Contributor.
That confirms the above two steps.


Navigate to the container and verify that the file is there in the blob container as intended. 


Download the same file: 

```
az storage blob download --account-name $STGE_ACCT --container-name $BLOB_CONT --name return_demo_test.txt --file demo_test.txt --auth-mode login
```

Confirmation JSON looks like this: 

```
Finished[#############################################################]  100.0000%
{
  "content": null,
  "deleted": false,
  "metadata": {},

  ...etcetera JSON...
  
  },
  "snapshot": null
}
```


To see what is in storage: 

az storage account list

Produces: 

```
[ a lot of JSON ]
```




Delete this storage account: 


az storage account delete -n $STGE_ACCT -g $RG



URL for LANDSAT data from ai4e: 

https://ai4edatasetspublicassets.blob.core.windows.net/assets/S2_TilingSystem2-1.txt



After getting the LANDSAT image: Push to blob: 


```
azureuser@demoVM2:~/CLASS-Examples/azure-landsat$ az storage blob upload --account-name $STGE_ACCT --container-name $BLOB_CONT --name test.png --file test.png --auth-mode login

Finished[#############################################################]  100.0000%
{
  etcetera
}
```
