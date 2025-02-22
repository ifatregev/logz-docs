---
layout: article
title: Setting Azure Blob Storage permissions
image: https://dytvr9ot2sszz.cloudfront.net/logz-docs/social-assets/docs-social.jpg
description: Azure permissions for log archiving
permalink: /user-guide/archive-and-restore/azure-blob-permissions/
tags:
  - Azure Blob Storage
  - archive-and-restore
contributors:
  - shalper
  - hidan
---

<!-- tabContainer:start
<div class="branching-container">

* [Archive to Blob](#permissions)
* [Limit access to trusted networks](#limits)
{:.branching-tabs}
-->
<!-- tab:start 
<div id="permissions"> -->

You can archive your logs for long-term storage by sending them to a Microsoft Azure Storage container.

On this page: 

* toc list
{:toc}

### Minimal permissions
{:.no_toc}

Logz.io will need the following:

* **Storage Blob Data Contributor** permissions to archive data to a Microsoft Azure Storage account.
* **Storage Blob Data Reader** permissions to restore data from a Microsoft Azure Storage account.


#### Setting up a Storage container and App registration {#grant-access-to-azure-storage}

**Before you begin, you'll need**: Permission to manage a Storage container and App registration in Microsoft Azure.

<div class="tasklist">

##### Create an App registration
{:.no_toc}

Open your [Azure Portal](https://portal.azure.com/).
Select **Azure Active Directory** > **App registrations** from the left-menu.

![Create Azure Storage account](https://dytvr9ot2sszz.cloudfront.net/logz-docs/archive-azure/azure-app-registration.png)

If you have an existing App registration you can use, select it. Otherwise create a new one.

Click **+ New registration** to create an **App registration**. Name it, leave the default settings and click **Register**.

![Create new App registration](https://dytvr9ot2sszz.cloudfront.net/logz-docs/archive-azure/azure-new-app.png)

##### Copy the App registration parameters
{:.no_toc}

The App **Overview** page provides 2 of the credentials required to fill-in the form in Logz.io: **Application (client) ID** & **Directory (tenant) ID**.

Copy them for future reference.

![Create Azure Storage account](https://dytvr9ot2sszz.cloudfront.net/logz-docs/archive-azure/azure-app-id.png)

##### Create & copy the Client secret password
{:.no_toc}

On the same **App registration** page, select **Certificates & secrets** from the left-menu.
Click **+ New client secret** to create a new one. Select a time frame for its expiration, add a description, and click **Add**.

Copy the secret for future reference. (Note that the password value will not be available once you leave the page.)

![Create Azure App Client secret](https://dytvr9ot2sszz.cloudfront.net/logz-docs/archive-azure/azure-certificates-secrets.png)

If the secret is set to expire, you will need to remember to renew the credentials and reconfigure archiving in Logz.io!
{:.info-box.note}

##### Create a Storage account
{:.no_toc}

Click the **main menu <i class="fas fa-bars"></i>** in the top-left corner, and select **Storage account**.

If you have an existing Storage account you can use, select it. Otherwise create a new one.

Click **+ Create** to create a new account.

![Create Azure Storage account](https://dytvr9ot2sszz.cloudfront.net/logz-docs/archive-azure/create-azure-storage-account.png)

##### Create a Storage container
{:.no_toc}

In the Storage account, create a storage container (or select an existing one).

![Create Azure Storage container](https://dytvr9ot2sszz.cloudfront.net/logz-docs/archive-azure/azure-container.png)

##### Assign App & role to your Storage container
{:.no_toc}

Still on the Storage container page, select **Access Control (IAM)** from the left-menu.

![Assign App & Role to your Storage container](https://dytvr9ot2sszz.cloudfront.net/logz-docs/archive-azure/azure-container-access.png)

Select **Add role assignments**.

![Add role assignment to your Storage container](https://dytvr9ot2sszz.cloudfront.net/logz-docs/archive-azure/azure-container-add-role-assignment.png)

Fill in the form:

* **Role** - Select **Storage Blob Data Contributor**.
* **Assign access to** - Leave the defaults unchanged. They should be **User, group, or service principal**
* **Select** - Start typing in the name of the app and select it from the dropdown list.
* Click **save**.

![Add role assignment to your Storage container](https://dytvr9ot2sszz.cloudfront.net/logz-docs/archive-azure/azure-container-add-role-assignment1.png)

##### Configure Logz.io Archive & Restore
{:.no_toc}

Open your [Logz.io app](https://app.logz.io/#/dashboard/tools/archive-and-restore).

In the **Archive configuration** tab, select the **Azure** tab, and fill in the form with the credentials you created and copied in the previous steps.

![Configure Logz.io connection to Azure](https://dytvr9ot2sszz.cloudfront.net/logz-docs/archive-azure/archive-to-azure.png)

</div>

#### Rehydrate Azure Blob Archive

If you're using Azure Blob Archive tier, there is an additional step you need to take before you can search or restore it to Logz.io.

Azure Blob Archive tier is an offline tier mainly used to store data you rarely need access to. If you want to read or modify its data, you will need to rehydrate the blob to an online tier, and set it to either Hot or Cool.

To access data stored in the archive tier, you'll need to rehydrate it through one of the following options:

* Copy an archived blob to an online tier - Use the **Copy Blob** option to copy it to a Hot or Cool tier. This is Microsoft's recommended option for most scenarios.
* Change an archived blob's access tier to an online tier - Use the **Set Blob** option to change the tier to Hot or Cool, which will rehydrate an archived blob. This option usually takes a few hours to complete.

Read more about rehydrating a blob in Microsoft's [**Blob rehydration from the Archive tier**](https://learn.microsoft.com/en-us/azure/storage/blobs/archive-rehydrate-overview) guide.


<!--</div>-->
<!-- tab:end -->
<!-- tab:start -->
<!-- <div id="limits"> -->


#### Limiting access to trusted networks

To make sure that your Azure Blob storage cannot be accessed by malicious actors who get hold of the access credentials, you can configure the Azure Blob account to only allow logins from trusted networks. Every storage account in Azure has Security rules, which define the access permissions. By default, the storage account is accessible by any network as long as the user has the access credentials.

**Before you begin, you'll need**: Azure CLI installed

To limit the access to trusted networks, add the network rules as follows:


```shell
az storage account network-rule add --subnet /subscriptions/ac7ee52c-3b51-43b5-b667-2498be58418b/resourceGroups/logzio-<REGION>-prod/providers/Microsoft.Network/virtualNetworks/logzio-westeurope-prod-vnet/subnets/logzio-westeurope-prod-vnet-subnet-archivers --account-name <CUSTOMER_STORAGE_ACCOUNT_NAME>

az storage account network-rule add --subnet /subscriptions/ac7ee52c-3b51-43b5-b667-2498be58418b/resourceGroups/logzio-<REGION>-prod/providers/Microsoft.Network/virtualNetworks/logzio-westeurope-prod-vnet/subnets/logzio-westeurope-prod-vnet-subnet-default --account-name <CUSTOMER_STORAGE_ACCOUNT_NAME>

az storage account network-rule add --subnet /subscriptions/ac7ee52c-3b51-43b5-b667-2498be58418b/resourceGroups/logzio-<REGION>-prod/providers/Microsoft.Network/virtualNetworks/logzio-westeurope-prod-vnet/subnets/logzio-westeurope-prod-vnet-subnet-aks-services --account-name <CUSTOMER_STORAGE_ACCOUNT_NAME>
```

Replace `<REGION>` with the region of your account and `<CUSTOMER_STORAGE_ACCOUNT_NAME>` with the name of your storage account.

<!--
</div> -->
<!-- tab:end -->

<!-- </div> -->
<!-- tabContainer:end -->

