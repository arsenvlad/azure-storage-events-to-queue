# Azure Storage Events to Azure Queue Service

0. Enable Event Grid resource provider for your subscription
```
az provider register --namespace Microsoft.EventGrid
az provider show -n Microsoft.EventGrid
```
1. Create storage account for queue
```
az group create --name avegs1 --location eastus2
az group deployment create --resource-group avegs1 --template-file 1-queue-storage.json
```
2. Create queue (since it is not possible to create it via ARM template)
```
az storage queue create --account-name avegs1 --name queue1

queueStorageAccountResourceId=$(az storage account show --name avegs1 --resource-group avegs1 --query id --output tsv)
```
3. Create ingest storage account with Event Grid subscription pointing to the created queue
```
az group deployment create --resource-group avegs1 --template-file 2-ingest-storage-event-subscription.json
```
4. Upload and delete files in storage account
```
export ak=$(az storage account keys list --resource-group avegs1 --account-name avegs1ingest --query "[0].value" -o tsv)

az storage blob upload --account-name avegs1ingest --account-key "$ak" --container "ingestcontainer" --name README.md --file README.md
az storage blob delete --account-name avegs1ingest --account-key "$ak" --container "ingestcontainer" --name README.md

az storage blob upload-batch --account-name avegs1ingest --account-key "$ak" --source "/mnt/c/temp" --destination "ingestcontainer/temp"
az storage blob delete-batch --account-name avegs1ingest --account-key "$ak" --source "ingestcontainer" --pattern "temp/*"
```
5. View queue contents in https://portal.azure.com

## Scale Experiments

Create many event subscriptions for same storage account (use minimum of 5 since that's the batch size in the template) to see if it possible to create more than 500 event subscriptions on each account.
```
az group deployment create --resource-group avegs1 --template-file 3-ingest-storage-many-event-subscriptions.json
```

After about ~505 event subscriptions getting the following error. This is expected based on the documentation of current limit of "500 event subscriptions per topic" https://docs.microsoft.com/en-us/azure/azure-subscription-service-limits#azure-event-grid-limits

Deployment failed. Correlation ID: bf6e4a31-62aa-4794-b11c-647d2a73fb3a. 
```json
{
  "error": {
    "code": "ResourceConflict",
    "message": "Quota limit reached."
  }
}
```

Create many storage accounts and one event subscription for each to see if possible to create more than 100 storage accounts as topics
```
az group deployment create --resource-group avegs1 --template-file 4-many-ingest-storage-accounts-event-subscription.json
```