# az

## intro


The Azure cloud command line interface is the command **`az`**. This page describes its use in relation to 
(or from) the bash shell.


## bash environment variables 


```
SUB=8888aa7a-dd88-4886-b888-7a888889d845
RG=demoRG
LOCATION=eastus
STGE_ACCT=mydemostorageaccount
BLOB_CONT=mydemoblobcontainer


echo $SUB
```


az group show --resource-group $RG


az ad signed-in-user show --query objectId -o tsv | az role assignment create --role "Storage Blob Data Contributor" --assignee @- --scope "/subscriptions/$SUB/resourceGroups/$RG/providers/Microsoft.Storage/storageAccounts/$STGE_ACCT"

produces the following: 


WARNING: The underlying Active Directory Graph API will be replaced by Microsoft Graph API in a future version of Azure CLI. Please carefully review all breaking changes introduced during this migration: https://docs.microsoft.com/cli/azure/microsoft-graph-migration
The underlying Active Directory Graph API will be replaced by Microsoft Graph API in a future version of Azure CLI. Please carefully review all breaking changes introduced during this migration: https://docs.microsoft.com/cli/azure/microsoft-graph-migration

```
{
  "canDelegate": null,
  "condition": null,
  "conditionVersion": null,
  "description": null,
  "id": "/subscriptions/8388aa7a-dd39-4996-b9a9-7adb5b69d845/resourceGroups/demoRG/providers/Microsoft.Storage/storageAccounts/demosarobfatland/providers/Microsoft.Authorization/roleAssignments/ed2cbf2e-4895-478a-9d54-c06b42814aae",
  "name": "ed2cbf2e-4895-478a-9d54-c06b42814aae",
  "principalId": "6aa1a880-0541-4467-bf63-2cd43481a9d7",
  "principalType": "User",
  "resourceGroup": "demoRG",
  "roleDefinitionId": "/subscriptions/8388aa7a-dd39-4996-b9a9-7adb5b69d845/providers/Microsoft.Authorization/roleDefinitions/ba92f5b4-2d11-453d-a403-e96b0029c9fe",
  "scope": "/subscriptions/8388aa7a-dd39-4996-b9a9-7adb5b69d845/resourceGroups/demoRG/providers/Microsoft.Storage/storageAccounts/demosarobfatland",
  "type": "Microsoft.Authorization/roleAssignments"
}
```

Then: 


```
az storage blob upload --account-name $STGE_ACCT --container-name $BLOB_CONT --name demo_file_upload.txt --file demo_file.txt --auth-mode login
```

Produces an error: 

You do not have the required permissions needed to perform this operation.
Depending on your operation, you may need to be assigned one of the following roles:
    "Storage Blob Data Contributor"
    "Storage Blob Data Reader"
    "Storage Queue Data Contributor"
    "Storage Queue Data Reader"

If you want to use the old authentication method and allow querying for the right account key, please use the "--auth-mode" parameter and "key" value.
                    

Ran it again and it worked, very strange.  One guesses this was propagation of the prior command through the system took some time. Result is now: 


az storage blob upload --account-name $STGE_ACCT --container-name $BLOB_CONT --name demo_file_upload.txt --file demo_file.txt --auth-mode login
Finished[#############################################################]  100.0000%
{
  "client_request_id": "6160010c-9f0f-11ec-abde-41c2af15d183",
  "content_md5": "GgLQLaEhKmVJ5hZ/CHEeag==",
  "date": "2022-03-08T18:41:35+00:00",
  "encryption_key_sha256": null,
  "encryption_scope": null,
  "etag": "\"0x8DA0133465AF4F1\"",
  "lastModified": "2022-03-08T18:41:36+00:00",
  "request_id": "64ce8cc6-c01e-000c-3f1c-33280f000000",
  "request_server_encrypted": true,
  "version": "2020-10-02",
  "version_id": null
}


In the Azure console: Navigate to containers for the Storage Account, look at the blob container, look at IAM, look at Role Assignments: I should be there as a Storage Blob Data Contributor.



Now to download the same file: 


az storage blob download --account-name $STGE_ACCT --container-name $BLOB_CONT --name demo_file_upload.txt --file demo_file_download.txt --auth-mode login


And we get confirmation JSON: 


Finished[#############################################################]  100.0000%
{
  "content": null,
  "deleted": false,
  "metadata": {},
  "name": "demo_file_upload.txt",
  "properties": {
    "appendBlobCommittedBlockCount": null,
    "blobTier": null,
    "blobTierChangeTime": null,
    "blobTierInferred": false,
    "blobType": "BlockBlob",
    "contentLength": 10,
    "contentRange": "bytes 0-9/10",
    "contentSettings": {
      "cacheControl": null,
      "contentDisposition": null,
      "contentEncoding": null,
      "contentLanguage": null,
      "contentMd5": "GgLQLaEhKmVJ5hZ/CHEeag==",
      "contentType": "application/octet-stream"
    },
    "copy": {
      "completionTime": null,
      "id": null,
      "progress": null,
      "source": null,
      "status": null,
      "statusDescription": null
    },
    "creationTime": "2022-03-08T18:41:36+00:00",
    "deletedTime": null,
    "etag": "\"0x8DA0133465AF4F1\"",
    "lastModified": "2022-03-08T18:41:36+00:00",
    "lease": {
      "duration": null,
      "state": "available",
      "status": "unlocked"
    },
    "pageBlobSequenceNumber": null,
    "remainingRetentionDays": null,
    "serverEncrypted": true
  },
  "snapshot": null
}



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
