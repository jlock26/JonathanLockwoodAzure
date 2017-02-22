# Submit a Hive job script to HDInsight cluster using CLI
## Requirements
* Azure subscription or [trial account](https://azure.microsoft.com/en-us/free/)
* Windows only: [Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

## Introduction
This is a tutorial that demonstrates how to create an HDInsight cluster using the Azure CLI. It will also show how to submit a hive job 
script and execute it and transfer the results to a blob storage container. Finally it will demonstrate how to close the HDInsight cluster.

## What is HDInsight?

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
![alt text](https://github.com/jlock26/JonathanLockwoodAzure/blob/master/azure%20cli%20login%202.JPG "azure cli authentication")
* enter the account login info when prompted

4. Switch modes to the Azure Resource Manager:
```
azure config mode arm
```
5. Create a resource group using the command:
```
azure group create groupname location
```
* Replace **groupname** and **location** with unique names
  * Find valid locations using the command *azure location list*
6. Create a storage account for the HDInsight cluster
```
azure storage account create -g groupname --sku-name RAGRS -l location --kind Storage storagename
```
* Replace **groupname** and **location** with the same name as before
* Replace **storagename** with a specific name
7. Get the key required for accessing the storage account
```
azure storage account keys list -g groupname storagename
```
* The data will output something like this
![alt text](https://github.com/jlock26/JonathanLockwoodAzure/blob/master/storage%20keys.JPG "storage keys")
* Save the **key1** value

8. To create a Hadoop HDInsight cluster:
```
azure hdinsight cluster create -g groupname -l location -y Linux --clusterType Hadoop --defaultStorageAccountName storagename.blob.core.windows.net --defaultStorageAccountKey storagekey --defaultStorageContainer clustername --workerNodeCount 2 --userName admin --password httppassword --sshUserName sshuser --sshPassword sshuserpassword clustername
```
* Replace **groupname** and **storagename** with their respective values
* Replace **storagekey** with the **key1** value from the previous step
* Replace **clustername**, **admin**, **httppassword**, **sshuser**, **sshuserpassword**, and **clustername** with unique values for each

## Submit a Hive job script to the cluster
1. Connect with SSH
* For windows clients, download [Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
 * Open PuTTYgen and click generate
 ![alt text]("puttygen")
 * Move the mouse over the blank area to create randomness
 ![alt text]("puttygen2")
 * Enter unique keyphrase and click save private key
 ![alt text]("puttygen3")
 * Open Putty
  * In **category**, expand **Connection**, expand **SSH**, select **Auth**. Click **Browse** and find the .ppk file you just saved
  ![alt text]("putty1")
  * In **category**, select **Session** and enter the SSH address in the **Host Name** box.
   > The SSH address is clustername-ssh.azurehdinsight.net where **clustername** is your unique name
   ![alt text]("putty2")
   * Click **Open** to connect
   * Enter the username and password while creating the cluster when prompted
2. Start the Hive CLI by entering
```
hive
```
