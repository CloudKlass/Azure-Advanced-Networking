---
lab:
    title: 'M01 - Unit 6 Configure DNS settings in Azure'
    module: 'Module 01 - Introduction to Azure Virtual Networks'
---

# M01 - Unit 6 Configure DNS settings in Azure

## Exercise scenario 
In this unit, you will configure DNS name resolution for Contoso Ltd. You will create a private DNS zone named contoso.com, link the VNets for registration and resolution, and then create two virtual machines and test the configuration.

In this exercise, you will:

+ Task 1: Create a private DNS Zone
+ Task 2: Link subnet for auto registration
+ Task 3: Create Virtual Machines to test the configuration
+ Task 4: Verify records are present in the DNS zone

>**Note**: An **[interactive lab simulation](https://mslabs.cloudguides.com/guides/AZ-700%20Lab%20Simulation%20-%20Configure%20DNS%20settings%20in%20Azure)** is available that allows you to click through this lab at your own pace. You may find slight differences between the interactive simulation and the hosted lab, but the core concepts and ideas being demonstrated are the same.

#### Estimated time: 25 minutes

## Task 1: Create a private DNS Zone

1. Go to [Azure Portal](https://portal.azure.com/).

2. On the Azure home page, in the search bar, enter **dns**, and then select **Private DNS zones**.  
   ‎![Azure Portal home page with dns search.](../media/create-private-dns-zone.png)

3. In Private DNS zones, select **+ Create**.

4. Use the information in the following table to create the private DNS zone.

| **Tab**         | **Option**                             | **Value**            |
| --------------- | -------------------------------------- | -------------------- |
| Basics          | Resource group                         | ContosoResourceGroup |
|                 | Name                                   | Contoso.com          |
| Tags            | No changes required

>**Important Note:** If you are prompted that **Contoso.com** is reserved, Check that you are creating a **Private DNS Zone** and not a **DNS Zone**.

Select **Review + create** Wait for validation to complete: Review your settings and select **Create** 

1. Wait until the deployment is complete, and then select **Go to resource**.

2. Verify that the zone has been created.

## Task 2: Link subnet for auto registration

1. In **Contoso.com**, under **Settings**, select **Virtual network links**.

2. On Contoso.com | Virtual network links, select **+ Add**.

![contoso.com | Virtual links with + Add highlighted.](../media/add-network-link-dns.png)

3. Use the information in the following table to add the virtual network link.

| **Option**                          | **Value**                               |
| ----------------------------------- | --------------------------------------- |
| Link name                           | CoreServicesVnetLink                    |
| Subscription                        | No changes required                     |
| Virtual Network                     | CoreServicesVnet (ContosoResourceGroup) |
| Enable auto registration            | Selected                                |
| Review your settings and select **Create**. |                                         |


>Note: You do not need to wait for completion, carry on with the next step.

4. On Contoso.com | Virtual network links, select **+ Add**. 

| **Option**                          | **Value**                                |
| ----------------------------------- | ---------------------------------------- |
| Link name                           | ManufacturingVnetLink                    |
| Subscription                        | No changes required                      |
| Virtual Network                     | ManufacturingVnet (ContosoResourceGroup) |
| Enable auto registration            | Selected                                 |
| Review your settings and select **Create**. |                                          |



5. On Contoso.com | Virtual network links, select **+ Add**. 

| **Option**                          | **Value**                           |
| ----------------------------------- | ----------------------------------- |
| Link name                           | ResearchVnetLink                    |
| Subscription                        | No changes required                 |
| Virtual Network                     | ResearchVnet (ContosoResourceGroup) |
| Enable auto registration            | Selected                            |
| Review your settings and select **Create**. |                                     |


6. Select **Refresh**: It may take upto 20 seconds before all 3 Network links show in the blade.

7. Verify that the **CoreServicesVnetLink**, **ManufacturingVnetLink** and the **ResearchVnetLink** has been created, and that auto-registration is **enabled**.

 

##  Task 3: Create Virtual Machines to test the configuration

In this section, you will create two test VMs to test the Private DNS zone configuration.

1. In the Azure portal, select the Cloud Shell icon (top right). If necessary, configure the shell.  
    + Select **PowerShell**.
    + Select **No Storage Account required** and your **Subscription**, then select **Apply**.
    + Wait for the terminal to create and a prompt to be displayed. 

1. On the toolbar of the Cloud Shell pane, select the **Manage Files** icon, in the drop-down menu, select **Upload** and upload the following files **azuredeploy.json** and **azuredeploy.parameters.json** into the Cloud Shell home directory one by one from the source folder **F:\Allfiles\Exercises\M01**.

3. Deploy the following ARM templates to create the VMs needed for this exercise:

>**Note**: You will be prompted to provide an Admin password.  Use **Pa55w.rd1234abc**

   ```powershell
   $RGName = "ContosoResourceGroup"
   
   New-AzResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile azuredeploy.json -TemplateParameterFile azuredeploy.parameters.json
   ```
  
4. When the deployment is complete, go to the Azure portal home page, and then select **Virtual Machines**.

5. Verify that both virtual machines have been created.

 

## Task 4: Verify records are present in the DNS zone

1. On the Azure Portal home page, select **Private DNS zones**.

2. On Private DNS zones, select **contoso.com**.

3. Verify that host (A) records are listed for both VMs, as shown:

![Contoso.com DNS zone showing auto-registered host A records.](../media/contoso_com-dns-zone.png)

 

4. Make a note of the names and IP addresses of the VMs.

 

### Connect to the Test VMs using RDP

1. On the Azure Portal home page, select **Virtual Machines**.

2. Select **TestVM1**.

3. On TestVM1, select **Connect &gt; Connect**.

![TestVM1 with Connect and RDP highlighted.](../media/connect-to-am.png)

4. On TestVM1 | Connect, select **Download RDP file**.

5. Save the RDP file to your desktop.

6. On the Azure Portal home page, select **Virtual Machines**.

7. Select **TestVM2**.

8. On TestVM2, select **Connect &gt; Connect**.

9. On TestVM2 | Connect, select **Download RDP file**.

10. Save the RDP file to your desktop.

11. Connect to **TestVM1** using the RDP file, and the username **TestUser** and the password **Pa55w.rd1234abc**.

12. If prompted, in **Networks**, select **Yes**.

15. On **TestVM1**, open a PowerShell console and enter the command **ipconfig /all**.

16. Verify that the IP address is the same as the one you noted in the DNS zone.

17. Enter the command **ping TestVM2.contoso.com**.

18. Verify that the FQDN resolves to the IP address that you noted in the Private DNS zone. The ping itself will timeout because of the Windows Firewall that is enabled on the VMs.

19. Enter the command **Test-NetConnection -Computername TestVM1.Contoso.com -port 3389** and verify that you receive a successful name resolution record and **TcpTestSucceeded : True** response from **TestVM2**
 

Congratulations! You have created a private DNS Zone, added a name resolution and auto-registration link, and tested name resolution in your configuration. 

>**Note:** Do not Delete any resources: you will use them in the next LAB Exercise.