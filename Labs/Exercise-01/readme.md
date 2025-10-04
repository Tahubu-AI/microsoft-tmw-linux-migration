# Exercise 1: Migrate a Linux VM to an Azure VM

## Task 1: Test the Application before the migration

> [!IMPORTANT]
> After launching the lab, ensure the the `LinuxVM-Ubuntu` is running in your Hyper-V

1- Login to the Hyper-V machine as the first step of this exercise.  The lab will provide the password in the intructions tab on the right, you can enter it manually or just click on the [T] in the instructions and the password will be typed in for you automatically to login.
2- Once logged in, double click the desktop icon of `Hyper-V Manager` to start the application

![Hyper-V Manager](./media/01-Launch-HyperV.png)

3- Ensure that Five virtual machines are available and running as you can see in screenshot below.  In this first Exercise, we will only use the `LinuxLabVM-Ubuntu` and the `AzMigrateAppliance-Test` Virtual machines.

![Five Virtual Machines running](./media/02-five-VMs-Running.png)

4- Click to highlight the `LinuxLabVM-Ubuntu` in the list and click on  `Connect` on the right sidebar

![Connect](./media/03-Connect.png)

5- As the `Administrator` enter the password provided to you on the `Resources` tab on the right of the lab and press the enter key to login to the Ubuntu machine.

6- Click on Use bar icon in top left in the Ubuntu VM, then search for `Visual Studio Code` and finally click on the Visual Studio Code icon to launch it.

![VSCode Ubuntu](./media/04-VSCode-Ubuntu.png)

7- The Visual Studio Code IDE will open and you should have the `LinuxApp` already open.  Click on the `views.py` file in the explorer.

8- On the desktop of the Hyper-V machine you will find a text file called `VM_IPs` open the file and copy the IP value of the `SQLPTO2022`.  Paste that IP address in the `views.py` file in the connection string line as show in the screenshot below.

![VM_IP](./media/05-VM-IP.png)

9- Open the file `runserver.py` in the explorer of Visual Studio code and run it by clicking on the play button in the top right of the editor, then `CTRL-Click` the `http://localhost:xxxx` link in the terminal to launch the LinuxApp in the default browser

![run server](./media/06-runserver.png)

10- The application will start in the browser. Navigate to the `People` tab

![LinuxApp](./media/07-LinuxApp.png)

11- Ensure data is populated on the people's page.  It should look like the screenshot below

![People Page](./media/08-PeoplePage.png)

## Task 2: Preparing the Hyper-V Host

1- Back in your Hyper-V host machine, open a browser window and navigate to `https://aka.ms/migrate/hyperv/script` to download the script needed to prepare the HyperV

![HyperV-Download](./media/09-download.png.png)

2- From the search bar at the bottom of the HyperV machine type `Powershell`.  A window will pop with multiple options, make sure to choose the Windows PowerShell app NOT  the ISE one and start it by running as Administrator.

![PS1 admin](./media/10-ps-admin.png)

3- In the terminal, run:
```powershell
Set-NetConnectionProfile -NetworkCategory Private
```
Then run the following to ensure that the NetworkCategory is set to `Private`

```powershell
Get-NetConnectionProfile
```

![Network-Private](./media/11-Network-Private.png)

4- Now head over to where you downloaded the script in step 1 and execute the script in the terminal, accepting ALL the prompts that will show up. There should be 7 prompts where you should agree to all.  At the last step you will be asked to choose a username and password for credentials/ Enter `MigrateLocal` as the username and `Pa$$w0rd` as the password.

![PS1 Execution](./media/12-ps1-execution.png)

5- Now you can reset the Network Connection to public by running the following command in the terminal

```powershell
Set-NetConnectionProfile -NetworkCategory Public
```
Then run the following to ensure that the NetworkCategory is set to `Public`

```powershell
Get-NetConnectionProfile
```
![Network Public](./media/13-Network-Public.png)

## Task 3: Azure Migrate Project creation

1- On the Hyper V Host Machine, open a browser window and navigate to `https://portal.azure.com`
 
2- Login using the credentials in the lab available on the `Resources` tab on the right of the lab screen.  There you will find the username and password you should use and also a TAP (Temporary Access Password) that we will use later.

> [!IMPORTANT]
> If prompted for Temporary Access Pass, Select use your password instead.

![Use Password Instead](./media/14-UsePasswordInstead.png)

3- Another Credential screen will show up, this time enter the TAP password you see in the `Resources` Tab

4- Teh Azure portal will load, start a cloud shell by click on the cloudshell icon in the top bar like in the screenshot below

![CloudShell](./media/15-CloudShell.png)

5- The Azure Cloud shell will ask whether you want to start a `Bash` or `Powershell` session, choose `Powershell`

6- The next screen will ask about storage, choose `No storage account required`

7- Select the only subscription you see in the dropdown and click `Apply`

8- You should see a cloudshell similar to the one in the screenshot below

![CloudShell-Apply](./media/16-CloudShell-Apply.png)

9- Run the following in the CloudShell to create the resources needed in Azure.

```powershell
# Login to Azure
# Write-Host "Logging in to Azure..."
# Connect-AzAccount

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

> [!TIP]
> You might need to press the ENTER key once during the `New-AzVirtualNetwork` command while running the script.

10- Back in your Azure Portal, search for `Azure Migrate` in the upper search bar and start `Azure Migrate`

![Azure Migrate](./media/17-AzureMigrate.png)

11- In the `Azure Migrate` blade, click on `All projects` and `Create Project`

![AZ Migrate Project](./media/18-AzMigrate_Project.png)

12- In the `Create Project` blade, fill in the resource group, it should be `rg-AzMigrateLab`, Give it a name for the project, make sure the Geograpgy is `United states` and finally in the Advanced section maker sure the connectivity method is `Public Endpoint`

13- Click on `Create` to start the create of the Azure Migrate Project.


