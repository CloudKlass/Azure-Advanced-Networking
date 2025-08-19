---
lab:
    title: 'M06 - Unit 9 Secure your virtual hub using Azure Firewall Manager'
    module: 'Module 06 - Design and implement network security'
---


# M06-Unit 9 Secure your virtual hub using Azure Firewall Manager

In this exercise, you will create the spoke virtual network and create a secured virtual hub, then you will connect the hub and spoke virtual networks and route traffic to your hub. Next you will deploy the workload servers, then create a firewall policy and secure your hub, and finally you will test the firewall.

## Create a hub and spoke architecture

In this part of the exercise, you will create the spoke virtual networks and subnets where you will place the workload servers. Then you will create the secured virtual hub and connect the hub and spoke virtual networks.

In this exercise, you will:

+ Task 1: Create two spoke virtual networks and subnets
+ Task 2: Create the secured virtual hub
+ Task 3: Connect the hub and spoke virtual networks
+ Task 4: Deploy the servers
+ Task 5: Create a firewall policy and secure your hub
+ Task 6: Associate the firewall policy
+ Task 7: Route traffic to your hub
+ Task 8: Test the application rule
+ Task 9: Test the network rule
+ Task 10: Clean up resources

#### Estimated time: 60 minutes

## Task 1: Create two spoke virtual networks and subnets

In this task, you will create the two spoke virtual networks each containing a subnet that will host your workload servers. 

1. On the Azure portal home page, in the search box, type **virtual network** and select **Virtual Network** when it appears.
1. Select **Create**.

>**Important Note:** 
>>If the CREATE VIRTUAL NETWORK - TAB menu order is:     
*"Basics* | **Security** | **IP Addressess** | *Tags* | *Review + Create*". Use **Option 1 instructions**.

>>If the CREATE VIRTUAL NETWORK - TAB menu order is:   
*"Basics* | **IP Addressess** | **Security** | *Tags* | *Review + Create*". Scroll down and use **Option 2 instructions**.

#### **Option 1 instructions**

1. On the Create virtual networks pane, on the **Basics** tab, use the information in the following table to create the VNet:

   | **Setting**          | **Value**                        |
   | -------------------- | -------------------------------- |
   | Resource Group       | Create new: Name: **fw-manager-rg**            |
   | Virtual Network Name | **Spoke-01**                 |
   | Region             | **East US**                          |

1. Click **Next**.

1. On the **Security** blade.Click **Next**.

1. On the **IP addresses** blade: In the **existing Address space section**, check that the address space is set to **10.0.0.0/16**

>**Note:** If the Address space is different, then delete and re-create a **10.0.0.0/16** address space.

5. In the Address space section, under **Subnets**, select **Default**.

1. On the **Edit subnet** blade, use the information in the following table to edit the current Subnet (leave others with their default values):

   | **Setting**          | **Value**                        |
   | -------------------- | -------------------------------- |
   | Name | **Workload-01-SN** |
   | Starting address | **10.0.1.0** |
   | Subnet size | **/24 (256 addresses)**
   
1. Click **Save**

1. Click **Review and Create**. Let validation occur, and click **Create** again to submit your deployment.

>**Repeat** Option 1 steps above to create another similar virtual network and subnet but using the following information:

   | **Setting**          | **Value**                        |
   | -------------------- | -------------------------------- |
   | Resource Group       | Select: **fw-managers-rg**            |
   | Virtual Network Name | **Spoke-02**                 |
   | Region             | **East US**                          |
   | Address space | ***Edit the default address space using the following settings***. |
   |**Address space details** | |
   | Starting address | **10.1.0.0** |
   | Address space size | **/16 (65536 addresses)**
   |**Subnet details** | |
   | Subnet template | **Default** |
   | Name | **Workload-02-SN** |
   | Starting address | **10.1.1.0** |
   | Subnet size | **/24 (256 addresses)**
   | **Save subnet** and then Select **Review and create**, then **Create**

>If you carried out **Option 1 instructions: GO TO Task 2**

#### **Option 2 instructions**

1. In **Resource group**, select **Create new**, and enter **fw-manager-rg** as the name and select **OK**.

