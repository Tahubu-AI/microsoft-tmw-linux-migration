# Exercise 1: Migrate a Linux VM to an Azure VM

Contoso Inc. is in the early stages of its infrastructure modernization journey, aiming to reduce datacenter overhead and improve operational agility by migrating key workloads to Azure. Among its legacy systems is a Linux-based personnel management application running on a Hyper-V virtual machine. While the application remains critical to daily operations, its underlying infrastructure is aging and difficult to scale. To preserve compatibility while gaining cloud-based resilience and manageability, Contoso has chosen a lift-and-shift migration strategy for this workload.

In this exercise, you will use the [Azure Migrate Service](https://learn.microsoft.com/azure/migrate/prepare-for-migration?view=migrate-classic) to assess, replicate, and migrate a Hyper-V–based Ubuntu VM to Azure. This process simulates the real-world steps Contoso's IT team would follow to transition legacy infrastructure into a cloud-first model.

## Objectives

After completing this exercise, you will be able to:

- Assess the readiness of a Linux VM for migration to Azure
- Configure replication of the VM using Azure Migrate
- Perform a lift-and-shift migration of the VM to Azure

## Duration

**Estimated time**: 1 Hour and 15 minutes

===

# Task 1: Test the application before migration

## Introduction

Before initiating a migration, it is critical to confirm that the source Linux VM is functioning properly in the Hyper-V environment. This includes verifying that the personnel management application is running and successfully connecting to the SQL Server database.

## Description

In this task, you will log in to the `LinuxLabVM-Ubuntu` virtual machine and test the personnel management application to ensure it is operational prior to migration.

## Success criteria

- You can connect to the Linux VM in Hyper-V Manager.
- You can run the server application from Visual Studio Code.
- You can confirm that data is returned successfully in the web app accessing the SQL database.

## Learning resources

- [Preparing for migrating Linux Virtual Machine to Azure](https://learn.microsoft.com/azure/migrate/prepare-for-migration?view=migrate-classic)

## Key tasks

1. [] Launch **Hyper-V Manager** from the desktop of your Lab VM.

    ![Screenshot of the Lab VM desktop with Hyper-V Manager highlighted and the Hyper-V Manager application open.](./media/01-Launch-HyperV.png)

2. [] Confirm that all five virtual machines are listed and show a status of **Running** in the **Virtual Machines** panel of Hyper-V Manager.

    ![In Hyper-V Manager, the five virtual machines with a running status are highlighted.](./media/02-five-VMs-Running.png)

3. [] Select the `LinuxLabVM-Ubuntu` in the list and choose **Connect** in the right-hand menu.

    ![In Hyper-V Manager, the LinuxLabVM-Ubuntu VM is highlighted in the Virtual Machines list and the Connect action is highlighted in the Actions panel.](./media/hyper-v-managed-connect-ubuntu.png)

4. [] At the Ubuntu login screen:

   - [] Select the `administrator` account
   - [] Enter `Pa$$w0rd` for the password
   - [] Select **Enter** to log in to the Ubuntu machine

5. [] Select the **Show Applications** button at the top left of the Ubuntu VM window, enter "visual studio" into the search box, and select **Visual Studio Code** from the search results.

    ![The Ubuntu desktop is displayed, with the Show Applications button highlighted. "Visual studio" is entered in the search box and Visual Studio Code is highlighted in the search results. The steps are numbers 1 - 3 to specify the order of operations.](./media/04-VSCode-Ubuntu.png)

    > Visual Studio Code will open to the `techsummit-LinuxApp` workspace.

6. [] From the **Explorer** in VS Code, open the `views.py` file. In this file, you need to update the IP address in the `connect_str` variable on line 58 to point to the `SQLPTO2022` VM in your environment.

7. [] Locate the IP address of the `SQLPTO2022` VM by opening the `VM_IPs.txt` file on the Lab VM's desktop.

    ![The Lab VM's desktop is displayed with the VM_IPs text file highlight and its contents displayed in the open file.](./media/lab-vm-desktop-vm-ips.png)

8. [] In the `VM_IPs.txt` file, copy the **private IP** address of the `SQLPTO2022` VM.

    ![The contents of the VM_IPs.txt file are displayed, with the private IP address of the SQLPTO2022 VM highlighted with a red box.](./media/vm-ips-txt-sql-private.png)

9. [] In the `views.py` file, go to **line 58** and replace the existing IP address in the `SERVER` field of the `connect_str` variable with the private IP of the `SQLPTO2022` VM.

    ![In VS Code, the views.py file is selected and highlighted in Explorer. Line 58 of views.py is highlighted and the SERVER component of the connection string is highlighted along with the pasted IP address.](./media/05-VM-IP.png)

10. [] Save the `views.py` file.

11. [] Open `runserver.py` from the VS Code Explorer and run it by selecting the **Run Python File** button in the top right corner of the file tab.

    ![In VS Code, the runserver.py file is selected and highlighted in Explorer. The Run Python File button is highlighted.](./media/vs-code-run-python-file-runserver-py.png)

12. [] In the **Terminal** panel of VS Code, select the **Running on** link (`http://localhost:5555`) to launch the application in your default browser.

    ![In the Terminal panel in VS Code, the Running on link is highlighted.](./media/06-runserver.png)

13. [] In the web app, select the **People** tab and verify that a list of people from the `Person.Person` table is displayed.

    ![In the AdventureWorksPTO application, the People tab is selected and highlighted and the list of people from the Person.Person table is highlighted on the page.](./media/08-PeoplePage.png)

    > If no data appears, double-check the IP address in `views.py` and ensure the `SQLPTO2022` VM is running and accessible.

14. [] Close the personnel management web app browser.

15. [] Close the `LinuxLabVM-Ubuntu` VM window.

===

# Task 2: Prepare the Hyper-V Host

## Introduction

Azure Migrate requires some preparation of the Hyper-V host before beginning a migration. For this lab, those preparation steps have been consolidated into a PowerShell script that enables required services and sets permissions. Running this script ensures the host is ready to support VM replication and migration.

## Description

In this task, you will run the Azure Migrate preparation script using PowerShell on the Hyper-V host (your Lab VM).

## Success criteria

- You have downloaded the Azure Migrate Hyper-V preparation script  
- You have executed the `MicrosoftAzureMigrate-Hyper-V.ps1` script on the Hyper-V host without errors  
- You have reset the network connectivity category to **Public**

## Learning resources

- [Preparing for migrating Linux Virtual Machine to Azure](https://learn.microsoft.com/azure/migrate/prepare-for-migration?view=migrate-classic)

## Key tasks

1. [] On the Lab VM, open the **Search Bar** and enter `"powershell"`.

    ![The Windows Search Bar is highlighted on the Task Bar with powershell entered into the search box.](media/lab-vm-search-bar-powershell.png)

2. [] In the search results, select **Windows PowerShell** under **Apps** and select **Run as administrator**.

    > **Important**: Do not select **Windows PowerShell ISE**.

    ![In the Windows Search window, PowerShell is highlighted and selected under Apps and Run as administrator is highlighted.](./media/10-ps-admin.png)

3. [] In the PowerShell terminal, set the network category to **Private** by running:

    ```powershell
    Set-NetConnectionProfile -NetworkCategory Private
    ```

4. [] Verify the change by running:

    ```powershell
    Get-NetConnectionProfile
    ```

    ![In the PowerShell terminal, the NetworkCategory setting of Private is highlighted.](./media/11-Network-Private.png)

5. [] Open a browser on the Lab VM and navigate to <https://aka.ms/migrate/hyperv/script> to download the `MicrosoftAzureMigrate-Hyper-V.ps1` script.

    ![Screenshot of the script download URL entered into the address bar in Microsoft Edge.](./media/09-download.png)

    > **Note**: Your Lab VM also serves as the Hyper-V host for this workshop.

6. [] Return to PowerShell and change to the **Downloads** directory:

    ```powershell
    cd Downloads
    ```

7. [] Execute the downloaded script:

    ```powershell
    .\MicrosoftAzureMigrate-Hyper-V.ps1
    ```

8. [] Respond to the script prompts as follows:

    - [] Do you want to run software from this untrusted publisher?: `[A] Always run`
    - [] Enable Remote Management (WinRM)?: `Y`
    - [] Make these changes?: `y`
    - [] Enable PowerShell Remoting?: `Y`
    - [] Configure firewall to open required ports?: `Y`
    - [] Use SMB share(s) to store VHDs?: `Y`
    - [] Create non-administrator local user for Azure Migrate?: `Y`
    - [] When prompted for credentials:
      - [] **Username**: `MigrateLocal`  
      - [] **Password**: `Pa$$w0rd`

9. [] After the script completes, reset the network category to **Public**:

    ```powershell
    Set-NetConnectionProfile -NetworkCategory Public
    ```

10. [] Confirm the reset by running:

    ```powershell
    Get-NetConnectionProfile
    ```

    ![In the PowerShell terminal, the NetworkCategory setting of Public is highlighted.](./media/13-Network-Public.png)

11. [] Close the PowerShell window.

===

# Task 3: Provision Azure Migrate and create a new project

## Introduction

Azure Migrate requires a supporting set of Azure resources before migration can begin. You will execute a PowerShell script in Azure Cloud Shell to provision the necessary infrastructure for Azure Migrate. After the script completes, you will create a new Azure Migrate project to begin the migration process.

## Description

In this task, you will use Azure Cloud Shell to run a PowerShell script that provisions a resource group, storage account, and virtual network. You will then create a new Azure Migrate project in the Azure portal.

## Success criteria

- You have executed the script in Azure Cloud Shell and provisioned the required Azure resources  
- You have created a new Azure Migrate project in the Azure portal  

## Learning resources

- [Preparing for migrating Linux Virtual Machine to Azure](https://learn.microsoft.com/azure/migrate/prepare-for-migration?view=migrate-classic)

## Key tasks

1. [] Open a web browser on the Lab VM and navigate to the [Azure portal](https://portal.azure.com/).

2. [] Sign in using your lab credentials from the **Resources** tab in the instructions panel.

    ![Screenshot of the resources tab in the instructions panel.](media/lab-resources-credentials.png)

    > **IMPORTANT**: You may be prompted to use a Temporary Access Pass (TAP) for login. This value is listed on the **Resources** tab.
    >
    > ![Screenshot of the login dialog for entering the Temporary Access Pass.](media/azure-portal-login-tap.png)
    >
    > If TAP does not work, choose **Use your password instead** and log in using the provided password.
    >
    > ![The Use your password instead link in highlighted on the TAP entry screen.](./media/14-UsePasswordInstead.png)

3. [] Select **Yes** if prompted to stay signed in.

4. [] In the Azure portal, start a Cloud Shell session by selecting the Cloud Shell icon in the top bar.

    ![The Azure Cloud Shell icon is highlighted on the top bar in the Azure portal.](./media/15-CloudShell.png)

5. [] When prompted, choose `PowerShell` as your shell type.

6. [] In the Cloud Shell setup dialog:

    - [] Select `No storage account required`
    - [] Choose the available subscription from the dropdown
    - [] Select **Apply**

    ![The Getting started dialog in the Cloud Shell is displayed, with no storage account required, the subscription, and the Apply button all highlighted.](media/azure-cloud-shell-getting-started.png)

7. [] At the Cloud Shell prompt, run the following script to create the required resources:

    > **TIP**:
    > You may need to press `ENTER` during the `New-AzVirtualNetwork` command to complete execution.

    ```powershell
    $location = "centralus"
    $resourceGroupName = "rg-AzMigrateLab"
    $storagePrefix = "storazmig"
    $vnetName = "vnet-AzMigrateLab"

    # Create Resource Group
    Write-Host "Creating resource group '$resourceGroupName' in location '$location'..."
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Generate unique storage account name
    $randomSuffix = -join ((48..57) + (97..122) | Get-Random -Count 8 | % {[char]$_})
    $storageAccountName = "$storagePrefix$randomSuffix"

    Write-Host "Creating storage account '$storageAccountName'..."
    New-AzStorageAccount -ResourceGroupName $resourceGroupName `
       -Name $storageAccountName `
       -Location $location `
       -SkuName Standard_GRS `
       -Kind StorageV2

    # Create Virtual Network and Subnet
    Write-Host "Creating virtual network '$vnetName' with subnet 'default'..."
    $subnetConfig = New-AzVirtualNetworkSubnetConfig -Name "default" -AddressPrefix "198.168.4.0/26"
    New-AzVirtualNetwork -Name $vnetName `
        -ResourceGroupName $resourceGroupName `
        -Location $location `
        -AddressPrefix "198.168.4.0/24" `
        -Subnet $subnetConfig
    ```

8. [] When the script finishes, verify output similar to:

    ```powershell
    ResourceGroupName Name              Location  ProvisioningState EnableDdosProtection DefaultPublicNatGateway
    ----------------- ----              --------  ----------------- -------------------- -----------------------
    rg-AzMigrateLab   vnet-AzMigrateLab centralus Succeeded         False
    ```

9. [] Close the Cloud Shell pane.

10. [] In the Azure portal, search for "Azure Migrate" in the search bar and select **Azure Migrate** under **Services**.

    ![Azure Migrate is displaed in the Azure Search bar and highlighted in the search results.](./media/17-AzureMigrate.png)

11. [] On the **Azure Migrate** blade:

    - [] Select **All projects** in the left menu  
    - [] Select **Create Project** in the toolbar

    ![On the Azure Migration All projects blade, the Create Project button is highlighted.](./media/18-AzMigrate_Project.png)

12. [] On the **Create Project** blade, enter the following:

    - [] **Subscription**: Accept the default  
    - [] **Resource group**: Select `rg-AzMigrateLab`  
    - [] **Project name**: `Linux-VM-Migration`  
    - [] **Geography**: `United States`  
    - [] Expand **Advanced** and verify **Connectivity method** is set to `Public Endpoint`  
    - [] Select **Create**

    ![The Create Project form is filled out with the information above and the steps are numbers 1-6.](media/azure-migrate-create-project.png)

13. [] After project creation completes, select **Refresh** on the **All projects** blade to view your new project.

    > Keep this blade open for the next task.

===

# Task 4: Register an Azure Migrate appliance

## Introduction

To begin discovery and assessment of on-premises virtual machines, Azure Migrate requires an appliance to be registered and connected to your Azure Migrate project. The appliance collects metadata about your Hyper-V environment and securely transmits it to Azure for analysis. You will use the Azure Migrate Appliance Configuration Manager to register your appliance with the Azure Migrate project created earlier. This step enables discovery of virtual machines hosted on your Hyper-V environment.

## Description

In this task, you will register the pre-provisioned Azure Migrate appliance using the Appliance Configuration Manager.

## Success criteria

- Your Azure Migrate project key has been verified  
- You have successfully logged in with your lab user credentials  
- Your appliance has been successfully registered  

## Learning resources

- [Azure Migrate appliance](https://learn.microsoft.com/azure/migrate/migrate-appliance?view=migrate-classic)  
- [Appliance requirements](https://learn.microsoft.com/azure/migrate/migrate-appliance?view=migrate-classic)  
- [Preparing for migrating Linux Virtual Machine to Azure](https://learn.microsoft.com/azure/migrate/prepare-for-migration?view=migrate-classic)

## Key tasks

1. [] Return to the Azure Migrate **All projects** blade in the Azure portal and select the **Linux-VM-Migration** project.

    ![The Azure Migration All project blade is displayed with the newly created project highlighted.](./media/20-labazmigrate.png)

2. [] On the **Overview** page, select **Start Discovery**, then choose `Using Appliance` → `For Azure`.

    ![On the Project blade, Start Discovery--> Using Appliance--> For Azure is highlighted.](./media/21-StartDiscovery.png)

3. [] On the **Discovery** blade:

    - [] **Are your servers virtualized?** → `Yes, with Hyper-V`  
    - [] **Name your appliance** → `LabAppliance`  
    - [] Select **Generate Key**

    ![The Discover blade in the Azure portal is displayed, with the settings above highlighted.](./media/22-Discover-steps.png)

4. [] After a minute or two, a **Project key** field will appear on the Discover screen. Copy the project key, then open the `project_key.txt` file on the desktop of the Lab VM and paste the key into the file.

    ![The Project Key section of the Discover screen is highlighted.](./media/23-ProjectKey.png)

    > **IMPORTANT**: Do not download the Azure Migrate appliance VHD or ZIP file.

5. [] Save the updated `project_key.txt` file.

6. [] On the Lab VM, open **Hyper-V Manager** and connect to the `AzMigrateAppliance-Test` VM.

    ![In Hyper-V Manager, the AzMigrateAppliance-Test VM is highlighted in the Virtual Machines list and the Connect action is highlighted in the Actions panel.](./media/24-MigrateVM-Connect.png)

7. [] Log in to the VM using the following credentials:

    - [] **Username**: `Administrator`
    - [] **Password**: `Pa$$w0rd`

8. [] On the `AzMigrateAppliance-Test` VM desktop, open the **Azure Migrate Appliance Configuration Manager**.

    ![Screenshot of the Azure Migrate Appliance Configuration Manager, with the desktop shortcut highlighted.](./media/25-ApplianceConfigManager.png)

9. [] Under **Verification of Azure Migrate project key**, paste the project key into the **Project key** field and select **Verify**.

    ![The Verify button for the Azure Migrate project key is highlighted.](./media/26-KeyVerify.png)

    > [!NOTE]
    > If you receive an internal server error while verifying the project key, periodically try again until it succeeds.

10. [] It may take a few minutes after selecting verify for the Appliance prerequisites to be installed. If you receive a **New updated installed** notification, select **Refresh** and then you will need to select **Verify** again to proceed.

    ![A New update installed dialog is displayed, with the Refresh button highlighted.](media/appliance-prerequisites-new-update-installed.png)

11. [] Scroll to the **Azure user login and appliance registration status** section and select **Login**.

    ![In the Azure user login and appliance registration status section, the Login button is highlighted.](./media/27-Login.png)

12. [] In the pop-up window, select **Copy code & Login**.

    ![The Continue with Azure login dialog is displayed with the Device Code.](./media/28-DeviceCode.png)

13. [] Enter the code in the next window and sign in using the Azure credentials from the **Resources** tab in the lab instructions.

14. [] Close the sign-in window and return to the **Appliance Configuration Manager** browser tab.

15. [] After successful login, you will see a message that the appliance has been successfully registered.

    ![The Appliance registered successfully message is displayed.](./media/appliance-registered.png)

===

# Task 5: Discover virtual machines and workloads on the Hyper-V host

## Introduction

Once the Azure Migrate appliance has been registered, you can begin the discovery process for virtual machines hosted on your Hyper-V environment. The discovery process collects metadata about VM configurations, operating systems, and resource usage, which helps assess readiness and plan migration strategies.

By enabling guest discovery, you can also identify workloads running inside the virtual machines—such as SQL Server, PostgreSQL, and other database platforms. This deeper insight allows you to plan workload-specific migrations and apply appropriate tooling and remediation steps. Guest discovery requires valid credentials and access to the VMs to collect software inventory, dependencies, and workload metadata.

## Description

In this task, you configure discovery credentials and host details in the Azure Migrate Appliance Configuration Manager, then initiate the discovery process. With guest discovery enabled, the appliance will identify both virtual machines and the workloads running inside them, including database platforms and installed software. Discovery results are sent to your Azure Migrate project for review.

## Success criteria

- You started a discovery in the Azure Migrate project  
- You identified virtual machines and their associated workloads using guest discovery  
- You reviewed the discovery results in the Azure portal

## Learning resources

- [Discover servers running on Hyper-V with Azure Migrate: Discovery and assessment](https://learn.microsoft.com/azure/migrate/tutorial-discover-hyper-v?view=migrate-classic)
- [Discover installed software inventory, web apps, and SQL Server instances and databases](https://learn.microsoft.com/azure/migrate/how-to-discover-applications?view=migrate-classic)
- [Provide server credentials to discover software inventory, dependencies, web apps, and SQL Server instances and databases](https://learn.microsoft.com/azure/migrate/add-server-credentials?view=migrate-classic)

## Key tasks

1. [] Return to the **Appliance Configuration Manager** on the `AzMigrateAppliance-Test` VM and continue with Section 2, **Manage credentials and discovery sources**. Select **Add credentials** under **Step 1: Provide Hyper-V host credentials for discovery of Hyper-V VMs**.

    ![The Add credentials button is highlighted in the Manage credentials and discovery sources section.](./media/30-Add-Credentials.png)

2. [] In the **Add credentials** popup, enter the following:

    - [] **Friendly name**: `LabCredentials`  
    - [] **Username**: `MigrateLocal`  
    - [] **Password**: `Pa$$w0rd`  
    - [] Select **Save**

    ![The Add Credentials dialog with the above settings is displayed.](./media/31-add-creds.png)

3. [] Under **Step 2: Provide Hyper-V host/cluster details**, select **Add discovery source**.

    ![The Add discovery source button is highlighted.](./media/32-Add-Discover-source.png)

4. [] In the **Add discovery source** dialog, select **Add single item**.

    ![Add single item is highlighted on the Add discovery source dialog.](media/add-discovery-source-single-item.png)

5. [] To complete the form, you'll need the IP address of your Lab VM (which acts as the Hyper-V host). On the Lab VM, open **Command Prompt** (`cmd.exe`) and run:

    ```bash
    ipconfig
    ```

6. [] Copy the `IPv4 Address` from the output.

7. [] Return to the **Appliance Configuration Manager** and complete the **Add discovery source** form:

    - [] **Discovery source**: Leave `Hyper-V Host/Cluster` selected  
    - [] **IP Address/FQDN**: Paste the IP address of your Lab VM  
    - [] **Map credentials**: Select `Lab Credentials`  
    - [] Select **Save**

    ![The values above are entered into the Add discovery source dialog.](./media/34-IPAddress.png)

8. [] You should receive a **Validation successful** message within a minute or so.

    ![The Validation successful message is highlighted for the added credentials.](media/appliance-add-credentials-successful.png)

9. [] After validation completes, ensure **Guest discovery is enabled by default** under **Step 3: Provide server credentials to perform guest discovery of installed software, dependencies, and workloads** is enabled.

    ![The Guest discovery is enabled by default toggle is highlighted.](./media/appliance-guest-discovery-enabled.png)

10. [] Scroll down and select **Add credentials**.

    ![The Add credentials button is highlighted under Step 3.](./media/guest-discovery-add-credentials.png)

11. [] In the **Add credentials** popup, enter the following:

    - [] **Credentials type**: `Windows (Non-domain)`  
    - [] **Friendly name**: `SqlWindows`
    - [] **Username**: `Administrator`  
    - [] **Password**: `P@$$w0rd1`

    ![The Add Credentials dialog with the above settings is displayed.](./media/guest-add-credentials-sql-windows.png)

12. [] Select **Add more** and add a second set of credentials:

    - [] **Credentials type**: `Linux (Non-domain)`  
    - [] **Friendly name**: `Ubuntu`
    - [] **Username**: `administrator`  
    - [] **Password**: `Pa$$w0rd`

13. [] Select **Add more** and add a third set of credentials:

    - [] **Credentials type**: `Linux (Non-domain)`  
    - [] **Friendly name**: `CentOS`
    - [] **Username**: `root`  
    - [] **Password**: `Pa$$w0rd`

14. [] Select **Add more** and add a fourth set of credentials:

    - [] **Credentials type**: `SQL Server Authentication`  
    - [] **Friendly name**: `SqlServer`
    - [] **Username**: `sqladmin`
    - [] **Password**: `Microsoft123`

15. [] Select **Add more** and add a fifth set of credentials:

    - [] **Credentials type**: `PostgreSQL Server (Password based)`  
    - [] **Friendly name**: `PostgreSQL`
    - [] **Username**: `pgadmin`
    - [] **Password**: `pgadmin123`

16. [] Select **Save** to close the **Add credentials** dialog.

17. [] Verify that all five sets of credentials appear under **Step 3**.

    ![The five sets of credentials are displayed under Step 3 in the Appliance Configuration Manager.](./media/guest-discovery-credentials.png)

18. [] Scroll down and select the **Start Discovery** button.

    ![The Start Discovery button is highlighted at the bottom of the Appliance Configuration Manager.](./media/start-discovery.png)

    > **Note**: This step can take up to 15 minutes to complete.

19. [] When discovery finishes, navigate to the Azure Migrate **Linux-VM-Migration** project in the Azure portal, expand **Explore Inventory** in the left navigation menu, then select **All Inventory** and confirm that your Hyper-V VMs appear in the list and that you see the PostgreSQL and SQL Server databases.

    ![The discovered VMs are highlighted on the All inventory blade in the Azure portal.](./media/38-validation.png)

===

# Task 6: Perform Assessments of discovered resources

## Introduction

With discovery complete, Contoso's migration team is ready to assess the readiness of key workloads for migration to Azure. This includes the Linux-based personnel system running on `LinuxLabVM-Ubuntu`, as well as SQL Server and PostgreSQL database instances discovered through guest-level inspection.

Azure Migrate assessments help estimate sizing, identify compatibility issues, and surface remediation guidance. By assessing both infrastructure and workload-level details, Contoso can make informed decisions about migration tooling, target platforms, and performance expectations.

## Description

In this task, you create assessments for the Linux VM, SQL Server, and PostgreSQL workloads discovered by the Azure Migrate appliance. These assessments will help validate sizing, compatibility, and migration readiness for each resource.

## Success criteria

- You created assessments for the Linux VM, SQL Server, and PostgreSQL workloads in the Azure portal  
- You reviewed the assessment results for sizing and compatibility recommendations

## Learning resources

- [Azure Migrate Assessment overview](https://learn.microsoft.com/azure/migrate/concepts-overview?view=migrate)
- [Azure Migrate application and code assessment](https://learn.microsoft.com/azure/migrate/appcat/overview?view=migrate-classic)
- [Create an Azure VM assessment](https://learn.microsoft.com/azure/migrate/how-to-create-assessment?view=migrate)
- [Assess SQL instances for migration to Azure SQL](https://learn.microsoft.com/azure/migrate/tutorial-assess-sql?view=migrate)
- [Assess PostgreSQL workloads for migration using Azure Migrate](https://learn.microsoft.com/azure/migrate/tutorial-assess-postgresql?view=migrate)

## Key tasks

1. [] On the Azure Migrate Project blade in the Azure portal, expand the **Decide and Plan** in the left menu, select **Assessments**, and then select **Create assessment**.

    ![The assessments blade is displayed with Create Assessment highlighted.](./media/39-Assessments.png)

2. [] Give enter `LabAssessment` for the **Assessment name** and then select **Add workloads**.

    ![On the Create assessment Basic's tab, the assessment name and Add workloads button are highlighted.](media/create-assessment-basics.png)

3. [] On the **Select workloads** page, check the boxes next to the following items:

    - [] `LinuxLabVM-CentOS-PostgreSQL` (this will automatically check the box next to the `localhost:5432` (PostgreSQL) database installed on that server)
    - [] `LinuxLabVM-Ubuntu`
    - [] `SQLPTO2022` (this will automatically check the box next to the `MSSQLSERVER` database installed on that server)
    - [] Select **Add**

    ![On the Select workloads page, the boxes next to LinuxLabVM-Ubuntu, localhost:5432 (PostgreSQL), and MSSQLSERVER are checked and highlighted and Add button is highlighted.](./media/azure-migrate-assessment-workloads.png)

4. [] Select **Next** and verify that **Sizing criteria** is set to `Performance-based`.

5. [] Select **Next** to go to the **Advanced** tab.

6. [] On the **Advanced** tab, select **Edit defaults** next to **SQL Server**.

    ![Edit default SQL Server settings](./media/azure-migrate-edit-sql-default.png)

7. [] On the **SQL Server settings** page, update the **Target services** by unchecking **Azure SQL MI (Managed Instance)** and checking **Azure SQL Database**, and then select **Save**.

    ![The SQL Server settings are populated with the values specified above.](./media/sql-server-settings.png)

8. [] Select **Review + Create assessment**, then select **Create** on the review tab.

9. [] On the **Assessments** blade, select **Refresh** after receiving the notification that the assessment was created.

    ![The newly created assessment is displayed on the Assessments blade.](./media/41-AssessmentReady.png)

===

# Task 7: Replicate the Linux Ubuntu VM to Azure

## Introduction

With the assessment complete, Contoso's migration team is ready to replicate the Linux-based personnel system to Azure. This task simulates the final stages of a lift-and-shift migration using Azure Migrate. You'll configure replication using the Azure Site Recovery provider and registration key.

## Description

In this task, you will download the Azure Site Recovery provider and registration key, install the provider on the Hyper-V host, and configure replication for the `LinuxLabVM-Ubuntu` VM.

## Success criteria

- You downloaded and installed the Azure Site Recovery provider on the Hyper-V host
- You configured replication for the `LinuxLabVM-Ubuntu` VM
- You completed the replication process successfully

## Learning resources

- [Preparing for migrating Linux Virtual Machine to Azure](https://learn.microsoft.com/azure/migrate/prepare-for-migration?view=migrate-classic)
- [Migrate Hyper-V VMs to Azure](https://learn.microsoft.com/azure/migrate/tutorial-migrate-hyper-v?view=migrate-classic)

## Key tasks

1. [] On the Azure Migrate Project page, expand **Execute** in the left menu, select **Migrations**, and on the Migration blade, select **Discover more**.

    ![The Discover more button is highlighted on the Migrations blade.](./media/42-Migration-dicover.png)

2. [] On the **Discover** blade:

   - [] **Where to you want to migrate to?**: Choose `Azure VM`
   - [] **Are your machines virtualized?**: Choose `Yes, with Hyper-V`
   - [] **Target region**: Accept the default value already selected
   - [] Check the box for **Confirm that the target region for migration is "centralus"**
   - [] Select **Create resources**

    ![The Discover blade is populated with the values specified above.](./media/43-CreateResources.png)

3. [] When the deployment completes, under **1. Prepare Hyper-V host servers**:

   1. [] Select the **Download** link to download the Hyper-V replication provider software installer.
   2. [] Select the **Download** button to download the registration key in step 2 in the screenshot.

    ![The Download link and button are highlighted and numbers 1 and 2 under Prepare Hyper-V host servers.](./media/44-Download.png)

    > **IMPORTANT**: You must be on the Lab VM when you download the software and the key above so that your Hyper-V host is configured correctly.

4. [] Run the `AzureSiteRecoveryProvider.exe` that you just downloaded.

5. [] Select **On (recommended)** and **Next**.

6. [] Install into the default location and select **Install**.

    > **IMPORTANT**: Do NOT select **Finish** yet.

7. [] After the installation is complete, select **Register**.

8. [] Browse to the registration key file you downloaded and select **Next**.

    ![Screenshot of the downloaded registration key file.](./media/45-Keyfile.png)

9. [] Select **Connect directly to Azure Site Recovery without a proxy server** and select **Next**.

10. [] After about 60 seconds the configuration will complete.

11. [] Select **Finish**.

12. [] Close the blade and return to the **Discover more** page on the Migrations blade.

    > You should see that you have one connected registration.

    ![The connected registrations are highlighted on the Discover more page.](./media/46-connected-registration.png)

13. [] Select **Finalize registration**.

    > This step may take up to 3 minutes.

14. [] Return to the Migration Project blade, expand **Execute** in the left menu, select **Migrations**, and select the **Replicate** button.

    ![The Replicate button is highlighted on the Migrations blade.](./media/47-Replicate.png)

15. [] On the **Specify intent** page:

    - [] **What do you want to migrate?**: Choose `Servers or virtual machines (VM)`
    - [] **Where do you want to migrate to?**: Select `Azure VM`
    - [] **Are your machines virtualized?**: Select `Yes, with Hyper-V`
    - [] Select **Continue**

    ![The Specify intent page is populated as specified above.](./media/48-SpecifyIntent.png)

16. [] On the **Replicate** Virtual machines tab:

    - [] **Target VM security type**: Choose `Standard or Trusted Launch Virtual machines`
    - [] **Import migration settings from an assessment**: Select `Yes, apply migration settings from an Azure Migrate assessment`
    - [] **Select assessment**: Select the `LabAssessment` you created earlier
    - [] Check the box next to `LinuxLabVM-Ubuntu`
    - [] Select **Next**

    ![The Replicate Virutal machines tab is populated with the values specified above and the steps are numbered 1-4.](./media/49-ReplicateVM.png)

17. [] On the **Target settings** tab:

    - [] **Resource group**: Select `rg-AzMigrateLab`
    - [] **Replication storage account**: Select the storage account that was created for you
    - [] **Virtual network**: Choose `vnet-AzMigrateLab`
    - [] Select **Next**

    ![The settings above are entered into the Target settings tab. An error message is highlighted.](./media/50-ErrorIAM.png)

    > **NOTE**: You may see an error that the Azure Migrate service cannot access the replication storage account. This error is because the storage account does not have the proper access privilege to the Recovery Service vault. To resolve this issue, first open the Recovery Service Vault resource in another browser page and under `Settings | Identity` turn ON the status on the `System assigned` and select **Save**.
    >
    > ![Screenshot of the steps to turn on the System Assigned identity for the recovery services vault.](./media/51-IAM-RecoveryVault.png)
    >
    > Next, you need to assign the `Contributor` and `Storage Blob Data Contributor` RBAC roles to the `Recovery Services Vault` identity on the storage account.
    >
    > The above steps should remove the error on the `Replicate` blade's Target settings tab and enable you to move to the next step. (You will need to redo the replicate steps after the IAM has been fixed).

18. [] On the **Compute** tab:

    - [] Select `Linux` in the **OS Type** drop down
    - [] Select any availability
    - [] Accept the default values for all other fields
    - [] Select **Next**

    ![Screenshot of the Compute tab, with the OS Type and its value of Linux highlighted.](./media/52-OSType.png)

19. [] Accept the default values on the remaining tabs, then select **Start** on the **Review + start replication** tab.

    > **NOTE**: The replication step can take 25 minutes or more. You can monitor the progress by opening the **Replications Summary** and selecting **Jobs** in the left menu under **Migration**.
    >
    > ![The Replications Summary button on the Migrations blade is highlighted.](./media/54-ReplicationSummary.png)

===

# Task 8: Migrate the Linux Ubuntu VM

## Introduction

With the Linux Ubuntu VM replicated, you are ready to migrate the Linux-based personnel system to Azure. This task simulates the final stages of a lift-and-shift migration using Azure Migrate. You will perform a planned migration to Azure Virtual Machines.

## Description

In this task, you initiate a planned migration of the Linux VM to Azure using the Azure Migrate portal.

## Success criteria

- You initiated the migration process successfully

## Learning resources

- [Preparing for migrating Linux Virtual Machine to Azure](https://learn.microsoft.com/azure/migrate/prepare-for-migration?view=migrate-classic)

## Key tasks

1. [] The replication is complete when you see the replication status for the `LinuxLabVM-Ubuntu` set to **Protected**.

2. [] Start the migration by selecting the ellipsis to the right of the `LinuxLabVM-Ubuntu` item and then selecting **Migrate** from the context menu.

    ![The ellipsis for the LinuxLabVM-Ubuntu is highlighted and Migrate is highlighted in the context menu.](./media/53-ProtectedItem.png)

3. [] Select **Yes** to shutting down the VM and performing a planned migration with no data loss.

4. [] Select **Migrate**.

    > **IMPORTANT**: The migration can take over two hours to complete. You can monitor the progress by opening the **Jobs** section on the Migration blade and selecting **Planned failover**.

    ![The Jobs page is displayed with the active and completed jobs shown in a list.](./media/55-jobs.png)

    ![Screenshot of the Planned failover job details.](./media/56-PlannedFailover.png)
