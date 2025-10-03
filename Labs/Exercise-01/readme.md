# Exercise 1: Migrate a Linux VM to an Azure VM

## Task 1: Test app before migration

1. Open Hyper-V Manager from the lab VM desktop or start menu.
2. Select the `LinuxLabVM-Ubuntu` VM from the Virutal Machines list and connect to the VM.
3. Open VS Code.
    1. Use bar icon in top left in Ubuntu VM
    2. Search for Visual Studio Code

    > [!TIP]
    > File > Auto Save

4. Update application code
    1. In a Terminal (in VS Code), run `git pull origin main`
    2. In `views.py` (with LinuxApp folder):
        1. Update connection string with (Line 58) with IP of `SQLPTO2022` (found in `VM_IPs` text file on Hyper-V host).
        2. Specify that they should use the private IP address of the SQL 2022 box (172.17.195.x)
5. Start Application locally
    1. Navigate to `runserver.py`
    2. Click Run icon (Run Python File)
6. Open a web browser and navigate to local server address (provided in console) - <http://localhost:5555>
    1. Navigate to People tab in the application
    2. Verify the page is populated with a list of people from the `Person.Person` table in the SQL database.

    > TODO: Provide a screenshot of what the populated pages should look like

## Task 2: Prepare the Hyper-V host

1. Download the [Hyper-V preparation script](https://aka.ms/migrate/hyperv/script) onto the lab VM (TODO: IMPORTANT: Do not download on the Ubuntu VM...)
   1. TODO: Provide a description of what this script is doing and why it matters...
   2. Enables WinRM and dependencies for Azure Migrate Hyper-V Host.

    > [!IMPORTANT]  
    > Donovan will work with Oren to see if when can have the script on the Hyper-V host so students do not have to download.

2. Open Powershell as Administrator

    > Ensure it is not PowerShell ISE as prompts may not show in console properly

3. Set Network connectivity to Private
4. Run `Set-NetConnectionProfile -NetworkCategory Private`
5. Run `Get-NetConnectionProfile` and ensure `NetworkCategory` is set to `Private`
6. Navigate to and run Hyper-V preparation script

   1. Navigate to location of `MicrosoftAzureMigrate-Hyper-V.ps1`

    > [!NOTE]
    > We can provide the location should be have the script downloaded already (i.e. `cd C:\Users\Administrator\Desktop\`)

7. Accept the prompts
8. Creating local user credentials

      ```PowerShell
      Username: (i.e. MigrateLocal)
      Password: (i.e. <same-as-machine>)
      ```

9. Reset Network Connection Type

   1. In PowerShell session (as administrator)
   2. Run `Set-NetConnectionProfile -NetworkCategory Public`
   3. Run `Get-NetConnectionProfile` and ensure `NetworkCategory` is set to `Public`

## Task 3: Create an Azure Migrate project

1. Provision via Azure Cloud Shell
2. Create project in Azure portal

## Task 4: Configure Appliance and Discover VMs

1. Configure appliance with Azure Migrate Applicance Configuration Manager
   1. Connect to appliance VM in Hyper-V (AzMigrateAppliance)
2. Start Discovery
3. Confirm discovery results

## Task 5: Assess, replicate, and migrate Linux Ubuntu VM

1. Assess
2. Replicate (TODO: Need to assess replication time here - Donovan stated it took a long time)

    TODO: Type of steps to configure MSI on the recovery vault resource and them ensure it is granted 'Contributor' and 'Storage Blob Data Contributor' roles on the target storage account.

        1. Go to your Storage account '/subscriptions/75080732-c8d0-4487-b43a-a06e7a9cdd8b/resourceGroups/rg-AzMigrateLab/providers/Microsoft.Storage/storageAccounts/storazmigexi5zsg2' -> Access Control (IAM).
        2. Add the below role-assignments (for ARM based storage account) to the Recovery services vault's MSI.
        a) "Contributor"
        And,
        b) "Storage Blob Data Contributor" for Standard storage or "Storage Blob Data Owner" for Premium storage
        Above permissions need to be added for System Assigned Managed Identity if only system MSI is present. If only user assigned managed identity is present or user assigned identity is present along with system assigned identity, add the above permissions to your storage account for user assigned identity.
        You can refer to https://aka.ms/asrGrantManagedIdentityToTheVault for more details.

3. Migrate the VM to Azure

## Task 6: Attach a Premium SSD 2 data disk to Linux VM on Azure

## Task 7: Test the application again to ensure migration worked and connections are still good to database.