1. In **Name**, enter **Spoke-01**.

1. In **Region**, select your region.

1. Select **Next: IP Addresses**.

1. In **IPv4 address space**, enter **10.0.0.0/16**. 

1. **Delete** any other address spaces listed here, such as **10.1.0.0/16**.

1.  Under **Subnet name**, select the word **default**.

1. In the **Edit subnet** dialog box, change the name to **Workload-01-SN**.

1. Change the **Subnet address range** to **10.0.1.0/24**.

1. Select **Save**.

1. Select **Review + create**.

1. Select **Create**.

Repeat steps 1 to 14 above to create another similar virtual network and subnet but using the following information:

- Resource Group: **fw-manager-rg** (select existing)
- Name: **Spoke-02**
- Address space: **10.1.0.0/16** - (delete any other listed address spaces)
- Subnet name: **Workload-02-SN**
- Subnet address range: **10.1.1.0/24**

## Task 2: Create the secured virtual hub

In this task you will create your secured virtual hub using Firewall Manager.

1. From the Azure portal home page, select **More services**.

2. In the search box, type **firewall manager** and select **Network security Keywords: Firewall Manager** when it appears.

3. On the **Network Security** page, In  the **Secure your resources** section, select **Virtual Hubs**.

4. On the **Network security \| Virtual hubs** page, select **Create new secured virtual hub**.

5. For **Resource group**, select **fw-manager-rg**.

6. For **Region**, select **East US**.

7. For the **Secured virtual hub name**, enter **Hub-01**.

8. For **Hub address space**, enter **10.2.0.0/16**.

9. Choose **New vWAN**.

10. In **Virtual WAN Name**, enter **Vwan-01**.

11. For **Type**, Leave the default **Standard**.

12. Select **Next: Azure Firewall**.
    ![Create new secured virtual hub - Basics tab](../media/create-new-secured-virtual-hub-1.png)

13. Select **Next: Security Partner Provider**.

14. Select **Next: Review + create.**, wait for validation to complete.

15. Select **Create**.

    > **[!NOTE]** 
    >
    > This can take up to 30 minutes to deploy.

    ​

    ![Create new secured virtual hub - Review + create tab](../media/create-new-secured-virtual-hub-2.png)

16. When the deployment completes, from the Azure portal home page, select **More services**.

17. In the search box, type **firewall** and select **Firewalls** when it appears.

18. On the **Network security \| Azure Firewalls** page, select **Virtual hubs**.

19. Select **Hub-01**.

20. Select **Public IP configuration** from the Azure Firewall section.

21. Note down the public IP address (e.g., **51.143.226.18**), which you will use later.

## Task 3: Connect the hub and spoke virtual networks

In this task you will connect the hub and spoke virtual networks. This is commonly known as peering.

1. From the Azure portal home page, select **Resource groups**.

2. Select the **fw-manager-rg** resource group, then select the **Vwan-01** virtual WAN.

3. Under **Connectivity**, select **Virtual network connections**.

4. Select **Add connection**.

5. For **Connection name**, enter **hub-spoke-01**.

6. For **Hubs**, select **Hub-01**.

7. For **Resource group**, select **fw-manager-rg**.

8. For **Virtual network**, select **Spoke-01**.

9. Select **Create**.
   ![Add hub and spoke connection to virtual WAN - Spoke 1](../media/connect-hub-spoke-vnet-1.png)

10. Repeat steps 4 to 9 above to create another similar connection but using the connection name of **hub-spoke-02** to connect the **Spoke-02** virtual network.

![Add hub and spoke connection to virtual WAN - Spoke 2](../media/connect-hub-spoke-vnet-2.png)

 >Wait for deployments to complete before doing the next Task.

## Task 4: Deploy the servers

1. On the Azure portal, open the **PowerShell** session within the **Cloud Shell** pane.

1. In the toolbar of the Cloud Shell pane, select the **Upload/Download files** icon, in the drop-down menu, select **Upload** and upload the following files **FirewallManager.json** and **FirewallManager.parameters.json** into the Cloud Shell home directory one by one from the source folder **F:\Allfiles\Exercises\M06**.

