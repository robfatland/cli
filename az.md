Working with the blob set up for anonymous access

SUB=8388aa7a-dd39-4996-b9a9-7adb5b69d845
RG=demoRG
LOCATION=eastus
STGE_ACCT=demosarobfatland
BLOB_CONT=demoblob
echo $SUB

az group show --resource-group $RG


az ad signed-in-user show --query objectId -o tsv | az role assignment create --role "Storage Blob Data Contributor" --assignee @- --scope "/subscriptions/$SUB/resourceGroups/$RG/providers/Microsoft.Storage/storageAccounts/$STGE_ACCT"

produces the following: 


WARNING: The underlying Active Directory Graph API will be replaced by Microsoft Graph API in a future version of Azure CLI. Please carefully review all breaking changes introduced during this migration: https://docs.microsoft.com/cli/azure/microsoft-graph-migration
The underlying Active Directory Graph API will be replaced by Microsoft Graph API in a future version of Azure CLI. Please carefully review all breaking changes introduced during this migration: https://docs.microsoft.com/cli/azure/microsoft-graph-migration
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


Then: 


az storage blob upload --account-name $STGE_ACCT --container-name $BLOB_CONT --name demo_file_upload.txt --file demo_file.txt --auth-mode login


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

[
  {
    "accessTier": "Hot",
    "allowBlobPublicAccess": true,
    "allowCrossTenantReplication": true,
    "allowSharedKeyAccess": true,
    "allowedCopyScope": null,
    "azureFilesIdentityBasedAuthentication": null,
    "blobRestoreStatus": null,
    "creationTime": "2022-03-08T18:07:50.760268+00:00",
    "customDomain": null,
    "defaultToOAuthAuthentication": false,
    "enableHttpsTrafficOnly": true,
    "enableNfsV3": null,
    "encryption": {
      "encryptionIdentity": null,
      "keySource": "Microsoft.Storage",
      "keyVaultProperties": null,
      "requireInfrastructureEncryption": false,
      "services": {
        "blob": {
          "enabled": true,
          "keyType": "Account",
          "lastEnabledTime": "2022-03-08T18:07:50.900898+00:00"
        },
        "file": {
          "enabled": true,
          "keyType": "Account",
          "lastEnabledTime": "2022-03-08T18:07:50.900898+00:00"
        },
        "queue": null,
        "table": null
      }
    },
    "extendedLocation": null,
    "failoverInProgress": null,
    "geoReplicationStats": null,
    "id": "/subscriptions/8388aa7a-dd39-4996-b9a9-7adb5b69d845/resourceGroups/demoRG/providers/Microsoft.Storage/storageAccounts/demosarobfatland",
    "identity": null,
    "immutableStorageWithVersioning": null,
    "isHnsEnabled": null,
    "isLocalUserEnabled": null,
    "isSftpEnabled": null,
    "keyCreationTime": {
      "key1": "2022-03-08T18:07:50.885263+00:00",
      "key2": "2022-03-08T18:07:50.885263+00:00"
    },
    "keyPolicy": null,
    "kind": "StorageV2",
    "largeFileSharesState": null,
    "lastGeoFailoverTime": null,
    "location": "eastus",
    "minimumTlsVersion": "TLS1_2",
    "name": "demosarobfatland",
    "networkRuleSet": {
      "bypass": "AzureServices",
      "defaultAction": "Allow",
      "ipRules": [],
      "resourceAccessRules": null,
      "virtualNetworkRules": []
    },
    "primaryEndpoints": {
      "blob": "https://demosarobfatland.blob.core.windows.net/",
      "dfs": "https://demosarobfatland.dfs.core.windows.net/",
      "file": "https://demosarobfatland.file.core.windows.net/",
      "internetEndpoints": null,
      "microsoftEndpoints": null,
      "queue": "https://demosarobfatland.queue.core.windows.net/",
      "table": "https://demosarobfatland.table.core.windows.net/",
      "web": "https://demosarobfatland.z13.web.core.windows.net/"
    },
    "primaryLocation": "eastus",
    "privateEndpointConnections": [],
    "provisioningState": "Succeeded",
    "publicNetworkAccess": null,
    "resourceGroup": "demoRG",
    "routingPreference": null,
    "sasPolicy": null,
    "secondaryEndpoints": {
      "blob": "https://demosarobfatland-secondary.blob.core.windows.net/",
      "dfs": "https://demosarobfatland-secondary.dfs.core.windows.net/",
      "file": null,
      "internetEndpoints": null,
      "microsoftEndpoints": null,
      "queue": "https://demosarobfatland-secondary.queue.core.windows.net/",
      "table": "https://demosarobfatland-secondary.table.core.windows.net/",
      "web": "https://demosarobfatland-secondary.z13.web.core.windows.net/"
    },
    "secondaryLocation": "westus",
    "sku": {
      "name": "Standard_RAGRS",
      "tier": "Standard"
    },
    "statusOfPrimary": "available",
    "statusOfSecondary": "available",
    "tags": {},
    "type": "Microsoft.Storage/storageAccounts"
  },
  {
    "accessTier": "Hot",
    "allowBlobPublicAccess": false,
    "allowCrossTenantReplication": null,
    "allowSharedKeyAccess": null,
    "allowedCopyScope": null,
    "azureFilesIdentityBasedAuthentication": null,
    "blobRestoreStatus": null,
    "creationTime": "2022-03-08T17:21:08.575275+00:00",
    "customDomain": null,
    "defaultToOAuthAuthentication": null,
    "enableHttpsTrafficOnly": true,
    "enableNfsV3": null,
    "encryption": {
      "encryptionIdentity": null,
      "keySource": "Microsoft.Storage",
      "keyVaultProperties": null,
      "requireInfrastructureEncryption": null,
      "services": {
        "blob": {
          "enabled": true,
          "keyType": "Account",
          "lastEnabledTime": "2022-03-08T17:21:08.684669+00:00"
        },
        "file": {
          "enabled": true,
          "keyType": "Account",
          "lastEnabledTime": "2022-03-08T17:21:08.684669+00:00"
        },
        "queue": null,
        "table": null
      }
    },
    "extendedLocation": null,
    "failoverInProgress": null,
    "geoReplicationStats": null,
    "id": "/subscriptions/8388aa7a-dd39-4996-b9a9-7adb5b69d845/resourceGroups/cloud-shell-storage-westus/providers/Microsoft.Storage/storageAccounts/cs410037ffe94229704",
    "identity": null,
    "immutableStorageWithVersioning": null,
    "isHnsEnabled": null,
    "isLocalUserEnabled": null,
    "isSftpEnabled": null,
    "keyCreationTime": {
      "key1": "2022-03-08T17:21:08.669024+00:00",
      "key2": "2022-03-08T17:21:08.669024+00:00"
    },
    "keyPolicy": null,
    "kind": "StorageV2",
    "largeFileSharesState": null,
    "lastGeoFailoverTime": null,
    "location": "westus",
    "minimumTlsVersion": "TLS1_2",
    "name": "cs410037ffe94229704",
    "networkRuleSet": {
      "bypass": "AzureServices",
      "defaultAction": "Allow",
      "ipRules": [],
      "resourceAccessRules": null,
      "virtualNetworkRules": []
    },
    "primaryEndpoints": {
      "blob": "https://cs410037ffe94229704.blob.core.windows.net/",
      "dfs": "https://cs410037ffe94229704.dfs.core.windows.net/",
      "file": "https://cs410037ffe94229704.file.core.windows.net/",
      "internetEndpoints": null,
      "microsoftEndpoints": null,
      "queue": "https://cs410037ffe94229704.queue.core.windows.net/",
      "table": "https://cs410037ffe94229704.table.core.windows.net/",
      "web": "https://cs410037ffe94229704.z22.web.core.windows.net/"
    },
    "primaryLocation": "westus",
    "privateEndpointConnections": [],
    "provisioningState": "Succeeded",
    "publicNetworkAccess": null,
    "resourceGroup": "cloud-shell-storage-westus",
    "routingPreference": null,
    "sasPolicy": null,
    "secondaryEndpoints": null,
    "secondaryLocation": null,
    "sku": {
      "name": "Standard_LRS",
      "tier": "Standard"
    },
    "statusOfPrimary": "available",
    "statusOfSecondary": null,
    "tags": {
      "ms-resource-usage": "azure-cloud-shell"
    },
    "type": "Microsoft.Storage/storageAccounts"
  }
]



Delete this storage account: 


az storage account delete -n $STGE_ACCT -g $RG



URL for LANDSAT data from ai4e: 

https://ai4edatasetspublicassets.blob.core.windows.net/assets/S2_TilingSystem2-1.txt



After getting the LANDSAT image: Push to blob: 


azureuser@demoVM2:~/CLASS-Examples/azure-landsat$ az storage blob upload --account-name $STGE_ACCT --container-name $BLOB_CONT --name test.png --file test.png --auth-mode login
Finished[#############################################################]  100.0000%
{
  "client_request_id": "f129fbd2-9f12-11ec-abde-41c2af15d183",
  "content_md5": "6uIiz96iDYk0dKtPq+rdyg==",
  "date": "2022-03-08T19:07:05+00:00",
  "encryption_key_sha256": null,
  "encryption_scope": null,
  "etag": "\"0x8DA0136D5D88B01\"",
  "lastModified": "2022-03-08T19:07:05+00:00",
  "request_id": "09b39a4f-801e-0040-421f-33b83f000000",
  "request_server_encrypted": true,
  "version": "2020-10-02",
  "version_id": null
}
