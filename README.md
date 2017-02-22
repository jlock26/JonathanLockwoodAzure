# Submit a Hive job script to HDInsight cluster using CLI
This is a tutorial that demonstrates how to create an HDInsight cluster using the Azure CLI. It will also show how to submit a hive job 
script and execute it and transfer the results to a blob storage container. Finally it will demonstrate how to delete the cluster.
## Requirements
* Azure subscription or [trial account](https://azure.microsoft.com/en-us/free/)
* Windows only: [Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

## Introduction
### What is HDInsight?
[HDInsight](https://azure.microsoft.com/en-us/services/hdinsight/) is an open source Hadoop cloud service for big data analysis. A variety of development environments can be used to help obtain accurate models from the data.

### What is a Hive job?
Hive is a data system for Hadoop which allows for data analysis to help structure big data. It can be run through a variety of different ways as seen in the table below. 

#### How to run Hive job
Method           | Cluster Operating System  |  Client Operating System |
---------------- | --------------------------|------------------------- |
Hive View        | Linux                     | Any                      |
SSH Session      | Linux                     | Linux, Unix, Mac, Windows|
Curl             | Linux, Windows            | Linux, Unix, Mac, Windows|
Query Console    | Windows                   | Any                      |
Visual Studio    | Linux, Windows            | Windows                  |
Powershell       | Linux, Windows            | Windows                  |
Remote Desktop   | Windows                   | Windwos                  |


## Creating an HDInsight cluster
1. Install the Azure CLI
  * Depending on whether you're using Windows or Mac, select one of the following installers:
  * [Windows installer](http://aka.ms/webpi-azure-cli)
  * [Mac installer](http://aka.ms/mac-azure-cli)
2. Once the Azure CLI has finished installing, open up the **Command Prompt**
 ![Command Prompt](https://github.com/jlock26/JonathanLockwoodAzure/blob/master/cmd.png "Command Prompt")
3. To login to azure through the CLI type:
 ```
 azure login
 ```
  * The output will provide a link and a code to autheticate the user login
  * Follow the link and enter the provided code. The page will look something like this:
 ![CLI auth](https://github.com/jlock26/JonathanLockwoodAzure/blob/master/azure%20cli%20login%202.JPG "azure cli authentication")
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
   * Find valid locations using the command ```azure location list```
   
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
 ![storage keys](https://github.com/jlock26/JonathanLockwoodAzure/blob/master/storage%20keys.JPG "storage keys")
 * Save the **key1** value

8. To create a Hadoop HDInsight cluster:
 ```
 azure hdinsight cluster create -g groupname -l location -y Linux --clusterType Hadoop --defaultStorageAccountName storagename.blob.core.windows.net --defaultStorageAccountKey storagekey --defaultStorageContainer clustername --workerNodeCount 2 --userName admin --password httppassword --sshUserName sshuser --sshPassword sshuserpassword clustername
 ```
 * Replace **groupname** and **storagename** with their respective values
 * Replace **storagekey** with the **key1** value from the previous step
 * Replace **clustername**, **admin**, **httppassword**, **sshuser**, **sshuserpassword**, and **clustername** with unique values for each
 
 > Note: This process will take ~10 minutes

## Submit a Hive job script to the cluster
1. Connect with SSH
 * For windows clients, download [Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
   * Open PuTTYgen and click generate  
   
   ![puttygen](https://github.com/jlock26/JonathanLockwoodAzure/blob/master/puttygen.JPG "puttygen")
   * Move the mouse over the blank area to create randomness
   
   ![](https://github.com/jlock26/JonathanLockwoodAzure/blob/master/puttygen2.JPG "puttygen2")
   
   * Enter unique keyphrase and click save private key
   
   ![](https://github.com/jlock26/JonathanLockwoodAzure/blob/master/puttygen3.JPG "puttygen3")
   
   * Open Putty
    * In **category**, expand **Connection**, expand **SSH**, select **Auth**. Click **Browse** and find the .ppk file you just saved
    
    ![](https://github.com/jlock26/JonathanLockwoodAzure/blob/master/putty1.JPG "putty1")
    
    * In **category**, select **Session** and enter the SSH address in the **Host Name** box.
     > The SSH address is clustername-ssh.azurehdinsight.net where **clustername** is your unique name
     
     ![](https://github.com/jlock26/JonathanLockwoodAzure/blob/master/putty2.JPG "putty2")
     
     * Click **Open** to connect
     * Enter the username and password while creating the cluster when prompted
2. Start the Hive CLI by entering
 ```
 hive
 ```
 
 * In this example, we use a log4j sample file 
  > Note: this can be found at **example/data/sample.log** in the blob storage container
   
 * Using the CLI, create a new table called **log4jLogs**
   * Using **DROP TABLE**: In case table exists, delete table and file
    
    ```
     DROP TABLE log4jLogs;
     ```
   * Create external table in Hive. Data remains in original location
     
     ```
     CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
     ```
   * Use **ROW FORMAT** to communicate to Hive how rows are formatted. Use space as a delimiter
    
     ```
     ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
     ```
   * Communicate to Hive where data is stored
    > Note: Data is stored as text
    
     ```
     STORED AS TEXTFILE LOCATION 'wasbs:///example/data/';
     ```
   * Select the number of rows where column t4 contains [ERROR]
     
     ```
     SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log' GROUP BY t4;
     ```
   * Create an internal table **errorLogs**  
    
     > Note: To create internal table, remove **EXTERNAL** keyword
     
     > Note: (ORC) is Optimized Row Columnar. Effective for storing Hive data
    
     ```
     CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
     ```
   
   * Select rows from **log4jLogs** that have [ERROR] and add them to the **errorLogs** table
     
     ```
     INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log';
     ```
   * To check output, ```SELECT * from errorLogs;``` should output 3 rows. 

     ![](https://github.com/jlock26/JonathanLockwoodAzure/blob/master/errorLogs.JPG "errorLogs")
     
## Transfer Hive results to blob storage
 1. To determine the location to transfer the results, in the Hive CLI write:
 
   ```
   INSERT OVERWRITE DIRECTORY 'wasb://containername@storagename.blob.core.windows.net/output'
   ```
    * Replace **containername** with the cluster name you came up with earlier
    * Replace **storagename** with the unique storage name from earlier
    * Replace **output** with a unique folder name for the results
    
         > Note: To point to an Azure blob storage, wasb:// or wasbs:// must be used for the uri
         
 2. In the next line type:
   ```
   SELECT * from errorLogs;
   ```
   * This transfers the 3 row Hive result output to the desired location
    
## Deleting the HDInsight cluster
 1. In the Azure CLI, use the following to delete the cluster
 
   ```
   azure hdinsight cluster delete clustername
   ```
    * Replace **clustername** with your unique cluster name
 2. Enter your Resource Group name when prompted
    
    > Note: This process will take ~10 minutes
     
    