1. Deploy the following ARM templates to create the VM needed for this exercise:

   ```powershell
   $RGName = "fw-manager-rg"
   
   New-AzResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile FirewallManager.json -TemplateParameterFile FirewallManager.parameters.json
   ```
1. When prompted for ***adminpassword*** enter **Pa55w.rd1234??** (where **??** = your initials.)

1. When the deployment is complete, go to the Azure portal home page, and then select **Virtual Machines**.

1. On the **Overview** page of **Srv-workload-01**, in the right-hand pane, under the **Networking** section, note down the **Private IP address** (e.g., **10.0.1.4**).

1. On the **Overview** page of **Srv-workload-02**, in the right-hand pane, under the **Networking** section, note down the **Private IP address** (e.g., **10.1.1.4**).


## Task 5: Create a firewall policy and secure your hub

In this task you will first create your firewall policy, then secure your hub. The firewall policy will define collections of rules to direct traffic on one or more Secured virtual hubs.

1. In the portal search bar type **firewalls** and select **Firewall Policies** when it appears.

1. Select **+ Create**.

1. On **Resource group**, select **fw-manager-rg**.

5. Under **Policy details**, for the **Name**, enter **Policy-01**.

1. On **Region** select **East US**.

1. On **Policy tier**, select **Standard**.

1. Select **Next : DNS Settings**.

1. Select **Next : TLS Inspection**.

1. Select **Next : Rules**.

1. On the **Rules** tab, select **Add a rule collection**.

1. On the **Add a rule collection** page, in **Name**, enter **App-RC-01**.

1. For **Rule collection type**, select **Application**.

1. For **Priority**, enter **100**.

1. Ensure **Rule collection action** is **Allow**.

1. Under **Rules**, in **Name** type **Allow-msft**.

1. For the **Source type**, select **IP Address**.

1. For **Source**, enter *.

1. For **Protocol**, enter **http,https**.

1. Ensure **Destination type** is **FQDN**.

1. For **Destination**, enter ***.microsoft.com**.

1. Select **Add**.

    ![Add application rule collection to firewall policy](../media/add-rule-collection-firewall-policy-1.png)

1. To add a **DNAT rule** so you can connect a remote desktop to the Srv-workload-01 VM, select **Add a rule collection**.

1. For **Name**, enter **dnat-rdp**.

1. For **Rule collection type**, select **DNAT**.

1. For **Priority**, enter **100**.

1. Under **Rules**, in **Name** enter **Allow-rdp**.

1. For the **Source type**, select **IP Address**.

1. For **Source**, enter *.

1. For **Protocol**, select **TCP**.

1. For **Destination Ports**, enter **3389**.

1. For **Destination (FirewallIP)**,enter the firewall virtual hub public IP address that you noted down earlier (e.g., **51.143.226.18**).

1. For **Translated address**, enter the private IP address for **Srv-workload-01** that you noted down earlier (e.g., **10.0.1.4**).

1. For **Translated port**, enter **3389**.

1. Select **Add**.

1. To add a **Network rule** so you can connect a remote desktop from Srv-workload-01 to Srv-workload-02 VM, select **Add a rule collection**.

1. For **Name**, enter **vnet-rdp**.

1. For **Rule collection type**, select **Network**.

1. For **Priority**, enter **100**.

1. For **Rule collection action**, select **Allow**.

1. Under **Rules**, in **Name** enter **Allow-vnet**.

1. For the **Source type**, select **IP Address**.

1. For **Source**, enter *.

1. For **Protocol**, select **TCP**.

1. For **Destination Ports**, enter **3389**.

1. For **Destination Type**, select **IP Address**.

1. For **Destination**, enter the private IP address for **Srv-workload-02** that you noted down earlier (e.g., **10.1.1.4**).

1. Select **Add**.

    ![List rule collections in the firewall policy](../media/list-rule-collections-firewall-policy.png)

1. You should now have 3 rule collections listed.

1. Select **Review + create**: Wait for validation to complete.

1. Select **Create**.

## Task 6: Associate the firewall policy

In this task you will associate the firewall policy with the virtual hub.

1. From the Azure portal home page, select **Firewall Policies**.
   
