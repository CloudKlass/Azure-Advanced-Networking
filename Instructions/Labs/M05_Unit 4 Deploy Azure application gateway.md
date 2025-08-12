---
lab:
    title: 'M05 - Unit 4 Deploy Azure Application Gateway'
    module: 'Module 05 - Load balancing HTTP(S) traffic in Azure'
---

# M05-Unit 4 Deploy Azure Application Gateway
 

In this exercise, you use the Azure portal to create an application gateway. Then you test it to make sure it works correctly.

   >**Note**: An **[interactive lab simulation](https://mslabs.cloudguides.com/guides/AZ-700%20Lab%20Simulation%20-%20Deploy%20Azure%20Application%20Gateway)** is available that allows you to click through this lab at your own pace. You may find slight differences between the interactive simulation and the hosted lab, but the core concepts and ideas being demonstrated are the same.


#### Estimated time: 25 minutes

The application gateway directs application web traffic to specific resources in a backend pool. You assign listeners to ports, create rules, and add resources to a backend pool. For the sake of simplicity, this article uses a simple setup with a public front-end IP, a basic listener to host a single site on the application gateway, a basic request routing rule, and two virtual machines in the backend pool.

For Azure to communicate between the resources that you create, it needs a virtual network. You can either create a new virtual network or use an existing one. In this example, you'll create a new virtual network while you create the application gateway. Application Gateway instances are created in separate subnets. You create two subnets in this example: one for the application gateway, and another for the backend servers.

In this exercise, you will:

+ Task 1: Create an application gateway
+ Task 2: Add a Subnet to the Virtual Network
+ Task 3: Create virtual machines
+ Task 4: Add backend servers to backend pool
+ Task 5: Test the application gateway


## Task 1: Create an application gateway

1. Sign in to the [Azure portal](https://portal.azure.com/) with your Azure account.

1. On any Azure Portal page, in **Search resources, services and docs (G+/)**, enter application gateway, and then select **Application gateways** from the results.
    ![Azure Portal search for application gateway](../media/search-application-gateway.png)    

1. on the **Application Gateways** blade, click **+ Create** then click **Application Gateway**..

1. On the Create application gateway **Basics** tab, enter, or select the following information: Leave other settings at their default settings.

   | **Setting**         | **Value**                                    |
   | ------------------- | -------------------------------------------- |
   | Subscription        | Select your subscription.                    |
   | Resource group      | Select **Create new** Name: **ContosoResourceGroup**       |
   | Application Gateway name| **ContosoAppGateway**                            |
   | Region              | Select **East US**                           |
   | Virtual Network     | Select **Create new**                        |

1. On the **Create virtual network** blade, enter, or select the following information:

   | **Setting**       | **Value**                          |
   | ----------------- | ---------------------------------- |
   | Name              | **ContosoVNet**                        |
   | **ADDRESS SPACE** |                                    |
   | Address range     | **10.0.0.0/16**                        |
   | **SUBNETS**       |                                    |
   | Subnet name       | Change **default** to **AGSubnet** |
   | Address range     | **10.0.0.0/24**                        |


1. Select **OK** to return to the Create application gateway Basics tab.

1. Accept the default values for the other settings and then select **Next: Frontends**.

1. On the **Frontends** tab, verify **Frontend IP address type** is set to **Public**.

1. Select **Add new** for the **Public IP address** and enter **AGPublicIPAddress** for the public IP address name, and then select **OK**.

1. Select **Next: Backends**.

1. On the **Backends** tab, select **Add a backend pool**.

1. On the **Add a backend pool** window that opens, enter the following values to create an empty backend pool:

    | **Setting**                      | **Value**   |
    | -------------------------------- | ----------- |
    | Name                             | **BackendPool** |
    | Add backend pool without targets | **Yes**         |

1. On the **Add a backend pool** window, select **Add** to save the backend pool configuration and return to the **Backends** tab.

1. On the **Backends** tab, select **Next: Configuration**.

1. On the **Configuration** tab, you'll connect the frontend and backend pool you created using a routing rule.

1. On the **Routing rules** column, select **Add a routing rule**.

1. On the **Add a routing rule** blade, enter the following information:

    | **Setting**   | **Value**         |
    | ------------- | ----------------- |
    | Rule name | **RoutingRule**          |
    | Priority      | **100**           |


1. On the **Listener** tab, enter or select the following information:

    | **Setting**   | **Value**         |
    | ------------- | ----------------- |
    | Listener name | **Listener**      |
    | Frontend IP   | Select **Public IPv4** |

1. Accept the default values for the other settings on the **Listener** tab.

    ![Azure Portal add an Application Gateway routing rule](../media/Routing-rule-listener-tab.png)

1. Select the **Backend targets** tab to configure the rest of the routing rule.

1. On the **Backend targets** tab, enter or select the following information:

    | **Setting**      | **Value**      |
    | -------------    | -------------- |
    | Target type      | **Backend pool**   |
    | Backend Target *(drop down list)* | Select  **BackendPool** |
    | Backend Settings | **Add new** |

1. In **Add Backend Setting**, enter or select the following information:

    | **Setting**          | **Value**   |
    | ------------------   | ----------- |
    | Backend setting name | **HTTPSetting** |
    | Backend port         | **80**          |

1. Accept the default values for the other settings in the **Add a Backend Setting** window, then select **Add** to return to **Add a routing rule**.

1. Select **Add** to save the routing rule and return to the **Configuration** tab of the **Create application gateway** blade.

1. Select **Add**

1. Select **Next: Tags** and then **Next: Review + create**.

1. Wait for validation to complete.

1.  Select **Create** to create the virtual network, the public IP address, and the application gateway. 

>It may take several minutes for Azure to create the application gateway. Wait until the deployment finishes successfully before moving on to the next section.

## Task 2: Add an additional subnet to the ContosoVnet Virtual Network.

1. On any Azure Portal page, in **Search resources, services and docs (G+/)**, enter **virtual networks**, and then select **Virtual networks** from the results.

1. In the **Virtual networks** blade: Select **ContosoVnet**

1. In the **ContosoVnet** blade, in the menu under **Settings**: Select **Subnets**

1. Select **+ Subnet**

1. On the **Add subnet** blade: enter the following information: Accept all other default settings.

    | **Setting**      | **Value**      |
    | -------------    | -------------- |
    | Name      | **BackendSubnet**   |
    | Subnet address range | **10.0.1.0/24** |

1. Click **Add**.

>Wait for the deployment to complete before moving on to the next task.

## Task 3: Create virtual machines

1. On the Azure portal, open the **PowerShell** session within the **Cloud Shell** pane.

1. On the toolbar of the Cloud Shell pane, select **Manage files** and then **Upload**. Upload the following files: **backend.json**, **backend.parameters.json**, and **install-iis.ps1**.The files are available for download from the repository, **C:\Allfiles\M05** folder.


1. Deploy the following ARM templates to create the VMs needed for this exercise:

   ```powershell
   $RGName = "ContosoResourceGroup"
   
   New-AzResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile backend.json -TemplateParameterFile backend.parameters.json
   ```

1. When prompted for the **adminpassword:** enter **Pa55w.rd1234abc**.

>**Note**: Do not close Cloud Shell. 

1. The command should complete successfully after a few minutes and list **BackendVM1** and **BackendVM2**.

### Install IIS on each virtual machine

1. Each backend server needs IIS installed.

1. Continue at the PowerShell prompt and use the provided script to install IIS on **BackendVM1**.

   ```powershell
   Invoke-AzVMRunCommand -ResourceGroupName 'ContosoResourceGroup' -Name 'BackendVM1' -CommandId 'RunPowerShellScript' -ScriptPath 'install-iis.ps1'
   ```

   >**Note**: While you wait review the PowerShell script. Notice that the IIS home page is being customized to provide the virtual machine name.

1. Run the command again, this time for **BackendVM2**.

   ```powershell
   Invoke-AzVMRunCommand -ResourceGroupName 'ContosoResourceGroup' -Name 'BackendVM2' -CommandId 'RunPowerShellScript' -ScriptPath 'install-iis.ps1'
   ```
   >**Note**: Each command will take a couple of minutes to complete.

1. Go to the Azure portal home page, and then select **Virtual Machines**.

1. Verify that both virtual machines have been created.

## Task 4: Add backend servers to backend pool

1. On the Azure portal menu, select **All resources** or search for and select All resources. Then select **ContosoAppGateway**.

1. Under **Settings**, select **Backend pools**.

1. Select **BackendPool**.

1. On the Edit backend pool page, under **Backend targets**, in **Target type**, select **Virtual machine**.

1. Under **Target**, select **BackendVM1-nic.** 

1. On **Target type**, select **Virtual machine**.

1. Under **Target**, select **BackendVM2-nic.** 

   ![Azure Portal add target backends to backend pool](../media/edit-backend-pool2.png)

1. Select **Save**.

Wait for the deployment to complete before proceeding to the next step.

## Task 5: Test the application gateway

Although IIS isn't required to create the application gateway, you installed it in this exercise to verify if Azure successfully created the application gateway.

### Use IIS to test the application gateway:

1. Find the public IP address for the application gateway on its **Overview** page. 

   ![Azure Portal look up Frontend Public IP address ](../media/app-gw-public-ip.png)

1. Copy the public IP address, and then paste it into the address bar of your browser to browse that IP address.

1. Check the response. A valid response verifies that the application gateway was successfully created and can successfully connect with the backend.

   ![Broswer - display BackendVM1 or BackendVM2 depending which backend server reponds to request.](../media/browse-to-backend.png)

1. Refresh the browser multiple times and you should see connections to both BackendVM1 and BackendVM2.

Congratulations! You have configured and tested an Azure Application Gateway.

## Clean up resources

   >**Note**: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

1. On the Azure portal, open the **PowerShell** session within the **Cloud Shell** pane.

1. Delete all resource groups you created throughout the labs of this module by running the following command:

   ```powershell
   Remove-AzResourceGroup -Name 'ContosoResourceGroup' -Force -AsJob
   ```

>**Note**: The command executes asynchronously (as determined by the -AsJob parameter), so while you will be able to run another PowerShell command immediately afterwards within the same PowerShell session, it will take a few minutes before the resource groups are actually removed.
