---
solution: Campaign Classic
product: campaign
title: Importing data
description: Learn how to import data in Adobe Campaign
audience: workflow
content-type: reference
topic-tags: -general-operation
---

# Importing data{#importing-data}

>[!CAUTION]
>
>Please keep in mind the SFTP storage, Database Storage and Active profile limits as per your Adobe Campaign contract while importing data.

## How to collect data {#how-to-collect-data}

### Using data from a list: Read list {#using-data-from-a-list--read-list}

The data sent in a workflow can come from lists whereby the data has been prepared and structured beforehand.

This list may have been directly created in Adobe Campaign or imported by the **[!UICONTROL Import a list]** option. For more on this option, refer to this [page](../../platform/using/generic-imports-and-exports.md).

For more on using the read list activity in a workflow, refer to [Read list](../../workflow/using/read-list.md).

### Loading data from a file {#loading-data-from-a-file}

The data processed in a workflow can be extracted from a structured file so that it can be imported into Adobe Campaign.

A description of the loading data activity can be found in the [Data loading (file)](../../workflow/using/data-loading--file-.md) section.

Example of structured file to import:

```
lastname;firstname;birthdate;email;crmID
Smith;Hayden;23/05/1989;hayden.smith@example.com;124365
Mars;Daniel;17/11/1987;dannymars@example.com;123545
Smith;Clara;08/02/1989;hayden.smith@example.com;124567
Durance;Allison;15/12/1978;allison.durance@example.com;120987
```

## Best practices when importing data {#best-practices-when-importing-data}

Being cautious and following the few simple rules detailed below will help a lot in ensuring data consistency within the database and in avoiding common errors during database update or data exports.

### Using import templates {#using-import-templates}

Most import workflows should contain the following activities: **[!UICONTROL Data loading (file)]**, **[!UICONTROL Enrichment]**, **[!UICONTROL Split]**, **[!UICONTROL Deduplication]**, **[!UICONTROL Update data]**.

