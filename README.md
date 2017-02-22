# Submit a Hive job script to HDInsight cluster using CLI
## Requirements
* Azure subscription or [trial account](https://azure.microsoft.com/en-us/free/)
* Windows only: [Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

## Introduction
This is a tutorial that demonstrates how to create an HDInsight cluster using the Azure CLI. It will also show how to submit a hive job 
script and execute it and transfer the results to a blob storage container. Finally it will demonstrate how to close the HDInsight cluster.

## What is HDInsight?

## Key definitions:
* cluster - 
* hive - 
* blob - 
* CLI - "Command Line Interface"

## Creating an HDInsight cluster
1. Install the Azure CLI
  * Depending on whether you're using Windows or Mac, select one of the following installers:
  * [Windows installer](http://aka.ms/webpi-azure-cli)
  * [Mac installer](http://aka.ms/mac-azure-cli)
2. Once the Azure CLI has finished installing, open up the **Command Prompt**
![alt text](https://github.com/jlock26/JonathanLockwoodAzure/blob/master/cmd.png "Command Prompt")
3. To login to azure through the CLI type:
```
azure login
```
* The output will provide a link and a code to autheticate the user login
* Follow the link and enter the provided code. The page will look something like this:
![alt text]("azure cli authentication")
* You are then prompted to enter the username and password for the account you want to use
* Once authenticated, the output will read
```
info:   login command OK
```
4. Switch modes to the Azure Resource Manager using:
```
azure config mode arm
```
5. Create a resource group using the command:
```
azure group create groupname location
```
* Replace **groupname** with a specific name for your group
* Replace **location** with a specific location
  * Find valid locations using the command *azure location list*
6. Create a storage account for the HDInsight cluster
```
azure storage account create -g groupname --sku-name RAGRS -l location --kind Storage storagename
```
* Replace **groupname** with the same group name as before
* Replace **location** with the same location as before
* Replace **storagename** with a specific storage name
7. Get the key required for accessing the storage account
```
azure storage account keys list -g groupname storagename
```
* Replace **groupname** with your group name
* Replace **storagename** with your unique storage name
* The data will output something like this
![alt text]("storage keys")
* Save the **key1** value

8. To create a Hadoop HDInsight cluster:
```
azure hdinsight cluster create -g groupname -l location -y Linux --clusterType Hadoop --defaultStorageAccountName storagename.blob.core.windows.net --defaultStorageAccountKey storagekey --defaultStorageContainer clustername --workerNodeCount 2 --userName admin --password httppassword --sshUserName sshuser --sshPassword sshuserpassword clustername
```
* Replace **groupname** and **storagename** with their respective values
* Replace **storagekey** with the **key1** value from the previous step
* Replace **clustername**, **admin**, **httppassword**, **sshuser**, **sshuserpassword**, and **clustername** with unique values for each
> Note: 
