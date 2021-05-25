---
layout: post
title:  "Local backup of an Azure SQL Server database"
tags:
-  .NET Core
-  microservices
-  database
-  SQL server
-  disaster recovery
-  backup
-  Azure
-  Azure CLI
author: Maurizio Attanasi
---

<div>
  <div style="float: left; margin-left: 12em;">
    <img src="/assets/images/azure-sql-database.png" alt="azure-sql-database">
  </div>
  <div style="float: left">
    <img src="/assets/images/azure-storage.png" alt="azure-storage">
  </div>
  <div style="clear: both"/>
</div>
Having databases on a cloud provider like [Microsoft Azure](https://portal.azure.com) virtually protects us from sudden data loss due to hardware failure, virus or malware attack, or whatever, because instance mirroring is generally provided even in the cheapest plans.
However, there may be technical or legal reasons why it is preferable to have a copy of the backups also on our local file system. A typical example is having a development server with data always aligned with the instance in production.

In this note, I will illustrate a possible approach to the problem in the case of a SQL Server database hosted on Azure.

To achieve our goal we will make use of the [Azure Command-Line Interface](https://docs.microsoft.com/it-it/cli/azure/) (a.k.a. azure CLI), a cross-platform set of commands used to create and manage Azure resources, but the same results can be achieved also from the Microsoft Azure Portal Dashboard [https://portal.azure.com](https://portal.azure.com).

First of all, from the terminal of our choice, we need to authenticate with the following command

```sh
~ az login
```

Once logged in, only for demo purpose, we can run the following script

```sh
#!/bin/bash
location="francecentral"
identifier="atechsqltest"

resource=rg-$identifier
storage=storage$identifier
container=container-$identifier

server=srv-$identifier
database=db-$identifier

login=sampleLogin
password=samplePassword123!

bacpac=backup.bacpac

echo "Creating resource group $resource in $location .."
az group create --name $resource --location "$location"

echo "Creating $storage in $resource resource group in $location location"
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

echo "Downloading $bacpac ..."
az storage blob download -c $container -f ./$bacpac -n $bacpac --account-key $key --account-name $storage

echo "Backup completed"
```

that, in order, will create:

- a resource group
- an Azure Storage Account
- a container in the Storage
- a database server
- a new database

With all the needed resources set,

<p align='center'>
  <img src='/assets/images/atech-sql-test-resources.png' alt='Created resources' style="max-width:100%">
</p>

we can continue backing up the created database in the container and downloading it on our local machine.

## Database Backup

The command responsible for backing up the database on the storage container is the following:

```sh

...

echo "Backing up $database..."
az sql db export --admin-password $password --admin-user $login --storage-key $key --storage-key-type StorageAccessKey --storage-uri "https://$storage.blob.core.windows.net/$container/$bacpac" --name $database --resource-group $resource --server $server

...

```

## Backup Download

The snippet which is responsible for downloading the backup file to our local file system is instead:

```sh

...

echo "Downloading $bacpac ..."
az storage blob download -c $container -f ./$bacpac -n $bacpac --account-key $key --account-name $storage

...

```

<p align=center>
  <img src='/assets/images/atech-sql-test-backup.png' alt='local backup' style="max-width:100%">
</p>


## Cleanup

At the end of the test, perhaps we would like to perform a cleanup of all the resources created.

```sh
~ az group delete --name $resource
```

<br/>

Enjoy, :fireworks:

---

<p align="center">
  <img src="/assets/images/keep-calm-wear-mask-red-small.jpg" alt="Stay Safe Wear a Mask" />
</p>