Using import templates makes it very convenient to prepare similar imports and ensure data consistency within the database. Learn how to build workflow templates in the [Workflow templates](../../workflow/using/building-a-workflow.md#workflow-templates) section.

In many projects, imports are built without **[!UICONTROL Deduplication]** activity because the files used in the project do not have duplicates. Duplicates sometimes appear from importing different files. De-duplication is then difficult. Therefore a deduplication step is a good precaution in all import workflows.

Do not rest on assumption that the incoming data is consistent and correct, or that the IT department or Adobe Campaign supervisor will take care of it. During the project, keep the data cleansing in mind. Deduplicate, reconcile, and maintain consistency when you import data.

An import template example is available in the [Setting up a recurring import](#setting-up-a-recurring-import) section.

### Using flat file formats {#using-flat-file-formats}

The most efficient format for imports is flat files. Flat files can be imported in bulk mode at the database level.

For example:

* Separator: tab or semicolon
* First line with headers
* No string delimiter
* Date format: YYYY/MM/DD HH:mm:SS

Adobe Campaign cannot import XML files using standard file import activities. It is possible to import XML files using JavaScript but only with small volumes: less than 10K records per file.

### Using compression and encryption {#using-compression-and-encryption}

Use zipped files for imports and exports when possible.

On Linux, it is possible to unzip a file and import at the same time using a command line. For example:

```
zcat nl6/var/vp/import/filename.gz
```

It is also a good practice to encrypt files that are sent across the network if it is unsecure. GPG can be used for this.

### Loading data in batch from files {#loading-data-in-batch-from-files}

Loading data in batch from a file is more effective than loading one line at a time and in real-time (for example via a Web service).

Imports using Web services are not efficient. It is best to use files whenever possible.

Calling external Web services to enrich profiles in real time is also known to cause performance problems and memory leaks, because it works at line level.

If you need to import data, it is better to do it in batch, using a workflow, than in real time, using a Web application or a Web service.

### Using Data Management {#using-data-management}

Loading in iterative mode (line by line) using JavaScript should be limited to small volumes.

For better efficiency, always use the **[!UICONTROL Data Loading (File)]** activity in data management workflows.

### Importing in Delta mode {#importing-in-delta-mode}

Regular Imports must be done in delta mode. It means that only modified or new data is sent to Adobe Campaign, instead of the whole table every time.

Full imports should be used for initial load only.

Import data using data management rather than JavaScript.

### Maintaining consistency {#maintaining-consistency}

To maintain data consistency in the Adobe Campaign database, follow the principles below:

* If the imported data matches a reference table in Adobe Campaign, then it should be reconciled with that table in the workflow. Records that do not match should be rejected.
* Ensure that the imported data is always **"normalized"** (email, phone number, direct mail address) and that this normalization is reliable and will not change over the years. If this is not the case, some duplicates are likely to appear in the database, and as Adobe Campaign does not provide tools to do "fuzzy" matching, it will be very difficult to manage and remove them.
* Transactional data should have a reconciliation key and be reconciled with the existing data in order to avoid creating duplicates.
* **Import related files in order**.

  If the import is composed of multiple files that depend on each other, the workflow should make sure that the files are imported in the correct order. When a file fails, the other files are not imported.

* **Deduplicate**, reconcile, and maintain consistency when you import data.

## Use case: setting up a recurring import {#setting-up-a-recurring-import}

Using an import template is a best practice if you need to regularly import files with the same structure.

This example shows how to pre-set a workflow that can be reused for importing profiles coming from a CRM in the Adobe Campaign database. For more information about all possible settings for each activity, refer to this [section](../../workflow/using/about-activities.md).

1. Create a new workflow template from **[!UICONTROL Resources > Templates > Workflow templates]**.
1. Add the following activities:

    * **[!UICONTROL Data loading (file)]**: Define the expected structure of the file containing the data to import.
    * **[!UICONTROL Enrichment]**: Reconcile the imported data with database data.
    * **[!UICONTROL Split]**: Create filters to process records differently depending on whether they could be reconciled or not.
    * **[!UICONTROL Deduplication]**: Deduplicate the data from the incoming file before it is inserted in the database.
    * **[!UICONTROL Update data]**: Update the database with the imported profiles.

   ![](assets/import_template_example0.png)

1. Configure the **[!UICONTROL Data Loading (file)]** activity:

    * Define the expected structure by uploading a sample file. The sample file should contain only a few lines but all the columns necessary for the import. Check and edit the file format to make sure that the type of each column is set correctly: text, date, integer, etc. For example:

      ```    
      lastname;firstname;birthdate;email;crmID
      Smith;Hayden;23/05/1989;hayden.smith@mailtest.com;123456
      ```

    * In the **[!UICONTROL Name of the file to load]** section, select **[!UICONTROL Upload a file from the local machine]** and leave the field blank. Each time a new workflow is created from this template, you can specify here the file you want, as long at it corresponds to the defined structure.

      You can use any of the options but you have to modify the template accordingly. For example, if you select **[!UICONTROL Specified in the transition]**, you can add a **[!UICONTROL File Transfer]** activity before to retrieve the file to import from a FTP/SFTP server. With S3 or SFTP connection, you can also import segment data to Adobe Campaign with Adobe Real-time Customer Data platform. For more on this, refer to this [documentation](https://docs.adobe.com/content/help/en/experience-platform/rtcdp/destinations/destinations-cat/adobe-destinations/adobe-campaign-destination.html).
    
      ![](assets/import_template_example1.png)

1. Configure the **[!UICONTROL Enrichment]** activity. The purpose of this activity in this context is to identify the incoming data.

    * In the **[!UICONTROL Enrichment]** tab, select **[!UICONTROL Add data]** and define a link between the imported data and the recipients targeting dimension. In this example, the **CRM ID** custom field is used to create the join condition. Use the field or combination of fields you need as long it allows to identify unique records.
    * In the **[!UICONTROL Reconciliation]** tab, leave the **[!UICONTROL Identify the document from the working data]** option unchecked.

   ![](assets/import_template_example2.png)

1. Configure the **[!UICONTROL Split]** activity to retrieve reconciled recipients in one transition and recipients that could not be reconciled but who have enough data in a second transition.

   The transition with reconciled recipients can then be used to update the database. The transition with unknown recipients can then be used to create new recipient entries in the database if a minimum set of information is available in the file.

   Recipients that cannot be reconciled and do not have enough data are selected in a complement outbound transition and can be exported in a separate file or simply ignored.

    * In the **[!UICONTROL General]** tab of the activity, select **[!UICONTROL Use the additional data only]** as filtering setting and make sure that the **[!UICONTROL Targeting dimension]** is automatically set to **[!UICONTROL Enrichment]**.

      Check the **[!UICONTROL Generate complement]** option to be able to see if any record cannot be inserted in the database. If you need, you can then apply further processing to the complementary data: file export, list update, etc.
    
    * In the first subset of the **[!UICONTROL Subsets]** tab, add a filtering condition on the inbound population to select only records for which the recipient primary key is not equal to 0. This way, data from the file that are reconciled with recipients from the database are selected in that subset.
    
      ![](assets/import_template_example3.png)

    * Add a second subset that selects unreconciled records that have enough data to be inserted in the database. For example: email address, first name and last name.

      Subsets are processed in their creation order, meaning that when this second subset is processed, all records that already exist in the database are already selected in the first subset.
    
      ![](assets/import_template_example3_2.png)

    * All records that are not selected in the first two subsets are selected in the **[!UICONTROL Complement]**.

1. Configure the **[!UICONTROL Update data]** activity located after the first outbound transition of the **[!UICONTROL Split]** activity configured previously.

    * Select **[!UICONTROL Update]** as **[!UICONTROL Operation type]** since the inbound transition only contains recipients already present in the database.
    * In the **[!UICONTROL Record identification]** section, select **[!UICONTROL Using reconciliation keys]** and define a key between the targeting dimension and the link created in the **[!UICONTROL Enrichment]**. In this example, the **CRM ID** custom field is used.
    * In the **[!UICONTROL Fields to update]** section, indicate the fields from the recipients dimension to update with the value of the corresponding column from the file. If the names of the file columns are identical or almost identical to the names of the recipients dimension fields, you can use the magic wand button to automatically match the different fields.
    
      ![](assets/import_template_example6.png)

1. Configure the **[!UICONTROL Deduplication]** activity located after the transition containing unreconciled recipients:

    * Select **[!UICONTROL Edit configuration]** and set the targeting dimension to the temporary schema generated from the **[!UICONTROL Enrichment]** activity of the workflow.
    
      ![](assets/import_template_example4.png)

    * In this is example, the email field is used to find unique profiles. You can use any field you are sure is filled and part of a unique combination.
    * In the **[!UICONTROL Deduplication method]** screen, select **[!UICONTROL Advanced parameters]** and check the **[!UICONTROL Disable automatic filtering of 0 ID records]** option to make sure records that have a primary key equal to 0 (which should be all records of this transition) are not excluded.

   ![](assets/import_template_example7.png)

1. Configure the **[!UICONTROL Update data]** activity located after the **[!UICONTROL Deduplication]** activity configured previously.

    * Select **[!UICONTROL Insert]** as **[!UICONTROL Operation type]** since the inbound transition only contains recipients not present in the database.
    * In the **[!UICONTROL Record identification]** section, select **[!UICONTROL Directly using the targeting dimension]** and choose the **[!UICONTROL Recipients]** dimension.
    * In the **[!UICONTROL Fields to update]** section, indicate the fields from the recipients dimension to update with the value of the corresponding column from the file. If the names of the file columns are identical or almost identical to the names of the recipients dimension fields, you can use the magic wand button to automatically match the different fields.
    
      ![](assets/import_template_example8.png)

1. After the third transition of the **[!UICONTROL Split]** activity, add a **[!UICONTROL Data extraction (file)]** activity and a **[!UICONTROL File transfer]** activity if you want to keep track of data not inserted in the database. Configure those activities to export the column you need and to transfer the file on a FTP or SFTP server where you can retrieve it.
1. Add an **[!UICONTROL End]** activity and save the workflow template.

The template can now be used and is available for every new workflow. All is needed is then to specify the file containing the data to import in the **[!UICONTROL Data loading (file)]** activity.

![](assets/import_template_example9.png)

## Unzipping or decrypting a file before processing {#unzipping-or-decrypting-a-file-before-processing}

### About pre-processing stages {#about-pre-processing-stages}

Adobe Campaign lets you import zipped or encrypted files. Before they can be read in a [Data loading (file)](../../workflow/using/data-loading--file-.md) activity, you can define a pre-processing to unzip or to decrypt the file.

To be able to do so:

1. Use the [Control Panel](https://docs.adobe.com/content/help/en/control-panel/using/instances-settings/gpg-keys-management.html#decrypting-data) to generate a public/private key pair.

    >[!NOTE]
    >
    >Control Panel is available to all customers hosted on AWS (excepted for customers who host their marketing instances on premise).

1. If your installation of Adobe Campaign is hosted by Adobe, contact Adobe Customer Care to have the necessary utilities installed on the server.
1. If your installation of Adobe Campaign is on premise, install the utility you want to use (for example: GPG, GZIP) as well as the necessary keys (encryption key) on the application server.

You can then use the desired pre-processing commands into your workflows:

1. Add and configure a **[!UICONTROL File transfer]** activity in your workflow.
1. Add a **[!UICONTROL Data loading (file)]** activity and define the file format.
1. Check the **[!UICONTROL Pre-process the file]** option.
1. Specify the pre-processing command you want to apply.
1. Add other activities to manage data coming from the file.
1. Save and execute your workflow.

An example is presented in the use case below.

**Related topics:**

* [Data loading (file) activity](../../workflow/using/data-loading--file-.md).
* [Zipping or encrypting a file](../../workflow/using/how-to-use-workflow-data.md#zipping-or-encrypting-a-file).

### Use case: Importing data encrypted using a key generated by Control Panel {#use-case-gpg-decrypt}

In this use case, we will build a workflow in order to import data that has been encrypted in an external system, using a key generated in the Control Panel.

A tutorial video showing how to use a GPG key to decrypt data is also available in [this section](https://docs.adobe.com/content/help/en/campaign-classic-learn/tutorials/administrating/control-panel-acc/gpg-key-management/decrypting-data.html).

The steps to perform this use case are as follows:

1. Use the Control Panel to generate a key pair (public/private). Detailed steps are available in [Control Panel documentation](https://docs.adobe.com/content/help/en/control-panel/using/instances-settings/gpg-keys-management.html#decrypting-data).

    * The public key will be shared with the external system, which will use it to  encrypt the data to send to Campaign.
    * The private key will be used by Campaign Classic to decrypt the incoming encrypted data.

    ![](assets/gpg_generate.png)

1. In the external system, use the public key downloaded from the Control Panel to encrypt the data to import into Campaign Classic.

1. In Campaign Classic, build a workflow to import the encrypted data and decrypt it using the private key that has been installed via the Control Panel. To do this, we will build a workflow as follows:

     ![](assets/gpg_import_workflow.png)

    * **[!UICONTROL File transfer]** activity: Transfers the file from an external source to Campaign Classic. In this example, we want transfer the file from an SFTP server.
    * **[!UICONTROL Data loading (file)]** activity: Loads the data from the file into the database and decrypt it using the private key generated in the Control Panel.

1. Open the **[!UICONTROL File transfer]** activity then specify the external account from which you want to import the encrypted .gpg file.

     ![](assets/gpg_key_transfer.png)

     Global concepts on how to configure the activity are available in [this section](../../workflow/using/file-transfer.md).

1. Open the **[!UICONTROL Data loading (file)]** activity, then configure it according to your needs. Global concepts on how to configure the activity are available in [this section](../../workflow/using/data-loading--file-.md).

    Add a pre-processing stage to the activity, in order to decrypt the incoming data. To do this, select the **[!UICONTROL Pre-process the file]** option, then copy-paste this decryption command in the **[!UICONTROL Command]** field :

    `gpg --batch --passphrase passphrase --decrypt <%=vars.filename%>`

     ![](assets/gpg_load.png)

    >[!CAUTION]
    >
    >In this example, we are using the passphrase used by default by Control Panel, which is "passphrase".
    >
    >If you have already had GPG keys installed on your instance through a Customer Care request in the past, the passphrase may have been changed and be different form the one by default.

1. Click **[!UICONTROL OK]** to confirm the activity configuration.

1. You can now run the workflow. Once it is executed, you can check in the workflow logs that the decryption has been executed, and that data from the file have been imported.

    ![](assets/gpg_run.png)
