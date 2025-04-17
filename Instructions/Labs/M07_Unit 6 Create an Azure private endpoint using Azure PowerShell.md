---
lab:
    title: 'M07 - Unit 6 Create an Azure private endpoint using Azure PowerShell'
    module: 'Module 07 - Design and implement private access to Azure Services'
---

# M07-Unit 6 Create an Azure private endpoint using Azure PowerShell

Get started with Azure Private Link by using a Private Endpoint to connect securely to an Azure web app. There are many ways to create Endpoints including Portal, CLI, PowerShell etc. 

#### Estimated time: 45 minutes

You'll create a Private Endpoint for an Azure web app and deploy a virtual machine to test the private connection.

Private Endpoints can be created for different kinds of Azure services, such as Azure SQL and Azure Storage.

**Prerequisites**

- An Azure account with an active subscription. Create an account for free.

- An Azure Web App with a PremiumV2-tier or higher app service plan deployed in your Azure subscription.

- Below steps will guide you through creating the required resource group and web app.

1. Find and open **parameters.json** in M07 folder. Open it in Notepad and find the line "value": "GEN-UNIQUE". Replace the placeholder GEN-UNIQUE string with a unique value for your webapp name. Save this change.

> You could use a name, for example: (***mywebapp***) + initials (***dpb***) + today's date (***311023***) = **mywebappdpb311023** to create a globally unique name.


1. On the Azure portal, open the **PowerShell** session within the **Cloud Shell** pane.

1. On the toolbar of the Cloud Shell pane, select the **Upload/Download files** icon, in the drop-down menu, select **Upload** and upload the following files **template.json** and **parameters.json** into the Cloud Shell home directory one by one.

If you choose to install and use PowerShell locally, this example requires the Azure PowerShell module version 5.4.1 or later. Run ```Get-Module -ListAvailable Az``` to find the installed version. If you need to upgrade, see [Install Azure PowerShell module](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps). If you're running PowerShell locally, you also need to run ```Connect-AzAccount``` to create a connection with Azure.

In this exercise, you will:

+ Task 1: Create a resource group
+ Task 2: Create a virtual network and bastion host
+ Task 3: Create a test virtual machine
+ Task 4: Create a Private Endpoint
+ Task 5: Configure the private DNS zone
+ Task 6: Test connectivity to the Private Endpoint
+ Task 7: Clean up resources

## Task 1: Create a resource group and deploy the prerequisite web app

An Azure resource group is a logical container into which Azure resources are deployed and managed.

