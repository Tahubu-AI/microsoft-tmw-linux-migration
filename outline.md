# Lab outline

1. Exercise 1: Migrate a Linux VM to an Azure VM
   1. Test app before migration (optional)
   2. Prepare Hyper-V Host
   3. Create an Azure Migrate project
      1. Provision via Azure Cloud Shell
      2. Create project in Azure portal
   4. Configure Appliance and Discover VMs
      1. Configure appliance with Azure Migrate Applicance Configuration Manager
         1. Connect to appliance VM in Hyper-V (AzMigrateAppliance)
      2. Start Discovery
      3. Confirm discovery results
   5. Assess, replicate, and migrate Linux Ubuntu VM
      1. Assess
      2. Replicate (TODO: Need to assess replication time here - Donovan stated it took a long time)
      3. Migrate the VM to Azure
   6. Attach a Premium SSD 2 data disk to Linux VM on Azure
   7. Test the application again to ensure migration worked and connections are still good to database.
2. Exercise 2: PostgreSQL migration to Azure Database for PostgreSQL Flexible Server
   1. TODO
3. Exercise 3: Enable replication of Linux VM to App Service Plan
   1. TODO
4. Exercise 4: SQL Server migration to Azure SQL Database
   1. TODO
