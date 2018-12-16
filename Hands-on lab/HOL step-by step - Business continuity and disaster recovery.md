![](https://github.com/Microsoft/MCW-Template-Cloud-Workshop/raw/master/Media/ms-cloud-workshop.png "Microsoft Cloud Workshops")

<div class="MCWHeader1">
Business continuity and disaster recovery
</div>

<div class="MCWHeader2">
Hands-on lab step-by-step
</div>

<div class="MCWHeader3">
September 2018
</div>

Information in this document, including URL and other Internet Web site references, is subject to change without notice. Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.

Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.

The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.
© 2018 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

**Contents**

<!-- TOC -->

- [Business continuity and disaster recovery hands-on lab step-by-step](#1-business-continuity-and-disaster-recovery-hands-on-lab-step-by-step)
    - [Abstract](#abstract)
    - [Overview](#overview)
    - [Solution architecture](#solution-architecture)
        - [Environment: On-premises (migrate to Azure)](#environment-on-premises-migrate-to-azure)
        - [Environment: Azure IaaS (failover region to region)](#environment-azure-iaas-failover-region-to-region)
        - [Environment: Azure PaaS (high-availably with seamless failover)](#environment-azure-paas-high-availably-with-seamless-failover)
    - [Requirements](#requirements)
    - [Exercise 1: Deploy Azure environments](#exercise-1-deploy-azure-environments)
        - [Task 1: Deploy Azure IaaS](#task-1-deploy-azure-iaas)
        - [Task 2: Deploy on-premises environment](#task-2-deploy-on-premises-environment)
        - [Task 3: Deploy Azure PaaS environment](#task-3-deploy-azure-paas-environment)
    - [Exercise 2: Configure BCDR services](#exercise-2-configure-bcdr-services)
        - [Task 1: Create Azure recovery services vault](#task-1-create-azure-recovery-services-vault)
        - [Task 2: Deploy Azure automation](#task-2-deploy-azure-automation)
    - [Exercise 3: Configure environments for failover](#exercise-3-configure-environments-for-failover)
        - [1.7.1. Task 1: Configure on-premises to Azure IaaS failover for migration](#task-1-configure-on-premises-to-azure-iaas-failover-for-migration)
        - [Task 2: Configure IaaS SQL Always On availability groups for region to region failover](#task-2-configure-iaas-sql-always-on-availability-groups-for-region-to-region-failover)
        - [Task 3: Configure IaaS for region to region failover](#task-3-configure-iaas-for-region-to-region-failover)
        - [Task 4: Configure PaaS for region to region failover](#task-4-configure-paas-for-region-to-region-failover)
    - [Exercise 4: Simulate failovers](#exercise-4-simulate-failovers)
        - [Task 1: Failover Azure IaaS region to region](#task-1-failover-azure-iaas-region-to-region)
        - [Task 2: Migrate the on-premises VM to Azure IaaS](#task-2-migrate-the-on-premises-vm-to-azure-iaas)
        - [Task 3: Failover and failback Azure PaaS](#task-3-failover-and-failback-azure-paas)
        - [Task 4: Failback Azure IaaS region to region](#task-4-failback-azure-iaas-region-to-region)
    - [After the hands-on lab](#after-the-hands-on-lab)
        - [Task 1: Disable replication in the recovery services vault](#task-1-disable-replication-in-the-recovery-services-vault)
        - [Task 2: Delete all BCDR resource groups](#task-2-delete-all-bcdr-resource-groups)

<!-- /TOC -->

# Business continuity and disaster recovery hands-on lab step-by-step 

## Abstract 

In this hands-on lab, you will implement three different environments and use Azure BCDR technologies to achieve three distinct goals for each environment type. These will include a migration to Azure, Azure region to region failover, and a PaaS implementation using BCDR technologies to ensure high availability of an application.

At the end of this hands-on lab, you will be better able to build a complex and robust IaaS BCDR solution.  

## Overview

The Business continuity and disaster recovery hands-on lab is an exercise that will challenge you to implement a BCDR solution that includes three different environments and uses Azure BCDR technologies to achieve three distinct goals for each environment type. These will include a migration to Azure, Azure region to region failover using Azure Site Recovery (ASR), and a PaaS implementation using BCDR technologies to ensure high availability of an application.

The hands-on lab can be implemented on your own, but it is highly recommended to pair up with other members at the lab to model a real-world experience and to allow each member to share their expertise for the overall solution.

## Solution architecture

Below are diagrams of the solution architecture you will build in this lab. Please study this carefully, so you understand the whole of the solution as you are working on the various components.

### Environment: On-premises (migrate to Azure)

-   **Background:** This environment will deploy a Hyper-V Host that will host a Linux VM to simulate a Linux, Apache, PHP, and MySQL (LAMP) based web application deployed into an on-premises datacenter on a single VM.

-   **Goal using Azure BCDR:** Your goal for this environment will be to migrate this application to Azure IaaS with a one-direction failover.

    ![The on-premises migration diagram has on-premises, azure platform, and secondary region sections. On-premises has a hyper-v host and a linux on-premises virtual machine. Azure Platform uses Azure Site Recovery. The secondary region has an on-premises Linux VM as well.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image2.png "On-premises migration diagram")

### Environment: Azure IaaS (failover region to region)

-   **Background:** This environment will consist of two Virtual Networks deployed to your Primary and Secondary site with an AD Domain, IIS Web Servers and Microsoft SQL Servers that you will configure into a SQL Always On Availability Group.

-   **Goal using Azure BCDR:** Your goal for this environment is to have the ability to have a one-click failover process using Azure Site Recovery in either direction. The users will have one URL that they will use to connect to your application regardless of where the application is running.

    ![Diagram of the Azure IaaS failover region to region solution.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image3.png "Azure IaaS failover region to region solution")

### Environment: Azure PaaS (high-availability with seamless failover)

-   **Background:** This environment will deploy an Azure Web App and Azure SQL Server in both the Primary and Secondary locations. You will configure SQL Database Failover groups to allow for seamless failover of the database.

-   **Goal using Azure BCDR:** Your goal for this environment is never to have the users experience any downtime if issues arise with your Web App or the SQL Database. The users will have one URL that they will use to connect to your application regardless of where the application or database is running.

![Diagram of the Azure PaaS high availability with seamless failover solution.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image4.png "Azure PaaS high availability with seamless failover diagram")

## Requirements

1.  An Azure Subscription with full access to the environment.

## Exercise 1: Deploy Azure environments

Duration: 15 minutes (Deployments can take as long as 75 minutes)

In this exercise, you will use Azure ARM Templates to deploy the following environments that will be used in this HOL:

-   **Azure IaaS:** This environment will consist of two Virtual Networks deployed to your Primary and Secondary site with an AD Domain, IIS Web Servers and Microsoft SQL Servers that you will configure into a SQL Always On Availability Group.

-   **On-premises:** This environment will deploy a Hyper-V Host that will host a Linux VM to simulate a web application deployed into on-premises datacenter on a single VM. Your goal for this environment will be to migrate this application to Azure IaaS with a one-direction failover.

-   **Azure PaaS:** This environment will deploy an Azure Web App and Azure SQL Server in both the Primary and Secondary locations.

### Task 1: Deploy Azure IaaS

1.  From the **LABVM**, open Internet Explorer and connect to the Azure portal at: <https://portal.azure.com>.

2.  Select **+Create a resource** and then search for **Template Deployment**.

    ![Template Deployment is typed in the New blade search field.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image27.png "Azure Portal New blade")

3.  Select **Template deployment** and then **Create**.

    ![Template deployment is selected in the Everything blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image28.png "Everything blade")

4.  On the Custom deployment blade, select **Build your own template in the editor**.

    ![In the Custom deployment blade, the link to build your own template in the editor is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image29.png "Custom deployment blade")

5.  On the Edit template blade, select **Load file**.

    ![In the Edit deployment blade top menu, Load file is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image30.png "Edit templat blade")

6.  From the **C:\\HOL\\Deployments** directory locate the **BCDRIaaSPrimarySite.json** file and select **Open**.

7.  This will load the template into the Azure portal. Select **Save**.

    ![Screenshot of the Save button.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image31.png "Save button")

8.  On the Custom deployment blade, next to **Resource group** select **Use existing** and then select your **BCDRIaaSPrimarySite** resource group. Notice how the template picked the deployment region based on the location of your **BCDRIaaSPrimarySite** resource group. Make sure that this is your **Primary** region.

    ![In the Custom deployment blade, fields are set to the previously defined values, and a callout points to the Location field.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image32.png "Custom deployment blade")

9.  Next, update the **Domain Controller DNS Name** in the **Settings** area. This will be the DNS name for the Active Directory Domain controller that will be your jump box into the IaaS environment. The name will need to be lowercase and 3-24 characters consisting of letters & numbers and be unique to all of Azure. In the example the name bcdrdc18675309 was used.

    ![In the Settings section, the Domain Controller Dns Name field is populated.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image33.png "Settings section")

10. Finally, select **I agree to the terms and conditions stated above** and **Pin to dashboard.** Select **Purchase** to start the deployment.

    ![The checkboxes for pin to dashboard, and I agree to the terms and conditions are selected, as is the Purchase button.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image34.png "Purchase button")

>**Note:** This deployment will take at least 60 minutes, but you can continue to the next task.

### Task 2: Deploy on-premises environment

1.  From the **LABVM**, open Internet Explorer and connect to the Azure portal at: <https://portal.azure.com>.

2.  Select **+Create a resource** and then search for **Template Deployment**.

    ![In the Azure portal New blade, Template Deployment is in the search field.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image27.png "Azure portal, New blade")

3.  Select **Template deployment** and then **Create**.

    ![Template deployment is selected in the Everything blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image28.png "Everything blade")

4.  On the Custom deployment blade, select **Build your own template in the editor**.

    ![The Build your own template in the editor link is selected in the Custom deployment blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image29.png "Custom deployment blade")

5.  On the Edit template blade, select **Load file**.

    ![In the Edit template blade top menu, Load file is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image30.png "Edit template blade")

6.  From the **C:\\HOL\\Deployments** directory locate the **BCDROnPremPrimarySite.json** file and select **Open**.

7.  This will load the template into the Azure portal. Select **Save**.

8.  On the Custom deployment blade, next to **Resource group** select **Use existing** and then select your **BCDROnPremPrimarySite** resource group. Notice how the template picked the deployment region based on the location of your **BCDROnPremPrimarySite** resource group. Make sure that this is your **Primary** region.

    ![In the Custom deployment blade, Basics section, fields are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image35.png "Custom deployment blade, Basics section")

9.  Next, update the **Hyper-V Host DNS Name** in the **Settings** area. This will be the DNS name for the Hyper-V Host that will you will use for this simulated on-premises environment. The name will need to be lowercase and 3-24 characters consisting of letters & numbers and be unique to all of Azure. In the example, the name hypervhost8675309 was used.

    ![In the Settings section, the Domain Controller Dns Name field is populated.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image36.png "Settings section")

10. Finally, select **I agree to the terms and conditions stated above** and **Pin to dashboard.** Select **Purchase** to start the deployment.

    ![The checkboxes for pin to dashboard, and I agree to the terms and conditions are selected, as is the Purchase button.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image34.png "Purchase button")

>**Note:** This deployment will take at least 20 minutes, but you can continue to the next task.

### Task 3: Deploy Azure PaaS environment

1.  From the **LABVM**, open Internet Explorer and connect to the Azure portal at <https://portal.azure.com>.

2.  Select **+Create a resource** and then search for **Template Deployment**.

    ![In the Azure Portal New blade, Template Deployment is in the Search field.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image27.png "Azure Portal, New blade")

3.  Select **Template deployment** and then **Create**.

    ![Template deployment is selected in the Everything blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image28.png "Everything blade")

4.  On the Custom deployment blade, select **Build your own template in the editor**.

    ![The Build your own template in the editor link is selected in the Custom deployment blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image29.png "Custom deployment blade")

5.  On the Edit template blade, select **Load file**.

    ![In the Edit template blade top menu, Load file is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image30.png "Edit template blade")

6.  From the **C:\\HOL\\Deployments** directory locate the **BCDRPaaSPrimarySite.json** file and select **Open**.

7.  This will load the template into the Azure portal. Select **Save**.

8.  On the Custom deployment blade, next to **Resource group** select **Use existing** and then select your **BCDRPaaSPrimarySite** resource group. Notice how the template picked the deployment region based on the location of your **BCDRPaaSPrimarySite** resource group. Make sure that this is your **Primary** region.

    ![In the Custom deployment blade, Basics section, fields are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image37.png "Custom deployment blade")

9.  Finally, select the **I agree to the terms and conditions stated above** and **Pin to dashboard.** Select **Purchase** to start the deployment.

    ![The checkboxes for pin to dashboard, and I agree to the terms and conditions are selected, as is the Purchase button.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image34.png "Purchase button")

>**Note:** This deployment will take at least 10 minutes, but you can continue to the next task.

## Exercise 2: Configure BCDR services

Duration: 30 minutes

In this exercise, you will create and configure the services that will make it possible to failover both the on-premises and Azure IaaS environments. These will include a Recovery Services Vault used for Azure Site Recovery and an Azure Automation account.

![The Automation region contains an Azure automation account, and two PowerShell scripts: ASRRunBookSQL, and ASRRunBookWEB. The Secondary region has an Azure Site Recovery, and a recovery services vault.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image38.png "Automation and Secondary Regions")

### Task 1: Create Azure recovery services vault

1.  Using the **LABVM** connect to the Azure portal using your web browser at <https://portal.azure.com>.

2.  Select **+Create a resource**, then enter **Backup and Site Recovery (OMS)**, and select **Create**.

    ![Screenshot of the Backup and Site Recovery (OMS) Screen with the Create button clicked.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image39.png "Backup and Site Recovery (OMS) Screen Create Button")

3.  Complete the **Recovery Services vault** blade using the following inputs, then select **Create**:

    -   **Name**: BCDRRSV

    -   **Resource Group**: Use existing / BCDRAzureSiteRecovery

    -   **Location**: Secondary Site (should be automatically selected based on your resource group)

4.  Once the **BCDRRSV** Recovery Service Vault has been created, open it in the Azure portal. Toggle the switch that selects Backup / Site Recovery.

    ![Screenshot of the Backup / Site Recovery toggle switch with Site Recovery selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image40.png "Backup / Site Recovery toggle switch")

5.  This is your dashboard for Azure Site Recovery (ASR), for the HOL.

    ![The Azure Site Recovery dashboard displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image41.png "Azure Site Recovery dashboard")

### Task 2: Deploy Azure automation

1.  Open the Azure portal at: <https://portal.azure.com>.

2.  Select **+Create a resource** and then enter **Automation** in the search box.

    ![In the Azure Portal New blade, Automation is typed in the search field.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image42.png "Azure Portal, New blade")

3.  Select **Automation** and then **Create**.

    ![In the Everything blade, Automation is in the Search field, and under Name, Automation is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image43.png "Everything blade") 
    
    ![Create button.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image44.png "Create button")

4.  Complete the **Add Automation Account** blade using the following inputs and then select **Create**:

    -   **Name**: Enter a Globally unique name starting with BCDR.

    -   **Resource group**: Use existing / BCDRAzureAutomation

    -   **Location**: Select a site in your Area, but NOT your Primary Site.

    -   **Create Azure Run As account**: Yes

        ![Fields in the Add Automation Account blade are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image45.png "Add Automation Account blade")

>**Note:** Azure Automation accounts are only allowed to be created in certain Azure regions, but they can act on any region in Azure (except Government, China or Germany). It is not a requirement to have your Azure Automation account in the same region as the **BCDRAzureAutomation** resource group but **CANNOT** be in your primary site.

5.  Once the Azure automation account has been created open the account and select **Modules gallery** under **Shared Resources**.

    ![Under Shared Resources, Modules gallery is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image46.png "Shared Resources section")

6.  When the Modules load, scroll down and locate and select **AzureRM.Profile**.

    ![The AzureRM.Profile link is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image47.png "AzureRM.Profile link")

7.  Select **Import**.

    ![Import is selected in the AzureRM.Profile blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image48.png "AzureRM.Profile blade")

8.  Select the **I agree to update all of the Azure modules** and then select **OK**.

    ![In the Import blade, the checkbox is selected for I agree to update all of the Azure modules.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image49.png "Import blade")

9.  It will take a few minutes to update the modules. Select **Modules** under Shared Resources, and you can wait the import has completed.

    ![In the Automation Account blade, under Shared Resources, Modules is selected. Under Add a module, Azure modules are being updated is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image50.png "Automation Account blade") 
    
    ![Screenshot of the Azure modules have been updated message.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image51.png "Azure modules have been updated")

10. Select back to **Modules Gallery**.

11. When the Modules load, scroll down and locate and select **AzureRM.Network**.

    ![Screenshot of the AzureRM.Network link.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image52.png "AzureRM.Network link")

12. Select **Import**.

    ![Import is selected in the AzureRM.Network blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image53.png "AzureRM.Network blade")

13. On the Import blade, select **OK**.

    ![Screenshot of the Import blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image54.png "Import blade")

14. The portal will begin the import process, but should only take about a minute.

    ![Screenshot of the Azure modules have been updated message.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image51.png "Azure modules have been updated")

15. Next select **Runbooks**.

    ![In the Automation Account blade, under Process Automation, Runbooks is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image55.png "Automation Account")

16. Select **+Add a runbook**.

    ![Screenshot of the Add a runbook button.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image56.png "Add a runbook button")

17. On the **Add Runbook** blade, select **Import an existing runbook**.

    ![Import an existing runbook is selected in the Add Runbook blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image57.png "Add Runbook blade")

18. Select the **Folder** icon on the Import blade and select the file **ASRRunbookSQL.ps1** from the C:\\HOL\\Deployments directory on the **LABVM**. The Runbook type should default to **PowerShell Workflow**. Notice that the Name can't be changed. This is the name of the Workflow inside of the Runbook script. Select **Create**.

    ![Fields in the Import blade are set to the previously defined settings. A callout points to the Name field.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image58.png "Import blade")

19. Once the Runbook is imported, select **ASRSQLFailover** to open the runbook.

    ![Under name, a callout points to the ASRQLFailover option.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image59.png "ASRQLFailover option")

20. On the **ASRSQLFailover** Runbook blade select **Edit**.

    ![In the Runbook blade top menu, Edit is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image60.png "Runbook blade")

21. The PowerShell runbook will load. If you wish, you can review the comments to better understand the runbook. Once completed select **Publish** to make the code available for use in the portal.

    ![On the top menu of the Edit PowerShell Workflow Runbook blade, Publish is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image61.png "Edit PowerShell Workflow Runbook blade")

>**Note:** You might notice the Test Pane link. This script can't be tested from here as there are more configurations required, and it relies of being called by Azure Site Recovery to feed its variables.

22. Select **Yes**, to configure that this Runbook will be published.

    ![In the Publish Runbook section, a message asks if you want to proceed, and the Yes button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image62.png "Publish Runbook section")

    ![Screenshot of the Published runbook message.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image63.png "Published runbook message")

23. Navigate back to the Azure Automation account.

    ![Screenshot of the Azure Automation account ASRSQLFailoverAG runbook heading.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image64.png "Azure Automation account")

24. Select **+Add a runbook**.

    ![Screenshot of the Add a runbook button.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image56.png "Add a runbook button")

25. On the **Add Runbook** blade, select **Import an existing runbook**.

    ![Import an existing runbook is selected in the Add Runbook blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image57.png "Add Runbook blade")

26. Select the Folder icon on the Import blade and select the file **ASRRunbookWEB.ps1** from the **C:\\HOL\\Deployments** directory on the **LABVM**. The Runbook type should default to **PowerShell Workflow**. Notice that the Name can't be changed. This is the name of the Workflow inside of the Runbook script. Select **Create**.

    ![Fields in the Import blade are set to the previously defined settings. A callout points to the Name field.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image65.png "Import blade")

27. Once the Runbook is imported select **ASRWebFailover** to open the runbook.

    ![Under name, a callout points to the ASRQLFailover option.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image66.png "ASRQLFailover option")

28. On the **ASRWebFailover** Runbook blade, select **Edit**.

    ![In the Runbook blade top menu, Edit is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image67.png "Runbook blade")

29. The PowerShell runbook will load. If you wish, you can review the comments to better understand the runbook. Once completed, select **Publish** to make the code available for use in the portal.

    ![On the top menu of the Edit PowerShell Workflow Runbook blade, Publish is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image68.png "Edit PowerShell Workflow Runbook blade")

30. Select **Yes** to configure that this Runbook will be published.

    ![In the Publish Runbook section, a message asks if you want to proceed, and the Yes button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image62.png "Publish Runbook section")

    ![Screenshot of the Published runbook message.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image69.png "Published runbook message")

31. Navigate back to **Runbooks**, and make sure that both Runbooks show as "**Published"**.

    ![Two runbooks have authoring status as published: ASRSQLFailover, and ASRWEBFailover.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image70.png "Runbooks")

32. From **LABVM**, select **Start** and select **PowerShell ISE** (make sure to right-click and run as Administrator).

    ![Screenshot of the Windows PowerShell ISE button.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image71.png "Windows PowerShell ISE button")

33. Open the file **C:\\HOL\\Deployments\\ASRRunBookVariable.ps1.** Review the script and enter the automation account name you created earlier. Then select the green play button to execute the script. You will need to authenticate to Azure.

    ![In the Windows PowerShell ISE window, the green play button is selected, and a callout points to \"Your-Automation-Account-Name-Here\" in the script window.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image72.png "Windows PowerShell ISE window")

>**Note**: If you are using an account that has multiple subscriptions you may need to change the context of your login. You can comment out the **Login-AzureRmAccount** command in the script and complete the login and the use the **Get-AzureRmSubscription** and **Select-AzureRmSubscription** cmdlets to create the proper context for this HOL. For help, you can review this article: <https://docs.microsoft.com/en-us/powershell/azure/authenticate-azureps?view=azurermps-5.1.1> & <https://docs.microsoft.com/en-us/powershell/azure/manage-subscriptions-azureps?view=azurermps-5.1.1>

34. Once the script has run, you will see the following output from PowerShell ISE. This script created a variable that will be used with the PowerShell Runbook in Azure Automation to help with the Failover and Failback of the Azure IaaS environment.

    ![Screenshot of PowerShell ISE output.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image73.png "PowerShell ISE output")

35. Move back to the Azure portal and locate the **Azure Automation** account. Select the **Variables** link in the **Shared Resources** section.

    ![In the Automation Account blade, Variables is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image74.png "Automation Account blade")

36. Notice that the variable **BCDRIaaSPlan** has been created. This variable will be used along with ASR to orchestrate part of the failover.

    ![In the Automation Account blade, under Name, a callout points to BCDRIaaSPlan.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image75.png "Automation Account blade")

>**Note:** When you configure the ASR Recovery Plan for the IaaS deployment you will use the SQL Runbook as a Pre-Failover Action and the Web Runbook as a Post-Failover action. They will run both ways and have been written to take the "Direction", of the failover into account when running.

## Exercise 3: Configure environments for failover

Duration: 90 minutes

In this exercise, you will configure the three environments to use BCDR technologies found in Azure. Each environment has unique configurations that must be completed to ensure their availability in the event of a disaster.

>**Note**: Make sure prior to starting each task that the deployment that you started in Exercise 1 has completed for each as you come to that task. This can be determined, but reviewing the deployments for each Resource group in the Azure portal. If it says Succeeded, then you can begin the task.

### Task 1: Configure on-premises to Azure IaaS failover for migration

In this task, the **OnPremVM** will be configured to replicate to Azure and be ready to failover to the **BCDRIaaSSecondarySite**. This will consist of configuring your Hyper-V host with the ASR provider and then enabling replication of the VM to the Recovery Service Vault.

1.  From the Azure portal, open the **BCDRRSV** Recovery Services Vault located in the **BCDRAzureSiteRecovery** resource group.

2.  Select **Site Recovery** in the **Getting Started** area of **BCDRRSV** blade.

    ![In the Recovery Services vault blade, under Getting Started, Site Recovery is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image76.png "Recovery Services vault blade")

3.  Next, select **Prepare Infrastructure** in the **For On-Premises Machines** section. This will start you down a path of various steps to configure your VM that is running on Hyper-V on-premises to be replicated to Azure.

    ![In the For On-Premises Machines section, Prepare Infrastructure is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image77.png "For On-Premises Machines section")

4.  On **Step 1 Protection Goal** select the following inputs and then select **OK**:

    -   **Where are your machines located?**: On-premises

    -   **Where do you want to replicate your machines to?**: To Azure

    -   **Are your machines virtualized?**: Yes, with Hyper-V (your VM is running as a nested VM in Azure).

    -   **Are you using System Center VMM to manage your Hyper-V hosts?**: No

    ![Fields in the Protection goal blade are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image78.png "Protection goal blade")

5.  On **Step 2 Deployment planning** select the following inputs and then select **OK**:

    -   Have you completed deployment planning: **Yes, I have done it**.

>**Note**: You can read more about planning an ASR to deployment here:

<https://docs.microsoft.com/en-us/azure/site-recovery/site-recovery-hyper-v-deployment-planner>

![In the Deployment planning blade, Yes, I have done it is selected in the drop-down menu.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image79.png "Deployment planning blade")

6.  On **Step 3 Prepare source** select **+Hyper-V Site**.

    ![In the top menu of the Prepare source blade, Hyper-V Site is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image80.png "Prepare source blade")

7.  On the **Create Hyper-V site** blade, enter the name: **OnPremHyperVSite**. Select **OK**.

    ![OnPremHyperVSite is in the Name field in the Create Hyper-V site blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image81.png "Create Hyper-V site blade")

8.  The portal will deploy the site providing you notifications. Wait for the creation process to complete, and the ASR portal will update once this is done. Your new site is now shown under **Step 1: Select Hyper-V site**.

    ![The Successfully completed creating Hyper-V site message displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image82.png "Success message")

    ![Under Step 1: Select Hyper-V site, OnPremHyperVSite is selected in the drop-down menu.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image83.png "Step 1: Select Hyper-V site")

9.  Next select **+Hyper-V Server**.

    ![In the top menu of the Prepare source blade, Hyper-V Server is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image84.png "Prepare source blade")

10. A new blade will appear. You will need to download the vault registration key to register the host in the Hyper-V site of ASR. Select the Download button which will save the file to your Downloads folder on the **LABVM**.

    ![In the Add Server blade, the Download button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image85.png "Add Server blade")

11. Open a **NEW** tab in your web browser and connect again to the Azure Portal at <https://portal.azure.com>.

12. Select **Resource groups**, then **BCDROnPremPrimarySite**.

    ![In the Resource groups blade, under Name, BCDROnPremPrimarySite is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image86.png "Resource groups blade")

13. Locate and select on the **HYPERVHOST** VM object.

    ![The HyperVHost button displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image87.png "HyperVHost button")

14. Select **Connect** and open the RDP file that is downloaded.

    ![In the HyperVHost blade, the Connect button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image88.png "HyperVHost blade")

15. Enter the credentials for the VM:

    -   **User Name**: mcwadmin

    -   **Password**: demo\@pass123

16. You will be prompted with a warning about a certificate. Select **Yes** to connection (you can always select yes to these prompts during this HOL).

    ![In the Remote Desktop Connection dialog box, the Yes button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image89.png "Remote Desktop Connection dialog box")

17. Select **Yes** on the Networks prompt.

    ![Screenshot of the Networks prompt.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image90.png "Networks prompt")

18. On the **HYPERVHOST** select **Configure this local server** in the Server Manager Dashboard.

    ![In Server Manager, the link to Configure this local server is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image91.png "Server Manager")

19. On the right side of the pane, select **On** by **IE Enhanced Security Configuration**.

    ![IE Enhanced Security Configuration is set to On.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image15.png "IE Enhanced Security Configuration ")

20. Change to **Off** for Administrators and select **OK**.

21. Open Internet Explorer on **HYPERVHOST** and browse to the following URL. This will download the Azure Site Recovery Provider for Hyper-V.

http://aka.ms/downloaddra_cus

22. Select **Run**.

    ![When asked if you want to run or save the file, the Run button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image92.png "Run button")

23. Select **On** and then **Next**.

    ![In the Microsoft Update screen, under Microsoft Update, On (recommended) is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image93.png "Microsoft Update screen")

24. Select **Install** on the Provider Installation screen.

    ![On the Provider Installation screen, Install is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image94.png "Provider Installation screen")

25. Once the Provider has been installed you will come to a screen that will request for you **Register** or **Finish**. Select **Register**.

    ![On the Provider Installation screen, the Register button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image95.png "Provider Installation screen")

26. Minimize your Remote Desktop window and locate the vault registration key which is in the **Downloads** directory of your **LABVM**. Right-click the file and copy it. Move back to your **HYPERVHOST** and paste the file to the desktop.

    ![In the menu, Paste is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image96.png "Menu") 
    
    ![Screenshot of the file being pasted.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image97.png "File being pasted")

27. On the Vault Setting screen select **Browse**.

    ![In the Vault Settings Screen, the Browse button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image98.png "Vault Settings Screen")

28. Locate the Vault file on the desktop and select **Open**.

    ![In File Explorer, the vault file is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image99.png "File Explorer")

29. The Vault Settings will now be populated. Select **Next**.

    ![Fields in the Vault Settings screen are populated.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image100.png "Vault Settings screen")

30. On the Proxy settings screen retain the settings **Connect directly to Azure Site Recovery without a proxy server** and select **Next**.

    ![On the Proxy Settings screen, the radio button is selected for Connect directly to Azure Site Recovery without a proxy server.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image101.png "Proxy Settings screen")

31. The provider will then connect and configure the Azure Site Recovery registration for the Hyper-V Server.

    ![On the Registration screen, a message indicates that azure site recovery is being configured for registration.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image102.png "Registration screen")

32. After a few minutes, the wizard will be complete with the message **The Server was registered in the Azure Site Recovery vault.** Select **Finish** to close the Provider installation.

    ![The message on the Registration screen now says the server was registered in the Azure Site Recovery vault.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image103.png "Registration screen")

33. Select **Start**, **Windows Administrative Tools**.

    ![Screenshot of the Windows Administrative Tools icon.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image104.png "Windows Administrative Tools icon")

34. Locate then double-click the **Hyper-V Manager**.

    ![Screenshot of the Hyper-V Manager icon.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image105.png "Hyper-V Manager icon")

35. When the Hyper-V Manager opens, select the Server name in the left pane and then you will see that the **OnPremVM** virtual machine is running on your **HYPERVHOST**. This is an Ubuntu 16.04.3 LTS server running Apache, PHP and MySQL. Double-click on the VM to open.

    ![In Hyper-V Manager, HyperVHOST is selected, and the OnPremVM virtual machine data displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image106.png "Hyper-V Manager")

36. The console for the **OnPremVM** will load. Press **Enter** to get a login prompt.

    ![The OnPremVM virtual machine console displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image107.png "OnPremVM virtual machine console")

37. Login to the VM using the following credentials:

    -   **User Name**: mcwadmin

    -   **Password**: demo\@pass123

38. Once logged in enter a few commands and notice that you can get to the internet and that the **local IP address of the VM is currently 192.168.0.10**.

    ```
    ping 8.8.8.8 -c 4

    ifconfig
    ```

    ![In the OnPremVM virtual machine console, the local IP address is called out.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image108.png "OnPremVM virtual machine console")

39. Open Internet Explorer on **HYPERVHOST** and browse to the following URL of the **OnPremVM**. This is very simple PHP application running on the **OnPremVM** connected to the MySQL Server that is running locally on the VM. This is to simulate an application is that running on one VM in the on-premises data center.

    ```
    http://192.168.0.10/bcdr.php
    ```

![In Hyper-V Manager, in the tree view, HyperVHost is selected. Under Virtual Machines, OnPremVM is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image109.png "Hyper-V Manager")

40. From the command prompt of the OnPremVM update the OS with the latest patches by using the following command. You will need to enter the password again.

    ```
    sudo apt-get update -y
    ```

    ![In the OnPremVM virtual machine console, the previous command is called out.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image110.png "OnPremVM virtual machine console")

41. After the updates complete, install the Azure Guest Agent for Linux on the VM using the following commands:

    ```
    sudo apt-get install walinuxagent -y
    ```

    ![In the OnPremVM virtual machine console, the previous command displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image111.png "OnPremVM virtual machine console")

>**Note**: You may see some errors, but this is normal behavior since the VM now thinks it should be in Azure.

42. Type exit into the terminal to log out of the **OnPremVM**.

43. Sign out of **HYPERVHOST** and return to the Azure portal running on your **LABVM**.

44. You will need to Return to the Prepare Source screen and select **+Hyper-V Server**. Notice the warning that it could take up to 30 mins for this server to appear, but in practice you should cancel out of this window by selecting Step 2 and then answer yes again with **I have done it** to the question of Deployment planning.

    ![Hyper-V Server is selected in the Prepare source blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image112.png "Prepare source blade")

45. Once the Server appears, select **OK**.

    ![In the Prepare source blade, a callout points to HYPERVHOST under Step 2.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image113.png "Prepare source blade")

46. On **Step 4 Target Prepare** review the screen to better understand the various steps. Select **+Storage** **account** to add a storage account that will be used for the **OnPremVM** when it is failed over to Azure.

    ![Add storage account is selected in the Target blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image114.png "Target blade")

47. Select **Create New** and provide a unique name for your storage account containing the name of the VM **OnPremVM** with added characters to make it unique. Also, select the Premium tier for the storage account and select **OK**.

    ![In the Choose storage account blade, Create new is selected. In the Create storage account blade, fields are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image115.png "Choose storage account and Create storage account blades")

48. The portal will submit a deployment, and you must wait until this completes. It will be created in the **BCDRAzureSiteRecovery** resource group.

    ![Screenshot of the Deployment succeeded message.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image116.png "Deployment succeeded message")

49. You will be failing the **OnPremVM** into the **Secondary** site that was deployed with your IaaS installation, so you already have a Virtual Network created. Select **OK** on the Target blade to continue.

    ![Add storage account is selected in the Target blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image117.png "Target blade")

50. On Step 5: Replication settings screen you will select the **+Create and Associate** to a **Replication policy**.

    ![Create and Associate is selected in the Replication Policy blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image118.png "Replication Policy blade")

51. Enter the name: **OnPremVM-POL**. Review the settings that you can configure and then select **OK**.

    ![In the Create and Associate blade, the Name field is set to OnPremVM-POL.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image119.png "Create and Associate blade")

52. Azure will run a deployment and start the process of creating and then associating the Hyper-V Site with the replication policy.

    ![In Step 1 of the replication policy, the Replication policy field is empty.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image120.png "Step 1 replication policy")

    ![In Step 1 of the replication policy, the Replication policy field is displays OnPremVM-POL.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image121.png "Step 1 replication policy")

>**Note**: This will take a couple of minutes to complete. Please wait until this completes prior to moving on.

Once complete select **OK**.

   ![The OK button is selected in the Replication policy blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image122.png "Replication policy blade")

53. Select **OK** again, and the process for adding the Hyper-V Server to the Recovery Services Vault will be complete.

    ![The OK button is selected in the Prepare infrastructure blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image123.png "Prepare infrastructure blade")

54. Next, select Step 1: **Replicate Application**.

    ![In the Recovery Services vault blade, Step 1: Replicate Application is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image124.png "Recovery Services vault blade")

55. On the Source blade select: Source: **On-premises** and Source location: **OnPremHyperVSite** and select **OK**.

    ![Fields in the Source blade are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image125.png "Source blade")

56. Complete the Target blade using the following inputs:

    -   **Post-failover resource group**: BCDRIaaSSecondarySite

    -   **Storage Account**: Select the account you just created (onpremvm8675309).

    -   **Storage Account for replication logs**: Create a new one using the prefix bcdrasrrepllogs.

    -   **Azure network**: Configure now for selected machines.

    -   **Post-failover Azure network**: BCDRFOVNET

    -   **Subnet**: WEB (172.16.1.0/24)

        ![Fields in the Target blade are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image126.png "Target blade")

57. Next on the **Select virtual machines** select **OnPremVM** and then select **OK**.

    ![In the Select virtual machines blade, OnPremVM is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image127.png "Select virtual machines blade")

58. Complete the **OnPremVM** selections of the **Configure properties** blade using these inputs and then select **OK**:

    -   **OS Type**: Linux

    -   **OS Disk**: OnPremLinuxVM

    -   **Disks to Replicate**: OnPremLinuxVM \[127.00GB\]

        ![Fields in the Configure properties blade are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image128.png "Configure properties blade")

59. On the **Configure replication settings** blade review the selections and notice that the **OnPremVM-POL** that you created has been selected. Select **OK**.

    ![The Configure replication settings blade displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image129.png "Configure replication settings blade")

60. On the final screen, select **Enable replication**.

    ![Enable replication is selected on the Enable replication blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image130.png "Enable replication blade")

61. The deployment will be submitted. You can select **Site Recovery Jobs** to review the process.

    ![In the Recovery Services vault blade, Jobs and Site Recovery Jobs are selected. The Site Recovery jobs blade displays process status.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image131.png "Recovery Services vault and Site Recovery jobs blade")

62. After a few minutes, the Enable replication will move to Successful. Select **Overview** on the **BCDRRSV**. You should now see that there is 1 Healthy Replicated Item.

    ![In the Recovery Services vault blade, a callout points to the healthy replicated item.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image132.png "Recovery Services vault blade")

63. Select **Healthy 1** and you will see the replicated items list. Notice it shows that the VM is healthy, but the replication has just started, so it shows **0% synchronized**. It will take a few minutes for the VM to replicate.

    ![In the Replicated items blade, a callout points to the status of 0 percent synchronized.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image133.png "Replicated items blade")

64. Select **OnPremVM**. Review the Replication details for **OnPremVM**. Once the VM has replicated, the selections across the top menu bar of the dashboard will allow you to work with this VM.

    ![In the Replicated items blade, a callout points to the status of Protected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image134.png "Replicated items blade")

>**Note**: It will take about 15 minutes for the VM to replicate, so you can move on and come back to review the environment later.

65. Scroll down and review the Infrastructure view of the BCDR environment you have created for **OnPremVM**. You can select **Hyper-V Virtual Machine** to review what is protected.

    ![In the Infrastructure view, Hyper-V Virtual machine and Log storage accounts are in On-premises, and connect to Azure Site Recovery and Storage accounts in Azure.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image135.png "Infrastructure view")

    ![In the Hyper-V sites blade, OnPremHyperVSite has one hyper-v host, and one protected VM.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image136.png "Hyper-V sites blade")

66. Next, select **Compute and Network**.

    ![In the Replicated items blade, Compute and Network is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image137.png "Replicated items blade")

67. Details of the VM will be shown, and you can make configuration changes. Change the **Size** of the VM to **DS2\_v2,** and then select **Save**.

    ![The Compute properties dialog box displays with size set to the previously mentioned value.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image138.png "Compute properties dialog box")

>**Note**: It could take a few minutes for these screens to populate, so be patient. You can come back to this step later to adjust the size if you wish.

### Task 2: Configure IaaS SQL Always On availability groups for region to region failover

In this task, you will build a Windows Failover Cluster and configure SQL Always On Availability Groups. This will be in place to ensure that if there is an issue in the **Primary** site in Azure you can failover to the **Secondary** site and have access to the data for the application. You will also configure the Traffic Manager to ensure that the Web Application will always answer to the same DNS name even when it is failed over to the **Secondary** site.

1.  From the **LABVM** navigate to the Azure Portal, and navigate to **Resource Groups** and then **BCDRAzureSiteRecovery**.

    ![In the Resource groups blade, under Name, BCDRAzureSiteRecovery is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image139.png "Resource groups blade")

2.  Select **+Add**.

    ![In the Resource Group blade, the Add button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image140.png "Resource Group blade")

3.  In the Search box, enter **Storage Account,** and then select **Storage account -- blob, file, table, queue**.

    ![In the Everything blade search field, Storage account - blob, file, table, queue is typed.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image141.png "Everything blade")

4.  Select **Create**.

    ![Screenshot of the Create button.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image142.png "Create button")

5.  Complete the **Create storage account** wizard using the following details, then select **Review + create**:

    -   **Name:** Unique name starting with bcdrcloudwitnessxxx.

    -   **Deployment model**: Resource manager

    -   **Account kind**: Storage (general purpose v2)

    -   **Performance**: Standard

    -   **Replication**: Locally-redundant storage (LRS)
   
    -   **Resource group**: Use existing / BCDRAzureSiteRecovery

    -   **Location**: Any location in your area that is NOT your Primary or Secondary Site.

        ![Fields in the Create storage account blade are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image143.png "Create storage account blade")

6.  Once the storage account is created, locate and select **Access keys** under **Settings**.

    ![Under Settings, Access keys is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image144.png "Settings section")

7.  Copy the name of the account and the first access key to notepad and save the file as **C:\\HOL\\Deployments\\CloudWitness.txt** on your **LABVM**.

    ![The Storage account name is selected, and under Default keys, the copy button for key1 is called out.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image145.png "Storage account section")

    ![Notepad displays the copied information.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image146.png "Notepad")

8.  From the **LABVM**, connect to the Azure portal and locate the **BCDRIaaSPrimarySite** Resource group.

    ![In the Resource groups blade, under Name, BCDRIaaSPrimarySite is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image147.png "Resource groups blade")

9.  Select **BCDRDC1** and then select **Connect**.

    ![In the top menu of the Virtual Machine blade, Connect is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image148.png "Virtual Machine blade")

10. Once in the RDP session to **BCDRDC1**, select **Start** and then **Remote Desktop**. You will need to connect to **SQLVM1** for the next steps.

    ![Screenshot of the Remote Desktop icon.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image149.png "Remote Desktop icon")

11. Connect to **SQLVM1** using the following credentials:

    -   **User Name**: CONTOSO\\mcwadmin

    -   **Password**: demo\@pass123

        ![In the Remote Desktop Connection window, fields are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image150.png "Remote Desktop Connection window")

12. Minimize your Remote Desktop window and locate three files: **ClusterCreateSQLVM1.ps1**, **ClusterUpdateSQLVM1.ps1** and **CloudWitness.txt** in the **C:\\HOL\\Deployments** directory of your **LABVM**. Right-click the files and copy them, then move back to your **SQLVM1** and **paste the files to the desktop**.

    ![Screenshot of the three file icons.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image151.png "Three file icons")

13. On **SQLVM1**, select Start and then select **PowerShell ISE**.

    ![Screenshot of the Windows PowerShell ISE icon.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image71.png "Windows PowerShell ISE icon")

14. Open the **ClusterCreateSQLVM1.ps1** in the PowerShell ISE window. Then select the **green play button**. This script will create the Windows Failover Cluster and add all the SQL VMs as nodes in the cluster. It will also assign a static IP address of 10.0.2.99 to the new Cluster named **AOGCLUSTER**.

    ![On the Windows PowerShell ISE top menu, the green play button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image152.png "Windows PowerShell ISE window")

    ![In the PowerShell ISE window, a callout points to AOGCLUSTER.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image153.png "PowerShell ISE window")

>**Note:** It is possible to use a wizard for this task, but the resulting cluster will require additional configuration to set the static IP address in Azure.

15. After the cluster has been created, select **Start** and then **Windows Administrative Tools**. Locate and open the **Failover Cluster Manager**.

    ![Screenshot of the Failover Cluster Manager shortcut icon.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image154.png "Failover Cluster Manager shortcut icon")

16. When the cluster opens, select **Nodes** and the SQL Server VMs will show as nodes of the cluster and show their status as **Up**.

    ![In Failover Cluster Manager, Nodes is selected in the tree view, and three nodes display in the details pane.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image155.png "Failover Cluster Manager")

17. If you select **Roles**, you will notice that currently, there aren't any roles assigned to the cluster.

    ![In Failover Cluster Manager, Roles is selected in the tree view, and zero roles display in the details pane.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image156.png "Failover Cluster Manager")

18. Select Networks, and you will see two networks: **Cluster Network 1** and **Cluster Network 2**. Both should show the status of **Up**. If you navigate through to the networks, you will see their IP address spaces, and on the lower tab you can select **Network Connections** and review the nodes.

    ![In Failover Cluster Manager, Networks is selected in the tree view, and two networks display in the details pane.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image157.png "Failover Cluster Manager")

    ![Details of Cluster Network 1 display the owner node, status, and name details.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image158.png "Cluster Network 1")

19. Right-click **AOGCLUSTER,** then select **More Actions**, **Configure Cluster Quorum Settings**.

    ![In Failover Cluster Manager, a callout says to right-click the cluster name in the tree view, then click More Actions, and then click Configure Cluster Quorum Settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image159.png "Failover Cluster Manager")

20. On the **Configure Cluster Quorum Wizard** select **Next**, then select **Select the quorum witness**. Then select **Next** again.

    ![On the Select Quorum Configuration Option screen of the Configure Cluster Quorum Wizard, the radio button for Select the quorum witness is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image160.png "Configure Cluster Quorum Wizard")

21. Select **Configure a cloud witness**.

    ![On the Select Quorum Witness screen, the radio button for Configure a cloud witness is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image161.png "Select Quorum Witness screen")

22. Open the **CloudWitness.txt** file on the desktop of **SQLVM1** and copy the **Storage account name** and **KEY1**. Leave the Azure Service endpoint as configured. Then select **Next**.

    ![Fields on the Configure cloud witness screenare set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image162.png "Configure cloud witness screen")

23. Select **Next** on the Confirmation screen.

    ![On the Confirmation screen, the Next button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image163.png "Confirmation screen")

24. Select **Finish**.

    ![Finish is selected on the Summary screen.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image164.png "Summary screen")

25. Select the name of the Cluster again and the **Cloud Witness** should now appear in the **Cluster Resources**. It's important to always use a third datacenter, in your case here a third Azure Region to be your Cloud Witness.

    ![Under Cluster Core Resources, the Cloud Witness status is Online.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image165.png "Cluster Core Resources section")

26. Select **Start** and Launch **SQL Server 2017 Configuration Manager**.

    ![Screenshot of the SQL Server 2017 Configuration Manager option on the Start menu.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image166.png "SQL Server 2017 Configuration Manager option")

27. Select **SQL Server Services**, then right-click **SQL Server (MSSQLSERVER)** and select **Properties**.

    ![In SQL Server 2017 Configuration Manager, in the left pane, SQL Server Services is selected. In the right pane, SQL Server (MSSQLSERVER) is selected, and from its right-click menu, Properties is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image167.png "SQL Server 2017 Configuration Manager")

28. Select the **AlwaysOn High Availability** tab and check the box for **Enable AlwaysOn Availability Groups**. Select **Apply** and then select **OK** on the message that notifies you that changes won't take effect until after the server is restarted.

    ![In the SQL Server Properties dialog box, on the Always On High Availability tab, the Enable AlwaysOn Availability Groups checkbox is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image168.png "SQL Server Properties dialog box")
    
    ![A pop-up warns that any changes made will not take effect until the service stops and restarts.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image169.png "Warning pop-up")

29. On the **Log On** tab, change the service account to **contoso\\mcwadmin** with the password **demo\@pass123**. Select **OK** to accept the changes, and then select **Yes** to confirm the restart of the server.

    ![In the SQL Server Properties dialog box, on the Log On tab, fields are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image170.png "SQL Server Properties dialog box")
    
    ![A pop-up asks you to confirm that you want to make the changes and restart the service.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image171.png "Confirm Account Change pop-up")

30. Open a new Remote desktop session (this can be done from within SQLVM1), and repeat these steps to **Enable SQL Always On**. Change the username to **contoso\\mcwadmin** on each of the other nodes **SQLVM2**, and **SQLVM3.** Make sure that you have restarted the SQL Service on each node prior to moving to the next node.

>**Note:** If you get confused what server you are on open a command prompt and simply enter the command *hostname.*

31. After you have completed the process on each SQLVM Node, reconnect to **SQLVM1** using Remote Desktop.

>**Note:** Remember that you must use the BCDRDC1 VM as your jumpbox to get into the environment. You can use the Azure portal to connect to BCDRDC1 and then use Remote desktop form there to SQLVM1.

32. Use the Start menu to launch **Microsoft SQL Server Management Studio 17** and connect to the local instance of SQL Server. (Located in the Microsoft SQL Server Tools 17 folder).

    ![Screenshot of Microsoft SQL Server Management Studio 17 on the Start menu.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image172.png "Microsoft SQL Server Management Studio 17")

33. Select **Connect** to sign on to **SQLVM1**.

    ![Screenshot of the Connect to Server dialog box.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image173.png "Connect to Server dialog box")

34. Right-click **Always On High Availability**, then select **New Availability Group Wizard**.

    ![In Object Explorer, Always On High Availiability is selected, and from its right-click menu, New Availability Group Wizard is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image174.png "SQ Server Management Studio, Object Explorer")

35. Select **Next** on the Wizard.

    ![On the New Availability Group Wizard begin page, Next is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image175.png "New Availability Group Wizard ")

36. Provide the name **BCDRAOG** for the **Availability group name**, then select **Next**.

    ![The Specify availability group options page displays the previous availability group pname.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image176.png "Specify availability group options page")

37. Select the ContosoInsurance Database, then select **Next**.

    ![ContosoInsurance is selected on the Select Databases page.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image177.png "Select Databases page")

38. On the Specify Replicas screen next to **SQLVM1**, select **Automatic Failover**.

    ![On the Replicas tab, for SQLVM1, the checkbox for Automatic Failover (Up to 3) is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image178.png "Specify Replicas screen")

39. Select **Add Replica**.

    ![Screenshot of the Add replica button.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image179.png "Add replica button")

40. On the Connect to Server enter the Server Name of **SQLVM2** and select **Connect**.

    ![Screenshot of the Connect to Server dialog box for SQLVM2.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image180.png "Connect to Server dialog box")

41. For **SQLVM2**, select Automatic Failover and Availability Mode of Synchronous commit.

    ![On the Replicas tab, for SQLVM2, the checkbox for Automatic Failover (Up to 3) is selected, and availability mode is set to synchronous commit.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image181.png "Specify Replicase Screen")

42. Select **Add Replica**.

    ![Screenshot of the Add replica button.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image179.png "Add replica button")

43. On the Connect to Server enter the Server Name of **SQLVM3** and select **Connect**.

    ![Screenshot of the Connect to Server dialog box for SQLVM3.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image182.png "Connect to Server dialog box")

44. At this point, the wizard should resemble the following:

    ![On the Replicas tab, for SQLVM3, the checkbox for Automatic Failover (Up to 3) is cleared, and availability mode is set to Asynchronous commit.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image183.png "Specify Replicas screen")

45. Select Endpoints and review these that the wizard has created.

    ![On the Endpoints tab, the three servers are listed.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image184.png "Specify Endpoints screen")

46. Next, select **Listener**. Then select the **Create an availability group listener**.

    ![On the Listener tab, the radio button for Create an availability group listener is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image185.png "Specify Listener screen")

47. Add the following details:

    -   **Listener DNS Name**: BCDRAOG

    -   **Port**: 1433

    -   **Network Mode**: Static IP

        ![Fields for the Listener details are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image186.png "Listener details")

48. Next, select **Add**.

    ![The Add button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image187.png "Add button")

49. Select the Subnet of **10.0.2.0/24** and then add IPv4 **10.0.2.100** and select **OK**. This is the IP address of the Internal Load Balancer that is in front of the **SQLVM1** and **SQLVM2** in the **BCDRVNET** **Data** Subnet running in the **Primary** Site.

    ![The Add IP Address dialog box fields are set to the prevoiusly defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image188.png "Add IP Address dialog box")

50. Select **Add**.

    ![The Add button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image189.png "Add button")

51. Select the Subnet of **172.16.2.0/24** and then add IPv4 **172.16.2.100** and select **OK**. This is the IP address of the Internal Load Balancer that is in front of the **SQLVM3** and **SQLVM4** in the **BCDRFOVNET** **Data** Subnet running in the **Secondary** Site.

    ![Fields in the Add IP Address dialog box are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image190.png "Add IP Address dialog box")

52. Select **Next**.

    ![The Next button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image191.png "Next button")

53. On the Select Initial Data Synchronization screen, make sure that **Automatic seeding** is selected and select **Next**.

    ![On the Select Initial Data Synchronization screen, the radio button for Automatic seeding is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image192.png "Select Initial Data Synchronization screen")

54. On the Validation screen, you should see all green. Select **Next**.

    ![The Validation screen displays a list of everything it is checking, and the results for each, which currently are success.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image193.png "Validation screen")

55. On the Summary page select **Finish**.

    ![On the Summary page, the Finish button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image194.png "Summary page")

56. The wizard will configure the AOG.

    ![On the New Availability Group Progress page, a progress bar shows the progress for configuring the endpoints.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image195.png "New Availability Group Progress page")

57. Once the AOG is built, select **Close**.

    ![On the New Availability Group Results page, a message says the wizard has completed successfully, and results for all steps is success.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image196.png "New Availability Group Results page")

58. Move back to **SQL Management Studio** on **SQLVM1** and open the **Always On High Availability** and then select on the **BCDRAOG (Primary)**. Expand the areas and review them.

    ![In SQL Management Studio, Always On High Availability is selected in the tree view.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image197.png "SQL Management Studio")

59. Right-click **BCDRAOG (Primary)** and then **Show Dashboard.** You should see that all the nodes have been added and are now "Green".

    ![The right-click menu for BCDRAOG displays, and Show Dashboard is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image198.png "Right-click menu")

    ![Screenshot of the BCDRAOG Dashboard.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image199.png "BCDRAOG Dashboard")

60. Next, select **Connect** and then **Database Engine** in SQL Management Studio.

    ![Connect / Database Engine is selected in Object Explorer.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image200.png "Object Explorer")

61. Enter **BCDRAOG** as the Server Name. This will be connected to the listener of the group that you created.

    ![In the Connect to Server Dialog box, Server name is BCDRAOG, and the connect button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image201.png "Connect to Server Dialog box")

62. Once connected to the **BCDRAOG**, you can select **Databases** and will be able to see the database there. Notice that you have no knowledge directly of which server this is running on.

    ![A callout points to ContosoInsurance (Synchronized) in SQL Management Studio.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image202.png "SQL Management Studio")

63. Move back to PowerShell ISE on **SQLVM1** and open the PowerShell script named **ClusterUpdateSQLVM1.ps1**. Select the **Play** button. This will update the Failover cluster with the IP Addresses of the Listener that you created for the AOG.

    ![In the Windows PowerShell ISE window, the play button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image203.png "Windows PowerShell ISE window")

    ![A PowerShell ISE command window displays the IP addresses for the Listener for AOG.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image204.png "PowerShell ISE command window")

64. Move back to SQL Management Studio and select **Connect** and then **Database Engine**.

    ![In Object Explorer, Connect / Database Engine is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image200.png "Object Explorer")

65. This time, put the following into the IP address of the Internal Load balancer of the **Primary** Site AOG Load Balancer: **10.0.2.100**. You again will be able to connect to the server which is up and running as the master.

    ![Fields in the Connect to Server dialog box are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image205.png "Connect to Server dialog box")

66. Once connected to **10.0.2.100**, you can select **Databases** and will be able to see the database there. Notice that you have no knowledge directly of which server this is running on.

    ![A callout points to the Databases folder in Object Explorer.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image206.png "Object Explorer")

>**Note:** It could take a minute to connect the first time as this is going through the Azure Internal Load Balancer.

67. Move back to Failover Cluster Manager on **SQLVM1**, and you can review the IP Addresses that were added by selecting Roles and **BCDRAOG** and viewing the Resources. Notice how the **10.0.2.100** is Online since the current primary replica is on the **Primary** Site.

    ![In the Failover Cluster Manager tree view, Roles is selected. Under Roles, BCDRAOG is selected, and details of the role display. A callout points to the IP Addresses, one of which is online, and the other is offline.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image207.png "Failover Cluster Manager")

68. Now that the AOG is up and running online the Contoso Insurance Web Application should be available. Minimize the RDP window and from the **LABVM** open the Azure portal and navigate to the resource group **BCDRIaaSPrimarySite**. Select **WWWEXTLB-PIP** which is the Public IP address for the external load balancer **WWWEXTLB** in front of **WEBVM1** and **WEBVM2** in your Primary region.

    ![In the Public IP address blade for WWWEXTLB-PIP, a callout points to the DNS name.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image208.png "Public IP address blade")

69. Locate the DNS name address and copy it to your clipboard.

    ![In the Public IP address blade, a callout points to the DNS name address.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image209.png "Public IP address blade")

70. Open a new tab in the browser and navigate to the DNS name. The **Contoso Insurance PolicyConnect** application should load in your browser. Select the **Current Policy Offerings** button to check for database connectivity.

    ![The Contoso Insurance PolicyConnect webpage displays, and the Current Policy Offerings button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image210.png "Contoso Insurance PolicyConnect webpage")

71. If you can see the offerings, then the application is able to access the database. You can also go back to the home page and interact with the application including adding, editing or deleting data.

    ![The Contoso Insurance Index webpage displays a list of offerings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image211.png "Contoso Insurance Index webpage")

>**Note**: If you see the following screen shot then something is not configured correctly with your environment. The connection string of the application is configured to use the name of **bcdraog.contoso.com** which is the name for the SQL AOG listener. This configuration is part of the connection string located in the **web.config** file which is on **WEBVM1** and **WEBVM2** in the **C:\\Inetpub\\wwwroot** directory. If you for some reason you named something incorrectly you can make a change to this file and then iisreset /restart from the command line on the WEBVMs.

![An Error message displays stating that an error occurred while processing your request.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image212.png "Error message")

72. Once you have verified that the application is up and running, you will need to build a Traffic Manager to direct traffic to the edge of your Primary and Secondary Site. Select **+Create a resource, Networking then Traffic Manager profile** in the Azure portal.

    ![In the Azure Portal, under Azure Marketplace, Networking and Traffic Manager profile are both selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image213.png "Azure Portal")

73. Complete the **Create Traffic Manager profile** using the following inputs, then select **Create**:

    -   **Name:** Unique name all lowercase using bcdriaasxxx.

    -   **Routing method:** Priority

    -   **Resource group:** Use existing / BCDRIaasPrimarySite

    -   **Location:** Automatically assigned based on the BCDRIaaSPrimarySite.

        ![Fields in the Create Traffic Manager profile blade are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image214.png "Create Traffic Manager profile blade")

74. Once the Traffic Manager profile is created, open it in the Azure portal. Notice the DNS name. This is the URL that you will use to connect to the application. Once configured this DNS name will always respond and doesn't matter if the IaaS application is running normally in the **Primary** site or failed over to the **Secondary site.**

    ![In the Traffic Manager profile blade, a callout points to the DNS name.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image215.png "Traffic Manager profile blade")

75. Select **Configuration** and review the configurations.

    ![Under Settings, Configuration is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image216.png "Settings section")

76. Next select **Endpoints**. This is where you will configure the two external load balancers that are located your **Primary** and **Secondary** sites.

    ![Under Settings, Endpoints is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image217.png "Settings section")

77. Select **+Add**.

    ![In the Traffic Manager profile blade top menu, Add is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image218.png "Traffic Manager profile blade")

78. Complete the **Add endpoint** using the following inputs and then select **OK:**

    -   **Type:** Azure endpoint

    -   **Name:** BCDRIaaSPrimarySiteLB

    -   **Target resource type:** Public IP address

    -   **Target resource:** Choose a public IP address.

    -   **Resource:** WWWEXTLB-PIP in the BCDRIaaSPrimarySite

    -   **Priority:** 1

        ![Fields in the Add endpoint blade are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image219.png "Add endpoint blade ")

79. Select **+Add**. Notice that the Primary endpoint was created as **"Enabled"**.

    ![The Add button is selected in the Traffic Manager profile blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image220.png "Traffic Manager profile blade")

80. Complete the **Add endpoint** using the following inputs and then select **OK:**

    -   **Type:** Azure endpoint

    -   **Name:** BCDRIaaSSecondarySiteLB

    -   **Target resource type:** Public IP address

    -   **Target resource:** Choose a public IP address.

    -   **Resource:** WWWEXTLB-PIP in the BCDRIaaSSecondarySite

    -   **Priority:** 2

        ![Fields in the Add endpoint blade are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image221.png "Add endpoint blade")

81. Once the second endpoint has been added, select **Overview**. Notice that the Primary endpoint is set to **Priority 1**. This means that traffic will always be directed to the **Primary** site unless it is down. The Traffic Manager will monitor the Endpoints, and if the **Primary** site moves to a **Monitor Status** of **Degraded**, then the Traffic Manager will direct traffic to the **Secondary** site. The current **Monitor Status** shows that the **Primary** site is **Online** and the Secondary site is **Degraded**. During a failover using ASR, the **Primary** site will move to **Degraded,** and the **Secondary** will move to **Online**. This will allow for traffic to flow to the failed over IaaS infrastructure now running at the **Secondary** site. This will, of course, revert during the failback to the **Primary** site.

    ![In the Traffic Manager profile blade, a callout points to the monitor status, which is Online.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image222.png "Traffic Manager profile blade")

>**Note:** All of this is automatic and easily configured with a vanity domain by adding a C NAME record in DNS to point to the DNS name of the Traffic Manager. This would allow for a site like [www.contoso.com](http://www.contoso.com) to resolve to the DNS name of the traffic manager. The users will never know that the site is failed over or failed back.

82. Select the DNS name of the Traffic manager the Policy Connect web application will load. This is connecting to the **WWWEXTLB** External Load Balancer that is in front of **WEBVM1** and **WEBVM2** running in the **Primary** Site in **BCDRIaaSPrimarySite** resource group and connecting to the SQL Always On Listener at the same location.

    ![The DNS name link is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image223.png "DNS name link")

    ![The Contoso Insurance PolicyConnect webpage displays with a callout pointing to the DNS name trafficmanager.net.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image224.png "Contoso Insurance PolicyConnect webpage ")

### Task 3: Configure IaaS for region to region failover

In this task the WEBVM1 and WEBVM2 will be configured to replicate from the Primary Site to the Secondary site to support an Azure region to region failover. This will consist of configuring the VMs to replicate and integrating with the Azure Automation to failover the SQL Always On group from the Primary Site to the Secondary. Once the failover is complete the website will again answer to the Traffic Manager DNS name.

1.  From the Azure portal on **LABVM**, open the **BCDRRSV** Recovery Services Vault located in the **BCDRAzureSiteRecovery** resource group.

2.  Select **Site Recovery** in the **Getting Started** area of **BCDRRSV** blade.

    ![In the Recovery Services vault blade, under Getting Started, Site Recovery is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image76.png "Recovery Services vault blade")

3.  Next, select **Step 1: Replicate Application** in the **For On-Premises Machines and Azure VMs** section. This will start you down a path of various steps to configure your WEBVMs that are running on Azure at your Primary Site.

    ![Under For On-Premises Machines and Azure VMs ,Step 1: Replicate Application is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image225.png "Step 1 selected")

4.  On **Step 1 Source** select the following inputs and then select **OK**:

-   **Source**: Azure

-   **Source Location**: Your Primary Region.

-   **Azure virtual machine deployment model**: Resource Manager

-   **Source resource group**: BCDRIaaSPrimarySite

    ![In the Source blade, fields are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image226.png "Source blade")

5.  On **Step 2 Virtual Machines** select **WEBVM1** and **WEBVM2** and select **OK**.

    ![In the Select virtual machines blade, the checkboxes for WebVM1 and WEBVM2 are selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image227.png "Select virtual machines blade")

6.  On the **Configure settings** blade, first select the **Target location** as your **Secondary** Site Azure Region.

    ![In the Configure settings blade, the Target location is set to Central US.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image228.png "Configure settings blade")

7.  Select **Customize**.

    ![In the Configure settings blade, the Customize button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image229.png "Configure settings blade")

8.  Update the rest of the blade using the following inputs and the select **OK**:

-   **Target resource group:** BCDRIaaSSecondarySite

-   **Target virtual network:** BCDRFOVNET

-   **Target storage:** accept the new account

-   **Target storage:** accept the new account

-   **Target Availability Set:** WEBAVSET

    ![In the Configure settings blade, under General Settings and VM Setting,s fields are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image230.png "Configure settings blade")

**Note**: Double check these selections. They are critical to your Failover.

9.  Then select **Create target resources**.

    ![In the Configure settings blade, the following following Network, Storage, and Availability sets are called out: Target resource group, Target virtual network, and Target availability sets.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image231.png "Configure settings blade")

    ![Screenshot of the Create target resources button.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image232.png "Create target resources button")

10. Then select **Enable replication**.

    ![Screenshot of the Enable replication button.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image233.png "Enable replication button")

11. The Azure portal will start the deployment. This will take about 10 minutes to complete and you will receive a notification once it has deployed. 

    ![The message Enabling replication for two vm(s) says it has successfully completed.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image234.png "Enabling replication for two vm(s)")

12. Select back to the Recovery Services Vault **BCDRRSV** and you will now see that 3 items are replicated.

    ![The Recovery Services Vault blade displays with information about three successfully replicated items.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image235.png "Recovery Services Vault blade")

    ![The Infrastructure view (machines replicating to Azure) displays a diagram for Azure virtual machines(s), which includes a Primary Region and a Recovery Region. The primary region has Create storage account(s) (1), and Virtual machines 2, which connect to each other, and to Azure Site Recovery, which connects to Storage Account(s) (1). Azure Site Recovery and Storage Accounts are located in the Recovery Region.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image236.png "Infrastructure view")

13. Select the **Replicated Items** link under **Protected Items**.

    ![Under Protected Items, Replicated items is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image237.png "Protected Items section")

14. You should see three items: **OnPremVM**, **WEBVM1** and **WEBVM2**. The **OnPremVM** will show as protected and the others may still need to synchronize a bit more.

    ![Under Replicated Items, the status for WEBVM1, WEBVM2 shows 93 percent synchronized, while the status for OnPremVM is Protected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image238.png "Replicated Items section")

15. Once **WEBVM1** and **WEBVM2** have reached Protected status, you can move on to the next step.

    ![Under Replicated Items, the status for WEBVM1, WEBVM2 is now Protected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image239.png "Replicated Items section")

>**Note**: It can take up to 30 minutes for this action to complete.

16. Under the **Manage** area select **Recovery Plans (Site Recovery)**.

    ![Under Manage, Recovery Plans (Site Recovery) is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image240.png "Manage Section")

17. Select **+Recovery plan**.

    ![On the Recovery Services vault blade top menu, Add a recovery plan is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image241.png "Recovery Services vault blade")

18. On the **Create recovery plan** blade, provide the Name: **BCDRIaaSPlan** and Source: select your **Primary** site.

    ![Fields in teh Create recovery plan blade are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image242.png "Create recovery plan blade")

19. Complete the rest of the blade using the following inputs and then select **OK**:

    -   **Target:** Secondary region

    -   **Allow items with deployment model:** Resource Manager

    -   **Select Items:** Select WEBVM1 and WEBVM2

    ![Fields in the Create recovery plan blade are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image243.png "Create recovery plan blade")

20. After a moment the **BCDRIaaSPlan** Recovery plan will appear, select it to review.

    ![In the Recovery Services vault blade , BCDRIaaSPlan is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image244.png "Recovery Services vault blade")

21. When the **BCDRIaaSPlan** loads **notice** that it shows **2 VMs in the Source** which is your **Primary** Site. Then select **Customize**.

    ![On the BCDSRV blade top menu, Customize is selected. Under Items in recovery plan, a callout points to Source, which shows two.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image245.png "BCDSRV blade")

22. Once the **BCDRIaaSPlan** blade loads, select the **ellipse** next to **All groups failover**.

    ![The All groups failover ellipses is selected in the Recovery plan blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image246.png "Recovery plan blade")

23. Select **Add pre-action**.

    ![In the Recovery plan blade, the right-click menu for All groups failover displays and Add pre-action is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image247.png "Recovery plan blade")

24. On the **Insert action** blade, select **Script** and then provide the name: **ASRSQLFailover.** Ensure that your Azure Automation account is selected and then chose the Runbook name: **ASRSQLFailover**. Select **OK**.

    ![Fields in the Insert action blade are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image248.png "Insert action blade")

25. Once the **BCDRIaaSPlan** blade loads, select the **ellipse** next to **Group 1: Start**.

    ![In the Recovery plan blade, the ellipses next to Group 1: Start is called out.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image249.png "Recovery plan blade")

26. Select **Add post action**.

    ![In the Recovery plan blade, the Group 1: Start right-click menu displays, and Add post action is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image250.png "Recovery plan blade")

27. On the **Insert action** blade, select **Script** and then provide the name: **ASRWEBFailover.** Ensure that your Azure Automation account is selected and then chose the Runbook name: **ASRWEBFailover**. Select **OK**.

    ![In the Insert Action blade, fields are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image251.png "Insert Action blade")

28. Make sure that your **Pre-steps** are running under **All groups failover** and the **Post-steps** are running under the **Group1: Start.** Select **Save**.

    ![In the Recovery plan blade, both instances of Script: ASRFSQLFailover are called out under both All groups failover: Pre-steps, and Group 1: Post-steps.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image252.png "Recovery plan blade")

29. After a minute, the portal will provide a successful update notification. This means that your machines are fully configured and ready to Failover and Back between the **Primary** and **Secondary** regions.

    ![The Updating recovery plan message shows that the update was successfully completed.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image253.png "Updating recovery plan message")

### Task 4: Configure PaaS for region to region failover

In this task you will deploy the website to App Services using Visual Studio, migrate a database to Azure SQL Database and configure it for high-availability using an Azure SQL Database Failover Group. The Traffic Manager will be used to direct traffic to the closest front end to the user. If there is a failover of the database it will happen transparently, and the users will never know there was an outage. There is no reconfiguration required for this to function properly.

1.  From the **LABVM**, open the Azure portal at: <https://portal.azure.com>.

2.  Navigate to the **BCDRPaaSPrimarySite** resource group. Select the SQL Server resource.

    ![In the Resource group blade, under Name, the SQL Server Resource is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image254.png "Resource group blade")

3.  This server will host your SQL Database for the Contoso Insurance Web App. Select **Properties** under the **Settings** area.

    ![Under Settings, Properties is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image255.png "Settings section")

4.  Copy the name of the SQL Server to notepad. Also, notice that the Server Admin Login is the same. Save this file as **C:\\HOL\\Deployments\\SQLSERVER.txt**. 

    ![In the SQL server blade, the Server Name is called out. Under Server Admin Login, mcadmin is called out as well.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image256.png "SQL server blade")

    ![The FQDN name displays in Notepad. ](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image257.png "Notepad")

5.  Use the Start menu to launch **Microsoft SQL Server Management Studio 17** and connect to the local instance of SQL Server (Located in the Microsoft SQL Server Tools 17 folder).

    ![In the Start menu, Microsoft SQL Server Management Studio 17 is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image172.png "Start menu")

6.  Select **Connect** to Sign On to the Local **SQLEXPRESS** on **LABVM**.

    ![In the Connect to Server dialog box fields are set to the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image258.png "Connect to Server dialog box")

7.  Right-click **Databases**, then select **Restore Database**.

    ![In the Object Explorer tree view, Databases is selected, and in its right-click menu, Restore Database is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image259.png "Object Explorer")

8.  Select **Device** and then the **Ellipse**.

    ![Under Source, the Device radio button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image260.png "Source section")

9.  Select **Add**.

    ![Under Select backup devices, the Add button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image261.png "Select backup devices section")

10. Navigate the folder menu and locate the **C:\\HOL\\Database** folder and then select on **ContosoInsurnace.bak** and then **OK**.

    ![In SQL Server, in the tree view, Databases is selected. In the right pane, ContosoInsurance.bak is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image262.png "SQL Server")

11. Select **OK**.

    ![In the Select backup devices dialog box, a callout points to the Backup media location.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image263.png "Select backup devices dialog box")

12. Select **OK** to restore the ContosoInsurance database.

    ![Screenshot of the Restore Database - ContosoInsurance dialog box.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image264.png "Restore Database - ContosoInsurance dialog box")

13. Select **OK** at the restored successfully prompt.

    ![The Microsoft SQL Server Management Studio pop-up shows that the ContosoInsurance database was successfully restored.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image265.png "Microsoft SQL Server Management Studio pop-up")

14. **Expand Databases** and then right-click on the **ContosoInsurance** database, select **Tasks**, then **Deploy Database to Microsoft Azure SQL Database**.

    ![In the Object Explorer tree view, ContosoInsurance is selected, and in its right-click menu, Deploy Database to Microsoft Azure SQL Database is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image266.png "Object Explorer")

15. Select **Next** on the Introduction screen.

    ![Screenshot of the Deploy Database to Microsoft Azure SQL Database introduction.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image267.png "Deploy Database to Microsoft Azure SQL Database introduction screen")

16. Select **Connect**.

    ![Screenshot of the Specify Target Connection screen.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image268.png "Specify Target Connection screen")

17. In the **Connect to SQL Server** screen, copy the name of your Azure SQL Server from the **SQLSERVER.TXT**. Change the Authentication to **SQL Server Authentication** and enter the credentials for the server then select **Connect.**

    -   **Login:** mcwadmin

    -   **Password:** demo\@pass123

        ![Fields in the Connect to Server dialog box display with the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image269.png "Connect to Server dialog box")

18. Update the remainder of the Deployment Settings screen using these inputs and then select **Next**:

    -   **Edition of Azure SQL database:** Standard

    -   **Maximum database size (GB):** 1

    -   **Service Objective:** S1

        ![Under Specify Target Connection, fields display the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image270.png "Specify Target Connection section")

19. Select **Finish** and allow the Database to be migrated to Azure.

    ![Screenshot of the Verify Specified Settings screen.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image271.png "Verify Specified Settings screen")

    ![An Importing database progress bar displays with individual steps below showing their progress.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image272.png "Importing database progress")

20. Select **Close** once the database has been migrated to Azure SQL Database.

    ![On the Operation Complete screen, under Summary, all steps in the database import process have results of Success.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image273.png "Operation Complete screen")

21. Move back to the Azure portal on **LABVM**. Open the **BCDRPaaSPrimarySite** resource group and notice that there is a new SQL Database called **ContosoInsurance**.

    ![In the Resource group blade, a callout points to ContosoInsurance.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image274.png "Resource group blade")

22. Select **ContosoInsurance** to open the resource then select **Show database connection strings**.

    ![Under Connection strings, the link for Show database connection strings is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image275.png "Connection strings setting")

23. Select the **Copy to clipboard** link to capture the connection string and then paste it to your **SQLSERVER.TXT** file.

    ![On the ADO.NET tab, the database connection string is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image276.png "ADO.NET tab")

    ![The Database connection string for the newly migrated ContosoInsurance SQL Database displays in Notepad.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image277.png "Notepad")

24. In the Azure portal, move back to the **BCDRPaaSPrimarySite** resource group and then select the **SQL Server** resource.

    ![In the Resource group blade, the SQL Server resource is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image278.png "Resource group blade")

25. Under **Settings**, select **Failover groups**.

    ![In the SQL server blade, under Settings, Failover groups is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image279.png "SQL server blade")

26. Select **+Add group**.

    ![In the SQL server blade, Add group is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image280.png "SQL server blade")

27. Complete the **Failover group** blade using these inputs and then select **Create:**

    -   **Failover group name:** Enter a lowercase unique name 3-24 characters using bcdrpassfogxxx.

    -   **Secondary Server:** Select the secondary SQL Server from your BCDRPaaSSecondarySite.

    -   **Database within the group:** ContosoInsurnace

        ![Fields in the Failover group blade display the previously defined settings, and in the Databases blade, the checkbox for the SQL Server database is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image281.png "Failover group and Databases blades")

28. The portal will submit a deployment.

    ![A Deployment in progress message displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image282.png "Deployment in progress message")

29. The portal will update once the Failover group has been deployed. Select the **group**.

    ![In the SQL Server blade, the failover group displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image283.png "SQL Server blade")

30. The Failover group (FOG) portal will appear. Copy the name of the Listener endpoint to your clipboard and then copy this to your **SQLSERVER.TXT** file.

    ![On the Configuration details tab, a map displays with dots marking the two server locations.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image284.png "Configuration details tab")

    ![In Notepad, a callout points to the Fog Listener Endpoint.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image285.png "Notepad")

31. In the **SQLSERVER.TXT** file, **update the server name** in the connection string with the name of the FOG listener endpoint. Also, **change the user name and password** to the credentials for the SQL Server:

    -   **Username:** mcwadmin

    -   **Password:** demo\@pass123

    ![The original SQLServer.TXT file displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image286.png "SQLServer.TXT file")

    ![The updated SQLServer.TXT file displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image287.png "SQLServer.TXT file")

32. The new connection string will be used for the Web App. This will ensure that when the SQL Database is failed over that the server is always pointing to the Failover group. Open the Web App in the **BCDRPaaSPrimarySite** resource group.

    ![The New Connection String for the Web App is called out.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image288.png "New Connection String")

33. Select the **URL**. The empty Web App will appear.

    ![The new URL link is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image289.png "URL")

    ![On the Microsoft Azure app service tab, a message says that your app service app is up and running.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image290.png "Microsoft Azure app service tab")

34. Under **Settings**, select **Application Settings**.

    ![Under Settings, Application settings is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image291.png "Settings section")

35. Scroll down to the **Connection strings** settings and add a new connection string using the following inputs, then select **Save**.

    -   **Name:** PolicyConnect

    -   **Value:** Paste in the updated string you created with the Failover group name from the SQLSERVER.TXT file.

        ![The connection string for PolicConnect displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image292.png "Connection string")

>**Note**: You must use the Name **PolicyConnect**. This is the name that the recognized by the Application in the source code.

36. Repeat the same procedure on the Web App located in the **BCDRPaaSSecondarySite** resource group using the same connection string:

    ![The New Connection String for the Web App is called out.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image293.png "New Connection String")

    -   **Name:** PolicyConnect

    -   **Value:** Paste in the updated string you created with the Failover group name from the SQLSERVER.TXT file.

37. On the LABVM open **Visual Studio**. You will be required to login to Visual Studio. If you don't have an account you can create a free account following the prompts.

    ![Visual Studio 2017 from the start menu displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image294.png "Visual Studio 2017")

38. Select **File**, **Open**, and **Project/Solution**.

    ![In Visual Studio, File / Open / Project/Solution is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image295.png "Visual Studio")

39. Open the Solution located at **C:\\HOL\\WebApp\\ContosoInsurnace.sln**.

    ![In the Open Project window, contosoinsurance.sln is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image296.png "Open Project")

40. Locate the ContosoInsurance application in the Solution Explorer on the right-hand area of Visual Studio.

    ![In Solution Explorer, ContosoInsurance is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image297.png "Solution Explorer")

>**Note:** If for some reason the Solution Explorer is not seen you can select **View -\> Solution Explorer** on the Menu bar of Visual Studio.

41. Right-click the **ContosoInsurance** Application and select **Publish**.

    ![In Solution Explorer, the right-click menu for ContosoInsurance displays, and Publish is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image298.png "Solution Explorer")

42. On the **Publish** screen select **Microsoft Azure App Service** and then **Select Existing** and finally **Publish**.

    ![On the Publish screen, Microsoft Azure App Service is selected. The radio button for Select Existing is selected as well.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image299.png "Publish screen")

43. On the App Service screen select the Web App under the **BCDRPaaSPrimarySite**. Then select **OK**.

    ![On the App Service screen, the web app under BCDRPaaSPrimarySite is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image300.png "App Service screen")

44. Visual Studio will build the app and then publish to your Web App. The browser should open to the application.

    ![The Contoso Insurance PolicyConnect webpage displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image301.png "Contoso Insurance PolicyConnect webpage ")

45. Select the **Current Policy Offerings** button, and the page should load with data showing. This means that you have successfully implemented the Web App and it has connected to the Failover Group database.

    ![The Index webpage displays the insurance options.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image302.png "Index webpage")

46. Right-click the **ContosoInsurance** Application and select **Publish**.

    ![In Solution Explorer, the right-click menu for ContosoInsurance displays, and Publish is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image298.png "Solution Explorer")

47. On the publish screen select **Create new profile**.

  ![On the Publish screen, New Profile is Selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image299.0.png "Publish screen New Profile")

48. On the **Publish** screen select **Microsoft Azure App Service** and then **Select Existing** and finally **Publish**.

    ![On the Publish screen, Microsoft Azure App Service is selected. The radio button for Select Existing is selected as well.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image299.png "Publish screen")

49. This time choose the Web App from the **Secondary** Site running in the **BCDRPaaSSecondarySite**. Select **OK**.

    ![On the App Service screen, the web app under BCDRPaaSSecondarySite is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image305.png "App Service screen")

50. Visual Studio will build the app and then publish to your Web App. The browser should open to the application.

    ![The Contoso Insurance PolicyConnect webpage displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image306.png "Contoso Insurance PolicyConnect webpage")

51. Select the **Current Policy Offerings** button, and the page should load with data showing. This means that you have successfully implemented the Web App, and it has connected to the Failover Group database.

    ![The Index webpage displays the insurance options.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image307.png "Index webpage")

52. Close Visual Studio and move back to the Azure Portal. The next step will be to deploy a Traffic Manager for this PaaS implementation. Select **+Create a resource, Networking then Traffic Manager profile** in the Azure portal.

    ![In the Azure Portal, under Azure Marketplace, Networking and Traffic Manager profile are both selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image213.png "Azure Portal")

53. Complete the **Create Traffic Manager profile** using the following inputs, then select **Create**:

    -   **Name:** Unique name all lowercase using bcdrpaasxxx.

    -   **Routing method:** Performance

    -   **Resource group:** Use existing / BCDRPaasPrimarySite

    -   **Location:** Automatically assigned based on the BCDRPaaSPrimarySite.

        ![In the Create Traffic Manager profile blade, fields display the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image308.png "Create Traffic Manager profile blade")

54. Once the Traffic Manager profile is created, open it in the Azure portal. Notice the DNS name. This is the URL that you will use to connect to the application. Once configured this DNS name will always respond and doesn't matter which location is responding or where if the current primary database is located. Since the **Performance** routing method was selected the closest site to the end users will be calculated and they will be sent to that location. If for some reason one of the sites is down the other will service all requests.

    ![The Routing method of Performance has a callout pointing to it.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image309.png "Routing method")

55. Select **Configuration** and review the configurations.

    ![Under Settings, Configuration is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image216.png "Settings section")

56. Next select **Endpoints**. This is where you will configure the two external load balancers that are located your **Primary** and **Secondary** sites.

    ![Under Settings, Endpoints is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image217.png "Settings section")

57. Select **+Add**.

    ![In the Traffic Manager profile blade, the Add button ](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image310.png "Traffic Manager profile blade")

58. Complete the **Add endpoint** using the following inputs and then select **OK**

    -   **Type:** Azure endpoint

    -   **Name**: BCDRPaaSPrimarySite

    -   **Target resource type**: App Service

    -   **Target resource**: Choose an app service.

    -   **Resource:** BCDRPrimarySiteContosoInsurancexxx in the BCDRPaaSPrimarySite

    ![In the Add endpoint blade, fields display the following settings: Type, Azure endpoint; Name, BCDRPaaSPrimarySite; Target resource type, App Service; Target resource, Choose an app service. In the Resource blade, the BCDRPaaSPrimarySite is called out.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image311.png "Add endpointand Resource blades")

59. Select **+Add**. Notice that the Primary endpoint was created as **"Enabled"**.

    ![The Add button is selected in the Traffic Manager profile blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image312.png "Traffic Manager profile blade")

60. Complete the **Add endpoint** using the following inputs and then select **OK**:

    -   **Type:** Azure endpoint

    -   **Name**: BCDRPaaSSecondarySite

    -   **Target resource type**: App Service

    -   **Target resource**: Choose an app service.

    -   **Resource:** BCDRSecondarySiteContosoInsurancexxx in the BCDRPaaSSecondarySite

        ![In the Add endpoint blade, fields display the previously defined settings.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image313.png "Add endpoint blade")

61. Once the second endpoint has been added select **Overview**. The Traffic Manager will monitor the Endpoints and if the **Primary** or **Secondary** site moves to a **Monitor Status** of **Degraded**, then the Traffic Manager will direct traffic only to the other site until the service is restored. The current **Monitor Status** shows that the **Primary** site and the **Secondary** site are **Online**. If there was an outage at the web layer for one of these sites then the site will move to **Degraded,** and the other site will service all requests.

    ![In the Overview section, a callout points to the status of enabled for the Primary and Secondary sites.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image314.png "Overview section")

>**Note:** All of this is automatic and easily configured with a vanity domain by adding a C NAME record in DNS to point to the DNS name of the Traffic manager. This would allow for a site like [www.contoso.com](http://www.contoso.com) to resolve to the DNS name of the traffic manager. The users will never know that the site is failed over or failed back as long as one site is up and the database is active in the Failover group.

62. Select the DNS name of the Traffic manager the Policy Connect web application will load. This is connecting to one of the two Web Apps running in the **Primary** Site or **Secondary** Site and talking to the Azure SQL Database Failover Group primary replica using the SQL FOG Listener.

    ![The DNS name link is called out.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image315.png "DNS name")

    ![The Contoso Insurance PolicyConnect webpage displays with a callout pointing to the URL in the address bar.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image316.png "Contoso Insurance PolicyConnect webpage")

## Exercise 4: Simulate failovers 

Duration: 75 minutes

Now, that your applications have been made ready for high-availability and BCDR you will now simulate their capabilities. First, you will Failover the **Azure IaaS environment** from your **Primary** to **Secondary** Region. Next, you will migrate the **On-Premises** environment to Azure. The **PaaS** environment will be tested to ensure that failing over the database doesn't cause an outage to the application. Finally, you will failback the Azure IaaS environment from the **Secondary** site to the **Primary** site.

### Task 1: Failover Azure IaaS region to region

![The Azure IaaS region to region failover diagram displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image317.png "Azure IaaS region to region failover diagram")

1.  Using the Azure portal, open the **BCDRIaaSPrimarySite** resource group. Locate the Traffic Manager DNS URL and select it to ensure that the application is up and running form the Primary Site. Pin the Traffic manager to your dashboard for easy access to this URL or make a favorite in your browser.

    ![The DNS name link is called out.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image318.png "DNS name")

2.  From the Azure portal, open the **BCDRRSV** Recovery Services Vault located in the **BCDRAzureSiteRecovery** resource group.

    ![Screenshot of the BCDRSRV Recovery Services Vault tile.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image319.png "BCDRSRV Recovery Services Vault tile")

3.  Select **Recovery Plans (Site Recovery)** in the **Manage** area.

    ![Under Manage, Recovery Plans (Site Recovery) is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image320.png "Manage section")

4.  Select **BCDRIaaSPlan**.

    ![In the Recovery Services vault blade, BCDRIaaSPlan is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image244.png "Recovery Services vault blade")

5.  Select **More** and then **Failover**.

    ![In the BCDRIaaSPlan blade, the ellipses right-click menu displays and Failover is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image321.png "BCDRIaaSPlan blade")

6.  On the warning about No Test Failover, select **I understand the risk, Skip test failover**.

    ![A warning displays that no test failover has been done in the past 180 days, and recommends that you do one before a failover. At the bottom, the I understand and skip test failover checkbox is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image322.png "Failover warning")

7.  Review the Failover direction. Notice that **From** is the **Primary** site, and **To** is the **Secondary** site. Select **OK**.

    ![Callouts in the Failover blade point to the From and To fields.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image323.png "Failover blade")

8.  After the Failover is initiated close the blade and move to **Site Recovery Jobs**. Select the **Failover** job to monitor the progress.

    ![Failover is selected in the Site Recover jobs blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image324.png "Site Recover jobs blade")

9.  You can monitor the progress of the Failover from here.

    ![Screenshot of the Site Recover jobs blade with progress status displaying.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image325.png "Site Recover jobs blade")

>**Note:** Do not make any changes to your VMs in the Azure portal during this process. Allow ASR to take the actions and wait for the failover notification prior to moving on to the next step. You can open another portal view in a new browser tab and review the output of the Azure Automation Jobs, by opening the jobs and selecting Output.

   ![Output is selected on the Job blade, and information displays in the Output blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image326.png "Job and Output blades")

10. Once the job has finished, it should show as successful for all tasks.

    ![Under Job, the status for the job steps all show as successful.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image327.png "Job status section")

11. There is also details of the failover VMs shown. Notice how they are running the **BCDRFOVNET**. This is the failover Virtual Network running in the **Secondary** Site.

    ![WEBVM1 and its network / subnet and Recovery point information are selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image328.png "Environment Details blade")

12. Select **Resource groups** and select **BCDRIaaSPrimarySite**. Locate the **WEBVM1** in the resource group and select to open.

    ![WEBVM1 is selected in the Resource group blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image329.png "Resource group blade")

13. Notice that it currently shows as **Status: Stopped (deallocated).** This shows that failover has stopped the VMs at the **Primary** site.

    ![A callout points to the Status of Stopped (deallocated) in the Virtual machine blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image330.png "Virtual machine blade")

>**Note:** Do not select Start! This is a task only for ASR.

14. Move back to the Resource group and select the **WWWEXTLB-PIP** Public IP address. Copy the DNS name and paste it into a new browser tab.

    ![On the Public IP Address blade, a callout points to the DNS name address.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image331.png "Public IP Address blade")

15. The site will be unreachable at the Primary location.

16. In the Azure portal, move to the **BCDRIaaSSecondarySite** resource group. Locate the **WEBVM1** in the resource group and select to open.

    ![WEBVM1 is selected in the Resource group blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image332.png "Resource group blade")

17. Take note that the **WEBVM1** in the **Secondary** site is running.

    ![In the Virtual Machine blade, a callout points to the status of WEBVM1, which is now running.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image333.png "Virtual Machine blade")

18. Move back to the Resource group and select the **WWWEXTLB-PIP** Public IP address. Copy the DNS name and paste it into a new browser tab.

    ![On the Public IP Address blade, a callout points to the DNS name address.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image334.png "Public IP Address blade")

19. The Application running on the WEBVM1 and WEBVM2 is now responding from the **Secondary** Site. Make sure to select the current Policy Offerings to ensure that there is connectivity to the SQL Always On group that was also failed over in the background.

    ![The Contoso Insurance PolicyConnect webpage displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image335.png "Contoso Insurance PolicyConnect webpage ")

    ![The Index webpage displays the insurance options.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image336.png "Index webpage")

20. Using the Azure portal to locate the **BCDRIaaS** **Traffic Manager** profile in the **BCDRIaaSPrimarySite** resource group. Notice that the Monitor Status has moved to **Degraded** for the **Primary** Site and moved to **Online** for the **Secondary** site.

    ![In the Traffic Manager profile blade, a callout points to the degraded status of the primary site.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image337.png "Traffic Manager profile blade")

21. Select **DNS Name URL**. The site load immediately and is failed over. The users will always be using this DNS URL, so there is no change in how they access the site even though it is failed over. There **will** be downtime as the failover happens, but once the site is back online the experience for them will be no different than when it is running in the **Primary** site.

    ![The Contoso Insurance PolicyConnect webpage displays with a callout pointing to the URL in the address bar.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image338.png "Contoso Insurance PolicyConnect webpage")

**Optional task**: If you wish, you can RDP to **SQLVM3** and open the SQL Management Studio to review the Failed over **BCDRAOG**. You will see that **SQLVM3** which is running the **Secondary** site is now the Primary Replica.

22. Now, that you have successfully failed over you need to prep ASR for the Failback. Move back to the **BCDRSRV** Recovery Service Vault using the Azure portal. Select Recovery Plans on the ASR dashboard.

    ![The Recovery Plans tile displays 1.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image339.png "Recovery Plans tile")

23. The BCDRIaaSPlan will show as **Failover completed.** Select the Plan.

    ![In the Recovery Plans blade, BCDIaaSPlan has the current job status of failover completed.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image340.png "Recovery Plans blade")

24. Notice that now 2 VMs are show in the **Target**.

    ![In the Recovery blade, a callout points to the Target tile with the number 2.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image341.png "Recovery blade")

25. Select **More**, then select **Re-protect**.

    ![The More ellipses is selected in the Recovery blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image342.png "Recovery blade")

26. On the **Re-protect** screen review the configuration and then select **OK**.

    ![Screenshot of the Re-protect blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image343.png "Re-protect blade")

27. The portal will submit a deployment. This process will take some time, by first committing the failover and then synchronizing the WEBVM1 and WEBVM2 back to the Primary Site. Once this process is complete, then you will be able to Fail back to the Primary.

    ![A Submitting deployment, and deployment in progress messages display.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image344.png "Submitting deployment, and deployment in progress")

>**Note:** You will perform the Failback later in the HOL, so it is safe to move on to the next task. You can check the status of the Re-protect using the Site Recovery Jobs area of the BCDRSRV.

![In the Recovery blade, Reprotect has a status of In progress.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image345.png "Recovery blade")

### Task 2: Migrate the on-premises VM to Azure IaaS

![The on-premises VM to Azure IaaS migration solution has on-premises, azure platform, and secondary region sections. On-premises has a hyper-v host and a linux on-premises virtual machine. Azure Platform uses Azure Site Recovery. The secondary region has an on-premises Linux VM as well.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image346.png "On-premises VM to Azure IaaS migration solution")

1.  From the Azure portal, open the **BCDRRSV** Recovery Services Vault located in the **BCDRAzureSiteRecovery** resource group.

    ![Screenshot of the BCDRRSV Recovery Services Vault tile.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image319.png "BCDRRSV Recovery Services Vault")

2.  Open the **BCDRSRV** and select **Replicated Items** under the **Protected Items** area. Make sure that **OnPremVM** shows up ad **Replication Heath**: **Healthy.** Select **OnPremVM.**

    ![Under Protected Items, Replicated items is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image347.png "Protected Items section")

    ![The OnPremVM status is Healthy.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image348.png "OnPremVM status")

3.  Right-click **OnPremVM** and then select **Failover**.

    ![The OnPremVM Healthy status right-click menu has Failover selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image349.png "Failover option")

4.  Select you understand the risk without a Test Failover and then on the Failover blade notice that the **From** is **OnPremHyperVSite** and the **To** is **Microsoft Azure**. Select **OK**.

    ![Callouts in the Failover blade point to both the From and To fields.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image350.png "Failover blade")

5.  The Azure portal will provide a notification that the Failover is starting.

    ![The Starting Failover notification explains that the operation is in progress.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image351.png "Starting Failover notification")

6.  Using Site Recovery Jobs, you can watch the progress of the failover.

    ![In the Properties section, the status of the various jobs display.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image352.png "Properties section")

7.  After the Failover Status is Successful, move back to **Replicated Items** in the **BCDRSRV** and right-click **OnPremVM**. Select **Complete Migration**.

    ![The OnPremVM Healthy status right-click menu has Complete Migration selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image353.png "Complete Migration option")

8.  Review the Complete Migration blade and then select **OK**.

    ![Screenshot of the Complete Migration blade message.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image354.png "Complete Migration blade")

    ![The Completing Migration notification explains that the operation is in progress.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image355.png "Completing Migration notification")

9.  Wait for this process to finish prior to continuing.

    ![In the Recovery group blade, OnPremVM is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image356.png "Recovery group blade")

10. Once the Migration is completed, you can move over to the **BCDRIaaSSecondarySite** Resource group and locate and select the **OnPremVM**.

    ![In the Resource group blade OnPremVM is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image357.png "Resource group blade")

11. Review the details of the VM. Notice that it is running in the **BCDRFOVNET** virtual network in the **WEB** Subnet.

    ![In the Virtual Machine blade, a callout points to the status of OnPremVM, which is running, and the Virtual network/subnet link is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image358.png "Virtual Machine blade")

12. Select the **Networking** link. Notice the IP address of the VM.

    ![In the Virtual machine blade, under Settings, Networking is selected, and a callout is pointing to the Private IP address.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image359.png "Virtual machine blade")

13. Remote desktop into **BCDRDC1**. Use the Server Manager to disable the **IE Enhanced Security Configuration**.

    ![In the Internet Explorer Enhanced Security Configuration dialog box, Administrators and Users are both set to Off.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image360.png "Internet Explorer Enhanced Security Configuration dialog box")

14. Point the browser of BCDRDC1 at the IP address of the **OnPremVM**. And it should load the sample web application.

    http://172.16.1.?/bcdr.php

    ![A sample webpage displays a message saying that you are connected successfully to MySQL.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image109.png "Sample Webpage")

15. Your OnPremVM has been successfully migrated to Azure.

**Optional Task**: If you wish, you can Remote Desktop back to the HYPERVHOST, and you will see that the migrated VM has shutdown.

### Task 3: Failover and failback Azure PaaS

![Diagram of the Azure PaaS failover and failback solution.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image361.png "Azure PaaS failover and failback solution")

1.  Using the Azure portal, open the **BCDRPaaSPrimarySite** resource group. Locate the Traffic Manager profile and then click the URL to ensure that the application is running.

    ![The DNS name link is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image362.png "DNS name link")

2.  Once you are sure that the website is active and connecting to the database, move back to the BCDRPaaSPrimarySite resource group. Select **SQL Server** resource.

    ![In the Resource group blade, the SQL Server resource is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image278.png "Resource group blade")

3.  Under **Settings**, select **Failover groups**.

    ![In the SQL Server blade, under Settings, Failover groups is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image279.png "SQL Server blade")

4.  Select the **Failover Group**.

    ![In the SQL Server blade, the Failover Group is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image363.png "SQL Server blade")

5.  Along the top of the page, select the **Failover** button. Then select **Yes** to confirm.

    ![The Failover button is selected above the Configuration details tab.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image364.png "Configuration details tab")

    ![A Warning pop-up explains that the action will make all secondary databases the primary server.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image365.png "Warning pop-up")

6.  The portal will notify you of the **Failover in progress**.

    ![Above the Configuration details tab, a message says Failover in progress.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image366.png "Configuration details tab")

7.  While, this is happening move back to your browser tab with the Contoso Insurance application is running on using the PaaS Traffic Manager link. Attempt to use the application. You should see no difference during the failover, but there could be some slowdown in the responses from the web pages that access the database.

8.  After a few minutes, move back to the Azure portal to the page where you performed the Failover. You should see that the Failover has completed and the Server running in the Secondary site will now show as the Primary replica. Also, there should be a notification from the Azure portal.

    ![The primary server name is called out.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image367.png "Primary server")

    ![Screenshot of the Failover group failover succeeded message.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image368.png "Failover group failover succeeded message")

9.  Next, to simulate a failover of one of the Azure regions you will stop the Primary Web App. Move to the **BCDRPaaSPrimarySite** resource group. Select the **Web App.**

    ![Screenshot of the Resource group list, with the web app selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image369.png "Resource group list")

10. Select the URL of the **Web App**. The browser should open to the direct link to the Application.

    ![The URL link is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image370.png "URL link")

    ![The Contoso Insurance PolicyConnect webpage displays with a callout pointing to the URL in the address bar.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image371.png "Contoso Insurance PolicyConnect webpage")

11. Back in the Azure portal, select **Stop** to terminate the Web App. Select **Yes** to confirm.

    ![The Stop button is selected in the App Service blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image372.png "App Service blade")

    ![In the Store web app section, you are asked to confirm you want to stop the web app, and the Yes button is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image373.png "Store web app section")

12. The Web App will now show as stopped. Select the **URL** again, and notice that the Web App shows as stopped.

    ![An Error message displays in the App Service blade, saying that the web app is stopped.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image374.png "App Service blade")

    ![An error message states that the web app is stopped.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image375.png "Error message")

13. Move back to the **BCDRPaaS Traffic Manager** profile and review the **Monitor Status**.

    ![A callout points to the Monitor Status, which is Online.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image376.png "Monitor Status")

14. Select the **DNS Name** URL.

    ![Screenshot of the DNS name URL link.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image362.png "DNS name URL link")

15. Notice that the website loads as normal. You can select refresh or F5, and you will always get back to the site.

    ![The Contoso Insurance PolicyConnect webpage displays with a callout pointing to the URL in the address bar.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image377.png "Contoso Insurance PolicyConnect webpage")

16. In the current configuration, you are completely failed over to the **Secondary** site. There were no configurations for you to do and this was completely transparent to the user of the application.

17. Move back to the **BCDRPaaSPrimarySite** and start the Web App.

    ![Screenshot of a Successfully started web app message.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image378.png "Successfully started message")

18. Open the **BCDRPaaS** Traffic Manager again, and the Monitor Status should show both as Online again.

    ![In the BCDRPaaS Traffic Manager, both web apps show as online.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image379.png "BCDRPaaS Traffic Manager")

19. Move back to your **BCDRPaaSPrimarySite** resource group and select through to your **SQL Server**. Select the **Failover group**. Select **Failover** and **Confirm**.

    ![A message above the Configuration details tab says a failover is in progress, and a map displays below with two dots connected with a dotted line.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image380.png "Configuration details tab")

20. Once the Failover back to the **Primary** site is completed, the **SQL Server** in the **Primary** site will show as the **Primary**.

    ![A list of servers display, with the primary server name called out.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image381.png "Servers")

### Task 4: Failback Azure IaaS region to region

![Diagram of the Azure IaaS region to region failback solution.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image3.png "Azure IaaS region to region failback solution")

1.  Open the **BCDRSRV** and select **Replicated Items** under the **Protected Items** area. Make sure that **WEBVM1** and **WEBVM2** show up ad **Replication Heath**: **Healthy.**

    ![Under Protected items, Replicated items is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image347.png "Protected items section")

    ![In the Recovery Services vault blade, a callout points to WEBVM1 and WEBVM2\'s replication health, which is Healthy. A second callout points to their Active location, which is Central US.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image382.png "Recovery Services vault blade")

2.  Select **Recovery Plans** under the **Manage** area.

    ![Under Manage, Recovery Plans (Site Recovery) is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image383.png "Manage section")

3.  Select the **BCDRIaaSPlan**.

    ![Screenshot of the BCDRIaaSPlan option.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image384.png "BCDRIaaSPlan option")

4.  Notice that the VMs are still at the Target, since they are Failed over to the Secondary Site. Select **More**, and then **Failover**.

    ![In the BCDRSRV blade, the right-click ellipses menu has Failover selected. Under Items in recovery plan, a callout points to the Target tile, with 2.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image385.png "BCDRSRV blade")

5.  On the warning about No Test Failover, select **I understand the risk, Skip test failover**.

    ![A warning displays that no test failover has been done in the past 180 days, and recommends that you do one before a failover. At the bottom, the I understand and skip test failover checkbox is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image322.png "Failover warning")

6.  Review the Failover direction. Notice that **From** is the **Secondary** site and **To** is the **Primary** site. Select **OK**.

    ![In the Failover blade, callouts point to the From and To fields.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image386.png "Failover blade")

7.  After the Failover is initiated close the blade and move to **Jobs**, then **Site Recovery Jobs**. Select the **Failover** job to monitor the progress.

    ![In the Site Recovery jobs blade, Failover is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image324.png "Site Recovery jobs blade")

8.  Once the job has finished, it should show as successful for all tasks.

    ![Under Job, the list of all jobs have a status of successful.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image327.png "Job section")

9.  There is also details of the failover VMs shown. Notice how they are running the **BCDRVNET**. This is the primary Virtual Network running in the **Primary** Site.

    ![A callout points to the bcdrvnet \\ WEB network \\ subnet in the Environment Details blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image387.png "Environment Details blade")

10. Select Resource groups and select **BCDRIaaSSecondarySite**. Locate the **WEBVM1** in the resource group and select to open.

    ![WEBVM1 is selected in the Resource group blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image388.png "Resource group blade")

11. Notice that is currently shows as **Status: Stopped (deallocated).** This shows that failover has stopped the VMs at the **Secondary** site.

    ![In the Virtual machine blade, a callout points out that the status is stopped (deallocated).](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image389.png "Virtual machine blade")

>**Note:** Do not select Start! This is a task only for ASR.

12. Move back to the Resource group and select the **WWWEXTLB-PIP** Public IP address. Copy the DNS name and paste it into a new browser tab.

    ![In the Public IP address blade, a callout points to the DNS name.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image390.png "Public IP address blade")

13. The site will be unreachable at the **Secondary** location.

14. In the Azure portal, move to the **BCDRIaaSPrimarySite** resource group. Locate the **WEBVM1** in the resource group and select to open.

    ![WEBVM1 is selected in the Resource group blade.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image391.png "Resource group blade")

15. Take note that the **WEBVM1** in the **Primary** site is running.

    ![In the Virtual machine blade, the status is now running.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image392.png "Virtual machine blade")

16. Move back to the Resource group and select the **WWWEXTLB-PIP** Public IP address. Copy the DNS name and paste it into a new browser tab.

    ![In the Public IP address blade, a callout points to the DNS name.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image393.png "Public IP address blade")

17. The Application running on the **WEBVM1** and **WEBVM2** is now responding from the **Primary** Site. Make sure to select the current **Policy Offerings** to ensure that there is connectivity to the SQL Always On group that was also failed back to primary in the background.

    ![The Contoso Insurance PolicyConnect webpage displays with a callout pointing to the URL in the address bar.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image394.png "Contoso Insurance PolicyConnect webpage")

    ![On the Index - Policy Connect tab webpage coverage options display.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image395.png "Index - Policy Connect tab webpage")

18. Using the Azure portal, locate the **BCDRIaaS** **Traffic Manager** profile in the **BCDRIaaSPrimarySite** resource group. Notice that the Monitor Status has moved to **Degraded** for the **Secondary** Site and moved to **Online** for **Primary** the site.

    ![In the Resource group blade, a callout points to the mointor status of degraded.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image396.png "Resource group blade")

19. Select the **DNS Name URL**. The site load immediately and is failed over. The users will always be using this DNS URL, so they there is no change in how they access the site even though it is failed over.

    ![Screenshot of the DNS name link.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image397.png "DNS name link")

    ![The Contoso Insurance PolicyConnect webpage displays with a callout pointing to the URL in the address bar.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image338.png "Contoso Insurance PolicyConnect webpage")

20. Now, that you have successfully failed back you need to prep ASR for the Failover again. Move back to the **BCDRSRV** Recovery Service Vault using the Azure portal. Select Recovery Plans on the ASR dashboard.

    ![A Recovery Plans tile with the number 1 displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image339.png "Recovery Plans tile")

21. The BCDRIaaSPlan will show as **Failover completed**. Select the Plan.

    ![In the BCDRSRV blade a callout points to the failover completed message.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image340.png "BCDRSRV blade")

22. Notice that now 2 VMs are shown in the **Source**.

23. Select **More**, then select **Re-protect**.

    ![In the BCDRSRV blade, the More ellipses is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image342.png "BCDRSRV blade")

24. On the **Re-protect** screen review the configuration and then select **OK**.

    ![The BCDRIaaSPlan blade displays.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image398.png "BCDRIaaSPlan blade")

25. The portal will submit a deployment. This process will take some time, by first committing the failover and then synchronizing the **WEBVM1** and **WEBVM2** back to the **Primary** Site. Once this process is complete, then you will be able to Fail over again from the **Primary** to **Secondary** site from the perspective of ASR.

26. Select **Resource groups** in the Azure portal and notice that two new Resource groups have been created with -asr in their names. These are used by ASR for the Failover and Failback, so they should be left in place.

    ![A list of Resource groups displays, two of which are: BCDRIaaSPrimarySite-asr, and BCDRIaaSPrimarySite-asr-1. ](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image399.png "Resource groups")

27. Next, you need to reset the SQL AOG environment to ensure a proper failover. To do this open Remote Desktop to **BCDRDC1** and then jump to **SQLVM1** using Remote desktop.

28. Once connected to **SQLVM1** open SQL Management Studio and Connect to **SQLVM1**. Expand the **Always On Availably Group**s and then right-click on **BCDRAOG** and then select **Show Dashboard**.

    ![In SQL Management Studio, the right-click menu for BCDRAOG (Primary) displays, and Show Dashboard is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image198.png "SQL Management Studio")

29. Notice that all the Replica partners are now Synchronous Commit with Automatic Failover Mode. You need to manually reset **SQLVM3** to be **Asynchronous** with **Manual Failover**.

    ![The Availability group dashboard displays with SQLVM3 and its properties called out.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image400.png "Availability group dashboard")

30. Right-click the **BCDRAOG** and select **Properties**.

    ![In Object Explorer, the right-click menu for BCDRAOG displays with Properties selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image401.png "Object Explorer")

31. Change **SQLVM3** to **Asynchronous** and **Manual Failover** and select **OK**.

    ![In the Availability Group Properties window, under Availability Replicas, the SQLVM3 server role is secondary, availability mode is asynchronous commit, and failover mode is manual.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image402.png "Availability Group Properties window")

32. Show the Availability Group Dashboard again. Notice that they change has been made and that the AOG is now reset.

    ![The Availability group dashboard displays, with SQLVM3 and its properties called out.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image403.png "Availability group dashboard")

>**Note:** This task could have been done using the Azure Automation script during Failback, but more DBAs would prefer a good, clean failback and then do this manually once they are comfortable with the failback.

## After the hands-on lab 

Duration: 15 minutes

There are many items that were created as a part of this lab, and they should be deleted once you no longer desire to retain the environments.

### Task 1: Disable replication in the recovery services vault

1.  To clean up the environment, you must first disable the replication of the **WEBVM1, WEBVM2** and **OnPremVM** in the **BCDRSRV** Recovery Service Vault. Open **BCDRSRV** in the Azure portal and select **Replicated Items** in the **Protected Items** area.

    ![Under Protected items, Replicated items is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image404.png "Protected items section")

2.  Select the **Ellipse** next to each replicated item and then select **Disable Replication**. On the following screen select **OK**.

    ![From the ellipses menu, Disable Replication is selected.](images/Hands-onlabstep-bystep-Businesscontinuityanddisasterrecoveryimages/media/image405.png "Disable Replication option")

3.  After this process is completed you can move on to the next task.

### Task 2: Delete all BCDR resource groups

1.  Using the Azure Portal delete each of the BCDR Resource Groups that you created:

|    |            |       
|----------|:-------------:|
| **Resource Group Name** | **Location** |
| **BCDRAzureAutomation** | Your Location |
| **BCDRAzureSiteRecovery** | Secondary |
| **BCDRIaaSPrimarySite** | Primary |
| **BCDRIaaSSecondarySite** | Secondary |
| **BCDRLabRG** | Your Location |
| **BCDROnPremPrimarySite** | Primary |
| **BCDROnPremPrimarySite-asr** | Primary |
| **BCDROnPremPrimarySite-asr-1** | Primary |
| **BCDRPaaSPrimarySite** | Primary |
| **BCDRPaaSSecondarySite** | Secondary |

You should follow all steps provided *after* attending the Hands-on lab.

