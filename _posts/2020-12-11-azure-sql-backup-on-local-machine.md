---
layout: post
title:  "Local backup of an Azure Sql Server database"
tags:
-  .NET Core
-  microservices
-  Continuous Integration/Continuous Deployment
-  GitHub Actions
-  Azure
-  Azure CLI
author: Maurizio Attanasi
---

[Install the Microsoft ODBC driver for SQL Server (macOS)](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/install-microsoft-odbc-driver-sql-server-macos?view=sql-server-ver15)

```sh
brew tap microsoft/mssql-release https://github.com/Microsoft/homebrew-mssql-release

brew update

HOMEBREW_NO_ENV_FILTERING=1 ACCEPT_EULA=Y brew install msodbcsql17 mssql-tools
```

[Use CLI to backup an Azure SQL single database to an Azure storage container](https://docs.microsoft.com/en-us/azure/sql-database/scripts/sql-database-backup-database-cli)

```sh
#!/bin/bash
location="francecentral"
idtentifier=atech-sql-test

resource=rg-$identifier
server=srv-$identifier
database=db-$identifier
storage=storage$identifier
container=container-$identifier

bacpac=backup.bacpac

login=sampleLogin
password=samplePassword123!

echo "Using resource group $resource with login: $login, password: $password..."

echo "Creating resource group $resource..."
az group create --name $resource --location "$location"

echo "Creating $storage..."
az storage account create --name $storage --resource-group $resource --location "$location" --sku Standard_LRS

echo "Creating $container on $storage..."
key=$(az storage account keys list --account-name $storage --resource-group $resource -o json --query [0].value | tr -d '"')
az storage container create --name $container --account-key $key --account-name $storage

echo "Creating $server..."
az sql server create --name $server --resource-group $resource --location "$location" --admin-user $login --admin-password $password
az sql server firewall-rule create --resource-group $resource --server $server --name AllowAzureServices --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0

echo "Creating $database..."
az sql db create --name $database --resource-group $resource --server $server --edition GeneralPurpose --sample-name AdventureWorksLT

echo "Backing up $database..."
az sql db export --admin-password $password --admin-user $login --storage-key $key --storage-key-type StorageAccessKey --storage-uri "https://$storage.blob.core.windows.net/$container/$bacpac" --name $database --resource-group $resource --server $server
```

Download

```sh
az storage blob download -c container-atechsqltest -f ./backup.bacpac -n https://storageatechsqltest.blob.core.windows.net/container-atechsqltest/backup.bacpac
```