Create a resource group with [New-AzResourceGroup](https://docs.microsoft.com/en-us/powershell/module/az.resources/new-azresourcegroup):

```PowerShell
New-AzResourceGroup -Name 'CreatePrivateEndpointQS-rg' -Location 'eastus'
```
Deploy the following ARM templates to create the PremiumV2-tier Azure Web App needed for this exercise:

   ```powershell
   $RGName = "CreatePrivateEndpointQS-rg"
   
   New-AzResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile template.json -TemplateParameterFile parameters.json
   ```
If you receive an error (for example while looking into the Deployment status in the Portal) like "Website with given name GEN-UNIQUE already exists." please make sure to go to the Prerequisites mentioned above regarding editing the template.

## Task 2: Create a virtual network and bastion host

You'll create a virtual network, subnet, and bastion host.

The bastion host will be used to connect securely to the virtual machine for testing the Private Endpoint.

Create a virtual network and bastion host with:

- New-AzVirtualNetwork

- New-AzPublicIpAddress

- New-AzBastion

 > This can take 15 minutes to deploy.

```PowerShell
## Create backend subnet config. ##

$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name myBackendSubnet -AddressPrefix 10.0.0.0/24

## Create Azure Bastion subnet. ##

$bastsubnetConfig = New-AzVirtualNetworkSubnetConfig -Name AzureBastionSubnet -AddressPrefix 10.0.1.0/24

## Create the virtual network. ##

$parameters1 = @{

 Name = 'MyVNet'

 ResourceGroupName = 'CreatePrivateEndpointQS-rg'

 Location = 'eastus'

 AddressPrefix = '10.0.0.0/16'

 Subnet = $subnetConfig, $bastsubnetConfig

}

$vnet = New-AzVirtualNetwork @parameters1

## Create public IP address for bastion host. ##

$parameters2 = @{

 Name = 'myBastionIP'

 ResourceGroupName = 'CreatePrivateEndpointQS-rg'

 Location = 'eastus'

 Sku = 'Standard'

 AllocationMethod = 'Static'

}

$publicip = New-AzPublicIpAddress @parameters2

## Create bastion host ##

$parameters3 = @{

 ResourceGroupName = 'CreatePrivateEndpointQS-rg'

 Name = 'myBastion'

 PublicIpAddress = $publicip

 VirtualNetwork = $vnet

}

New-AzBastion @parameters3
```


> Don't forget to press enter to run the last command (***New-AzBastion @parameters3***) when prompted, then wait for the deployment to finish before moving on to the next task.

> Bastion Host deployment can take up to 15 minutes ***(good time for a coffee break)***.

## Task 3: Create a test virtual machine

In this section, you'll create a virtual machine that will be used to test the Private Endpoint.

- Create the virtual machine with:

- Get-Credential 

- New-AzNetworkInterface

- New-AzVM

- New-AzVMConfig

- Set-AzVMOperatingSystem

- Set-AzVMSourceImage

- Add-AzVMNetworkInterface

```PowerShell
## Set credentials for server admin and password. ##

$cred = Get-Credential

## Command to get virtual network configuration. ##

$vnet = Get-AzVirtualNetwork -Name myVNet -ResourceGroupName CreatePrivateEndpointQS-rg

## Command to create network interface for VM ##

$parameters1 = @{

 Name = 'myNicVM'

 ResourceGroupName = 'CreatePrivateEndpointQS-rg'

 Location = 'eastus'

 Subnet = $vnet.Subnets[0]

}

$nicVM = New-AzNetworkInterface @parameters1

## Create a virtual machine configuration.##

$parameters2 = @{

 VMName = 'myVM'

 VMSize = 'Standard_DS1_v2'

}

$parameters3 = @{

 ComputerName = 'myVM'

 Credential = $cred

}

$parameters4 = @{

 PublisherName = 'MicrosoftWindowsServer'

 Offer = 'WindowsServer'

 Skus = '2019-Datacenter-gensecond'

 Version = 'latest'

}

$vmConfig = New-AzVMConfig @parameters2 | Set-AzVMOperatingSystem -Windows @parameters3 | Set-AzVMSourceImage @parameters4 | Add-AzVMNetworkInterface -Id $nicVM.Id

## Create the virtual machine ##

New-AzVM -ResourceGroupName 'CreatePrivateEndpointQS-rg' -Location 'eastus' -VM $vmConfig 


```
> (Note: **when prompted enter a local admin account credentials for the VM** (i.e. ***Testuser and Pa55w.rd1234??*** (where ?? are your initials))).

> Don't forget to press enter when prompted to run the last command (***New-AzVM etc.***). Then wait for the deployment to complete (Should take about 5 minutes).


Azure provides an ephemeral IP for Azure Virtual Machines which aren't assigned a public IP address, or are in the backend pool of an internal Basic Azure Load Balancer. The ephemeral IP mechanism provides an outbound IP address that isn't configurable.

The ephemeral IP is disabled when a public IP address is assigned to the virtual machine or the virtual machine is placed in the backend pool of a Standard Load Balancer with or without outbound rules. If a Azure Virtual Network NAT gateway resource is assigned to the subnet of the virtual machine, the ephemeral IP is disabled.

For more information on outbound connections in Azure, see Using Source Network Address Translation (SNAT) for outbound connections.

## Task 4: Create a Private Endpoint

In this section, you'll create the Private Endpoint and connection using:

- New-AzPrivateLinkServiceConnection

- New-AzPrivateEndpoint

 

```PowerShell
## Place web app into variable. This assumes that only one web app exists in the resource group. ##

$webapp = Get-AzWebApp -ResourceGroupName CreatePrivateEndpointQS-rg

## Create Private Endpoint connection. ##

$parameters1 = @{

 Name = 'myConnection'

 PrivateLinkServiceId = $webapp.ID

 GroupID = 'sites'

}

$privateEndpointConnection = New-AzPrivateLinkServiceConnection @parameters1

## Place virtual network into variable. ##

$vnet = Get-AzVirtualNetwork -ResourceGroupName 'CreatePrivateEndpointQS-rg' -Name 'myVNet'

## Disable private endpoint network policy ##

$vnet.Subnets[0].PrivateEndpointNetworkPolicies = "Disabled"

$vnet | Set-AzVirtualNetwork

## Create private endpoint

$parameters2 = @{

 ResourceGroupName = 'CreatePrivateEndpointQS-rg'

 Name = 'myPrivateEndpoint'

 Location = 'eastus'

 Subnet = $vnet.Subnets[0]

 PrivateLinkServiceConnection = $privateEndpointConnection

}

New-AzPrivateEndpoint @parameters2 
```

> Don't forget to press enter when prompted for the last command (***New-AzPrivateEndpoint @parameters 2***). Wait for the deployment to complete (should take about 2 minutes).


## Task 5: Configure the private DNS zone

In this section you'll create and configure the private DNS zone using:

- New-AzPrivateDnsZone

- New-AzPrivateDnsVirtualNetworkLink

- New-AzPrivateDnsZoneConfig

- New-AzPrivateDnsZoneGroup

```PowerShell
## Place virtual network into variable. ##

$vnet = Get-AzVirtualNetwork -ResourceGroupName 'CreatePrivateEndpointQS-rg' -Name 'myVNet'

## Create private dns zone. ##

$parameters1 = @{

 ResourceGroupName = 'CreatePrivateEndpointQS-rg'

 Name = 'privatelink.azurewebsites.net'

}

$zone = New-AzPrivateDnsZone @parameters1

## Create dns network link. ##

$parameters2 = @{

 ResourceGroupName = 'CreatePrivateEndpointQS-rg'

 ZoneName = 'privatelink.azurewebsites.net'

 Name = 'myLink'

 VirtualNetworkId = $vnet.Id

}

$link = New-AzPrivateDnsVirtualNetworkLink @parameters2

## Create DNS configuration ##

$parameters3 = @{

 Name = 'privatelink.azurewebsites.net'

 PrivateDnsZoneId = $zone.ResourceId

}

$config = New-AzPrivateDnsZoneConfig @parameters3

## Create DNS zone group. ##

$parameters4 = @{

 ResourceGroupName = 'CreatePrivateEndpointQS-rg'

 PrivateEndpointName = 'myPrivateEndpoint'

 Name = 'myZoneGroup'

 PrivateDnsZoneConfig = $config

}

New-AzPrivateDnsZoneGroup @parameters4 
```
> Don't forget to press enter when prompted to run the last command (***New-AzPrivateDnsZoneGroup @parameters4***). Then wait for the deployment to complete (Should take about 2 minutes).

1. On Completion of deployment, close **Cloud Shell**

## Task 6: Test connectivity to the Private Endpoint

In this section, you'll use the virtual machine you created in the previous step to connect to the web app across the Private Endpoint.

1. Sign in to the [Azure portal](https://portal.azure.com/)

1. Select **Resource groups** in the left-hand navigation pane.

1. Select **CreatePrivateEndpointQS-rg**.

1. Select **myVM**.

1. On the overview page for **myVM**, select **Connect** then select **Go to Bastion**.

1. Enter the username and password that you entered during the virtual machine creation and select **Connect**.

1. Open Windows PowerShell on the server after you connect.

1. Enter **nslookup ***&lt;your- webapp-name&gt;***.azurewebsites.net**. Replace &lt;your-webapp-name&gt; with the name of the web app you created in the previous steps. You'll receive a message similar to what is displayed below:

  ```
  Server: UnKnown
  
  Address: 168.63.129.16
  
  Non-authoritative answer:
  
  Name: mywebapp8675.privatelink.azurewebsites.net
  
  Address: 10.0.0.5
  
  Aliases: mywebapp8675.azurewebsites.net 
  ```  


A private IP address of **10.0.0.5** is returned for the web app name. This address is in the subnet of the virtual network you created previously.

1. In the bastion connection to **myVM**, open Internet Explorer.
1. Enter the url of your web app, **https://&lt;your-webapp-name&gt;.azurewebsites.net**
1. You'll receive the default web app page if your application has been deployed:
  ![screen shot of page in Azure indicating an app service is up and running](../media/web-app-default-page.png)
1. Close the connection to **myVM**. 

## Task 7: Clean up resources

When you're done using the Private Endpoint and the VM, use [Remove-AzResourceGroup](https://docs.microsoft.com/en-us/powershell/module/az.resources/remove-azresourcegroup) to remove the resource group and all the resources it has:

```PowerShell
Remove-AzResourceGroup -Name CreatePrivateEndpointQS-rg -Force -AsJob
```





