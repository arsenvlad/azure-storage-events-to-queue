# Azure Storage Events to Azure Queue Service

1. Create storage account for queue
```
az group create --name avegs1 --location eastus2
az group deployment create --resource-group avegs1 --template-file 1-queue-storage.json
```
2. Create queue (since it is not possible to create it via ARM template)
```
az storage queue create --resource-group avegs1 --account-name avegs1

queueStorageAccountResourceId=$(az storage account show --name avegs1 --resource-group avegs1 --query id --output tsv)
```
3. Create ingest storage account with Event Grid subscription pointing to the created queue
```
az group deployment create --resource-group avegs1 --template-file 2-ingest-storage-event-subscription.json
```
4. Upload and delete files in storage account
```
export ak=$(az storage account keys list --resource-group avegs1 --account-name avegs1ingest --query "[0].value" -o tsv)

az storage blob upload --account-name avegs1ingest --account-key "$ak" --container "ingestcontainer" --name create-queue.sh --file create-queue.sh
az storage blob delete --account-name avegs1ingest --account-key "$ak" --container "ingestcontainer" --name create-queue.sh

az storage blob upload-batch --account-name avegs1ingest --account-key "$ak" --source "/mnt/c/temp" --destination "ingestcontainer/temp"
az storage blob delete-batch --account-name avegs1ingest --account-key "$ak" --source "ingestcontainer" --pattern "temp/*"
```
5. View queue contents in https://portal.azure.com