1. On **Network security \| Azure Firewall Policies**, under **Secure your resources**, select **Virtual Hubs**.

1. Select the checkbox for **Hub-01**.

1. Select **Manage security** and from the drop down menu select **Choose another firewall policy**.

1. Select the checkbox for **Policy-01**.

1. Select **Next : Review + confirm**.

1. Select **Confirm** 

1. Wait for the deployment to complete.



## Task 7: Route traffic to your hub

In this task you will ensure that network traffic gets routed through your firewall.
 
1. From the Azure portal home page select **Firewalls**

1. On the Network security \| Azure Firewalls page select **Virtual Hubs** from the **Secure your resources** section 

1. Click on  **Hub-01**.

1. Select **Security configuration**.

1. On **Internet traffic**, select **Azure Firewall**.

1. On **Private traffic**, select **Send via Azure Firewall**.

1. Select **Save**. 

>Click **OK** for the warning message.

> Once configuration has completed (This can take up to 10 minutes), ensure that under **INTERNET TRAFFIC** and **PRIVATE TRAFFIC**, it says **Secured by Azure Firewall** for both hub-spoke connections.


## Task 8: Test the application rule

In this part of the exercise, you will connect a remote desktop to the firewall public IP address, which is NATed to Srv-Workload-01. You will then use a web browser to test the application rule and connect a remote desktop to Srv-Workload-02 to test the network rule.

In this task you will test the application rule to confirm that it works as expected.

1. Open **Remote Desktop Connection** on your PC.

1. On the **Computer** box, enter the **firewall's public IP address** (e.g., **51.143.226.18**).

1. Select **Show Options**.

1. On the **Username** box, enter **TestUser**.

1. Select **Connect**.

   ![RDP connection to srv-workload-01](../media/rdp-srv-workload-01.png)

1. On the **Enter your credentials** dialog box, log into the **Srv-workload-01** server virtual machine, by using the password you created during the VM deployment.

1. Select **OK**.

1. (If it appears), Select **Yes** on the Network blade.

1. Open Internet Explorer and select **Ok** in the **Set up Internet Explorer 11** dialog box.

1. Browse to **https://www.microsoft.com**.

1. On the **Security Alert** dialog box, select **OK**.

1. Select **Close** on the Internet Explorer security alerts that may pop-up.

1. You should see the Microsoft home page.

    ![RDP session browsing microsoft.com](../media/microsoft-home-page.png)

1. Browse to **https://www.google.com**.

1. On the **Security Alert** dialog box, select **OK**.

1. You should be blocked by the firewall.

    ![RDP session browser blocked on google.com](../media/google-home-page-fail.png)

1. So, you have verified that you can connect to the one allowed FQDN but are blocked from all others.

## Task 9: Test the network rule

In this task you will test the network rule to confirm that it works as expected.

1. While still logged in to the **Srv-workload-01** RDP session, from this remote computer, open **Remote Desktop Connection**.

1. On the **Computer** box, enter the **private IP address** of **Srv-workload-02** (e.g., **10.1.1.4**).

1. On the **Enter your credentials** dialog box, log in to the **Srv-workload-02** server by using the username **TestUser**, and the password you provided during the deployment.

1. Select **OK**.

1. (If it appears), Select **Yes** on the Network blade.

   ![RDP session from srv-workload-01 to another RDP session on srv-workload-02](../media/rdp-srv-workload-02-from-srv-workload-01.png)

1. So, now you have verified that the firewall network rule is working, as you have connected a remote desktop from one server to another server located in another virtual network.

1. Close both RDP sessions to disconnect them.


## Task 10: Clean up resources 

>**Note**: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

1. On the Azure portal, open the **PowerShell** session within the **Cloud Shell** pane.

1. Delete all resource groups you created throughout the labs of this module by running the following command:

   ```powershell
   Remove-AzResourceGroup -Name 'fw-manager-rg' -Force -AsJob
   ```

    >**Note**: The command executes asynchronously (as determined by the -AsJob parameter), so while you will be able to run another PowerShell command immediately afterwards within the same PowerShell session, it will take a few minutes before the resource groups are actually removed.
