# Azure Data Factory et procedure stockée (Azure SQL Database)

In a project with a client, one step was to transform files, deposited in blob storage, using an existing stored procedure, and then move the result to an "Azure Files" storage. I found it interesting to use the copy activity with a stored procedure sourced.

This article aims to share the different steps to achieve this transformation pipeline and the various tricks used to carry out this part of the project.



## Prerequisite

- [An Azure subscription](https://azure.microsoft.com/fr-fr/free/)
- [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) 



# Creating Azure services
## Resource group
We will start by creating a group of resources to host the different services of our solution.

From [Azure portal](https://portal.azure.com), click "**Create a resource**"

![sparkle](Pictures/001.png)

 Then, look for "**Resource group**"

 ![sparkle](Pictures/002.png)


Click "**Create**"

![sparkle](Pictures/003.png)


In the "**Resource group**" field, give a name to your resource group. Click on the "**Review + Create**" button


![sparkle](Pictures/004.png)

In the validation screen, click on the "**Create**" button

![sparkle](Pictures/005.png)

Return to the Azure portal home. Click on the burger menu at the top left then on "**Resource groups**"

![sparkle](Pictures/006.png)

Click on the resource group created previously

![sparkle](Pictures/007.png)

## Azure Storage account

Once in the resource group, click the button  "**Add**"

![sparkle](Pictures/008.png)

Search for storage account 

![sparkle](Pictures/009.png)

Click "**Create**"

![sparkle](Pictures/010.png)


Complete the account storage creation and click "**Review** + **create**"

![sparkle](Pictures/011.png)

After checking the storage account creation information, click the button "**Create**"

![sparkle](Pictures/012.png)

## Azure SQL database creation
Here we will create a database only to host and execute our stored procedure. So, if you wish, you can use an existing Azure database.

Go back to the resource group. You must have your storage account as your first resource.

Click "**Add**"

![sparkle](Pictures/013.png)

Then, look for "**Azure SQL**" 

![sparkle](Pictures/014.png)

Click **Create**

![sparkle](Pictures/015.png)

Select **SQL Database** then click **Create**

![sparkle](Pictures/016.png)

Choose the previously created resource group, set the name of the database and create a new SQL server (it's also possible to use an existing server)

A **"Basic"** tier will be more than enough for our demonstration

Click **"Review + create"**


![sparkle](Pictures/017.png)

Click **"Create"**

![sparkle](Pictures/018.png)

After deploying your Azure SQL database and Azure SQL server you need to have 3 services in your resource group

![sparkle](Pictures/019.png)

## Azure Data Factory (ADF)

In your resource group, click **" + Add"**

![sparkle](Pictures/020.png)

In the search bar enter **"Data Factory"**

![sparkle](Pictures/021.png)

Then click **"Create"**

![sparkle](Pictures/022.png)

Make sure you've selected the right resource group and name your ADF service.

Select **"V2"**

Click **"Next: Git configuration "**

![sparkle](Pictures/023.png)

Check the **"Configure Git Later"** box and click **"Review + create"**

![sparkle](Pictures/024.png)

In the validation page, click**"Create"**

![sparkle](Pictures/025.png)

After creating the Azure Data Factory service, you should have 4 services in your resource group

![sparkle](Pictures/026.png)

## Preparing the stored procedure

In our example, the stored procedure will read data in blob storage and perform transformations. The transformations made here will be extremely basic. The aim here is to illustrate the use of procedures stored with Azure Data Factory.

### Azure SQL server setting

Set up the Azure SQL server Firewall so you can connect to it with tools like SQL Server Management Studio or Azure Data Studio

From the Azure portal, select your Azure SQL server, then click **"Firewalls and virtual networks"**

Enter the necessary ip addresses.

![sparkle](Pictures/027.png)


After setting up ip addresses, click the button **"Save"**

![sparkle](Pictures/028.png)

### File format creation process

The stored procedure will use the function [OPENROWSET](https://docs.microsoft.com/en-us/sql/t-sql/functions/openrowset-transact-sql?view-sql-server-ver15). And because we want to retrieve the information from the file in order to do data operations, we need to define a [format file](https://docs.microsoft.com/en-us/sql/t-sql/functions/openrowset-transact-sql?view-sql-server-ver15)..

The format of the files we will process for this example is very simple. It consists of 3 columns:

- Name
- First Name
- Sale

The creation of the format file will be done in 3 steps

- Creating a SQL table that matches the file format
- Using the BCP tool to create the format file
- Download the file to the storage account


#### SQL Table creation

With [Azure Data Studio](https://docs.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio?view-sql-server-ver15), connect to your Azure SQL database, and then create a new SQL file with a combination of **"Ctrl"** and **"N"** keys.


![sparkle](Pictures/029.png)


Then copy the script below. Click on the **"Play"** button



```Javascript
CREATE TABLE [dbo].[MyFirstImport](
	[LastName] [varchar](30) NULL,
	[FirstName] [varchar](25) NULL,
	[Sales] [int] NULL
) ON [PRIMARY]
GO
```


![sparkle](Pictures/030.png)

If all goes well you should have the following message and access to your table via the menu on the left


![sparkle](Pictures/031.png)

#### Création du fichier de format

Make sure you have the latest version of the BCP tool. For this example, I used [version 15](https://docs.microsoft.com/en-us/sql/tools/bcp-utility?view-sql-server-ver15).

To make sure you're using the right version of the BCP tool, go to the installation directory. In my case the directory is:

C:\Program Files\Microsoft SQL Server\Client SDK\ODBC\170\Tools\Binn

Then use the following command (I have a **"Temp"** directory on my C drive)

```javascript

bcp dbo.MyFirstImport format nul -c -x -f C:\Temp\format.xml -t, -U <Your User> -S tcp:<Your Server Name>.database.windows.net -d <Your Database name> -P <Your Password>

```

See below:
![sparkle](Pictures/032.png)


You need to get the format file in the specified directory with the BCP command

![sparkle](Pictures/033.png)


#### Download the format file in the Azure storage account

From the Azure portal, go to your storage account


![sparkle](Pictures/034.png)

Click **"Containers"**

![sparkle](Pictures/035.png)

And click **"+ Container"**


![sparkle](Pictures/036.png)

Give a name and click **"Create"**

![sparkle](Pictures/037.png)

We will create a directory for our format file.

Click on **"Storage Explorer (preview),"** select the container created previously, and then click **"New Folder"**

![sparkle](Pictures/038.png)

Name the directory and click the button **"Ok"**

![sparkle](Pictures/039.png)

Click the **"Upload"** button and download the format file previously created with the BCP function.

![sparkle](Pictures/040.png)

The file is ready

![sparkle](Pictures/041.png)

### Azure File setup

From the Azure portal, select your storage account and click **"File shares"**

![sparkle](Pictures/0411.png)

Click **"+ File share"**

![sparkle](Pictures/0412.png)

Give your file sharing a name and then click the button **"Create"**

![sparkle](Pictures/0413.png)

You should get a result similar to the screen copy below:

![sparkle](Pictures/0414.png)


## Stored procedure

Our stored procedure will read files that are in our storage account and perform operations on the data it will recover.

Preliminary steps are therefore needed to allow the stored procedure to access the storage account.

- Creating a shared access signature (storage account) [(Documentation)] (https://docs.microsoft.com/en-us/azure/storage/common storage-sas-overview)

- Creating a database master key [(Documentation)](https://docs.microsoft.com/en-us/sql/t-sql/statements/create-master-key-transact-sql?view-sql-server-ver15)

- Creating credentials to access the storage account [(Documentation)](https://docs.microsoft.com/en-us/sql/t-sql/statements/create-database-scoped-credential-transact-sql?view-sql-server-ver15)

- Creating an external source [(Documentation)]) (https://docs.microsoft.com/en-us/sql/t-sql/statements/create-external-data-source-transact-sql?view-sql-server-ver15)



### Creating a shared access signature (Storage Account)

From the Azure portal, go to your storage account and click **"Shared Access Signature."**

Set the options for the shared access signature and then click the **"Generate SAS and connection string"** button.


![sparkle](Pictures/042.png)


Copy the contents of the field **"SAS Token"** and then keep it on hand, we'll need it a little bit later.

![sparkle](Pictures/043.png)


### Database master key creation (Azure SQL)

From Azure Data Studio, copy the query below:

```javascript
CREATE MASTER KEY ENCRYPTION BY PASSWORD='<EnterStrongPasswordHere>';

```

Then click the button **Run**

![sparkle](Pictures/044.png)


### Creating credentials to access the storage account

From Azure Data Studio, run the script below:

**"WARNING!!!! "** Remove the sign **"?"** After copying your shared access signature.

```javascript
CREATE DATABASE SCOPED CREDENTIAL AccessAzureStorage
WITH
  IDENTITY = 'SHARED ACCESS SIGNATURE',
  -- Remove ? from the beginning of the SAS token
  SECRET = '<YOUR SHARED ACCESS SIGNATURE>' ;

```

For clarity, here's a screenshot

![sparkle](Pictures/045.png)

### Creating credentials to access the storage account

From Azure Data Studio, run the script below:

```javascript
CREATE EXTERNAL DATA SOURCE AzureStorageExternalData
WITH
  ( LOCATION = '<YOUR LOCATION>' ,
    CREDENTIAL = AccessAzureStorage ,
    TYPE = BLOB_STORAGE
  ) ;

```

Replace <YOUR LOCATION> with your container path. This information can be found in the Azure portal, in the container's properties

![sparkle](Pictures/046.png)

Below is a screenshot in Azure Data Studio:

![sparkle](Pictures/047.png)

### Creating the stored procedure

In Azure Data Studio, copy the script below:

```javascript
CREATE PROCEDURE Franmer
       @MyFileName nvarchar(MAX)
AS
BEGIN
       declare @query nvarchar(MAX)
       set @query = 'Select LastName, sum(Sales) as TotalSales FROM OPENROWSET(BULK ''' + @MyFileName + ''', 
       DATA_SOURCE = ''AzureStorageExternalData'',
       FORMAT=''CSV'',
       FORMATFILE=''Format/format.xml'',
       FORMATFILE_DATA_SOURCE = ''AzureStorageExternalData'') as products
       GROUP BY LastName;'
 
       EXEC sp_executesql @query
 END

```


![sparkle](Pictures/048.png)

It is possible to test the stored procedure by downloading the sample file (which is in the [github](https://github.com/franmer2/ADFandStoredProcedProcedure/blob/master/Resources/test.csv)) at the root of the container.


![sparkle](Pictures/049.png)

Then in Azure Data Studio, enter the script below:

```Javascript
EXECUTE franmer @MyFileName='test.csv'
```

If everything goes according to plan, you should get the following result:

![sparkle](Pictures/050.png)

## Azure Data Factory Pipeline

From the Azure portal, find your Azure Data Factory service, then click **"Author & Monitor"**

![sparkle](Pictures/051.png)

## Linked services création
### Azure Blob storage linked service

Once on the Azure Data Factory homepage, click the **"Manage"** button to the left of the screen


![sparkle](Pictures/052.png)


Click on **"Linked services"** and then on the **"New"** button  

![sparkle](Pictures/053.png)

In the list of related services, select **"Azure Blob Storage"**

![sparkle](Pictures/054.png)

Name the linked service, select the storage account and then test the connection by clicking on **"Test connection"** (1). Once the test is successful, click the **"Create"** button (2).

![sparkle](Pictures/055.png)

### Azure File Storage linked service

Create a new  **"Azure File Storage"** linked service

![sparkle](Pictures/056.png)

Then complete the login information

![sparkle](Pictures/057.png)

### Azure SQL Database linked service

Finally, create an **"Azure SQL Database"** linked service

![sparkle](Pictures/058.png)

Then complete the login information

![sparkle](Pictures/059.png)

You should have 3 linked services

![sparkle](Pictures/060.png)

## Pipeline creation

Below is a global view of the pipeline we're going to create 


![sparkle](Pictures/061.png)


### Datasets
#### Blob Storage Dataset

We'll start by creating our "Datasets" to access our stored procedure, our "blob storage" and our "file storage"

In the Azure Data Factory console on the left, click the **"Dataset"** button

![sparkle](Pictures/062.png)

Choose an **"Azure Blob Storage"** Dataset and click **"Continue"**

![sparkle](Pictures/063.png)

Choose the format **"DelimitedText"** and then click the **"Continue"** button.
(You can't choose the binary type because the source must also be of the binary type. Now the copying activity that we will do in our pipeline will have a procedure stored as a source)

![sparkle](Pictures/064.png)

Enter your blob storage information and click the button **"OK"**

![sparkle](Pictures/065.png)

Once the Dataset is created, click **"Parameters"** and then the **"New"** button. Give the setting a name. Here I'm going to name my setting "FileName"

![sparkle](Pictures/066.png)

Click on the **"Connection"** tab, then in the field **"File"**. Then click on the link **"Add dynamic content"**


![sparkle](Pictures/067.png)

The **"Add dynamic content"** component appears. Add the following expression:


```Javascript
 @dataset().FileName 
```

 Click the **"Finish"** button

![sparkle](Pictures/068.png)

 Our first Dataset is ready. Click the **"Publish all"** button and publish the dataset


![sparkle](Pictures/069.png)


#### Azure File Storage Dataset

In the Azure Data Factory console, on the left, click **"+"** and then click **"Dataset"**

![sparkle](Pictures/062.png)

Choose **"Azure File Storage"** Dataset and then click the **"Continue"** button.

![sparkle](Pictures/070.png)

Choose the format **"DelimitedText"** and then click the **"Continue"** button.

![sparkle](Pictures/064.png)

Entrez les informations de votre stockage *"file"* puis cliquez sur le bouton **"OK"**

![sparkle](Pictures/071.png)

Une fois le *"Dataset"* créé, cliquez sur **"Parameters"**, puis sur le bouton **"+ New"** 3 fois afin de créer 3 paramètres. Donnez un nom aux paramètres 

![sparkle](Pictures/072.png)

Cliquez sur l'onglet **"Connection"**, puis dans le champ **"File"**. Cliquez ensuite sur le lien **"Add dynamic content"**

![sparkle](Pictures/073.png)

Le volet **"Add dynamic content"** apparaît. rajoutez l'expression

```Javascript
 @concat(dataset().Prefix,'-',dataset().Date,'-',dataset().Name,'.csv')
 ```
 
 Cliquez sur le bouton **"Finish"**

![sparkle](Pictures/074.png)

Cliquez sur le bouton **"Publish all"**

![sparkle](Pictures/075.png)

#### Création du "Dataset" Azure SQL Database

Dans la console Azure Data Factory, sur la gauche, cliquez sur le bouton **"+"** puis sur **"Dataset"**

![sparkle](Pictures/062.png)

Choisissez un *"Dataset"* de type **"Azure SQL Database"** puis cliquez sur le bouton **"Continue"**

![sparkle](Pictures/076.png)

Renseignez les données concerant Azure SQL puis cliquez sur le bouton **"OK"**

![sparkle](Pictures/077.png)

Cliquez sur le bouton **"Publish"**


![sparkle](Pictures/078.png)

### Création des activités du pipeline

Depuis le portail d'Azure Data Factory, cliquez sur le bouton **"+"** puis sur **"Pipeline"**

![sparkle](Pictures/079.png)

Au niveau de votre pipeline, créez un nouveau paramètre. Dans l'onglet **"Parameters"**. Cliquez sur le bouton **"+ New"** puis puis donnez un nom au paramètre

![sparkle](Pictures/080.png)


Cliquez ensuite sur l'onglet **"Variables"** afin de rajouter une variable pour capturer la date. Cliquez sur le bouton **"+ New"** puis puis donnez un nom à la variable

![sparkle](Pictures/081.png)



Comme nous avons une contrainte fonctionnelle au niveau du nom du fichier, qui doit comporter la date dans son nom, nous devons la capturer au début du pipeline afin de s'assurer d'avoir la même valeur durant l'exécution du pipeline.


De plus, comme la destination de la copie est un *"File storage"*, il va falloir formater la date afin de ne conserver que des caractères supportés par le *"File storage"*.

Rajoutez une activité **"Set variable"** dans votre pipeline, qui se trouve dans la rubrique **"General"**. Puis cliquez sur l'onglet **"Variables"**. Dans la liste déroulante **"Name"**, choisissez la variable **"Date"**, cliquez dans le champ **"Value"**, puis cliquez sur **"Add dynamic content"**

![sparkle](Pictures/082.png)

Dans le volet **"Add dynamic content"**, rajoutez la fonction 

```Javascript
@formatDateTime(utcnow(),'yyyy-MM-ddTHH-mm-ss')
```


Cliquez sur le bouton **"Finish"**

![sparkle](Pictures/083.png)


Depuis le volet **"Activities"**, dans la rubrique **"Move and transform"**, rajoutez l'activité **"Copy data"** dans votre pipeline juste après l'activité **"Set variable 1"** 

![sparkle](Pictures/084.png)

Cliquez sur l'onglet **"Source"**. Dans la liste déroulante **"Source Dataset"**, sélectionnez votre *"Dataset"* Azure SQL. Puis cliquez sur **"Stored pocedure"**, et sélectionnez votre procédure stockée dans la liste déroulante **"Name"**.

Cliquez ensuite sur le bouton **"Import parameter"** afin de récupérer le paramètre de la procédure stockée. Cliquez dans le champ **"Value"** pour faire apparaître le lien **"Add dynanic content"**. Cliquez sur ce lien.

![sparkle](Pictures/085.png)

Dans le volet **"Add dynanic content"**, rajoutez l'expression 

```Javascript
@pipeline().parameters.FileName
```


Cliquez sur le bouton **"Finish"**.

![sparkle](Pictures/086.png)


Cliquez sur l'onglet **"Sink"**. Dans la liste déroulante **"Sink dataset"**, sélectionnez le dataset du *"File storage"*. Pour chacun des paramètres, rajoutez les valeurs suivantes, à chaque fois en cliquant sur le lien **"Add dynamic content"**

Pour le paramètre **"Prefix"** rajoutez l'expression

```Javascript
@pipeline().parameters.FileName
```

Pour le paramètre **"Date"** rajoutez l'expression

```Javascript
@variables('Date')
```

![sparkle](Pictures/087.png)





Pour le paramètre **"Name"** rajoutez l'expression que vous souhaitez. Ici pour illustrer cet exemple je vais tout simplement mettre la valeur **MonParamètre**.

![sparkle](Pictures/088.png)


Voici ce que va donner la partie **"Sink"** de l'activité de copie

![sparkle](Pictures/089.png)

Nous allons enfin rajouter une activité **"Delete"** afin de le *"blob storage"* après l'exécution du pipeline.

Depuis le volet **"Activities"**, dans la rubrique **"General"**, rajoutez l'activité **"Delete"** dans votre pipeline

![sparkle](Pictures/090.png)

Cliquez sur l'onglet **"Source"**. Sélectionnez le *"dataset"* correspondant au *"blob storage"*. Cliquez dans le champ **"value"** correspondant au paramètre puis cliquez sur **"Add dynamic content"**.

![sparkle](Pictures/091.png)

Dans le volet **"Add dynamic content"**, rajoutez l'expression

```Javascript
@pipeline().parameters.FileName
```

Puis cliquez sur le bouton **"Finish"**

![sparkle](Pictures/092.png)

Cliquez sur l'onglet **"Logging settings"** pour désactiver le logging en décochant la case **"Enable logging"**. On ne va pas en avoir besoin pour cet exemple.

![sparkle](Pictures/093.png)

Cliquez sur le bouton **"publish"**

![sparkle](Pictures/094.png)

#### ajout d'un déclencheur de type évènement


Nous allons maintenant rajouter un déclencheur pour que le pipeline se déclenche dès qu'un nouveau fichier arrive.

Cliquez sur **"Add trigger"** puis sur **"New/Edit"**

![sparkle](Pictures/095.png)

dans la fenêtre **"Add trigger"**, dans la liste déroulante, cliquez sur **"+ New"**

![sparkle](Pictures/096.png)

Dans la rubrique **"Type"** sélectionnez **"Event"**. Puis donnez les informations de connexion à votre compte de stockage. Comme nous allons surveiller l'arrivée des fichiers à la racine du conteneur, nous allons laisser le champ **"Blob path begins with"** vide.

=======================================

**ATTENTION !!!** Le déclencheur va surveiller l'ensemble du conteneur, ce qui veut dire que si un fichier csv arrive dans un sous dossier, le pipeline sera déclenché.
Dans le cas où vous souhaitez surveiller un dossier bien particulier, et ainsi isoler le traitement des fichiers dans une zone précise de votre conteneur, vous pouvez tout simplement mettre le nom du répertoire à surveiller). 

![sparkle](Pictures/200.png)

=======================================


Dans le chanp **"Blob path ends with"**, nous allons indiquer l'extension des fichiers que l'on souhaite traiter. Ici on indiquera **".csv"**

Dans la rubrique **"Event"**, cliquez sur **"Blob created"**. Cliquez sur le bouton **"Continue"**

![sparkle](Pictures/097.png)

Si vous avez déjà des fichiers csv dans votre compte de stockage, ils devraient être affichés dans cette fenêtre. C'est aussi un bon moyen de vérifier la syntaxe utilisé dans les champs **"Blob path begins with"** et **"Blob path ends with"** du volet précédent. Cliquez sur le bouton **"Continue"**.

![sparkle](Pictures/098.png)

Dans le volet **"Trigger run paramètre"**, il est possible de définir une valeur au paramètre de notre pipeline. C'est ce que nous allons faire avec l'expression suivante :

```Javascript
@trigger().outputs.body.fileName
```

Cliquez sur le bouton **OK"**

![sparkle](Pictures/099.png)

Puis publiez le pipeline en cliquant sur le bouton **"Publish all"**

![sparkle](Pictures/100.png)

### Test du pipeline

Téléchargez le fichier d'exemple dans votre conteneur Blob. Par exemple avec Azure Storage Explorer

![sparkle](Pictures/101.png)

Puis ensuite, aller dans votre stockage *"file storage"* pour vérifier si un fichier est présent avec la bonne nomenclature au niveau de son nom


![sparkle](Pictures/102.png)

Vérifiez aussi si le fichier à bien été effacé du stockage blob en fin d'exécution du pipleine

![sparkle](Pictures/103.png)


Du côté du portail Azure Data Factory, vous pouvez monitorer la bonne exécution du déclencheur et du pipeline en allant dans **"Monitor"** puis **"Trigger runs"** ou **"Pipeline runs"**

Ci dessous un exemple de monitoring du déclencheur


![sparkle](Pictures/104.png)



Si vous devez faire des tests de votre pipeline sans utilser le déclencheur, il est possible de l'arréter en allant dans **"Manage"**, **"Triggers"** puis en cliquant sur **"Deacticate"**

![sparkle](Pictures/105.png)